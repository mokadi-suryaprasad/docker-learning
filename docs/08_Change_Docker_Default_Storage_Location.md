# 08) Change Docker Default Storage Location

Docker stores images, containers, layers, and volumes in its *data-root* directory. By default this is:

```
/var/lib/docker
```

If the root filesystem is small and gets full, Docker stops working. To avoid this, move Docker storage to a larger disk such as `/data/docker`.

---

## Step-by-Step Real-Time Example (Linux)

### 1. Stop Docker Services
```bash
sudo systemctl stop docker
sudo systemctl stop containerd
```

### 2. Create a New Directory on Larger Disk
```bash
sudo mkdir -p /data/docker
```

### 3. Copy Existing Docker Data to New Location
```bash
sudo rsync -aHAX /var/lib/docker/ /data/docker/
```

### 4. Update Docker Configuration
Edit or create:
```bash
sudo nano /etc/docker/daemon.json
```

Add:
```json
{
  "data-root": "/data/docker"
}
```

### 5. Restart Docker
```bash
sudo systemctl daemon-reload
sudo systemctl start containerd
sudo systemctl start docker
```

### 6. Verify New Storage Path
```bash
docker info | grep "Docker Root Dir"
```

Expected output:
```
Docker Root Dir: /data/docker
```

---

## Why Use This Method?
- Prevents root partition from filling up
- Keeps Docker data intact
- Works in production environments

---

## Rollback (If Needed)
If Docker fails to start, revert:
```bash
sudo nano /etc/docker/daemon.json
# Change data-root back to /var/lib/docker
sudo systemctl restart docker
```

---

Done ✅
Docker storage is now moved safely.
