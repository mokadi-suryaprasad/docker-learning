# Docker Commands Cheat Sheet

This file provides commonly used Docker commands with simple explanations and real examples.

---

## 1) Check Docker Version
```bash
docker --version
```

## 2) Download an Image
```bash
docker pull nginx
```

## 3) List All Images
```bash
docker images
```

## 4) Run a Container (Example: Nginx)
```bash
docker run -d -p 8080:80 --name my-nginx nginx
```
- `-d` → Run in background
- `-p 8080:80` → Map port
- `--name my-nginx` → Assign name

## 5) List Running Containers
```bash
docker ps
```

## 6) List All Containers (Including Stopped)
```bash
docker ps -a
```

## 7) Stop a Container
```bash
docker stop my-nginx
```

## 8) Start a Container
```bash
docker start my-nginx
```

## 9) Restart a Container
```bash
docker restart my-nginx
```

## 10) View Logs of a Container
```bash
docker logs my-nginx
```

## 11) Enter into a Container Shell
```bash
docker exec -it my-nginx /bin/bash
```

## 12) Remove a Container
```bash
docker rm my-nginx
```

## 13) Remove an Image
```bash
docker rmi nginx
```

## 14) Build Custom Image from Dockerfile

Dockerfile Example:
```
FROM python:3.9
COPY app.py .
CMD ["python", "app.py"]
```

Build Image:
```bash
docker build -t demo-image .
```

Run the Image:
```bash
docker run demo-image
```

## 15) Inspect a Container
```bash
docker inspect my-nginx
```

## 16) Resource Usage of Containers
```bash
docker stats
```

---

# 10) Cleaning Docker Resources

Over time, Docker stores unused containers, images, networks, and cache which occupy disk space. We use prune commands to clean them safely.

---

## Check Disk Usage
```bash
docker system df
```

---

## Basic Cleanup
Removes:
- Stopped containers
- Unused networks
- Dangling images
- Build cache

```bash
docker system prune
```

---

## Full Cleanup (Removes All Unused Images)
```bash
docker system prune -a
```

> Use when disk is almost full.  
> Unused images will need to be downloaded again.

---

## Remove Only Stopped Containers
```bash
docker container prune
```

---

## Remove Only Unused Images
```bash
docker image prune
```

Remove all unused images:
```bash
docker image prune -a
```

---

## Remove Only Unused Volumes (Be Careful)
```bash
docker volume prune
```

> Warning: Volumes may contain important data. Removing them can delete data permanently.

---

## Recommended Cleanup Sequence
```bash
docker system df
docker system prune -f
docker system prune -a -f
docker volume prune -f
```

---

## Summary

| Command | What It Removes | Safe for Data? |
|--------|-----------------|----------------|
| `docker system prune` | Stopped containers, cache, dangling images, unused networks | ✅ Yes |
| `docker system prune -a` | Above + all unused images | ✅ Yes |
| `docker volume prune` | Unused volumes | ❌ Be careful |
| `docker image prune -a` | All unused images | ✅ Yes |

---
