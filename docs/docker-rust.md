# Dockerfiles for Rust

## When to use
- **Single‑stage**: quick tests.
- **Multi‑stage**: small final image.

### Layout
```
app/
├─ Cargo.toml
├─ Cargo.lock
├─ src/main.rs
└─ Dockerfile
```

---

## Single‑Stage
```dockerfile
FROM rust:1.81-alpine
WORKDIR /src
COPY . .
RUN cargo build --release
EXPOSE 8080
CMD ["./target/release/app"]
```

---

## Multi‑Stage (distroless)
```dockerfile
FROM rust:1.81-alpine AS build
RUN apk add --no-cache musl-dev
WORKDIR /src
COPY . .
RUN cargo build --release

FROM gcr.io/distroless/cc
COPY --from=build /src/target/release/app /app
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/app"]
```
