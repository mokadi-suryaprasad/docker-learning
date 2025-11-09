# Docker Architecture and Installing Docker & Networking

## 1) What is Docker?

Docker allows applications to run inside **containers**.  
A container bundles the application and everything it needs.

---

## 2) Docker Architecture (With Easy Diagrams)

### Components:

| Component | Meaning |
|----------|---------|
| **Docker Client** | Where you type commands (`docker run`) |
| **Docker Daemon** | Background service that runs containers |
| **Image** | Blueprint of the application |
| **Container** | Running instance of the image |
| **Registry** | Online image storage (e.g., Docker Hub) |

### Diagram

```
+-----------------+        commands        +-------------------+
|   Docker Client  | --------------------> |  Docker Daemon     |
+-----------------+                        +---------+---------+
                                                      |
                                                      |
                                            manages    | creates containers
                                                      |
                                                      v
                                            +-------------------+
                                            |    Containers     |
                                            +-------------------+
                                                      ^
                                                      |
                                              uses     |
                                                      |
                                            +-------------------+
                                            |     Images        |
                                            +-------------------+
                                                      ^
                                                      |
                                            pulled from registry
                                                      |
                                            +-------------------+
                                            |   Docker Hub      |
                                            +-------------------+
```

---

## 3) Installing Docker (Ubuntu Example)

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```

---

## 4) Docker Networking (With Diagrams)

When Docker installs, it creates a **virtual network bridge** (`docker0`).

### Diagram

```
+----------------------------+
|        Your Machine        |
|                            |
|   +--------------------+   |
|   |    docker0 Bridge   | (Virtual Switch)
|   +---------+----------+   |
|             |              |
|     +-------+-------+      |
|     |               |      |
| +---v---+     +-----v---+  |
| | Cont1 |     |  Cont2  |  |
| +-------+     +---------+  |
| 172.17.0.2     172.17.0.3  |
+----------------------------+
```

Containers get **private IPs**, but cannot be reached directly from outside your machine.

### To access containers, use **port mapping**:

```bash
docker run -d -p 8080:80 nginx
```

| Host port | Container port |
|----------|----------------|
| 8080     | 80             |

Open:
```
http://localhost:8080
```

---

## 5) Docker Network Types

| Network | What It Does |
|--------|--------------|
| **bridge** (default) | Containers talk to each other internally |
| **host** | Shares host network directly (no port mapping needed) |
| **none** | No network at all |

---

## 6) Summary

- **Image** = Blueprint
- **Container** = Running app
- **Daemon** = Engine controlling containers
- **Registry** = Storage for images
- **Networks** = How containers communicate
- **Port Mapping** = Allow browser to connect to app

---

End of Document.
