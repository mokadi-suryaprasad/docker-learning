# 09) Data Persistence (Volumes & Bind Mounts)

Containers are **temporary**. When a container stops or is removed, the data inside it is **lost**. To solve this, Docker provides **Volumes** and **Bind Mounts** to store data **permanently**.

---

## Why Data Persistence Is Needed

### Without Persistence
```
Container Filesystem
└── Data (Deleted when container is removed)
```

### With Persistence
```
Container ─── (mapped storage) ─── Host Disk
```

This ensures data **survives container restarts**.

---

# 1) Docker Volumes (Managed by Docker)

Volumes are stored under Docker's data location:
```
/var/lib/docker/volumes/
```

### Diagram (Volumes)

```
        Docker Engine (Host)
        -------------------------
        |   /var/lib/docker/volumes/mydata/  
        -------------------------
                    ▲
                    │
            (Volume Mapping)
                    │
        -------------------------
        | Container (MySQL)     |
        |   /var/lib/mysql      |
        -------------------------
```

### Real-Time Example: MySQL with Persistent Data

```bash
docker volume create mysql_data

docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql
```

#### Verify Data Persists

Create DB:
```bash
docker exec -it mysql mysql -uroot -p
CREATE DATABASE company;
exit;
```

Stop & Remove Container:
```bash
docker stop mysql && docker rm mysql
```

Run again using same volume:
```bash
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql
```

Check DB:
```bash
docker exec -it mysql mysql -uroot -p -e "SHOW DATABASES;"
```
✔ **Database still exists** → Data persisted

---

# 2) Bind Mounts (Direct Host Path Sharing)

A **Bind Mount** maps a **host directory** to a **container directory**.

### Diagram (Bind Mount)

```
 Host System                      Container
-------------------            ---------------------
| /home/user/webapp |  <---->  | /usr/share/nginx/html |
-------------------            ---------------------
```

### Real-Time Example: Live Editing With Nginx

```bash
mkdir webapp
echo "Hello from Docker!" > webapp/index.html

docker run -d -p 8080:80 --name web \
  -v $(pwd)/webapp:/usr/share/nginx/html \
  nginx
```

Open browser:
```
http://localhost:8080
```

Edit host file:
```bash
echo "Updated Content!" > webapp/index.html
```

Refresh browser → change appears ✅

---

# 3) Special Case: Bind Mounting docker.sock (Troubleshooting / CI)

This allows the container to run Docker commands using host's Docker Engine.

### Diagram

```
Container (app1) ---> /var/run/docker.sock ---> Docker Engine (Host)
```

### Example: Run Docker Commands Inside Container

```bash
docker run --rm -d --name app1 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  kiran2361993/troubleshootingtools:v1
```

Enter container:
```bash
docker exec -it app1 bash
docker ps
docker images
```
✔ Works, because it is using **host Docker Engine**

> ⚠ Security Warning: Containers with docker.sock access can control the entire host.

---

# When to Use Which?

| Scenario | Use Volume | Use Bind Mount |
|---------|:----------:|:---------------:|
| Database storage | ✅ Yes | ❌ No |
| Local development (live reload) | ❌ No | ✅ Yes |
| Production | ✅ Recommended | ⚠ Use Carefully |
| Need exact host folder path | ❌ No | ✅ Yes |

---

# Final Summary

| Feature | Volumes | Bind Mounts |
|--------|---------|------------|
| Managed by Docker | ✅ | ❌ |
| Best For | Databases, durable storage | Development, config sharing |
| Path Location | Internal to Docker | Any host path |
| Live file update from host | ❌ No | ✅ Yes |

---
