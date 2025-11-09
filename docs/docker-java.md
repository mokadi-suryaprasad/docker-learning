# Dockerfiles for Java (Spring Boot)

## When to Use
- **Single‑stage**: Use when you have already built the JAR (via `mvn package`).
- **Multi‑stage**: Use when you want Docker to build the JAR (recommended for CI/CD & Production).

---

## Project Layout
```
app/
├─ pom.xml
├─ src/
│   └─ main/java/...
└─ Dockerfile
```

---

## Before Single‑Stage Build
Run this locally:
```bash
mvn clean package -DskipTests
```

This creates:
```
target/app.jar
```

---

## Single‑Stage Dockerfile (Simple but Larger Image)
```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /appx
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

**Build & Run**
```bash
docker build -t spring-app:single .
docker run -d -p 8080:8080 --name spring-single spring-app:single
```

---

## Multi‑Stage Dockerfile (Recommended for CI/CD & Production)
```dockerfile
# Stage 1: Build the JAR
FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /app

# Cache dependencies
COPY pom.xml .
RUN mvn -B -DskipTests dependency:go-offline

# Copy source code and build
COPY src ./src
RUN mvn -B -DskipTests package

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

**Build & Run**
```bash
docker build -t spring-app:multi .
docker run -d -p 8080:8080 --name spring-multi spring-app:multi
```

---

## Comparison Summary
| Feature | Single‑Stage | Multi‑Stage |
|--------|-------------|-------------|
| Build inside Docker | ❌ No | ✅ Yes |
| Image Size | Larger | Smaller |
| Best For | Local testing | Production / CI/CD |
| Requires running `mvn package` first | ✅ Yes | ❌ No |

---

## Interview One‑Liner
> Single‑stage uses an already built JAR, while multi‑stage builds the JAR inside Docker and produces a smaller final image.
