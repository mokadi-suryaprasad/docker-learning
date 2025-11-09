# Dockerfiles for Ruby (Rails)

## When to use
- **Single‑stage**: smaller apps, dev.
- **Multi‑stage**: precompile assets, reduce size.

### Layout
```
app/
├─ Gemfile Gemfile.lock
├─ app/ config/ ...
└─ Dockerfile
```

---

## Single‑Stage
```dockerfile
FROM ruby:3.3-alpine
RUN apk add --no-cache build-base postgresql-dev nodejs yarn
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY . .
EXPOSE 3000
CMD ["bin/rails","server","-b","0.0.0.0","-p","3000"]
```

---

## Multi‑Stage
```dockerfile
FROM ruby:3.3-alpine AS build
RUN apk add --no-cache build-base postgresql-dev nodejs yarn
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY . .
RUN rake assets:precompile

FROM ruby:3.3-alpine
RUN apk add --no-cache postgresql-client nodejs
WORKDIR /app
COPY --from=build /app .
EXPOSE 3000
CMD ["bin/rails","server","-b","0.0.0.0","-p","3000"]
```
