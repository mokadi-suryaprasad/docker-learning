# 09) Data Persistence (Volumes & Bind Mounts)

Containers are temporary. If they stop, data inside them is lost. To keep data permanently, Docker provides:

| Method | Use Case | Managed By |
|-------|----------|------------|
| **Volume** | Databases, production deployments | Docker |
| **Bind Mount** | Local development, live code changes | You (Host system) |

---

## A) Named Volumes (Recommended for Databases)

### Example: Persistent MySQL Database

```bash
docker volume create mysql_data

docker run -d --name mysql   -e MYSQL_ROOT_PASSWORD=root   -v mysql_data:/var/lib/mysql   mysql
```

### Test Persistence
Create DB:
```bash
docker exec -it mysql mysql -uroot -p
CREATE DATABASE company;
exit;
```

Recreate container:
```bash
docker stop mysql && docker rm mysql

docker run -d --name mysql   -e MYSQL_ROOT_PASSWORD=root   -v mysql_data:/var/lib/mysql   mysql
```

Verify:
```bash
docker exec -it mysql mysql -uroot -p -e "SHOW DATABASES;"
```
You will still see:
```
company
```

✅ **Data persisted successfully.**

---

## B) Bind Mounts (Live Editing From Host)

### Example: Serve Local Website Using Nginx
```bash
mkdir webapp
echo "Hello from Docker!" > webapp/index.html

docker run -d -p 8080:80 --name web   -v $(pwd)/webapp:/usr/share/nginx/html   nginx
```

Now open:
```
http://localhost:8080
```

Edit file:
```bash
echo "Content Updated!" > webapp/index.html
```
Refresh browser → change reflects immediately.

✅ Perfect for development.

---

## When to Use What

| Scenario | Use Volume | Use Bind Mount |
|---------|:----------:|:---------------:|
| Database storage | ✅ Yes | ❌ No |
| Local coding/project editing | ❌ No | ✅ Yes |
| Production environments | ✅ Yes | ⚠️ Use carefully |
| Need exact host path control | ❌ No | ✅ Yes |

---

Data persistence done ✔
