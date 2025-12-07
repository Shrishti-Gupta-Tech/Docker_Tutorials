# 100 Docker Scenarios: From Basic to Advanced

This comprehensive guide provides 100 distinct Docker scenarios with complete, working Dockerfiles for each.

---

## Table of Contents
- [Basic Scenarios (1-35)](#basic-scenarios-1-35)
- [Intermediate Scenarios (36-70)](#intermediate-scenarios-36-70)
- [Advanced Scenarios (71-100)](#advanced-scenarios-71-100)

---

## Basic Scenarios (1-35)

### Scenario 1: Simple Node.js Application
**Complexity Level:** Basic

**Explanation:** Run a basic Node.js application in a container. Demonstrates fundamental Docker concepts like FROM, COPY, and CMD.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

---

### Scenario 2: Python Flask Application
**Complexity Level:** Basic

**Explanation:** Deploy a simple Flask web application. Shows how to work with Python dependencies using pip.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

---

### Scenario 3: Static HTML Website
**Complexity Level:** Basic

**Explanation:** Serve static HTML files using nginx. Perfect for learning basic web server deployment.

**Dockerfile:**
```dockerfile
FROM nginx:alpine
COPY ./html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Scenario 4: Node.js with Environment Variables
**Complexity Level:** Basic

**Explanation:** Use environment variables to configure the application at runtime.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV PORT=3000
ENV NODE_ENV=production
EXPOSE ${PORT}
CMD ["node", "server.js"]
```

---

### Scenario 5: Python Script Runner
**Complexity Level:** Basic

**Explanation:** Run a standalone Python script in a container. Useful for batch processing or cron jobs.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY script.py .
CMD ["python", "script.py"]
```

---

### Scenario 6: Ruby Sinatra Application
**Complexity Level:** Basic

**Explanation:** Deploy a simple Sinatra web application demonstrating Ruby in Docker.

**Dockerfile:**
```dockerfile
FROM ruby:3.2-alpine
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY . .
EXPOSE 4567
CMD ["ruby", "app.rb"]
```

---

### Scenario 7: Go Binary Application
**Complexity Level:** Basic

**Explanation:** Build and run a simple Go application. Shows compilation within Docker.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main .
EXPOSE 8080
CMD ["./main"]
```

---

### Scenario 8: Java Spring Boot Application
**Complexity Level:** Basic

**Explanation:** Deploy a Spring Boot application using Maven.

**Dockerfile:**
```dockerfile
FROM maven:3.9-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

---

### Scenario 9: PHP Apache Application
**Complexity Level:** Basic

**Explanation:** Run a PHP application with Apache web server.

**Dockerfile:**
```dockerfile
FROM php:8.2-apache
COPY ./src /var/www/html
RUN chown -R www-data:www-data /var/www/html
EXPOSE 80
CMD ["apache2-foreground"]
```

---

### Scenario 10: Node.js with Custom Port
**Complexity Level:** Basic

**Explanation:** Demonstrate port mapping and configuration.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ARG APP_PORT=8080
ENV PORT=${APP_PORT}
EXPOSE ${APP_PORT}
CMD ["node", "app.js"]
```

---

### Scenario 11: Python Data Processing
**Complexity Level:** Basic

**Explanation:** Run data processing scripts with pandas and numpy.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install --no-cache-dir pandas numpy
COPY process_data.py .
COPY data/ ./data/
CMD ["python", "process_data.py"]
```

---

### Scenario 12: Node.js TypeScript Application
**Complexity Level:** Basic

**Explanation:** Build and run a TypeScript Node.js application.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json tsconfig.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

### Scenario 13: React Development Server
**Complexity Level:** Basic

**Explanation:** Run a React development server in a container.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

### Scenario 14: Python Jupyter Notebook
**Complexity Level:** Basic

**Explanation:** Run Jupyter Notebook for data science work.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install --no-cache-dir jupyter pandas numpy matplotlib
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root", "--no-browser"]
```

---

### Scenario 15: Node.js Express API
**Complexity Level:** Basic

**Explanation:** Simple REST API with Express framework.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV PORT=3000
EXPOSE 3000
CMD ["node", "api.js"]
```

---

### Scenario 16: Python Django Application
**Complexity Level:** Basic

**Explanation:** Deploy a Django web application.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python manage.py collectstatic --noinput
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

---

### Scenario 17: Angular Production Build
**Complexity Level:** Basic

**Explanation:** Build Angular app and serve with nginx.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist/my-app /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Scenario 18: Vue.js Production Build
**Complexity Level:** Basic

**Explanation:** Build Vue.js application and serve statically.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Scenario 19: Perl Script Runner
**Complexity Level:** Basic

**Explanation:** Run Perl scripts in a containerized environment.

**Dockerfile:**
```dockerfile
FROM perl:5.38-slim
WORKDIR /app
COPY cpanfile .
RUN cpanm --installdeps .
COPY . .
CMD ["perl", "script.pl"]
```

---

### Scenario 20: Rust Application
**Complexity Level:** Basic

**Explanation:** Build and run a Rust application.

**Dockerfile:**
```dockerfile
FROM rust:1.75-slim AS build
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=build /app/target/release/app /usr/local/bin/app
CMD ["app"]
```

---

### Scenario 21: Node.js with Health Check
**Complexity Level:** Basic

**Explanation:** Add health check endpoint for container monitoring.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js
CMD ["node", "server.js"]
```

---

### Scenario 22: Python FastAPI Application
**Complexity Level:** Basic

**Explanation:** Deploy a modern FastAPI application.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### Scenario 23: .NET Core Application
**Complexity Level:** Basic

**Explanation:** Run a .NET Core web application.

**Dockerfile:**
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/out .
EXPOSE 80
CMD ["dotnet", "MyApp.dll"]
```

---

### Scenario 24: Elixir Phoenix Application
**Complexity Level:** Basic

**Explanation:** Deploy an Elixir Phoenix web framework application.

**Dockerfile:**
```dockerfile
FROM elixir:1.15-alpine
WORKDIR /app
RUN mix local.hex --force && mix local.rebar --force
COPY mix.exs mix.lock ./
RUN mix deps.get
COPY . .
RUN mix compile
EXPOSE 4000
CMD ["mix", "phx.server"]
```

---

### Scenario 25: Swift Vapor Application
**Complexity Level:** Basic

**Explanation:** Run a Vapor web framework application in Swift.

**Dockerfile:**
```dockerfile
FROM swift:5.9-focal
WORKDIR /app
COPY Package.* ./
RUN swift package resolve
COPY . .
RUN swift build -c release
EXPOSE 8080
CMD [".build/release/Run", "serve", "--hostname", "0.0.0.0", "--port", "8080"]
```

---

### Scenario 26: Clojure Application
**Complexity Level:** Basic

**Explanation:** Run a Clojure application with Leiningen.

**Dockerfile:**
```dockerfile
FROM clojure:temurin-21-lein-alpine
WORKDIR /app
COPY project.clj .
RUN lein deps
COPY . .
RUN lein uberjar
EXPOSE 3000
CMD ["java", "-jar", "target/app-standalone.jar"]
```

---

### Scenario 27: Scala Application
**Complexity Level:** Basic

**Explanation:** Build and run a Scala application using sbt.

**Dockerfile:**
```dockerfile
FROM hseeberger/scala-sbt:11.0.16.1_1.8.2_2.13.10
WORKDIR /app
COPY build.sbt .
COPY project ./project
RUN sbt update
COPY . .
RUN sbt compile stage
EXPOSE 9000
CMD ["./target/universal/stage/bin/app"]
```

---

### Scenario 28: Haskell Application
**Complexity Level:** Basic

**Explanation:** Build a Haskell application using Stack.

**Dockerfile:**
```dockerfile
FROM haskell:9.4-slim AS build
WORKDIR /app
COPY stack.yaml package.yaml ./
RUN stack setup
COPY . .
RUN stack build --copy-bins --local-bin-path /app/bin

FROM debian:bookworm-slim
COPY --from=build /app/bin/app /usr/local/bin/app
CMD ["app"]
```

---

### Scenario 29: R Shiny Application
**Complexity Level:** Basic

**Explanation:** Deploy an R Shiny dashboard application.

**Dockerfile:**
```dockerfile
FROM rocker/r-ver:4.3.2
RUN R -e "install.packages(c('shiny', 'dplyr', 'ggplot2'), repos='https://cran.rstudio.com/')"
WORKDIR /app
COPY app.R .
EXPOSE 3838
CMD ["R", "-e", "shiny::runApp('/app', host='0.0.0.0', port=3838)"]
```

---

### Scenario 30: Node.js with npm Cache
**Complexity Level:** Basic

**Explanation:** Optimize builds by leveraging npm cache layers.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

---

### Scenario 31: Python with pip Cache
**Complexity Level:** Basic

**Explanation:** Use pip caching for faster rebuilds.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install --no-cache-dir --upgrade pip
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

### Scenario 32: Node.js with Yarn
**Complexity Level:** Basic

**Explanation:** Use Yarn package manager instead of npm.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
RUN corepack enable
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --frozen-lockfile
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

---

### Scenario 33: Node.js with pnpm
**Complexity Level:** Basic

**Explanation:** Use pnpm for efficient package management.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

---

### Scenario 34: MongoDB Backup Script
**Complexity Level:** Basic

**Explanation:** Run a MongoDB backup utility in a container.

**Dockerfile:**
```dockerfile
FROM mongo:7.0
WORKDIR /backup
COPY backup-script.sh .
RUN chmod +x backup-script.sh
CMD ["./backup-script.sh"]
```

---

### Scenario 35: Redis CLI Container
**Complexity Level:** Basic

**Explanation:** Create a container with Redis CLI tools for debugging.

**Dockerfile:**
```dockerfile
FROM redis:7-alpine
COPY redis-scripts/ /scripts/
WORKDIR /scripts
CMD ["redis-cli", "--help"]
```

---

## Intermediate Scenarios (36-70)

### Scenario 36: Multi-Stage Node.js Build
**Complexity Level:** Intermediate

**Explanation:** Use multi-stage builds to reduce final image size by separating build dependencies from runtime.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

### Scenario 37: Python with .dockerignore
**Complexity Level:** Intermediate

**Explanation:** Optimize build context by excluding unnecessary files with .dockerignore.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python -m pytest
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**.dockerignore:**
```
__pycache__/
*.py[cod]
*$py.class
*.so
.pytest_cache/
.coverage
htmlcov/
.git/
.gitignore
README.md
tests/
docs/
```

---

### Scenario 38: Node.js with Volume Mounting for Development
**Complexity Level:** Intermediate

**Explanation:** Use volumes for hot-reloading during development.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
VOLUME ["/app/node_modules"]
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

---

### Scenario 39: Multi-Stage Go Build with Alpine
**Complexity Level:** Intermediate

**Explanation:** Create minimal Go binary image using multi-stage builds.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

---

### Scenario 40: Python with Build Arguments
**Complexity Level:** Intermediate

**Explanation:** Use build arguments to customize image build.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
ARG ENVIRONMENT=production
ARG APP_VERSION=1.0.0
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV APP_ENV=${ENVIRONMENT}
ENV VERSION=${APP_VERSION}
LABEL version="${APP_VERSION}"
LABEL environment="${ENVIRONMENT}"
EXPOSE 8000
CMD ["python", "app.py"]
```

---

### Scenario 41: Node.js with Custom Network
**Complexity Level:** Intermediate

**Explanation:** Configure container to use custom Docker networks.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
ENV DATABASE_HOST=db
ENV REDIS_HOST=redis
EXPOSE 3000
CMD ["node", "server.js"]
```

---

### Scenario 42: React with nginx Custom Configuration
**Complexity Level:** Intermediate

**Explanation:** Deploy React app with custom nginx configuration for SPA routing.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}
```

---

### Scenario 43: Python with Secrets Management
**Complexity Level:** Intermediate

**Explanation:** Securely handle secrets using Docker secrets.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8000
CMD ["python", "app.py"]
```

---

### Scenario 44: Java with Layered JARs
**Complexity Level:** Intermediate

**Explanation:** Optimize Java builds with layered JAR approach.

**Dockerfile:**
```dockerfile
FROM maven:3.9-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests spring-boot:repackage

FROM openjdk:17-jre-slim
WORKDIR /app
ARG JAR_FILE=target/*.jar
COPY --from=build /app/${JAR_FILE} app.jar
RUN java -Djarmode=layertools -jar app.jar extract
EXPOSE 8080
CMD ["java", "org.springframework.boot.loader.JarLauncher"]
```

---

### Scenario 45: Node.js with Health Check and Metrics
**Complexity Level:** Intermediate

**Explanation:** Implement health checks and expose metrics endpoint.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
EXPOSE 9090
HEALTHCHECK --interval=10s --timeout=3s --start-period=30s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
CMD ["node", "server.js"]
```

---

### Scenario 46: Python with Multiple Stages for Testing
**Complexity Level:** Intermediate

**Explanation:** Separate testing stage from production build.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS test
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt
COPY . .
RUN pytest

FROM base AS production
COPY . .
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "wsgi:app"]
```

---

### Scenario 47: Go with Scratch Base Image
**Complexity Level:** Intermediate

**Explanation:** Create minimal Go image using scratch base for maximum size reduction.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o main .

FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/main /main
EXPOSE 8080
CMD ["/main"]
```

---

### Scenario 48: Node.js with npm Audit
**Complexity Level:** Intermediate

**Explanation:** Include security scanning in build process.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS audit
WORKDIR /app
COPY package*.json ./
RUN npm audit --audit-level=high

FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=build /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

### Scenario 49: Python with Distroless Base
**Complexity Level:** Intermediate

**Explanation:** Use Google's distroless images for minimal attack surface.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/dependencies -r requirements.txt

FROM gcr.io/distroless/python3-debian12
COPY --from=builder /app/dependencies /app/dependencies
ENV PYTHONPATH=/app/dependencies
WORKDIR /app
COPY . .
EXPOSE 8000
CMD ["app.py"]
```

---

### Scenario 50: Next.js with Standalone Output
**Complexity Level:** Intermediate

**Explanation:** Optimize Next.js deployment with standalone output mode.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

---

### Scenario 51: Django with PostgreSQL
**Complexity Level:** Intermediate

**Explanation:** Django application configured to connect to PostgreSQL database.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python manage.py collectstatic --noinput
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "3", "myproject.wsgi:application"]
```

---

### Scenario 52: Ruby on Rails with Asset Precompilation
**Complexity Level:** Intermediate

**Explanation:** Rails app with proper asset pipeline setup.

**Dockerfile:**
```dockerfile
FROM ruby:3.2-alpine AS builder
RUN apk add --no-cache build-base postgresql-dev nodejs yarn
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install --without development test
COPY . .
RUN RAILS_ENV=production bundle exec rake assets:precompile

FROM ruby:3.2-alpine
RUN apk add --no-cache postgresql-client
WORKDIR /app
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /app ./
EXPOSE 3000
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

---

### Scenario 53: Rust with Cargo Chef for Layer Caching
**Complexity Level:** Intermediate

**Explanation:** Optimize Rust builds with cargo-chef for better layer caching.

**Dockerfile:**
```dockerfile
FROM rust:1.75-slim AS chef
RUN cargo install cargo-chef
WORKDIR /app

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/app /usr/local/bin/app
CMD ["app"]
```

---

### Scenario 54: Node.js with PM2 Process Manager
**Complexity Level:** Intermediate

**Explanation:** Use PM2 for production-grade Node.js process management.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
RUN npm install -g pm2
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["pm2-runtime", "start", "ecosystem.config.js"]
```

---

### Scenario 55: PHP-FPM with nginx
**Complexity Level:** Intermediate

**Explanation:** PHP application with PHP-FPM and nginx separation.

**Dockerfile:**
```dockerfile
FROM php:8.2-fpm-alpine
RUN docker-php-ext-install pdo pdo_mysql mysqli
WORKDIR /var/www/html
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader
COPY . .
RUN chown -R www-data:www-data /var/www/html
EXPOSE 9000
CMD ["php-fpm"]
```

---

### Scenario 56: Angular with Build Optimization
**Complexity Level:** Intermediate

**Explanation:** Optimized Angular production build with custom nginx config.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production --output-hashing=all

FROM nginx:alpine
COPY --from=build /app/dist/my-app /usr/share/nginx/html
COPY nginx-custom.conf /etc/nginx/conf.d/default.conf
RUN apk add --no-cache curl
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/ || exit 1
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Scenario 57: Python with Celery Worker
**Complexity Level:** Intermediate

**Explanation:** Celery worker container for asynchronous task processing.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -m celeryuser && chown -R celeryuser:celeryuser /app
USER celeryuser
CMD ["celery", "-A", "tasks", "worker", "--loglevel=info"]
```

---

### Scenario 58: Gradle-based Java Application
**Complexity Level:** Intermediate

**Explanation:** Build Java app using Gradle instead of Maven.

**Dockerfile:**
```dockerfile
FROM gradle:8.5-jdk17-alpine AS build
WORKDIR /app
COPY build.gradle settings.gradle ./
COPY gradle ./gradle
RUN gradle dependencies --no-daemon
COPY src ./src
RUN gradle build --no-daemon -x test

FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
CMD ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

---

### Scenario 59: Node.js Microservice with gRPC
**Complexity Level:** Intermediate

**Explanation:** Node.js gRPC microservice with protocol buffers.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
RUN apk add --no-cache python3 make g++
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY proto ./proto
COPY . .
EXPOSE 50051
CMD ["node", "grpc-server.js"]
```

---

### Scenario 60: Python with SQLite Database
**Complexity Level:** Intermediate

**Explanation:** Python app with embedded SQLite database and volume persistence.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN mkdir -p /app/data
VOLUME ["/app/data"]
ENV DATABASE_PATH=/app/data/app.db
EXPOSE 8000
CMD ["python", "app.py"]
```

---

### Scenario 61: Go with Wire Dependency Injection
**Complexity Level:** Intermediate

**Explanation:** Go application using Wire for compile-time dependency injection.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine AS builder
RUN go install github.com/google/wire/cmd/wire@latest
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN wire gen ./...
RUN CGO_ENABLED=0 go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/main /main
EXPOSE 8080
CMD ["/main"]
```

---

### Scenario 62: NestJS Application
**Complexity Level:** Intermediate

**Explanation:** NestJS TypeScript framework with production optimization.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=development /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/main"]
```

---

### Scenario 63: Laravel PHP Application
**Complexity Level:** Intermediate

**Explanation:** Laravel framework with proper optimization and caching.

**Dockerfile:**
```dockerfile
FROM php:8.2-fpm-alpine AS base
RUN docker-php-ext-install pdo pdo_mysql
WORKDIR /var/www/html
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

FROM base AS builder
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-autoloader
COPY . .
RUN composer dump-autoload --optimize

FROM base AS production
COPY --from=builder /var/www/html .
RUN php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache
RUN chown -R www-data:www-data /var/www/html
EXPOSE 9000
CMD ["php-fpm"]
```

---

### Scenario 64: Svelte/SvelteKit Application
**Complexity Level:** Intermediate

**Explanation:** SvelteKit application with adapter-node.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/build ./build
COPY --from=builder /app/package.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "build"]
```

---

### Scenario 65: Kotlin Spring Boot Application
**Complexity Level:** Intermediate

**Explanation:** Kotlin-based Spring Boot with Gradle.

**Dockerfile:**
```dockerfile
FROM gradle:8.5-jdk17-alpine AS build
WORKDIR /app
COPY build.gradle.kts settings.gradle.kts ./
COPY gradle ./gradle
RUN gradle dependencies --no-daemon
COPY src ./src
RUN gradle bootJar --no-daemon

FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### Scenario 66: Nuxt.js SSR Application
**Complexity Level:** Intermediate

**Explanation:** Nuxt.js with server-side rendering enabled.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/.output ./.output
EXPOSE 3000
ENV NUXT_HOST=0.0.0.0
ENV NUXT_PORT=3000
CMD ["node", ".output/server/index.mjs"]
```

---

### Scenario 67: Express with TypeScript and ESLint
**Complexity Level:** Intermediate

**Explanation:** TypeScript Express API with linting in build process.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json tsconfig.json ./
RUN npm ci
COPY . .
RUN npm run lint
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

### Scenario 68: Python with Multiple Workers
**Complexity Level:** Intermediate

**Explanation:** Python WSGI app with configurable worker count.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ARG WORKERS=4
ENV WORKERS_COUNT=${WORKERS}
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8000
CMD gunicorn --bind 0.0.0.0:8000 --workers ${WORKERS_COUNT} --timeout 60 wsgi:app
```

---

### Scenario 69: Go with Air for Hot Reload
**Complexity Level:** Intermediate

**Explanation:** Development container with Air for hot reloading Go applications.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine
RUN go install github.com/cosmtrek/air@latest
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
EXPOSE 8080
CMD ["air", "-c", ".air.toml"]
```

---

### Scenario 70: Elixir Release with Mix Release
**Complexity Level:** Intermediate

**Explanation:** Production Elixir release using mix release.

**Dockerfile:**
```dockerfile
FROM elixir:1.15-alpine AS builder
WORKDIR /app
ENV MIX_ENV=prod
RUN mix local.hex --force && mix local.rebar --force
COPY mix.exs mix.lock ./
RUN mix deps.get --only prod
COPY config ./config
COPY lib ./lib
COPY priv ./priv
RUN mix compile
RUN mix release

FROM alpine:latest
RUN apk add --no-cache openssl ncurses-libs
WORKDIR /app
COPY --from=builder /app/_build/prod/rel/myapp ./
EXPOSE 4000
CMD ["bin/myapp", "start"]
```

---

## Advanced Scenarios (71-100)

### Scenario 71: Docker Compose Multi-Service Application
**Complexity Level:** Advanced

**Explanation:** Full-stack application with frontend, backend, database, and cache.

**Dockerfile (Backend):**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

---

### Scenario 72: Secured Python Application with Non-Root User
**Complexity Level:** Advanced

**Explanation:** Security-hardened Python container with non-root user and minimal permissions.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM python:3.11-slim
RUN apt-get update && apt-get install -y --no-install-recommends \
    dumb-init \
    && rm -rf /var/lib/apt/lists/*
RUN groupadd -r appgroup && useradd -r -g appgroup -u 1001 appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /root/.local /home/appuser/.local
COPY --chown=appuser:appgroup . .
ENV PATH=/home/appuser/.local/bin:$PATH
USER appuser
EXPOSE 8000
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

---

### Scenario 73: PostgreSQL with Custom Init Scripts
**Complexity Level:** Advanced

**Explanation:** Custom PostgreSQL image with initialization scripts and extensions.

**Dockerfile:**
```dockerfile
FROM postgres:16-alpine
RUN apk add --no-cache postgresql-contrib
COPY init-scripts/ /docker-entrypoint-initdb.d/
COPY custom-postgresql.conf /etc/postgresql/postgresql.conf
RUN chown postgres:postgres /etc/postgresql/postgresql.conf
EXPOSE 5432
CMD ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf"]
```

**init-scripts/01-create-extensions.sql:**
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "btree_gin";
```

---

### Scenario 74: MySQL with Replication Setup
**Complexity Level:** Advanced

**Explanation:** MySQL container configured for replication.

**Dockerfile:**
```dockerfile
FROM mysql:8.0
COPY custom-my.cnf /etc/mysql/conf.d/
COPY init-replication.sh /docker-entrypoint-initdb.d/
RUN chmod +x /docker-entrypoint-initdb.d/init-replication.sh
EXPOSE 3306
```

**custom-my.cnf:**
```
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-format=ROW
gtid-mode=ON
enforce-gtid-consistency=ON
```

---

### Scenario 75: Jenkins CI/CD Pipeline Container
**Complexity Level:** Advanced

**Explanation:** Custom Jenkins image with pre-installed plugins and Docker-in-Docker.

**Dockerfile:**
```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*
USER jenkins
RUN jenkins-plugin-cli --plugins \
    git:latest \
    workflow-aggregator:latest \
    docker-workflow:latest \
    kubernetes:latest \
    blueocean:latest
EXPOSE 8080 50000
```

---

### Scenario 76: GitLab Runner with Docker Executor
**Complexity Level:** Advanced

**Explanation:** GitLab Runner configured for CI/CD with Docker executor.

**Dockerfile:**
```dockerfile
FROM gitlab/gitlab-runner:latest
RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*
COPY config.toml /etc/gitlab-runner/config.toml
VOLUME ["/etc/gitlab-runner", "/home/gitlab-runner"]
CMD ["run", "--user=gitlab-runner", "--working-directory=/home/gitlab-runner"]
```

---

### Scenario 77: Kubernetes Init Container Pattern
**Complexity Level:** Advanced

**Explanation:** Application with init container for database migrations.

**Dockerfile (App):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Dockerfile (Init):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY migrations ./migrations
COPY migrate.js .
CMD ["node", "migrate.js"]
```

---

### Scenario 78: Monitoring Stack with Prometheus and Grafana
**Complexity Level:** Advanced

**Explanation:** Complete monitoring solution with custom metrics.

**Dockerfile (Custom Exporter):**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install --no-cache-dir prometheus-client psutil
COPY exporter.py .
EXPOSE 8000
CMD ["python", "exporter.py"]
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

  exporter:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"

volumes:
  prometheus-data:
  grafana-data:
```

---

### Scenario 79: Elasticsearch Cluster Setup
**Complexity Level:** Advanced

**Explanation:** Multi-node Elasticsearch cluster with custom configuration.

**Dockerfile:**
```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:8.11.0
COPY elasticsearch.yml /usr/share/elasticsearch/config/
RUN elasticsearch-plugin install analysis-icu
EXPOSE 9200 9300
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  es01:
    build: .
    environment:
      - node.name=es01
      - cluster.name=docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es01-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200

  es02:
    build: .
    environment:
      - node.name=es02
      - cluster.name=docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es02-data:/usr/share/elasticsearch/data

  es03:
    build: .
    environment:
      - node.name=es03
      - cluster.name=docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es03-data:/usr/share/elasticsearch/data

volumes:
  es01-data:
  es02-data:
  es03-data:
```

---

### Scenario 80: Apache Kafka with Zookeeper
**Complexity Level:** Advanced

**Explanation:** Kafka message broker setup with Zookeeper coordination.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    volumes:
      - zk-data:/var/lib/zookeeper/data
      - zk-logs:/var/lib/zookeeper/log

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
      - kafka-data:/var/lib/kafka/data

volumes:
  zk-data:
  zk-logs:
  kafka-data:
```

---

### Scenario 81: RabbitMQ with Management Plugin
**Complexity Level:** Advanced

**Explanation:** RabbitMQ message broker with custom configuration and plugins.

**Dockerfile:**
```dockerfile
FROM rabbitmq:3.12-management-alpine
RUN rabbitmq-plugins enable rabbitmq_shovel rabbitmq_shovel_management
COPY rabbitmq.conf /etc/rabbitmq/
COPY definitions.json /etc/rabbitmq/
EXPOSE 5672 15672
```

**rabbitmq.conf:**
```
management.load_definitions = /etc/rabbitmq/definitions.json
vm_memory_high_watermark.relative = 0.6
disk_free_limit.absolute = 2GB
```

---

### Scenario 82: MongoDB Replica Set
**Complexity Level:** Advanced

**Explanation:** MongoDB replica set for high availability.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  mongo1:
    image: mongo:7.0
    command: mongod --replSet rs0 --bind_ip_all
    ports:
      - "27017:27017"
    volumes:
      - mongo1-data:/data/db
    networks:
      - mongo-cluster

  mongo2:
    image: mongo:7.0
    command: mongod --replSet rs0 --bind_ip_all
    ports:
      - "27018:27017"
    volumes:
      - mongo2-data:/data/db
    networks:
      - mongo-cluster

  mongo3:
    image: mongo:7.0
    command: mongod --replSet rs0 --bind_ip_all
    ports:
      - "27019:27017"
    volumes:
      - mongo3-data:/data/db
    networks:
      - mongo-cluster

  mongo-init:
    image: mongo:7.0
    depends_on:
      - mongo1
      - mongo2
      - mongo3
    command: >
      bash -c "
        sleep 10 &&
        mongosh --host mongo1:27017 --eval \"
          rs.initiate({
            _id: 'rs0',
            members: [
              {_id: 0, host: 'mongo1:27017'},
              {_id: 1, host: 'mongo2:27017'},
              {_id: 2, host: 'mongo3:27017'}
            ]
          })\"
      "
    networks:
      - mongo-cluster

volumes:
  mongo1-data:
  mongo2-data:
  mongo3-data:

networks:
  mongo-cluster:
    driver: bridge
```

---

### Scenario 83: Nginx Reverse Proxy with SSL
**Complexity Level:** Advanced

**Explanation:** Nginx as reverse proxy with SSL termination.

**Dockerfile:**
```dockerfile
FROM nginx:alpine
RUN apk add --no-cache openssl
COPY nginx.conf /etc/nginx/nginx.conf
COPY ssl-params.conf /etc/nginx/snippets/
COPY generate-certs.sh /
RUN chmod +x /generate-certs.sh && /generate-certs.sh
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```
events {
    worker_connections 1024;
}

http {
    upstream backend {
        least_conn;
        server backend1:3000;
        server backend2:3000;
        server backend3:3000;
    }

    server {
        listen 80;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;
        
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        include /etc/nginx/snippets/ssl-params.conf;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

---

### Scenario 84: Traefik Reverse Proxy with Auto SSL
**Complexity Level:** Advanced

**Explanation:** Traefik as modern reverse proxy with automatic SSL certificates.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=admin@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt

  app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`example.com`)"
      - "traefik.http.routers.app.entrypoints=websecure"
      - "traefik.http.routers.app.tls.certresolver=myresolver"

volumes:
  letsencrypt:
```

---

### Scenario 85: Full-Stack App with Build Arguments
**Complexity Level:** Advanced

**Explanation:** Application with build-time configuration for different environments.

**Dockerfile:**
```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS builder
ARG API_URL
ARG ENVIRONMENT=production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_PUBLIC_API_URL=${API_URL}
ENV NEXT_PUBLIC_ENV=${ENVIRONMENT}
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

**Build commands:**
```bash
# Development
docker build --build-arg API_URL=http://localhost:3001 --build-arg ENVIRONMENT=development -t app:dev .

# Production
docker build --build-arg API_URL=https://api.production.com --build-arg ENVIRONMENT=production -t app:prod .
```

---

### Scenario 86: Python with Poetry for Dependency Management
**Complexity Level:** Advanced

**Explanation:** Use Poetry for advanced Python dependency management.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim AS builder
ENV POETRY_VERSION=1.7.0
ENV POETRY_HOME=/opt/poetry
ENV POETRY_VENV=/opt/poetry-venv
ENV POETRY_CACHE_DIR=/opt/.cache
RUN python3 -m venv $POETRY_VENV \
    && $POETRY_VENV/bin/pip install -U pip setuptools \
    && $POETRY_VENV/bin/pip install poetry==${POETRY_VERSION}
ENV PATH="${PATH}:${POETRY_VENV}/bin"
WORKDIR /app
COPY poetry.lock pyproject.toml ./
RUN poetry install --no-interaction --no-ansi --no-root --only main
COPY . .
RUN poetry build

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app/dist/*.whl .
RUN pip install --no-cache-dir *.whl
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

---

### Scenario 87: Multi-Architecture Build
**Complexity Level:** Advanced

**Explanation:** Build images for multiple architectures (amd64, arm64).

**Dockerfile:**
```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/main /main
EXPOSE 8080
CMD ["/main"]
```

**Build command:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

---

### Scenario 88: Image Scanning and Security
**Complexity Level:** Advanced

**Explanation:** Security-focused build with vulnerability scanning.

**Dockerfile:**
```dockerfile
FROM python:3.11-slim AS base
RUN apt-get update && apt-get install -y --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

FROM base AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM base AS scanner
COPY --from=aquasec/trivy:latest /usr/local/bin/trivy /usr/local/bin/trivy
COPY --from=builder / /scan-target
RUN trivy filesystem --exit-code 1 --severity HIGH,CRITICAL --no-progress /scan-target

FROM base AS final
RUN groupadd -r appgroup && useradd -r -g appgroup -u 1001 appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /root/.local /home/appuser/.local
COPY --chown=appuser:appgroup . .
USER appuser
ENV PATH=/home/appuser/.local/bin:$PATH
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

---

### Scenario 89: Distroless Production Image
**Complexity Level:** Advanced

**Explanation:** Ultra-minimal production image using Google's distroless.

**Dockerfile:**
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o main .

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder --chown=nonroot:nonroot /app/main /main
EXPOSE 8080
USER nonroot:nonroot
ENTRYPOINT ["/main"]
```

---

### Scenario 90: Docker-in-Docker for CI/CD
**Complexity Level:** Advanced

**Explanation:** CI/CD container that can build Docker images.

**Dockerfile:**
```dockerfile
FROM docker:24-dind
RUN apk add --no-cache \
    git \
    bash \
    curl \
    jq \
    python3 \
    py3-pip \
    nodejs \
    npm
RUN pip3 install --no-cache-dir docker-compose
COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh
VOLUME /var/lib/docker
EXPOSE 2375 2376
ENTRYPOINT ["entrypoint.sh"]
CMD []
```

---

### Scenario 91: Microservices with Service Mesh
**Complexity Level:** Advanced

**Explanation:** Microservice with Istio sidecar injection ready.

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
RUN apk add --no-cache curl
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myservice
  labels:
    app: myservice
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myservice
  template:
    metadata:
      labels:
        app: myservice
        version: v1
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - name: myservice
        image: myservice:latest
        ports:
        - containerPort: 3000
```

---

### Scenario 92: Serverless Container with AWS Lambda
**Complexity Level:** Advanced

**Explanation:** Container optimized for AWS Lambda runtime.

**Dockerfile:**
```dockerfile
FROM public.ecr.aws/lambda/python:3.11
COPY requirements.txt ${LAMBDA_TASK_ROOT}
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py ${LAMBDA_TASK_ROOT}
COPY lib/ ${LAMBDA_TASK_ROOT}/lib/
CMD ["app.lambda_handler"]
```

---

### Scenario 93: Apache Airflow with Custom Operators
**Complexity Level:** Advanced

**Explanation:** Airflow instance with custom operators and plugins.

**Dockerfile:**
```dockerfile
FROM apache/airflow:2.8.0-python3.11
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
USER airflow
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=airflow:root dags/ ${AIRFLOW_HOME}/dags/
COPY --chown=airflow:root plugins/ ${AIRFLOW_HOME}/plugins/
COPY --chown=airflow:root config/airflow.cfg ${AIRFLOW_HOME}/airflow.cfg
EXPOSE 8080
```

---

### Scenario 94: Apache Spark Cluster
**Complexity Level:** Advanced

**Explanation:** Spark cluster with master and worker nodes.

**Dockerfile (Base):**
```dockerfile
FROM openjdk:11-jre-slim
ARG SPARK_VERSION=3.5.0
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*
RUN curl -fSL "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop3.tgz" -o spark.tgz \
    && tar -xvf spark.tgz -C /opt/ \
    && mv /opt/spark-${SPARK_VERSION}-bin-hadoop3 /opt/spark \
    && rm spark.tgz
ENV SPARK_HOME=/opt/spark
ENV PATH=$PATH:$SPARK_HOME/bin
WORKDIR /opt/spark
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  spark-master:
    build: .
    command: /opt/spark/bin/spark-class org.apache.spark.deploy.master.Master
    ports:
      - "8080:8080"
      - "7077:7077"
    environment:
      - SPARK_MODE=master

  spark-worker-1:
    build: .
    command: /opt/spark/bin/spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077
    depends_on:
      - spark-master
    environment:
      - SPARK_MODE=worker
      - SPARK_WORKER_CORES=2
      - SPARK_WORKER_MEMORY=2g

  spark-worker-2:
    build: .
    command: /opt/spark/bin/spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077
    depends_on:
      - spark-master
    environment:
      - SPARK_MODE=worker
      - SPARK_WORKER_CORES=2
      - SPARK_WORKER_MEMORY=2g
```

---

### Scenario 95: TensorFlow with GPU Support
**Complexity Level:** Advanced

**Explanation:** TensorFlow container with CUDA GPU support.

**Dockerfile:**
```dockerfile
FROM tensorflow/tensorflow:2.15.0-gpu
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root", "--no-browser"]
```

---

### Scenario 96: MinIO Object Storage
**Complexity Level:** Advanced

**Explanation:** Self-hosted S3-compatible object storage.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio-data:/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  createbuckets:
    image: minio/mc:latest
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
      sleep 5;
      /usr/bin/mc alias set myminio http://minio:9000 minioadmin minioadmin;
      /usr/bin/mc mb myminio/mybucket;
      /usr/bin/mc anonymous set public myminio/mybucket;
      exit 0;
      "

volumes:
  minio-data:
```

---

### Scenario 97: Keycloak Identity Provider
**Complexity Level:** Advanced

**Explanation:** Keycloak for authentication and authorization.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data

  keycloak:
    image: quay.io/keycloak/keycloak:23.0
    command: start-dev
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres-data:
```

---

### Scenario 98: HashiCorp Vault for Secrets Management
**Complexity Level:** Advanced

**Explanation:** Vault server for centralized secrets management.

**Dockerfile:**
```dockerfile
FROM vault:1.15
COPY config.hcl /vault/config/
COPY policies/ /vault/policies/
EXPOSE 8200
ENTRYPOINT ["vault", "server", "-config=/vault/config/config.hcl"]
```

**config.hcl:**
```hcl
storage "file" {
  path = "/vault/data"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}

api_addr = "http://0.0.0.0:8200"
ui = true
```

---

### Scenario 99: Full E-Commerce Stack
**Complexity Level:** Advanced

**Explanation:** Complete e-commerce application with all services.

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/ecommerce
      REDIS_URL: redis://redis:6379
      ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
      - postgres
      - redis
      - elasticsearch

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: ecommerce
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - es-data:/usr/share/elasticsearch/data

  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    ports:
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq

  worker:
    build: ./worker
    environment:
      RABBITMQ_URL: amqp://rabbitmq:5672
      DATABASE_URL: postgresql://user:pass@postgres:5432/ecommerce
    depends_on:
      - rabbitmq
      - postgres

volumes:
  postgres-data:
  redis-data:
  es-data:
  rabbitmq-data:
```

---

### Scenario 100: Comprehensive DevOps Pipeline
**Complexity Level:** Advanced

**Explanation:** Complete CI/CD pipeline with testing, building, and deployment.

**Dockerfile (Build Agent):**
```dockerfile
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    curl \
    git \
    docker.io \
    kubectl \
    python3 \
    python3-pip \
    nodejs \
    npm \
    openjdk-17-jdk \
    maven \
    && rm -rf /var/lib/apt/lists/*
RUN curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
RUN pip3 install ansible awscli
WORKDIR /workspace
COPY pipeline-scripts/ /usr/local/bin/
RUN chmod +x /usr/local/bin/*.sh
CMD ["/bin/bash"]
```

**.gitlab-ci.yml:**
```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

test:
  stage: test
  image: node:18-alpine
  script:
    - npm ci
    - npm run lint
    - npm run test
    - npm run test:coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kubectl rollout status deployment/myapp
  only:
    - main
  when: manual
```

---

## Conclusion

This document provides 100 comprehensive Docker scenarios covering:
- **Basic (1-35)**: Fundamental concepts and simple applications
- **Intermediate (36-70)**: Multi-stage builds, optimization, and best practices
- **Advanced (71-100)**: Complex architectures, security, and production deployments

Each scenario includes a complete, working Dockerfile that can be used as-is or adapted to your specific needs. These scenarios demonstrate real-world use cases and industry best practices for containerization.

### Key Takeaways

1. **Start Simple**: Begin with basic scenarios to understand Docker fundamentals
2. **Layer Optimization**: Use multi-stage builds to reduce image size
3. **Security First**: Always run containers as non-root users in production
4. **Health Checks**: Implement health checks for production containers
5. **Environment Variables**: Use build arguments and environment variables for configuration
6. **Networking**: Understand Docker networking for multi-container applications
7. **Volumes**: Use volumes for data persistence
8. **.dockerignore**: Always use .dockerignore to optimize build context
9. **Caching**: Leverage Docker layer caching for faster builds
10. **Production Ready**: Use distroless or minimal base images for production

### Additional Resources

- Official Docker Documentation: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Docker Best Practices: https://docs.docker.com/develop/dev-best-practices/
- Multi-stage Builds: https://docs.docker.com/build/building/multi-stage/
- Security Best Practices: https://docs.docker.com/engine/security/
