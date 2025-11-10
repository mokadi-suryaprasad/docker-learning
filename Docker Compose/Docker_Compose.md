# Docker Compose — Simple Explanation with Real‑Time Example

Docker Compose is a tool that allows us to **run multiple containers together** using **one configuration file**.  
Instead of running each container manually, we describe everything inside a **`docker-compose.yml`** file and start all services with one command.

---

## Why Docker Compose?
When building real applications, we rarely work with just one container.  
Example: A Spring Boot application usually needs a **database**.

Without Docker Compose:
- You start app container manually
- You start database container manually
- You connect them manually
- You handle restart manually

With Docker Compose:
- Everything is defined in one YAML file
- Start all services using one command:
  ```bash
  docker compose up -d
  ```

---

## Key Benefits
| Feature | Description |
|--------|-------------|
| Multi‑service setup | Run app + database + cache together |
| Easy networking | Containers can talk to each other by service name |
| Easy restart & scaling | `docker compose up -d --scale app=3` |
| Persistent storage | Databases use volumes to keep data safe |

---

## Basic Commands
```bash
docker compose up -d       # Start containers in background
docker compose down        # Stop and remove containers
docker compose logs -f app # View app logs
docker compose ps          # Show running services
```

---

## Real‑Time Example: Spring Boot + PostgreSQL

### Project Structure
```
project/
├─ src/
├─ pom.xml
└─ Dockerfile
└─ docker-compose.yml
```

### Dockerfile (Single‑Stage: Uses pre‑built JAR)
```dockerfile
FROM openjdk:21-jdk
WORKDIR /app
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml
```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: spring-app
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD: apppassword
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    container_name: postgres-db
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

---

## How to Run
```bash
# 1) Build JAR
mvn clean package -DskipTests

# 2) Start services
docker compose up -d

# 3) Open app
http://localhost:8080
```

---

## Explanation of Networking
- Compose automatically creates a **network**
- The application can connect to the database using the service name `db` (not IP)
- So in Spring Boot:
  ```
  jdbc:postgresql://db:5432/appdb
  ```

---

## One‑Line Interview Answer
> Docker Compose is used to run multi‑container applications by defining all services in a single YAML file. It simplifies deployment, networking, and scaling for development and CI environments.

---

Done! Now you have a clear explanation and a real‑time working example.
