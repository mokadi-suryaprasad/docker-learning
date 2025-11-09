# Dockerfiles for Go (Single‑Stage & Multi‑Stage)

## When to use
- **Single‑stage**: quick demos.
- **Multi‑stage**: tiny, secure runtime (recommended).

### Layout
```
app/
├─ go.mod
├─ go.sum
├─ cmd/server/main.go
└─ Dockerfile
```

---

## Single‑Stage
```dockerfile
FROM golang:1.23-alpine
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o /bin/app ./cmd/server
EXPOSE 8080
CMD ["/bin/app"]
```

---

## Multi‑Stage (distroless)
```dockerfile
FROM golang:1.23-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /bin/app ./cmd/server

FROM gcr.io/distroless/base-debian12
USER nonroot:nonroot
COPY --from=build /bin/app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```
