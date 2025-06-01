# 📦 Docker Image Import & Export Guide

This guide explains how to **export a Docker container to a `.tar` file**, and then
**import it as a Docker image**. This is useful for sharing containers with their file changes but not necessarily their full Dockerfile history.

---

## 🐳 Step 1: Pull a Base Image

### 📌 Command:

```bash
docker pull nginx
```

### 🧠 Syntax Breakdown:

* `docker pull <image-name>`: Downloads a Docker image from Docker Hub or another registry.
* `nginx`: The image name (defaults to the `latest` tag if not specified).

---

## 🔍 Step 2: Verify Downloaded Images

### 📌 Command:

```bash
docker images
```

### 🧠 Syntax Breakdown:

* Lists all images available locally on the host.
* Shows REPOSITORY, TAG, IMAGE ID, CREATED time, and SIZE.

---

## 🚀 Step 3: Run a Container

### 📌 Command:

```bash
docker run -itd --name test1 nginx:latest
```

### 🧠 Syntax Breakdown:

* `docker run`: Run a new container.
* `-i`: Interactive mode.
* `-t`: Allocate a pseudo-TTY (terminal).
* `-d`: Run in detached mode (in background).
* `--name test1`: Name the container `test1`.
* `nginx:latest`: Use the `nginx` image, tag `latest`.

---

## 📁 Step 4: Modify the Running Container

### 📌 Command:

```bash
docker exec -it test1 /bin/bash
```

### 🧠 Syntax Breakdown:

* `docker exec`: Run a command inside a running container.
* `-i -t`: Interactive terminal.
* `test1`: The name of the container.
* `/bin/bash`: Run the Bash shell inside the container.

Inside the container, run:

```bash
mkdir /my-data
touch /my-data/file{1..10}.txt
ls /my-data/
```

### 🧠 Explanation:

* `touch /my-data/file{1..10}.txt`: Bash brace expansion creates 10 files (`file1.txt` to `file10.txt`).
* This works because the `nginx` image is Debian-based and includes `bash`.

Exit container:

```bash
exit
```

---

## 📤 Step 5: Export the Container

### 📌 Command:

```bash
docker export test1 > custom-container.tar
```

### 🧠 Syntax Breakdown:

* `docker export <container-name>`: Exports the container’s filesystem as a tar archive.
* `>`: Redirects output to a file.
* `custom-container.tar`: Name of the exported file.

---

## 📥 Step 6: Import the Container as an Image

### 📌 Command:

```bash
docker import custom-container.tar custom-image:v1
```

### 🧠 Syntax Breakdown:

* `docker import <file> <image-name>:<tag>`: Creates an image from a tar archive.
* `custom-image`: New image repository name.
* `v1`: Tag version.

Check the new image:

```bash
docker images
```

---

## ⚙️ Step 7: Run a New Container from the Imported Image

Try to run without a command:

```bash
docker run -it --name test2 custom-image:v1
```

### ❗ You’ll get an error:

```text
docker: Error response from daemon: no command specified
```

That’s because `docker import` doesn't preserve entrypoint or CMD settings.

### ✅ Correct Command:

```bash
docker run -it --name test2 custom-image:v1 /bin/bash
```

### 🧠 Syntax Breakdown:

* Appends `/bin/bash` so Docker knows what command to run inside the container.

Verify the files:

```bash
ls /my-data/
```

You should see the files you created earlier.

---

## ✅ Summary Table

| Purpose                       | Command                                                 |
| ----------------------------- | ------------------------------------------------------- |
| Pull a Docker image           | `docker pull nginx`                                     |
| View local images             | `docker images`                                         |
| Run a container               | `docker run -itd --name test1 nginx:latest`             |
| Enter a running container     | `docker exec -it test1 /bin/bash`                       |
| Create files inside container | `touch /my-data/file{1..10}.txt`                        |
| Export container to `.tar`    | `docker export test1 > custom-container.tar`            |
| Import `.tar` as Docker image | `docker import custom-container.tar custom-image:v1`    |
| Run new image with command    | `docker run -it --name test2 custom-image:v1 /bin/bash` |
| List files in container       | `ls /my-data/`                                          |

---
