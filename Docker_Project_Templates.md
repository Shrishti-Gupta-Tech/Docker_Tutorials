# 📦 Docker Project Templates
### Production-Ready Templates for Common Tech Stacks

---

## Table of Contents
1. [MERN Stack (MongoDB, Express, React, Node.js)](#1-mern-stack)
2. [Django + PostgreSQL + Redis](#2-django--postgresql--redis)
3. [Spring Boot + MySQL](#3-spring-boot--mysql)
4. [Laravel + MySQL + Redis](#4-laravel--mysql--redis)
5. [Flask + PostgreSQL](#5-flask--postgresql)
6. [Next.js + PostgreSQL](#6-nextjs--postgresql)

---

## 1. MERN Stack

### Project Structure
```
mern-app/
├── docker-compose.yml
├── .dockerignore
├── .env.example
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── frontend/
    ├── Dockerfile
    ├── package.json
    └── src/
```

### Backend Dockerfile
```dockerfile
# backend/Dockerfile
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "run", "dev"]

FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app ./
RUN adduser -D appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 5000
CMD ["node", "server.js"]
```

### Frontend Dockerfile
```dockerfile
# frontend/Dockerfile
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # MongoDB Database
  mongodb:
    image: mongo:7
    container_name: mern-mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USERNAME:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD:-password}
      MONGO_INITDB_DATABASE: ${MONGO_DB:-mernapp}
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    networks:
      - mern-network
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  backend:
    build:
      context: ./backend
      target: ${NODE_ENV:-development}
    container_name: mern-backend
    restart: unless-stopped
    environment:
      NODE_ENV: ${NODE_ENV:-development}
      PORT: 5000
      MONGODB_URI: mongodb://${MONGO_USERNAME:-admin}:${MONGO_PASSWORD:-password}@mongodb:27017/${MONGO_DB:-mernapp}?authSource=admin
      JWT_SECRET: ${JWT_SECRET:-your-secret-key}
      CORS_ORIGIN: ${CORS_ORIGIN:-http://localhost:3000}
    ports:
      - "5000:5000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - mern-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Frontend React App
  frontend:
    build:
      context: ./frontend
      target: ${NODE_ENV:-development}
    container_name: mern-frontend
    restart: unless-stopped
    environment:
      REACT_APP_API_URL: ${REACT_APP_API_URL:-http://localhost:5000}
      WATCHPACK_POLLING: true
      CHOKIDAR_USEPOLLING: true
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    networks:
      - mern-network
    stdin_open: true
    tty: true

volumes:
  mongodb-data:

networks:
  mern-network:
    driver: bridge
```

### .env.example
```env
# Database
MONGO_USERNAME=admin
MONGO_PASSWORD=secure_password_here
MONGO_DB=mernapp

# Backend
NODE_ENV=development
JWT_SECRET=your-super-secret-jwt-key-change-this
CORS_ORIGIN=http://localhost:3000

# Frontend
REACT_APP_API_URL=http://localhost:5000
```

### nginx.conf (for production frontend)
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

### .dockerignore
```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
.DS_Store
coverage
.vscode
dist
build
```

---

## 2. Django + PostgreSQL + Redis

### Project Structure
```
django-app/
├── docker-compose.yml
├── .dockerignore
├── .env.example
├── Dockerfile
├── requirements.txt
├── manage.py
└── myproject/
```

### Dockerfile
```dockerfile
FROM python:3.11-slim AS base
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

FROM base AS development
WORKDIR /app
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

FROM base AS production
WORKDIR /app
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -m -u 1000 django
COPY requirements.txt .
RUN pip install -r requirements.txt gunicorn
COPY . .
RUN python manage.py collectstatic --noinput
USER django
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "myproject.wsgi:application"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: django-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-djangodb}
      POSTGRES_USER: ${POSTGRES_USER:-django}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - django-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-django}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: django-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - django-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # Django Application
  web:
    build:
      context: .
      target: ${DJANGO_ENV:-development}
    container_name: django-web
    restart: unless-stopped
    environment:
      DEBUG: ${DEBUG:-True}
      SECRET_KEY: ${SECRET_KEY:-change-me-in-production}
      DATABASE_URL: postgresql://${POSTGRES_USER:-django}:${POSTGRES_PASSWORD:-password}@postgres:5432/${POSTGRES_DB:-djangodb}
      REDIS_URL: redis://redis:6379/0
      ALLOWED_HOSTS: ${ALLOWED_HOSTS:-localhost,127.0.0.1}
    ports:
      - "8000:8000"
    volumes:
      - .:/app
      - static-files:/app/staticfiles
      - media-files:/app/media
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - django-network
    command: >
      sh -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             python manage.py runserver 0.0.0.0:8000"

  # Celery Worker (for async tasks)
  celery:
    build:
      context: .
      target: development
    container_name: django-celery
    restart: unless-stopped
    environment:
      DEBUG: ${DEBUG:-True}
      SECRET_KEY: ${SECRET_KEY:-change-me-in-production}
      DATABASE_URL: postgresql://${POSTGRES_USER:-django}:${POSTGRES_PASSWORD:-password}@postgres:5432/${POSTGRES_DB:-djangodb}
      REDIS_URL: redis://redis:6379/0
    volumes:
      - .:/app
    depends_on:
      - redis
      - postgres
    networks:
      - django-network
    command: celery -A myproject worker -l info

  # Nginx (for production)
  nginx:
    image: nginx:alpine
    container_name: django-nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - static-files:/static:ro
      - media-files:/media:ro
    depends_on:
      - web
    networks:
      - django-network
    profiles:
      - production

volumes:
  postgres-data:
  redis-data:
  static-files:
  media-files:

networks:
  django-network:
    driver: bridge
```

### requirements.txt
```txt
Django==4.2.0
psycopg2-binary==2.9.6
redis==4.5.5
celery==5.2.7
django-environ==0.10.0
gunicorn==20.1.0
pillow==9.5.0
```

---

## 3. Spring Boot + MySQL

### Project Structure
```
spring-boot-app/
├── docker-compose.yml
├── .dockerignore
├── Dockerfile
├── pom.xml
└── src/
```

### Dockerfile
```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine AS production
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN addgroup -g 1000 spring && \
    adduser -D -u 1000 -G spring spring && \
    chown -R spring:spring /app
USER spring
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: springboot-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootpassword}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-springdb}
      MYSQL_USER: ${MYSQL_USER:-springuser}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-password}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - spring-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD:-rootpassword}"]
      interval: 10s
      timeout: 5s
      retries: 5
    command: --default-authentication-plugin=mysql_native_password

  # Spring Boot Application
  app:
    build:
      context: .
      target: production
    container_name: springboot-app
    restart: unless-stopped
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE:-springdb}?useSSL=false&serverTimezone=UTC
      SPRING_DATASOURCE_USERNAME: ${MYSQL_USER:-springuser}
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD:-password}
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
      SPRING_JPA_SHOW_SQL: ${SPRING_JPA_SHOW_SQL:-false}
      SERVER_PORT: 8080
      JAVA_OPTS: -Xmx512m -Xms256m
    ports:
      - "8080:8080"
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - spring-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M

volumes:
  mysql-data:

networks:
  spring-network:
    driver: bridge
```

---

## 4. Laravel + MySQL + Redis

### Dockerfile
```dockerfile
FROM php:8.2-fpm-alpine AS base
RUN apk add --no-cache \
    mysql-client \
    nodejs \
    npm \
    && docker-php-ext-install pdo pdo_mysql

FROM base AS development
WORKDIR /var/www/html
RUN apk add --no-cache $PHPIZE_DEPS \
    && pecl install redis xdebug \
    && docker-php-ext-enable redis xdebug
COPY . .
RUN composer install
EXPOSE 9000
CMD ["php-fpm"]

FROM base AS production
WORKDIR /var/www/html
RUN pecl install redis && docker-php-ext-enable redis
COPY . .
RUN composer install --optimize-autoloader --no-dev \
    && php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache
RUN chown -R www-data:www-data /var/www/html
USER www-data
EXPOSE 9000
CMD ["php-fpm"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # Nginx Web Server
  nginx:
    image: nginx:alpine
    container_name: laravel-nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./:/var/www/html
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - laravel-network

  # PHP-FPM Application
  app:
    build:
      context: .
      target: ${APP_ENV:-development}
    container_name: laravel-app
    restart: unless-stopped
    environment:
      APP_ENV: ${APP_ENV:-local}
      APP_DEBUG: ${APP_DEBUG:-true}
      APP_KEY: ${APP_KEY}
      DB_CONNECTION: mysql
      DB_HOST: mysql
      DB_PORT: 3306
      DB_DATABASE: ${DB_DATABASE:-laravel}
      DB_USERNAME: ${DB_USERNAME:-laravel}
      DB_PASSWORD: ${DB_PASSWORD:-password}
      REDIS_HOST: redis
      REDIS_PASSWORD: null
      REDIS_PORT: 6379
      CACHE_DRIVER: redis
      SESSION_DRIVER: redis
      QUEUE_CONNECTION: redis
    volumes:
      - ./:/var/www/html
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - laravel-network

  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: laravel-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-rootpassword}
      MYSQL_DATABASE: ${DB_DATABASE:-laravel}
      MYSQL_USER: ${DB_USERNAME:-laravel}
      MYSQL_PASSWORD: ${DB_PASSWORD:-password}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - laravel-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: laravel-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - laravel-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # Queue Worker
  queue:
    build:
      context: .
      target: development
    container_name: laravel-queue
    restart: unless-stopped
    volumes:
      - ./:/var/www/html
    depends_on:
      - redis
      - mysql
    networks:
      - laravel-network
    command: php artisan queue:work --sleep=3 --tries=3

volumes:
  mysql-data:
  redis-data:

networks:
  laravel-network:
    driver: bridge
```

---

## 5. Flask + PostgreSQL

### Dockerfile
```dockerfile
FROM python:3.11-slim AS base
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

FROM base AS development
WORKDIR /app
RUN apt-get update && apt-get install -y postgresql-client \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]

FROM base AS production
WORKDIR /app
RUN apt-get update && apt-get install -y postgresql-client \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -m -u 1000 flask
COPY requirements.txt .
RUN pip install -r requirements.txt gunicorn
COPY . .
USER flask
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "wsgi:app"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: flask-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-flaskdb}
      POSTGRES_USER: ${POSTGRES_USER:-flask}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - flask-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-flask}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Flask Application
  web:
    build:
      context: .
      target: ${FLASK_ENV:-development}
    container_name: flask-web
    restart: unless-stopped
    environment:
      FLASK_APP: app.py
      FLASK_ENV: ${FLASK_ENV:-development}
      SECRET_KEY: ${SECRET_KEY:-change-me-in-production}
      DATABASE_URL: postgresql://${POSTGRES_USER:-flask}:${POSTGRES_PASSWORD:-password}@postgres:5432/${POSTGRES_DB:-flaskdb}
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - flask-network
    command: >
      sh -c "flask db upgrade &&
             flask run --host=0.0.0.0"

volumes:
  postgres-data:

networks:
  flask-network:
    driver: bridge
```

---

## 6. Next.js + PostgreSQL

### Dockerfile
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
USER nextjs
EXPOSE 3000
CMD ["npm", "start"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: nextjs-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-nextjsdb}
      POSTGRES_USER: ${POSTGRES_USER:-nextjs}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - nextjs-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-nextjs}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Next.js Application
  app:
    build:
      context: .
      target: production
    container_name: nextjs-app
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${POSTGRES_USER:-nextjs}:${POSTGRES_PASSWORD:-password}@postgres:5432/${POSTGRES_DB:-nextjsdb}
      NEXTAUTH_SECRET: ${NEXTAUTH_SECRET:-change-me}
      NEXTAUTH_URL: ${NEXTAUTH_URL:-http://localhost:3000}
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - nextjs-network

volumes:
  postgres-data:

networks:
  nextjs-network:
    driver: bridge
```

---

## Quick Start Commands

### For any template:

```bash
# 1. Copy template files to your project
# 2. Create .env from .env.example
cp .env.example .env

# 3. Start development environment
docker-compose up -d

# 4. View logs
docker-compose logs -f

# 5. Stop services
docker-compose down

# 6. Production build
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 7. Clean everything
docker-compose down -v
```

---

**End of Docker Project Templates**

These production-ready templates provide a solid foundation for common tech stacks. Customize them according to your specific requirements!
