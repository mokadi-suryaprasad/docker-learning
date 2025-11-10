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

FROM maven:3.8.5-openjdk-21

# Set work directory
WORKDIR /app

# Copy project files
COPY pom.xml .
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Expose port
EXPOSE 8080

# Run the application (assumes jar will be created inside target/)
ENTRYPOINT ["java", "-jar", "target/app.jar"]

```

**Build & Run**
```bash
docker build -t spring-app:single .
docker run -d -p 8080:8080 --name spring-single spring-app:single
```

---

## Multi‑Stage Dockerfile (Recommended for CI/CD & Production)
```dockerfile

# ---- Stage 1: Build the JAR using Maven ----
FROM maven:3.8.5-openjdk-21 AS build

WORKDIR /app

# Copy pom.xml and download dependencies (cache optimization)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy the source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# ---- Stage 2: Run with OpenJDK ----
FROM openjdk:21-jdk

WORKDIR /app

# Copy the JAR from the build stage
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]

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
