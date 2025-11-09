# Building Images (Dockerfile Instructions)

This guide explains **what a Dockerfile is**, how Docker builds images from it, and every common instruction with **practical use cases** and examples.

---

## 1) What is a Dockerfile?

A **Dockerfile** is a plain‑text recipe that tells Docker how to build an **image**. Each line is an **instruction** (e.g., `FROM`, `RUN`, `COPY`). Docker executes them top‑to‑bottom to assemble a **layered** image.

```
Build Context (folder you pass to `docker build`)
├── Dockerfile        <-- instructions
├── app/              <-- source code, assets
└── requirements.txt  <-- files you COPY into the image
```

**Key concepts:**
- **Layered filesystem:** Each instruction adds a layer. Layers are cached and reused to speed up builds.
- **Build context:** Everything under the build directory is sent to the Docker daemon. Use **.dockerignore** to exclude big/unnecessary files.
- **Image vs Container:** An image is a template; a container is a running instance of that image.

---

## 2) The Build Process (at a glance)

1. You run: `docker build -t myapp:1.0 .`
2. Docker reads **Dockerfile** in the current directory (`.` = context).
3. For each instruction, Docker creates a **layer**; if the inputs didn’t change, Docker reuses the **cache**.
4. Final output is an **image** you can run: `docker run myapp:1.0`.

**Simple diagram:**

```
Dockerfile  -->  Layers  -->  Image  -->  Container
   FROM         (cache)       myapp:1.0      (runtime)
   RUN
   COPY
   CMD
```

---

## 3) Most‑Used Dockerfile Instructions (with use cases)

### 3.1 `FROM`
Sets the **base image**.
```dockerfile
FROM python:3.12-slim
```
**Use case:** Choose a runtime or OS base (e.g., Debian slim, Alpine, distroless). Always prefer **slim** or **alpine** to reduce size.

> Tip: You can specify platform: `FROM --platform=$BUILDPLATFORM node:20-alpine` (helpful on Apple Silicon).

---

### 3.2 `ARG`
Build‑time variables. Available **only during build** (not at runtime).
```dockerfile
ARG NODE_VERSION=20-alpine
FROM node:${NODE_VERSION}
ARG APP_ENV=prod
RUN echo "Building for $APP_ENV"
```
**Use case:** Toggle versions or optional packages: `docker build --build-arg APP_ENV=dev .`

---

### 3.3 `LABEL`
Metadata (key/value) for images.
```dockerfile
LABEL maintainer="you@example.com" \
      org.opencontainers.image.source="https://github.com/user/repo"
```
**Use case:** Ownership, links, scanners, SBOM tooling.

---

### 3.4 `RUN`
Execute a command **at build time**; result is baked into the image layer.
```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```
**Use case:** Install OS packages, compile assets. Combine commands to reduce layers and clean caches for smaller images.

> Shell vs Exec form:
> - Shell (uses `/bin/sh -c`): `RUN echo hi`
> - Exec (no shell processing): `RUN ["bash","-lc","echo hi"]`

---

### 3.5 `COPY` (vs `ADD`)
Copies files from **build context** into the image.
```dockerfile
COPY requirements.txt ./
COPY app/ /app/
```
**Use case:** Bring in code, configs.  
**Prefer `COPY`**. Use `ADD` **only** when you need its extras (auto‑extract tar, remote URLs).

---

### 3.6 `WORKDIR`
Sets the working directory for subsequent instructions.
```dockerfile
WORKDIR /app
COPY . .
```
**Use case:** Cleaner paths; avoids `cd` in `RUN`.

---

### 3.7 `ENV`
Sets environment variables **in the image** (available at runtime).
```dockerfile
ENV PORT=8080 APP_ENV=production
```
**Use case:** Defaults that containers can override via `-e` flags.

---

### 3.8 `EXPOSE`
**Documentation** of intended ports (does not publish them).
```dockerfile
EXPOSE 8080
```
**Use case:** Helps others know which ports to map (`-p 8080:8080`).

---

### 3.9 `USER`
Set default user for subsequent instructions and at runtime.
```dockerfile
# create non-root user and switch
RUN useradd -m appuser
USER appuser
```
**Use case:** Security hardening (avoid running as root).

---

### 3.10 `ENTRYPOINT` and `CMD`
Define the default **process** and **arguments**.

- `ENTRYPOINT` = **what** to run
- `CMD` = **default args** (can be overridden with `docker run ... <args>`)

```dockerfile
ENTRYPOINT ["python","-m","uvicorn","main:app"]
CMD ["--host","0.0.0.0","--port","8080"]
```
**Use case:** Provide a stable command (`ENTRYPOINT`) and flexible args (`CMD`).

> If you use only `CMD ["..."]`, that entire command can be overridden by `docker run IMAGE other_command`.

---

### 3.11 `HEALTHCHECK`
Tell Docker how to check if the container is healthy.
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
**Use case:** Orchestrators can react to unhealthy containers.

---

### 3.12 `VOLUME`
Declare a mount point that **should** be a volume.
```dockerfile
VOLUME ["/var/lib/data"]
```
**Use case:** Remind operators to persist data. (You can still bind‑mount if you prefer.)

---

### 3.13 `SHELL`
Change default shell for `RUN`.
```dockerfile
SHELL ["/bin/bash","-c"]
```
**Use case:** Complex bash features during build.

---

### 3.14 `ONBUILD` (advanced)
Triggers additional instructions **when your image is used as a base**.
```dockerfile
ONBUILD COPY . /src
ONBUILD RUN make /src
```
**Use case:** Framework‑style base images. Use sparingly; can be surprising.

---

### 3.15 Deprecated: `MAINTAINER`
Use `LABEL maintainer="..."` instead.

---

## 4) End‑to‑End Examples

### 4.1 Python (FastAPI) — production‑ready, multi‑stage
```dockerfile
# syntax=docker/dockerfile:1

FROM python:3.12-slim AS base
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
WORKDIR /app

# layer caching: install deps first
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir -r requirements.txt

# now copy the source
COPY . .

# create a non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8080
ENTRYPOINT ["python","-m","uvicorn","main:app"]
CMD ["--host","0.0.0.0","--port","8080"]
```
**Build & run:**
```bash
docker build -t fastapi-app:1.0 .
docker run -d -p 8080:8080 --name api fastapi-app:1.0
```

---

### 4.2 Node.js (PNPM) — small image, reproducible
```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm i --frozen-lockfile
COPY . .
EXPOSE 3000
CMD ["pnpm","start"]
```
**Use case:** `COPY` lockfile and install deps before app code for better caching.

---

### 4.3 Go — static binary + distroless
```dockerfile
# Build stage
FROM golang:1.23-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /bin/app ./cmd/server

# Runtime stage (very small, no shell)
FROM gcr.io/distroless/base-debian12
USER nonroot:nonroot
COPY --from=build /bin/app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```
**Use case:** Multi‑stage builds produce tiny, secure runtime images.

---

### 4.4 Java (JAR) — layered with Spring Boot
```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
**Tip:** With Spring Boot 3 you can use layered jars to improve cache reuse.

---

## 5) .dockerignore (crucial for fast & secure builds)

Create a `.dockerignore` next to your Dockerfile to keep the context small and avoid leaking secrets:
```
.git
.gitignore
node_modules
dist
venv
.env
*.log
**/__pycache__/
**/.DS_Store
```

Benefits: Faster builds, smaller attack surface, fewer cache busts.

---

## 6) Caching & Reproducibility Tips

- Order instructions from **least** to **most** frequently changed (e.g., install deps before copying app code).
- Use lockfiles (`requirements.txt`, `package-lock.json`, `pnpm-lock.yaml`) to pin versions.
- Clean package caches after install (apt, apk, pip) or use cache mounts:  
  `RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt`
- Pin base images to major versions and update regularly.

---

## 7) Security Best Practices

- Prefer **slim**/alpine/distroless images.
- Run as **non‑root** (`USER appuser`).
- Avoid embedding secrets; pass them at runtime via env/secret stores.
- Scan images (Trivy, Snyk):  
  `trivy image myapp:1.0`

---

## 8) Building & Running (hands‑on)

```bash
# Build with a tag
docker build -t myapp:1.0 .

# Run publishing a port
docker run --rm -d -p 8080:8080 --name myapp myapp:1.0

# View logs
docker logs -f myapp

# Exec into container
docker exec -it myapp sh

# Stop & remove
docker rm -f myapp
```

---

## 9) Troubleshooting

- **“File not found” on COPY:** The file must be **inside** the build context. Check `.dockerignore` too.
- **Cache not used:** Any change above an instruction invalidates the cache for all steps below it.
- **Large image:** Remove build tools in final stage; use multi‑stage; prefer slim bases; clean package caches.
- **Permission errors at runtime:** Ensure correct `USER`, file ownership (`chown`), and volume permissions.

---

## 10) TL;DR

- Dockerfile = **recipe** for building layered images.
- Master these core instructions: `FROM`, `ARG`, `RUN`, `COPY`, `WORKDIR`, `ENV`, `EXPOSE`, `USER`, `ENTRYPOINT`, `CMD`, `HEALTHCHECK`.
- Use **.dockerignore**, **multi‑stage builds**, and **non‑root users** for performance and security.
- Keep layers small, cache smartly, and scan images.

Happy building! 🐳
