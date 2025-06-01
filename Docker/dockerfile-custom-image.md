# 🛠️ Creating a Custom Docker Image Using a Dockerfile (Apache Server)

This guide shows how to build a custom Docker image using a **Dockerfile** that sets up a basic Apache 
web server using Red Hat’s UBI (Universal Base Image).

---

## 📄 Step 1: Prepare the Content

### 📌 Command:

```bash
echo "What's up! from apache server.." > index.html
```

### 🧠 Syntax Breakdown:

* `echo`: Prints text to standard output.
* `>`: Redirects output into a file.
* `index.html`: This will be the default web page served by Apache inside the container.

---

## 📝 Step 2: Write the Dockerfile

### 📌 Command:

```bash
vim Dockerfile
```

Paste the following content:

```Dockerfile
# Use Red Hat UBI (Universal Base Image)
FROM registry.access.redhat.com/ubi8/ubi
LABEL maintainer="Gaurav Chauhan <gaurav.cloud000@gmail.com>"
RUN dnf update -y
RUN dnf install -y httpd && \
    dnf clean all
COPY index.html /var/www/html/
EXPOSE 80
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

```

---

### 🧠 Dockerfile Syntax Explanation:

| Line                                        | Purpose                                              |
| ------------------------------------------- | ---------------------------------------------------- |
| `FROM registry.access.redhat.com/ubi8/ubi`  | Base image (Red Hat UBI 8)                           |
| `MAINTAINER`                                | (Deprecated) Specifies image author. Prefer `LABEL`. |
| `RUN dnf update -y`                         | Updates all packages inside the image                |
| `RUN dnf install -y httpd && dnf clean all` | Installs Apache HTTP server and cleans up cache      |
| `COPY index.html /var/www/html/`            | Copies your custom HTML to the Apache web directory  |
| `EXPOSE 80`                                 | Declares that the container will use port 80         |
| `CMD [...]`                                 | Sets the default command (runs Apache in foreground) |

---

## 🏗️ Step 3: Build the Docker Image

```bash
docker build -t custom-image:v1 .
```

### 🧠 Syntax Breakdown:

* `docker build`: Builds an image from a Dockerfile.
* `-t custom-image:v1`: Tags the image with name `custom-image` and version `v1`.
* `.`: Context directory (the current directory containing `Dockerfile` and `index.html`).

---

## 🖼️ Step 4: Verify the Image

### 📌 Command:

```bash
docker images
```

**Output:**

```text
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
custom-image      v1        <image-id>     few seconds ago  238MB
```

---

## 🚀 Step 5: Run the Container

### 📌 Command:

```bash
docker run -d -p 8081:80 custom-image:v1
```

### 🧠 Syntax Breakdown:

* `-d`: Detached mode (runs in background)
* `-p 8081:80`: Maps host port `8081` to container port `80`
* `custom-image:v1`: The image to run

Verify it's running:

```bash
docker ps
```

---

## 🌐 Step 6: Test the Web Server

Open your browser or use `curl`:

```bash
curl http://<your-host-ip>:8081
```

**Expected Output:**

```text
What's up! from apache server..
```

---

## ✅ Summary Table

| Task                     | Command                                    |
| ------------------------ | ------------------------------------------ |
| Create `index.html`      | `echo "..." > index.html`                  |
| Write Dockerfile         | `vim Dockerfile`                           |
| Build image              | `docker build -t custom-image:v1 .`        |
| Run container            | `docker run -d -p 8081:80 custom-image:v1` |
| Check running containers | `docker ps`                                |
| Test via curl            | `curl http://<host-ip>:8081`               |

---
