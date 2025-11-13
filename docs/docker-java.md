# Dockerfiles for Java (Spring Boot)

## When to Use
- **Single‑stage**: Use when you have already built the JAR (via `mvn package`).
- **Multi‑stage**: Use when you want Docker to build the JAR (recommended for CI/CD & Production).

# Single-Stage vs Multi-Stage Docker Build  
### (Clear and Interview-Ready Explanation)

---

## 🟦 Single-Stage Docker Build

In a **single-stage build**, the entire process happens inside **one base image**.

This image contains:
- Build dependencies (compilers, SDKs, tools)
- Runtime dependencies
- Source code
- Final build artifacts

### ❗ Why is the image size very large?

Because the final image includes **both**:
- **Build environment** (example: Maven, JDK, Node modules, Go build tools)
- **Runtime environment**

Nothing is removed → the final image becomes **heavy**.

Example:
```dockerfile
FROM maven:3.9-jdk17
COPY . .
RUN mvn package
CMD ["java", "-jar", "target/app.jar"]
```

### ⛔ Size Problem
Final image may be:
- 800MB – 1GB  
Due to unnecessary build tools inside the image.

---

## 🟩 Multi-Stage Docker Build

A multi-stage Docker build uses **two or more stages**.

### **Stage 1 — Build Stage**
- Uses a heavy base image (e.g., `maven:3.9-jdk17`)
- Installs dependencies
- Builds the application

### **Stage 2 — Runtime Stage**
- Uses a lightweight image (e.g., `openjdk:17-jre`, `alpine`, or `distroless`)
- Copies only the **final artifact** (JAR / binary / build output)  
- Does **not** include build tools

### ✔ Why is the final image small?
Because it contains:
- Only runtime environment  
- Only final artifact  

And **excludes**:
- Build dependencies  
- Source code  
- Compilation tools  
- Temporary files & caches  

This reduces image size by **70–90%**.

Example:
```dockerfile
# Stage 1: Build App
FROM maven:3.9-jdk17 AS builder
COPY . .
RUN mvn package

# Stage 2: Run App
FROM openjdk:17-jre
COPY --from=builder /target/app.jar /app.jar
CMD ["java", "-jar", "app.jar"]
```

### ✅ Final Image Benefits
- Very small  
- More secure  
- Faster to pull  
- Faster deployments  
- Best for production  

---

## 🟧 Summary

| Feature | Single-Stage | Multi-Stage |
|--------|--------------|-------------|
| Image Size | Very Large | Much Smaller |
| Contains Build Tools? | ✔ Yes | ❌ No |
| Production Ready | ❌ No | ✔ Yes |
| Security | Low | High |
| Best Use Case | Local testing | CI/CD & Production |

---

## ⭐ Interview-Ready Answer

> In a single-stage build, the image contains both build and runtime dependencies, so it becomes very large. In a multi-stage build, we use a heavy image only for building the application, and then copy the final artifact into a lightweight runtime image, which makes the final image much smaller and production-ready.


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
CMD ["java", "-jar", "target/app.jar"]

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

CMD ["java", "-jar", "app.jar"]

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
