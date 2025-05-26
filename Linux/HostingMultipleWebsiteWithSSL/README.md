# Hosting Multiple Websites with SSL Encryption in RHEL 8

## Overview

This guide explains how to host multiple websites on an Apache web server with SSL
encryption using self-signed certificates on RHEL 8. We will cover the installation 
and configuration of Apache, SSL certificate generation, and setting up virtual hosts 
for multiple websites with HTTPS.

### Prerequisites:

* A fresh RHEL 8 server.
* Root or sudo privileges.
* The `firewalld` service should be running.
* DNS or `/etc/hosts` entries for your domains.
* Internet access for downloading necessary packages.

---

## Step 1: Set Hostname

First, set your server hostname using the following command:

```bash
sudo hostnamectl set-hostname www.gaurav.com
```

To confirm the hostname has been set:

```bash
hostnamectl
hostnamectl | grep -i Static
```

**Expected output:**

```
Static hostname: www.gaurav.com
```

---

## Step 2: Check and Configure Repositories

Ensure that the necessary repositories are configured:

```bash
dnf repolist
```

If needed, configure the repository based on your RHEL version.

---

## Step 3: Install Apache and SSL Modules

Install Apache and SSL support:

```bash
sudo dnf install -y httpd openssl mod_ssl
```

---

## Step 4: Generate SSL Certificates

Generate a self-signed SSL certificate for the domain:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
```

Enter the required information as shown below:

```plaintext
Country Name (2 letter code) [XX]: IN
State or Province Name (full name) []: Punjab
Locality Name (eg, city) [Default City]: Rajpura
Organization Name (eg, company) [Default Company Ltd]: GauravChauhan
Organizational Unit Name (eg, section) []: HO
Common Name (eg, your name or your server's hostname) []: Gaurav
Email Address []: sysops@192.168.70.133
```

Now copy the certificate and key files to the correct directories:

```bash
sudo cp server.crt /etc/pki/tls/certs/
sudo cp server.key /etc/pki/tls/private/
```

Check the files to ensure they have been copied correctly:

```bash
ls -lh /etc/pki/tls/private/
ls -lh /etc/pki/tls/certs/
```

---

## Step 5: Configure SSL in Apache

Before editing the SSL configuration file, take a backup:

```bash
sudo cp /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.backup
```

Edit the `/etc/httpd/conf.d/ssl.conf` file:

```bash
sudo vim /etc/httpd/conf.d/ssl.conf
```

Change the following lines to match the paths to your certificate and key:

```apache
SSLCertificateFile /etc/pki/tls/certs/server.crt
SSLCertificateKeyFile /etc/pki/tls/private/server.key
```

Test Apache configuration:

```bash
httpd -t
```

You may see the message:
Syntax OK

---

## Step 6: Configure Virtual Hosts

Now, configure virtual hosts for your websites. Edit the `/etc/httpd/conf.d/httpd.conf` file:

```bash
sudo vim /etc/httpd/conf.d/httpd.conf
```

Add the following configuration for two websites:

```apache
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/server.crt
    SSLCertificateKeyFile /etc/pki/tls/private/server.key
    ServerName www.gaurav.com
    DocumentRoot /var/www/html/website1
</VirtualHost>

<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/server.crt
    SSLCertificateKeyFile /etc/pki/tls/private/server.key
    ServerName www.gaurav.net
    DocumentRoot /var/www/html/website2
</VirtualHost>
```

---

## Step 7: Create Website Directories

Create directories for both websites:

```bash
sudo mkdir -p /var/www/html/website1
sudo mkdir -p /var/www/html/website2
```

Ensure the directories have the correct SELinux context:

```bash
sudo chcon -t httpd_sys_content_t /var/www/html/website1
sudo chcon -t httpd_sys_content_t /var/www/html/website2
```

Verify:

```bash
ls -lZd /var/www/html/
ls -lZd /var/www/html/website1
ls -lZd /var/www/html/website2
```

---

## Step 8: Download and Deploy Website Templates

You can download free website templates from [TemplateMo](https://templatemo.com/). For this example, we'll use:

* `templatemo_586_scholar` for website1.
* `templatemo_591_villa_agency` for website2.

Download and copy the contents into the respective directories:

```bash
sudo cp -r templatemo_586_scholar/* /var/www/html/website1
sudo cp -r templatemo_591_villa_agency/* /var/www/html/website2
```

---

## Step 9: Configure Firewall

Allow HTTPS traffic through the firewall:

```bash
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

---

## Step 10: Start Apache and Enable on Boot

Enable and start the Apache service:

```bash
sudo systemctl enable httpd.service
sudo systemctl start httpd.service
```

Check the status:

```bash
sudo systemctl status httpd.service
```

---

## Step 11: Update `/etc/hosts` for Local DNS

Add the following entries to your `/etc/hosts` file to map the domain 
names to your server’s IP address:

```bash
sudo vim /etc/hosts
```

Add the following line (assuming `192.168.70.133` is your server's IP):

```plaintext
192.168.70.133 www.gaurav.com
192.168.70.133 www.gaurav.net
```

Restart Apache:

```bash
sudo systemctl restart httpd.service
```

---

## Step 12: Verify the Websites

Finally, you can check the websites in your browser:

* [https://www.gaurav.com](https://www.gaurav.com)
  ![]()
* [https://www.gaurav.net](https://www.gaurav.net)
  ![]()
  
Both should be accessible over HTTPS with the self-signed SSL certificates.

---

This completes the configuration of hosting multiple websites with SSL encryption in RHEL 8. 
If you run into any issues, review the logs or try restarting the Apache service.


