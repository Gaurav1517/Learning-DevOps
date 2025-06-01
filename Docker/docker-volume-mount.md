# 🐳 Docker Volume Mount: Host-Container File Sharing

This guide demonstrates how to **create a Docker container that shares a directory with the host system**, enabling seamless file exchange between the host and the container.

---

## 📁 Step 1: Create a Host Directory for Mounting

Create a directory on the host machine that will be mounted inside the container:

```bash
mkdir /host-volume
```

Create a test file inside this directory:

```bash
echo "What's up! from docker-host" > /host-volume/host.txt
```

---

## 🔐 Step 2: Ensure Docker Is Authenticated (If Required)

If pulling images from a private registry (like Docker Hub), log in first:

```bash
docker login -u your-username
# Enter your password when prompted
```

---

## 🚀 Step 3: Create and Run the Docker Container

Now, run a new container using the Alpine Linux image and mount the host directory into it:

```bash
docker run -dit --name=demo -v /host-volume/:/opt/new:Z alpine:latest
```

* `-dit`: Run in detached, interactive mode with a terminal
* `--name=demo`: Names the container "demo"
* `-v /host-volume/:/opt/new:Z`: Mounts the host directory to `/opt/new` in the container
* `:Z`: Ensures proper SELinux context (only needed on SELinux-enabled systems)
* `alpine:latest`: Lightweight Linux image

Check that the container is running:

```bash
docker ps
```

---

## 🔍 Step 4: Inspect the Mounted Directory from Inside the Container

Access the running container:

```bash
docker exec -it demo sh
```

Once inside, check the mounted files:

```sh
ls /opt/new/
```

**Output:**

```text
host.txt
```

---

## 🛠️ Step 5: Create Files from Inside the Container

### ⚠️ Problem:

Alpine uses `sh`, which does **not support brace expansion** like `{1..10}`. So the following won't work as expected:

```sh
touch /opt/new/file{1..10}.txt
```

This would create a single file named `file{1..10}.txt`.

---

## ✅ Solution: Install and Use Bash

To enable Bash features:

### Install Bash:

```sh
apk add --no-cache bash
```

### Run Bash:

```sh
bash
```

### Create Multiple Files:

```bash
touch /opt/new/file{1..10}.txt
ls /opt/new/
```

**Expected Output:**

```text
file1.txt
file2.txt
...
file10.txt
host.txt
```

Exit the container:

```bash
exit
```

---

## 🖥️ Step 6: Verify on the Host

Check the host-mounted directory to confirm the new files are created:

```bash
ls /host-volume/
```

**Output:**

```text
host.txt
file1.txt
file2.txt
...
file10.txt
```

---

## ✅ Summary

| Task                   | Command                                                                 |
| ---------------------- | ----------------------------------------------------------------------- |
| Create mount directory | `mkdir /host-volume`                                                    |
| Add test file          | `echo "..." > /host-volume/host.txt`                                    |
| Login to Docker        | `docker login -u your-username`                                         |
| Run container          | `docker run -dit --name=demo -v /host-volume/:/opt/new:Z alpine:latest` |
| Enter container        | `docker exec -it demo sh`                                               |
| Install bash           | `apk add --no-cache bash`                                               |
| Create multiple files  | `touch /opt/new/file{1..10}.txt`                                        |
| Verify from host       | `ls /host-volume/`                                                      |

---

