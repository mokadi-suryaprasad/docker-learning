# Dockerfiles for Node.js (Single‑Stage & Multi‑Stage)

## When to use
- **Single‑stage**: simple APIs (Express) without build step.
- **Multi‑stage**: Next.js/React SSR or when you build assets.

### Project layout
```
app/
├─ package.json
├─ package-lock.json (or yarn/pnpm lockfile)
├─ src/...
└─ Dockerfile
```

---

## Single‑Stage (Express)
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json* pnpm-lock.yaml* yarn.lock* ./
RUN corepack enable && (pnpm i --frozen-lockfile || npm ci || yarn --frozen-lockfile)
COPY . .
EXPOSE 3000
CMD ["npm","start"]
```

---

## Multi‑Stage (Next.js / build step)
```dockerfile
# Build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json* pnpm-lock.yaml* yarn.lock* ./
RUN corepack enable && (pnpm i --frozen-lockfile || npm ci || yarn --frozen-lockfile)
COPY . .
RUN npm run build || yarn build || pnpm build

# Runtime
FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev || true
# For Next.js adjust path; for Express copy dist/build
COPY --from=build /app .
EXPOSE 3000
CMD ["npm","start"]
```
