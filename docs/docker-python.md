# Dockerfiles for Python (Single‑Stage & Multi‑Stage)

## When to use
- **Single‑stage**: dev, quick prototypes.
- **Multi‑stage**: prod, smaller images, faster deploys.

### Project layout (example)
```
app/
├─ main.py
├─ requirements.txt
└─ Dockerfile
```

---

## Single‑Stage (FastAPI/Flask style)
```dockerfile
# Dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
ENV PORT=8080
EXPOSE 8080
# Example: uvicorn main:app for FastAPI
CMD ["python","-m","uvicorn","main:app","--host","0.0.0.0","--port","8080"]
```

**Build & run**
```bash
docker build -t py-app:dev .
docker run -d -p 8080:8080 --name py py-app:dev
```

---

## Multi‑Stage (smaller runtime)
```dockerfile
# syntax=docker/dockerfile:1

FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir -r requirements.txt
COPY . .

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app /app
ENV PORT=8080
EXPOSE 8080
CMD ["python","-m","uvicorn","main:app","--host","0.0.0.0","--port","8080"]
```

**Why better?** Keeps only what you need at runtime.
