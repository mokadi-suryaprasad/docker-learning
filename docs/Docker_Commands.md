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
