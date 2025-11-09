# Docker Architecture & Installing Docker & Networking

## 1) What is Docker?

Docker allows applications to run inside **containers**.  
A container bundles the application and everything it needs.

---

## 2) Docker Architecture

### Docker Architecture Diagram — Step‑by‑Step

```
+-----------------+        commands        +-------------------+
|   Docker Client | -------------------->  |  Docker Daemon    |
+-----------------+                        +---------+---------+
                                                      |
                                                      |
                                            manages   | creates containers
                                                      |
                                                      v
                                            +-------------------+
                                            |    Containers     |
                                            +-------------------+
                                                      ^
                                                      |
                                              uses    |
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

## 1) Docker Client (You)
- This is **where you type commands** like `docker run`, `docker build`, `docker ps`.
- Examples:
  ```bash
  docker pull nginx
  docker run -d -p 8080:80 nginx
  docker ps
  ```
- The client sends these commands to the **Docker Daemon**.

## 2) Docker Daemon (Engine)
- A **background service** that listens for requests and **does the heavy work**.
- It **builds images**, **runs containers**, **manages networks/volumes**.
- You don’t interact with it directly; the **client talks to it** over a local socket or TCP.

## 3) Images (Blueprints)
- An **image** is a **read‑only template** with your app and its dependencies.
- You **build** images from a `Dockerfile` or **pull** them from a registry.
- Example:
  ```bash
  docker build -t myapp:1.0 .
  docker images
  ```

## 4) Docker Hub / Registry
- A **registry** is an **online store** of images. **Docker Hub** is the default public one.
- You **pull** (download) images to run them, or **push** (upload) your images to share.
- Examples:
  ```bash
  docker pull python:3.12
  docker push myrepo/myapp:1.0
  ```

## 5) Containers (Running Apps)
- A **container** is a **running instance of an image**.
- Many containers can be created **from the same image**.
- Example:
  ```bash
  docker run -d --name web -p 8080:80 nginx
  docker exec -it web sh
  docker stop web && docker rm web
  ```

## 6) How it all fits together (Flow)
1. **You (Client)** run a command: `docker run nginx`  
2. **Daemon** checks if the **image** exists locally. If not → **pull from Docker Hub**.  
3. Daemon **creates a container** **from the image**.  
4. The container **runs your app** (e.g., NGINX).  
5. You can **see/manage** it with commands like `docker ps`, `docker logs`, `docker stop`.  

## 7) One‑Line Interview Summary
> The **Client** sends commands to the **Daemon**, which **pulls images** from a **registry** and **creates containers** from those images. Images = blueprint; Containers = running apps.


---

## 3)Install Docker on Ubuntu 

### 1) Update system packages
``` bash
sudo apt update
```
### 2) Install Docker
``` bash
sudo apt install -y docker.io
```

### 3) Start Docker service
``` bash
sudo systemctl start docker
```

### 4) Enable Docker to start on boot
``` bash
sudo systemctl enable docker
```

### 5) Check Docker version
``` bash
docker --version
```

### 6) Allow running docker without sudo (optional)
``` bash
sudo usermod -aG docker $USER
```

### *** Logout and login again OR restart the system ***

### Test Docker

``` bash
docker run hello-world
```

---

## Docker Networking Diagram Explanation (Step-by-Step)

## How Containers Connect to Outside Network

When Docker is installed, it creates a **virtual network bridge** called `docker0`.  
This acts like a **small internal switch** inside your machine.

Each container gets a **private IP** and connects to `docker0`.

---

## Step-by-Step Flow

### 1) Containers Receive Private IPs
When containers start, Docker assigns internal IPs:
- C1 → 172.17.0.2
- C2 → 172.17.0.3
- C3 → 172.17.0.4

These IPs **only work inside your computer**.

### 2) Containers Communicate With Each Other
```
C1 → docker0 → C2
C2 → docker0 → C3
C3 → docker0 → C1
```
`docker0` works like a **virtual switch**.

### 3) Container Sends Traffic to Internet
```
Container → docker0 → Host Ethernet → Internet
```
Docker uses **NAT (Network Address Translation)** so:
- The **container’s private IP** becomes
- The **host machine’s IP** when going to the internet

### 4) Internet Sends Data Back
```
Internet → Host → docker0 → Container
```

### 5) To Allow Outside Access to Containers → Use Port Mapping
Example:
```bash
docker run -d -p 8080:80 nginx
```

Meaning:
| Host Port | Container Port |
|----------|----------------|
| 8080     | 80             |

Open in browser:
```
http://localhost:8080
```

---

## Diagram

```
+---------------------------------------------------+
|                   Docker Host                     |
|                                                   |
|   +---------+   +---------+   +---------+         |
|   |   C1    |   |   C2    |   |   C3    |         |
|   +----+----+   +----+----+   +----+----+         |
|        |             |             |              |
|        +-------------+-------------+              |
|                      |                            |
|                 +----v----+                       |
|                 | docker0 |   (Virtual Switch)    |
|                 +----+----+                       |
|                      |                            |
+----------------------|----------------------------+
                       |
                  +----v----+
                  | Ethernet|
                  +----+----+
                       |
                  +----v----+
                  | Internet|
                  +---------+
```

---

## One-Line Interview Answer

> Containers talk to each other through the `docker0` virtual bridge.  
> To reach the internet, Docker uses **NAT**.  
> To allow outside access to a container, we use **port mapping** (`-p hostPort:containerPort`).

---

# Docker Networking — All Types 

## Docker Bridge Network 

## What is the Bridge Network?

When Docker is installed, it automatically creates a built‑in network called **bridge**.  
This network allows containers on the **same machine** to communicate with each other.

But, there are **two** types of bridge networks:
1. **Default bridge** (pre‑created by Docker)
2. **User‑defined bridge** (created by us)

---

## Default Bridge Network (Not Recommended)

- Containers **can communicate**, but usually need to use **IP addresses**.
- Name-to-name communication is **not reliable** by default.
- So, containers talk like this:

```
Container A → http://172.17.0.2
```

This is harder to manage because container IPs can change.

---

## User‑Defined Bridge Network (Recommended)

When we create our own network, Docker enables **built‑in DNS**.

This means containers can talk using **both IP and container names**.

### Create a user‑defined bridge:
```bash
docker network create mynet
```

### Run containers in this network:
```bash
docker run -d --name api --network mynet nginxdemos/hello
docker run -it --rm --network mynet curlimages/curl:8.10.1 curl http://api
```

### Here:
| Access Method | Works? | Example |
|--------------|-------|---------|
| Using IP | ✅ Yes | `curl http://172.18.0.2` |
| Using Container Name | ✅ Yes | `curl http://api` |

This is **simpler** and **more stable**, so recommended for microservices.

---

## Port Mapping in Bridge Network

If you want to access a container from **your laptop / browser**, use `-p`:

```bash
docker run -d --name web -p 8080:80 --network mynet nginx
```

Now open:
```
http://localhost:8080
```

| Host Port | Container Port |
|----------|----------------|
| 8080     | 80             |

---

## One‑Line Interview Answer

> On Docker’s **default bridge**, containers communicate mostly via **IP**.  
> On a **user‑defined bridge**, containers communicate using **both IP and container names**, because Docker enables internal DNS.

---


### Port Mapping
```bash
docker run -d --name web -p 8080:80 --network mynet nginx
# Access: http://localhost:8080
```

---

## Docker Host Network

## What is Host Network?

In **host** network mode, the container **shares the host machine’s network** directly.
This means there is **no virtual network** and **no port mapping is needed**.

The container and the host use **the same network interface and IP**.

---

## Key Points

| Feature | Explanation |
|--------|-------------|
| No `-p` needed | Container directly uses host ports |
| Faster networking | No NAT (no translation) →
 Less overhead |
| Risk of port conflicts | If host port is in use, container cannot bind |
| Good for exporters | Example: Node Exporter, Prometheus Agents |

---

## Example (Node Exporter)

```bash
docker run -d --name node-exp --network host   quay.io/prometheus/node-exporter
```

Now metrics are available at:

```
http://<HOST-IP>:9100/metrics
```

No port mapping needed because container **uses host networking directly**.

---

## Diagram

```
[ Container ] ------ shares ------> [ Host Network ]
(same IP, same ports)
```

---

## One-Line Interview Answer

> Host network mode lets the container share the host’s network directly, so there is **no need for port mapping**, but it can cause **port conflicts**.


## Docker None Network 

## What is None Network?

The **none** network mode gives the container **no external network access**.

Only the **loopback interface (`lo`)** exists inside the container.

---

## Key Points

| Feature | Explanation |
|--------|-------------|
| No internet | Container cannot reach outside |
| No access from outside | You cannot access container from host |
| Maximum isolation | Used for security or offline compute tasks |
| Only `127.0.0.1` works | Only loopback works inside container |

---

## Example

```bash
docker run -it --network none alpine sh
```

Inside the container:

```bash
ip a          # Only `lo` will show
ping google.com  # Will NOT work
```

---

## Diagram

```
[ Container ]
     |
   (lo only)
     |
No external communication
```

---

## One-Line Interview Answer

> None network gives the container **zero network connectivity**, except **loopback**, providing **maximum isolation**.


### Key Point
- Used when isolation is required.

### Example
```bash
docker run -it --network none alpine sh
```

---

## Docker Overlay Network 

## What is Overlay Network?

Overlay network allows containers running on **different Docker hosts** (different machines)
to communicate **as if they are on the same network**.

It creates a **virtual network** that spans across multiple hosts.

---

## Requirement

Overlay network **requires Docker Swarm mode**.

---

## Key Points

| Feature | Explanation |
|--------|-------------|
| Cross‑host communication | Containers on different machines can talk |
| Uses VXLAN encapsulation | Traffic tunneled across hosts |
| Needs Swarm | Must `docker swarm init` first |
| Provides service discovery | Containers communicate by **service name** |

---

## Example Setup

### Step 1: Initialize Swarm
```bash
docker swarm init
```

### Step 2: Create overlay network
```bash
docker network create -d overlay appnet
```

### Step 3: Deploy a service
```bash
docker service create --name web --network appnet -p 8080:80 nginx
```

Now if another node joins swarm and runs a service on `appnet`,
containers can talk **using service names** like:

```
curl http://web
```

---

## Diagram

```
 Host A                          Host B
+----------+                 +----------+
| web svc  | --- overlay --- | api svc  |
+----------+                 +----------+
       \                         /
        \       appnet         /
         \--------------------/
```

---

## One-Line Interview Answer

> Overlay network allows **containers on different machines** to communicate using a **shared virtual network**, and it requires **Docker Swarm**.


---

## 5) macvlan

Gives container an **IP in your LAN network**, like a real device.

### Example
```bash
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 lan-net

docker run -d --name cam --network lan-net nginx
```

Container will have LAN IP like:
```
192.168.1.50
```

---

## Summary Table

| Network | Best Use Case | Can talk by container name? | Needs Port Mapping? |
|--------|---------------|-----------------------------|--------------------|
| bridge | Normal single-host apps | Yes (user-defined bridge) | Yes |
| host | Exporters / avoid NAT overhead | No | No |
| none | Fully isolated containers | No | No |
| overlay | Multi-node container apps | Yes | Sometimes |
| macvlan | Containers need LAN IPs | Yes | No |

---

## 6) Summary

- **Image** = Blueprint
- **Container** = Running app
- **Daemon** = Engine controlling containers
- **Registry** = Storage for images
- **Networks** = How containers communicate
- **Port Mapping** = Allow browser to connect to app

---


