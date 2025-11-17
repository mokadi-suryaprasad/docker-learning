# Docker Troubleshooting Checklist 

## 1. Check if Docker is running
Make sure Docker is actually running.

### Command:
```
docker info
docker version
```

### Common issue:
```
Cannot connect to the Docker daemon
```

### Fix (Linux):
```
sudo systemctl start docker
sudo systemctl status docker
```

### Fix (Mac/Windows):
- Open Docker Desktop
- Restart Docker if needed

---

## 2. Build Errors
Happens when Dockerfile has mistakes or missing files.

### Example error:
```
COPY failed: file not found in build context
```

### Fix:
```
docker build --no-cache -t myapp:latest .
```

---

## 3. Container Starts and Stops Immediately
Check logs:
```
docker ps -a
docker logs <container-id>
```

Example:
```
Error: Port 3000 already in use
```

---

## 4. App Not Opening in Browser
Check ports:
```
docker ps
```

Example:
```
0.0.0.0:3000->80/tcp
```
Open: http://localhost:3000

---

## 5. Permission Issues
```
Permission denied
```

Fix:
```
sudo chown -R $USER:$USER data/
```

---

## 6. Docker Compose Errors
Validate:
```
docker compose config
```

---

## 7. Network Issues
Use service names:
```
DB_HOST=db
```

---

## 8. Image Pull Issues
```
docker login
```

---

## 9. Disk Full
```
docker system prune
docker system prune -a
```

---

## 10. View Logs
```
docker logs -f <container-id>
```

---

## 11. Enter Container
```
docker exec -it <id> bash
```

---

## 12. Restart Services
```
docker compose down
docker compose up --build -d
```

---
