##  Steps to Change Jenkins Port on Debian/Ubuntu

### 1. **Edit Jenkins configuration file**

Open the Jenkins default configuration file using a text editor like `nano`:

```bash
sudo nano /etc/default/jenkins
```

### 2. **Find the HTTP\_PORT line**

Look for this line:

```bash
HTTP_PORT=8080
```

### 3. **Change the port number**

Change `8080` to your desired port, for example:

```bash
HTTP_PORT=9090
```

### 4. **Save and exit**

* In nano: Press `CTRL + O`, then `Enter` to save, and `CTRL + X` to exit.

### 5. **Restart Jenkins**

Apply the change by restarting the Jenkins service:

```bash
sudo systemctl restart jenkins
```

### 6. **Verify Jenkins is running on the new port**

Open a browser and go to:

```
http://<your-server-ip>:9090
```

or use `curl`:

```bash
curl http://localhost:9090
```

---

##  Optional: Adjust Firewall (UFW)

If you're using a firewall like UFW, allow the new port:

```bash
sudo ufw allow 9090
```

---
