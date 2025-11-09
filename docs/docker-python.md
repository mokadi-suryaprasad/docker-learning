# Single-Stage vs Multi-Stage Dockerfile

## Single-Stage Dockerfile
A single-stage Dockerfile builds and runs the application in one image.
Everything (installing packages, building code, running the app) happens in the same layer.

**Result:**
- Easy to write
- But the final image is larger

### Example (Python)
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["python","main.py"]
```

---

## Multi-Stage Dockerfile
A multi-stage Dockerfile uses two or more images.
The first stage builds the app, the final stage keeps only the files needed to run it.

**Result:**
- Final image is smaller and cleaner
- Better for production

### Example (Python)
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app /app
EXPOSE 8080
CMD ["python","main.py"]
```

---

## Simple Visual
```
Single-Stage:
Build + Run in one image → Large Image

Multi-Stage:
Stage 1: Build (with tools)
Stage 2: Copy only final output → Small Image
```

## Interview One-Liner
Single-stage builds everything in one image. Multi-stage builds first, then copies only what's needed, resulting in a smaller production-ready image.
