# Dockerfiles for PHP (Apache)

## When to use
- **Single‑stage**: simple PHP apps.
- **Multi‑stage**: composer build separate from runtime.

### Layout
```
app/
├─ composer.json
├─ public/index.php
└─ Dockerfile
```

---

## Single‑Stage
```dockerfile
FROM php:8.3-apache
WORKDIR /var/www/html
COPY . .
RUN docker-php-ext-install pdo_mysql
EXPOSE 80
```

---

## Multi‑Stage (composer vendors)
```dockerfile
FROM composer:2 AS vendor
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --prefer-dist --no-interaction --no-progress
COPY . .
RUN composer dump-autoload -o

FROM php:8.3-apache
WORKDIR /var/www/html
COPY --from=vendor /app .
RUN docker-php-ext-install pdo_mysql
EXPOSE 80
```
