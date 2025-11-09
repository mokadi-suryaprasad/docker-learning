# Dockerfiles for Java (Spring Boot)

## When to use
- **Single‑stage**: you already built a JAR.
- **Multi‑stage**: build with Maven/Gradle in container.

### Layout
```
app/
├─ pom.xml
├─ src/...
└─ Dockerfile
```

---

## Single‑Stage (prebuilt JAR)
```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

---

## Multi‑Stage (build + run)
```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /src
COPY pom.xml .
RUN mvn -B -DskipTests dependency:go-offline
COPY src ./src
RUN mvn -B -DskipTests package

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /src/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
