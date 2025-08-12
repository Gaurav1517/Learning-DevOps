# **Complete Ansible Setup for Normal User**

###  Ansible Installation & Configuration (Normal User Setup)

This guide explains how to install and configure Ansible for a non-root user (`sysops`),
set up passwordless SSH, and define the inventory and configuration files.

---

##  Environment Overview

| Machine Name        | IP Address        | Role               |
|---------------------|-------------------|--------------------|
| ansible-controller  | 192.168.157.150   | Control Node       |
| ansible-worker01    | 192.168.157.149   | Managed Node       |

---

##  1. Add Local DNS Entries

On **both** the controller and worker nodes, update `/etc/hosts`:

```bash
sudo tee -a /etc/hosts <<EOF

192.168.157.150 ansible-controller
192.168.157.149 ansible-worker01
EOF
```

---

##  2. Set Hostnames

Run the following on each system:

### Controller:

```bash
sudo hostnamectl set-hostname ansible-controller
```

### Worker:

```bash
sudo hostnamectl set-hostname ansible-worker01
```

---

##  3. Verify Connectivity

From each machine, run:

```bash
ping -c 4 192.168.157.150 && ping -c 4 ansible-controller
ping -c 4 192.168.157.149 && ping -c 4 ansible-worker01
```

---

##  4. Install Required Packages (Controller)

```bash
sudo dnf update -y
sudo dnf install -y python3 python3-pip
python3 --version
pip3 --version

# Install Ansible
python3.9 -m pip install ansible
ansible --version
```

---

##  5. Create and Configure `sysops` User

```bash
sudo useradd sysops
sudo passwd sysops
sudo usermod -aG wheel sysops
```

### (Optional) Enable Passwordless Sudo:

```bash
sudo vim /etc/sudoers
```

Add this line:

```bash
sysops ALL=(ALL) NOPASSWD: ALL
```

---

##  6. Configure SSH Key-Based Authentication

### On Controller (as `sysops`):

```bash
ssh-keygen
# Press Enter for default path and no passphrase
```

### Copy SSH Key to All Managed Nodes:

```bash
ssh-copy-id sysops@192.168.44.132
ssh-copy-id sysops@192.168.44.134
```

### Verify Passwordless SSH:

```bash
ssh sysops@192.168.44.132
ssh sysops@192.168.44.134
```

You should connect without being prompted for a password.

---

##  7. Create Ansible Configuration and Inventory

Create the directory structure:

```bash
mkdir -p /home/sysops/ansible
cd /home/sysops/ansible
```

### Inventory File: `sysops-inv-host`

```ini
[sysops-server]
192.168.44.132
192.168.44.134
```

### Ansible Configuration File: `ansible.cfg`

```ini
[defaults]
inventory = /home/sysops/ansible/sysops-inv-host
remote_user = sysops  # Replace with remote username
host_key_checking = False
```

---

##  8. Set Ansible Config Environment Variable

Add this to `~/.bashrc` (for `sysops`):

```bash
cat <<EOF >> ~/.bashrc

# Ansible configuration path
export ANSIBLE_CONFIG=/home/sysops/ansible/ansible.cfg
EOF
```
# Reload bash config
```bash
source ~/.bashrc
```

---

##  9. Test Ansible Setup

### List Hosts:

```bash
ansible all --list-hosts
```

Expected:

```
  hosts (2):
    192.168.44.132
    192.168.44.134
```

### Ping All Hosts:

```bash
ansible all -m ping
```

Expected:

```
192.168.44.132 | SUCCESS => { "ping": "pong" }
192.168.44.134 | SUCCESS => { "ping": "pong" }
```

---

##  Troubleshooting

* **SSH Permission Denied**: Check `ansible.cfg` for correct `remote_user` and remove inline comments.
* **Hosts Unreachable**: Confirm SSH key exists on both managed nodes.
* **No inventory parsed**: Ensure `ANSIBLE_CONFIG` is exported and paths are correct.

---

##  Summary

| Step                      | Completed |
| ------------------------- | --------- |
| Hostnames & DNS           | ✅         |
| SSH Key Auth Setup        | ✅         |
| Ansible Installed         | ✅         |
| `ansible.cfg` & Inventory | ✅         |
| Ansible Commands Work     | ✅         |

---

