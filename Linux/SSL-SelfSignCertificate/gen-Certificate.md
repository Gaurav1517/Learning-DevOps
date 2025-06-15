## 🎯 GOAL: Create a self-signed CA, then use it to sign a certificate for `etcd`

---

### ✅ STEP 1: Create a CA Private Key

```bash
openssl genrsa -out ca.key 2048
```

| Parameter     | Meaning                                               |
| ------------- | ----------------------------------------------------- |
| `genrsa`      | Generate an RSA private key                           |
| `-out ca.key` | Output file name for the CA’s private key             |
| `2048`        | Key size in bits (2048 is standard; 4096 is stronger) |

🔐 This creates your **CA’s private key**. It’s like the master key for signing other certificates.

---

### ✅ STEP 2: Create a Certificate Signing Request (CSR) for CA

```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

| Parameter                   | Meaning                                                                           |
| --------------------------- | --------------------------------------------------------------------------------- |
| `req`                       | Start a certificate request                                                       |
| `-new`                      | Generate a new request                                                            |
| `-key ca.key`               | Use the CA’s private key                                                          |
| `-subj "/CN=KUBERNETES-CA"` | Set subject (Common Name = "KUBERNETES-CA") directly (skip interactive questions) |
| `-out ca.csr`               | Output the CSR file for CA                                                        |

📄 This creates a **CSR file** (`ca.csr`) which says:

> “I am Kubernetes-CA and here’s my public key.”

---

### ✅ STEP 3: Create a **Self-Signed Certificate** for CA

```bash
openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial -out ca.crt -days 1000
```

| Parameter         | Meaning                                       |
| ----------------- | --------------------------------------------- |
| `x509`            | Generate an X.509 certificate                 |
| `-req`            | Input is a CSR                                |
| `-in ca.csr`      | Use the request file you just created         |
| `-signkey ca.key` | Sign with your own private key (self-signing) |
| `-CAcreateserial` | Create a serial number file for signing       |
| `-out ca.crt`     | Output file — this is your CA's certificate   |
| `-days 1000`      | Valid for 1000 days                           |

📜 Now you have a **self-signed CA certificate** (`ca.crt`), ready to sign others.

---

### ✅ STEP 4: Create a Private Key for `etcd`

```bash
openssl genrsa -out etcd.key 2048
```

| Parameter       | Meaning                      |
| --------------- | ---------------------------- |
| `genrsa`        | Generate an RSA private key  |
| `-out etcd.key` | Save as `etcd.key`           |
| `2048`          | 2048-bit encryption strength |

🔐 This creates the **private key for etcd** (used for secure communication).

---

### ✅ STEP 5: Create a CSR for `etcd` (With Extra Config)

```bash
openssl req -new -key etcd.key -subj "/CN=etcd" -out etcd.csr -config etcd.cnf
```

| Parameter          | Meaning                                           |
| ------------------ | ------------------------------------------------- |
| `-new`             | New CSR                                           |
| `-key etcd.key`    | Use the `etcd` private key                        |
| `-subj "/CN=etcd"` | Common Name = `etcd`                              |
| `-out etcd.csr`    | Output file for etcd's request                    |
| `-config etcd.cnf` | Use a config file (to add extensions, SANs, etc.) |

📄 `etcd.csr` is the request for a certificate for `etcd`.

---

### ✅ STEP 6: Sign `etcd.csr` with the CA to Issue `etcd.crt`

```bash
openssl x509 -req -in etcd.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd.crt -extensions v3_req -extfile etcd.cnf -days 1000
```

| Parameter            | Meaning                                            |
| -------------------- | -------------------------------------------------- |
| `x509`               | Create certificate                                 |
| `-req`               | Input is a CSR                                     |
| `-in etcd.csr`       | CSR from previous step                             |
| `-CA ca.crt`         | CA’s certificate used to sign it                   |
| `-CAkey ca.key`      | CA’s private key to actually sign                  |
| `-CAcreateserial`    | Create serial number file                          |
| `-out etcd.crt`      | Output: Signed certificate for `etcd`              |
| `-extensions v3_req` | Use the `v3_req` section in config file (etcd.cnf) |
| `-extfile etcd.cnf`  | Config file with extension settings                |
| `-days 1000`         | Validity of the certificate                        |

📜 You now have a **valid certificate (`etcd.crt`) for etcd**, signed by your **trusted CA (`ca.crt`)**.

---

## 🗂 Final File List (After `ls`)

| File       | Description                                                             |
| ---------- | ----------------------------------------------------------------------- |
| `ca.key`   | CA private key                                                          |
| `ca.csr`   | CSR for CA                                                              |
| `ca.crt`   | CA certificate (self-signed)                                            |
| `ca.srl`   | CA serial number file (auto-created)                                    |
| `etcd.key` | Private key for etcd                                                    |
| `etcd.csr` | CSR for etcd                                                            |
| `etcd.crt` | Final certificate for etcd (signed by CA)                               |
| `etcd.cnf` | OpenSSL config file for extra settings (like Subject Alternative Names) |

---

## ✅ Summary of the Flow:

```
[etcd.key]  --> Used to create -->  [etcd.csr] --> Sent to CA
                                         |
                            CA uses [ca.key] + [ca.crt]
                                         ↓
                                Signs and returns:
                                    [etcd.crt]
```

---

