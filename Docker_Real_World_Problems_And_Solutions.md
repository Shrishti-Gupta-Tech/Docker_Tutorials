# 🔧 Docker Real-World Problems & Solutions
### Common Issues, Root Causes, and Practical Fixes

---

## Table of Contents
1. [Container Startup & Runtime Issues](#container-startup--runtime-issues)
2. [Image Building Problems](#image-building-problems)
3. [Data Persistence & Volume Issues](#data-persistence--volume-issues)
4. [Networking & Connectivity Problems](#networking--connectivity-problems)
5. [Performance Issues](#performance-issues)
6. [Security & Permissions Problems](#security--permissions-problems)
7. [Debugging & Logging Issues](#debugging--logging-issues)
8. [CI/CD & Deployment Challenges](#cicd--deployment-challenges)
9. [Development Environment Problems](#development-environment-problems)
10.[Database Container Issues](#database-container-issues)

---

## Container Startup & Runtime Issues

### Problem 1: Container Exits Immediately After Starting

**Symptom:**
```bash
docker run -d my-app
# Container starts but stops immediately

docker ps -a
# CONTAINER ID   STATUS                     
# abc123def456   Exited (0) 2 seconds ago
```

**Root Causes:**
1. Application exits normally (completes task)
2. No foreground process running
3. CMD/ENTRYPOINT misconfiguration

**Solutions:**

**Solution 1: Keep Container Running (Development)**
```dockerfile
# ❌ Wrong: Container exits after echo
FROM alpine
CMD echo "Hello World"

# ✅ Correct: Keep container running
FROM alpine
CMD tail -f /dev/null
# OR
CMD sleep infinity
```

**Solution 2: Run Application in Foreground**
```dockerfile
# ❌ Wrong: nginx runs as daemon
FROM nginx
CMD ["nginx"]

# ✅ Correct: Run in foreground
FROM nginx
CMD ["nginx", "-g", "daemon off;"]
```

**Solution 3: Check Application Logs**
```bash
# View logs to see why it exited
docker logs container-name

# Run in foreground to see output
docker run --rm my-app

# Override CMD to debug
docker run --rm my-app sh -c "ls -la && cat config.json"
```

**Real-World Example:**
```dockerfile
# Node.js application that exits immediately
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# ❌ Wrong: Runs npm install and exits
CMD ["npm", "install"]

# ✅ Correct: Starts the server
CMD ["node", "server.js"]
# OR for development
CMD ["npm", "start"]
```

---

### Problem 2: "Cannot Connect" or "Connection Refused" to Container

**Symptom:**
```bash
docker run -d -p 8080:3000 my-app
curl http://localhost:8080
# curl: (7) Failed to connect to localhost port 8080: Connection refused
```

**Root Causes:**
1. Application not listening on correct interface (0.0.0.0)
2. Wrong port mapping
3. Application not started yet
4. Firewall blocking

**Solutions:**

**Solution 1: Listen on 0.0.0.0, Not localhost**
```javascript
// ❌ Wrong: Only accessible inside container
app.listen(3000, 'localhost', () => {
  console.log('Server running');
});

// ✅ Correct: Accessible from outside
app.listen(3000, '0.0.0.0', () => {
  console.log('Server running on 0.0.0.0:3000');
});
```

**Solution 2: Verify Port Mapping**
```bash
# Check which ports are mapped
docker port my-app

# Output should show:
# 3000/tcp -> 0.0.0.0:8080

# Check if application is listening
docker exec my-app netstat -tuln | grep 3000
# OR
docker exec my-app ss -tuln | grep 3000
```

**Solution 3: Wait for Application Startup**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install

# Add health check
HEALTHCHECK --interval=5s --timeout=3s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

**Solution 4: Verify Container Network Settings**
```bash
# Check container's network configuration
docker inspect my-app | grep -A 10 NetworkSettings

# Test from inside container
docker exec my-app curl http://localhost:3000
# If this works but external doesn't, it's a port mapping issue
```

---

### Problem 3: Container Consuming Too Much Memory/CPU

**Symptom:**
```bash
docker stats
# CONTAINER   CPU %   MEM USAGE / LIMIT
# my-app      150%    1.8GiB / 2GiB
```

**Root Causes:**
1. Memory leaks in application
2. No resource limits set
3. Too many processes

**Solutions:**

**Solution 1: Set Resource Limits**
```bash
# Limit memory and CPU
docker run -d \
  --name my-app \
  --memory="512m" \
  --memory-swap="512m" \
  --cpus="1.0" \
  my-app:latest

# With Docker Compose
```

```yaml
version: '3.8'
services:
  app:
    image: my-app
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

**Solution 2: Optimize Application**
```dockerfile
# Node.js memory optimization
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Set Node.js memory limits
ENV NODE_OPTIONS="--max-old-space-size=384"

CMD ["node", "server.js"]
```

**Solution 3: Monitor and Profile**
```bash
# Continuous monitoring
docker stats my-app

# Get detailed process info
docker exec my-app ps aux

# Check for memory leaks
docker exec my-app cat /proc/meminfo
```

---

## Image Building Problems

### Problem 4: Build Takes Forever / Very Slow Builds

**Symptom:**
```bash
docker build -t my-app .
# Step 5/10 : RUN npm install
#  ---> Running in abc123
# [Hangs for 10+ minutes]
```

**Root Causes:**
1. Poor layer caching
2. Large build context
3. Downloading large dependencies repeatedly

**Solutions:**

**Solution 1: Optimize Layer Caching**
```dockerfile
# ❌ Bad: Cache breaks on any file change
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install  # Reinstalls everything on ANY change

# ✅ Good: Cache dependencies separately
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Cached unless package.json changes
COPY . .  # Source changes don't invalidate dependency cache
```

**Solution 2: Use .dockerignore**
```
# .dockerignore
node_modules/
npm-debug.log
.git/
.gitignore
*.md
.env
.DS_Store
dist/
build/
coverage/
.vscode/
.idea/
*.log
*.tmp
.cache/
```

**Solution 3: Use BuildKit for Parallel Builds**
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or set in daemon.json
{
  "features": {
    "buildkit": true
  }
}

# Build with BuildKit
DOCKER_BUILDKIT=1 docker build -t my-app .
```

**Solution 4: Multi-Stage Builds with Caching**
```dockerfile
FROM node:18 AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci
# Dependencies layer is cached

FROM node:18 AS builder
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build
# Build layer uses cached dependencies

FROM node:18-alpine
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

---

### Problem 5: "No Space Left on Device" Error

**Symptom:**
```bash
docker build -t my-app .
# ERROR: failed to solve: failed to copy files: write /var/lib/docker/...: no space left on device
```

**Root Causes:**
1. Docker using too much disk space
2. Too many unused images/containers
3. Large build cache

**Solutions:**

**Solution 1: Clean Up Docker Resources**
```bash
# Check disk usage
docker system df

# Output:
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          15        5         4.2GB     2.8GB (66%)
# Containers      10        2         100MB     90MB (90%)
# Local Volumes   5         2         1GB       800MB (80%)
# Build Cache     50        0         2GB       2GB (100%)

# Remove unused data
docker system prune

# Remove all unused data (aggressive)
docker system prune -a

# Remove specific resources
docker image prune -a       # Remove unused images
docker container prune      # Remove stopped containers
docker volume prune         # Remove unused volumes
docker builder prune        # Remove build cache
```

**Solution 2: Limit Docker Storage**
```bash
# Configure Docker storage limit (daemon.json)
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.size=50G"
  ]
}

# Restart Docker daemon
sudo systemctl restart docker
```

**Solution 3: Regular Cleanup Schedule**
```bash
# Create cron job for daily cleanup
# Add to crontab: crontab -e
0 2 * * * docker system prune -f --filter "until=24h"
```

---

### Problem 6: "Cannot Locate Package" During apt-get Install

**Symptom:**
```dockerfile
FROM ubuntu:22.04
RUN apt-get install -y curl
# E: Unable to locate package curl
```

**Root Causes:**
1. Missing `apt-get update`
2. Package name incorrect
3. Package repository not available

**Solutions:**

**Solution 1: Always Run apt-get update First**
```dockerfile
# ❌ Wrong: No update
FROM ubuntu:22.04
RUN apt-get install -y curl

# ✅ Correct: Update package lists first
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**Solution 2: Combine Commands to Avoid Cache Issues**
```dockerfile
# ❌ Wrong: Update and install in separate layers
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl  # May use stale cache

# ✅ Correct: Combined in one layer
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        vim \
    && rm -rf /var/lib/apt/lists/*
```

**Solution 3: Handle Non-Interactive Installation**
```dockerfile
FROM ubuntu:22.04

# Set non-interactive mode
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        tzdata \
        curl \
    && ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    dpkg-reconfigure --frontend noninteractive tzdata && \
    rm -rf /var/lib/apt/lists/*
```

---

## Data Persistence & Volume Issues

### Problem 7: Data Lost After Container Restart

**Symptom:**
```bash
# Insert data into database
docker exec postgres psql -U user -c "INSERT INTO users VALUES (1, 'Alice')"

# Restart container
docker restart postgres

# Data is lost!
docker exec postgres psql -U user -c "SELECT * FROM users"
# (0 rows)
```

**Root Cause:**
Container filesystem is ephemeral - data stored inside container is lost on removal.

**Solutions:**

**Solution 1: Use Named Volumes**
```bash
# ❌ Wrong: No volume
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# ✅ Correct: With named volume
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data persists even after container removal
docker rm -f postgres
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
# Data is still there!
```

**Solution 2: Docker Compose with Volumes**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
```

**Solution 3: Backup and Restore Volumes**
```bash
# Backup volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres-backup.tar.gz -C /data .

# Restore volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres-backup.tar.gz -C /data
```

---

### Problem 8: Permission Denied When Accessing Volume

**Symptom:**
```bash
docker run -d -v $(pwd)/data:/app/data my-app
# Permission denied: cannot write to /app/data
```

**Root Causes:**
1. User ID mismatch between host and container
2. Volume owned by root
3. SELinux restrictions

**Solutions:**

**Solution 1: Match User IDs**
```dockerfile
FROM node:18-alpine

# Create user with specific UID/GID matching host
ARG USER_ID=1000
ARG GROUP_ID=1000

RUN addgroup -g ${GROUP_ID} appgroup && \
    adduser -D -u ${USER_ID} -G appgroup appuser

WORKDIR /app
COPY --chown=appuser:appgroup . .

USER appuser
CMD ["node", "server.js"]
```

```bash
# Build with host user ID
docker build \
  --build-arg USER_ID=$(id -u) \
  --build-arg GROUP_ID=$(id -g) \
  -t my-app .

# Run with volume
docker run -d -v $(pwd)/data:/app/data my-app
```

**Solution 2: Fix Permissions on Host**
```bash
# Create directory with correct permissions
mkdir -p ./data
chmod 777 ./data  # Or more restrictive as needed

# Or change ownership
sudo chown -R $(id -u):$(id -g) ./data
```

**Solution 3: Use Named Volumes Instead**
```bash
# Named volumes handle permissions automatically
docker run -d \
  -v app-data:/app/data \
  my-app
```

**Solution 4: SELinux Context (Linux)**
```bash
# Add :z or :Z flag for SELinux
docker run -d -v $(pwd)/data:/app/data:z my-app

# :z = shared context
# :Z = private context
```

---

## Networking & Connectivity Problems

### Problem 9: Containers Can't Communicate with Each Other

**Symptom:**
```bash
docker run -d --name backend my-backend
docker run -d --name frontend my-frontend

# Frontend can't reach backend
docker exec frontend curl http://backend:3000
# curl: (6) Could not resolve host: backend
```

**Root Cause:**
Containers on default bridge network can't use DNS resolution.

**Solutions:**

**Solution 1: Create Custom Bridge Network**
```bash
# Create custom network
docker network create app-network

# Run containers on same network
docker run -d --name backend \
  --network app-network \
  my-backend

docker run -d --name frontend \
  --network app-network \
  my-frontend

# Now they can communicate by name
docker exec frontend curl http://backend:3000
# Works!
```

**Solution 2: Use Docker Compose**
```yaml
version: '3.8'
services:
  backend:
    image: my-backend
    networks:
      - app-network
    expose:
      - "3000"

  frontend:
    image: my-frontend
    networks:
      - app-network
    environment:
      - API_URL=http://backend:3000
    ports:
      - "80:80"

networks:
  app-network:
    driver: bridge
```

**Solution 3: Link Containers (Legacy)**
```bash
# Using --link (deprecated but still works)
docker run -d --name backend my-backend
docker run -d --name frontend --link backend:api my-frontend

# Frontend can now access backend via 'api' hostname
```

---

### Problem 10: Can't Access Container from Host Browser

**Symptom:**
```bash
docker run -d -p 8080:80 nginx
# Browser shows "This site can't be reached" at localhost:8080
```

**Root Causes:**
1. Port not properly mapped
2. Application not listening on 0.0.0.0
3. Firewall blocking
4. Using Docker Desktop on Mac/Windows (different networking)

**Solutions:**

**Solution 1: Verify Port Mapping**
```bash
# Check running containers and ports
docker ps

# Check specific port mapping
docker port nginx

# Test from command line
curl -v http://localhost:8080

# Test from inside container
docker exec nginx curl http://localhost:80
```

**Solution 2: Bind to All Interfaces**
```bash
# If application binds to localhost only
# ❌ Wrong
app.listen(3000, 'localhost')  # Only accessible inside container

# ✅ Correct
app.listen(3000, '0.0.0.0')  # Accessible from outside
```

**Solution 3: Check Firewall (Linux)**
```bash
# Check if port is allowed
sudo ufw status
sudo firewall-cmd --list-all

# Allow port
sudo ufw allow 8080
sudo firewall-cmd --add-port=8080/tcp
```

**Solution 4: Docker Desktop (Mac/Windows)**
```bash
# Use host.docker.internal instead of localhost in some cases
# Or ensure port forwarding is working

# Check Docker Desktop settings
# Settings > Resources > Advanced
# Ensure enough resources allocated
```

---

### Problem 11: Container Can't Access External Network/Internet

**Symptom:**
```bash
docker run --rm alpine ping google.com
# ping: bad address 'google.com'

docker run --rm alpine wget https://example.com
# wget: can't connect to remote host: Network is unreachable
```

**Root Causes:**
1. DNS resolution failure
2. Docker daemon DNS configuration
3. Corporate proxy/firewall

**Solutions:**

**Solution 1: Configure DNS**
```bash
# Test current DNS
docker run --rm alpine nslookup google.com

# Run with custom DNS
docker run --rm --dns 8.8.8.8 --dns 8.8.4.4 alpine ping google.com

# Set default DNS in daemon.json
```

```json
{
  "dns": ["8.8.8.8", "8.8.4.4", "1.1.1.1"]
}
```

```bash
# Restart Docker
sudo systemctl restart docker
```

**Solution 2: Configure Docker Proxy**
```bash
# Create proxy configuration
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080"
Environment="HTTPS_PROXY=http://proxy.example.com:8080"
Environment="NO_PROXY=localhost,127.0.0.1"
```

```bash
# Reload and restart
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**Solution 3: Check Docker Network**
```bash
# Inspect Docker network
docker network inspect bridge

# Check iptables rules
sudo iptables -L -n

# Reset Docker network
sudo systemctl restart docker
```

---

## Performance Issues

### Problem 12: Slow File I/O on Mounted Volumes (Mac/Windows)

**Symptom:**
```bash
# Extremely slow npm install or file operations
docker run -v $(pwd):/app node:18 npm install
# Takes 10+ minutes for what should take 1 minute
```

**Root Cause:**
Docker Desktop on Mac/Windows uses filesystem sync which is slow for many small files.

**Solutions:**

**Solution 1: Use Named Volumes for Dependencies**
```dockerfile
# Don't mount node_modules from host
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

```yaml
# Docker Compose with anonymous volume
version: '3.8'
services:
  app:
    build: .
    volumes:
      - ./src:/app/src          # Only source code
      - /app/node_modules       # Don't sync node_modules
```

**Solution 2: Use Cached or Delegated Mounting (Mac)**
```yaml
version: '3.8'
services:
  app:
    volumes:
      - ./:/app:cached     # Faster on Mac
      # OR
      - ./:/app:delegated  # Even faster but may have consistency issues
```

**Solution 3: Move Build Inside Container**
```dockerfile
# Build everything inside container
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

---

### Problem 13: Image Pull is Extremely Slow

**Symptom:**
```bash
docker pull ubuntu:22.04
# Takes 30+ minutes for a small image
```

**Root Causes:**
1. Network bandwidth limitations
2. Docker Hub rate limiting
3. Large image layers

**Solutions:**

**Solution 1: Use Mirror/Registry Cache**
```json
// daemon.json
{
  "registry-mirrors": ["https://mirror.gcr.io"]
}
```

**Solution 2: Check Rate Limiting**
```bash
# Check your Docker Hub rate limit
TOKEN=$(curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:ratelimitpreview/test:pull" | jq -r .token)
curl -s --head -H "Authorization: Bearer $TOKEN" https://registry-1.docker.io/v2/ratelimitpreview/test/manifests/latest | grep -i ratelimit

# Authenticate to increase limit
docker login
```

**Solution 3: Use Smaller Base Images**
```dockerfile
# ❌ Large: 200MB+
FROM ubuntu:22.04

# ✅ Small: 5MB
FROM alpine:3.18

# ✅ Medium: 70MB
FROM debian:bookworm-slim
```

---

## Security & Permissions Problems

### Problem 14: Running as Root User (Security Risk)

**Symptom:**
```bash
docker exec my-app whoami
# root
```

**Root Cause:**
Default container user is root, which is a security risk.

**Solutions:**

**Solution 1: Create Non-Root User**
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

CMD ["node", "server.js"]
```

**Solution 2: Use Official Image's User**
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY --chown=node:node . .

# Use built-in node user
USER node

CMD ["node", "server.js"]
```

**Solution 3: Read-Only Root Filesystem**
```bash
# Run with read-only root filesystem
docker run -d \
  --name my-app \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  my-app:latest
```

---

### Problem 15: Secrets Exposed in Image

**Symptom:**
```dockerfile
# ❌ NEVER DO THIS
FROM node:18
COPY .env /app/.env
ENV API_KEY=super-secret-key-12345
```

```bash
# Secrets visible in image history
docker history my-app
# Shows API_KEY in plain text!
```

**Root Cause:**
Secrets baked into image layers.

**Solutions:**

**Solution 1: Use Environment Variables at Runtime**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# Don't set secrets in Dockerfile!
# ENV API_KEY=...  # NEVER DO THIS

CMD ["node", "server.js"]
```

```bash
# Pass secrets at runtime
docker run -d \
  -e API_KEY=${API_KEY} \
  -e DATABASE_URL=${DATABASE_URL} \
  my-app
```

**Solution 2: Use Docker Secrets (Swarm)**
```bash
# Create secret
echo "my-secret-value" | docker secret create api_key -

# Use in service
docker service create \
  --name my-app \
  --secret api_key \
  my-app:latest
```

```javascript
// Read secret in application
const fs = require('fs');
const apiKey = fs.readFileSync('/run/secrets/api_key', 'utf8');
```

**Solution 3: Use Build-Time Secrets (BuildKit)**
```dockerfile
# syntax=docker/dockerfile:1.4

FROM node:18-alpine
WORKDIR /app

# Mount secret during build only
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci

COPY . .
CMD ["node", "server.js"]
```

```bash
# Build with secret
DOCKER_BUILDKIT=1 docker build \
  --secret id=npmrc,src=$HOME/.npmrc \
  -t my-app .
```

**Solution 4: Use .dockerignore**
```
# .dockerignore
.env
.env.*
*.key
*.pem
secrets/
credentials.json
```

---

## Debugging & Logging Issues

### Problem 16: Can't See Application Logs

**Symptom:**
```bash
docker logs my-app
# (empty output)
```

**Root Causes:**
1. Application logging to file instead of stdout/stderr
2. Logs buffered
3. Wrong log driver

**Solutions:**

**Solution 1: Log to stdout/stderr**
```javascript
// ❌ Wrong: Logging to file
const fs = require('fs');
fs.appendFileSync('/var/log/app.log', 'Log message\n');

// ✅ Correct: Log to stdout
console.log('Log message');
console.error('Error message');
```

```python
# ❌ Wrong: Logging to file
logging.basicConfig(filename='/var/log/app.log')

# ✅ Correct: Log to stdout
logging.basicConfig(
    stream=sys.stdout,
    level=logging.INFO
)
```

**Solution 2: Disable Output Buffering**
```dockerfile
# Python: Disable buffering
FROM python:3.11-slim
ENV PYTHONUNBUFFERED=1
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

```dockerfile
# Node.js: Use process.stdout.write
FROM node:18-alpine
ENV NODE_ENV=production
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
```

**Solution 3: Configure Logging Driver**
```bash
# Use json-file driver (default)
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  my-app

# Or in daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**Solution 4: Follow Logs in Real-Time**
```bash
# Follow logs
docker logs -f my-app

# Show timestamps
docker logs -t my-app

# Show last 100 lines
docker logs --tail 100 my-app

# Show logs since specific time
docker logs --since 2023-01-01T10:00:00 my-app
```

---

### Problem 17: Need to Debug Inside Container

**Symptom:**
Container is running but behaving incorrectly, need to investigate.

**Solutions:**

**Solution 1: Execute Shell in Running Container**
```bash
# Bash shell
docker exec -it my-app bash

# If bash not available, try sh
docker exec -it my-app sh

# Run as root (if needed)
docker exec -it -u root my-app sh
```

**Solution 2: Inspect Running Processes**
```bash
# List processes
docker exec my-app ps aux

# Check environment variables
docker exec my-app env

# Check network connections
docker exec my-app netstat -tuln

# Check disk usage
docker exec my-app df -h
```

**Solution 3: Copy Files Out for Inspection**
```bash
# Copy file from container
docker cp my-app:/app/logs/error.log ./error.log

# Copy entire directory
docker cp my-app:/app/config ./config

# Copy file into container
docker cp ./fixed-config.json my-app:/app/config.json
```

**Solution 4: Run With Override**
```bash
# Override CMD to start with shell
docker run -it --rm my-app sh

# Override ENTRYPOINT
docker run -it --rm --entrypoint sh my-app

# Run specific command
docker run --rm my-app cat /app/config.json
```

**Solution 5: Attach to Running Container**
```bash
# Attach to container's stdout/stderr
docker attach my-app

# Detach without stopping: Ctrl+P, Ctrl+Q
```

---

## CI/CD & Deployment Challenges

### Problem 18: Docker Build Fails in CI/CD Pipeline

**Symptom:**
```bash
# Works locally
docker build -t my-app .

# Fails in CI/CD
ERROR: failed to solve: executor failed running [...]
```

**Root Causes:**
1. Build context too large
2. Network timeouts
3. Authentication issues
4. Platform differences

**Solutions:**

**Solution 1: Optimize Build Context**
```
# .dockerignore
.git/
.github/
node_modules/
*.md
*.log
.env
.vscode/
.idea/
coverage/
dist/
build/
*.tar.gz
```

**Solution 2: Use Build Cache in CI**
```yaml
# GitHub Actions
name: Build Docker Image
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
        
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: user/app:latest
          cache-from: type=registry,ref=user/app:buildcache
          cache-to: type=registry,ref=user/app:buildcache,mode=max
```

**Solution 3: Handle Authentication**
```yaml
# GitLab CI
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

**Solution 4: Platform-Specific Builds**
```dockerfile
# Handle different architectures
FROM --platform=$BUILDPLATFORM node:18-alpine AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "Building on $BUILDPLATFORM for $TARGETPLATFORM"

WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:18-alpine
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

---

### Problem 19: Image Size Too Large for Registry

**Symptom:**
```bash
docker push my-app:latest
# Error: blob upload invalid: blob exceeds maximum size
```

**Solutions:**

**Solution 1: Multi-Stage Build**
```dockerfile
# Before: 1.2GB
FROM node:18
COPY . .
RUN npm install
CMD ["node", "server.js"]

# After: 180MB
FROM node:18 AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

**Solution 2: Remove Unnecessary Files**
```dockerfile
FROM node:18-alpine
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Copy only necessary files
COPY dist/ ./dist/
COPY config/ ./config/

CMD ["node", "dist/server.js"]
```

**Solution 3: Use Smaller Base Images**
```dockerfile
# 900MB
FROM node:18

# 170MB
FROM node:18-alpine

# 40MB (if you can compile to standalone)
FROM alpine:latest
COPY --from=builder /app/binary /bin/app
CMD ["app"]
```

---

## Development Environment Problems

### Problem 20: Hot Reload Not Working

**Symptom:**
```bash
# Change source code
vim src/app.js

# Server doesn't reload
# No changes reflected
```

**Root Causes:**
1. File watchers not working with volumes
2. Polling not enabled
3. node_modules mounted from host

**Solutions:**

**Solution 1: Enable Polling**
```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - ./src:/app/src
      - /app/node_modules  # Don't mount node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true
      - WATCHPACK_POLLING=true
    ports:
      - "3000:3000"
    command: npm run dev
```

**Solution 2: React Development**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
ENV WATCHPACK_POLLING=true
ENV CHOKIDAR_USEPOLLING=true
COPY . .
CMD ["npm", "start"]
```

**Solution 3: Node.js with Nodemon**
```json
// nodemon.json
{
  "watch": ["src"],
  "ext": "js,json",
  "ignore": ["node_modules"],
  "legacyWatch": true,
  "polling": true,
  "pollingInterval": 1000
}
```

**Solution 4: Next.js Configuration**
```javascript
// next.config.js
module.exports = {
  webpack: (config, { dev }) => {
    if (dev) {
      config.watchOptions = {
        poll: 1000,
        aggregateTimeout: 300,
      };
    }
    return config;
  },
};
```

---

## Database Container Issues

### Problem 21: Database Initialization Scripts Not Running

**Symptom:**
```bash
docker run -d \
  -v $(pwd)/init.sql:/docker-entrypoint-initdb.d/init.sql \
  postgres:15

# Tables not created
```

**Root Causes:**
1. Volume already contains data
2. Scripts only run on first initialization
3. Wrong directory

**Solutions:**

**Solution 1: Clean Volume for Fresh Start**
```bash
# Remove existing volume
docker volume rm postgres-data

# Recreate container
docker run -d \
  -v postgres-data:/var/lib/postgresql/data \
  -v $(pwd)/init.sql:/docker-entrypoint-initdb.d/init.sql \
  postgres:15
```

**Solution 2: Use Multiple Init Scripts**
```bash
# Scripts run in alphabetical order
docker run -d \
  -v $(pwd)/init-scripts:/docker-entrypoint-initdb.d \
  postgres:15

# Directory structure:
# init-scripts/
#   01-create-tables.sql
#   02-insert-data.sql
#   03-create-users.sql
```

**Solution 3: Docker Compose with Initialization**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"

volumes:
  postgres-data:
```

---

### Problem 22: Can't Connect to Database Container

**Symptom:**
```bash
docker run -d --name postgres -e POSTGRES_PASSWORD=secret postgres:15
psql -h localhost -U postgres
# psql: error: connection to server at "localhost", port 5432 failed
```

**Root Causes:**
1. Port not exposed
2. Database not ready
3. Authentication issue

**Solutions:**

**Solution 1: Expose Port and Wait for Ready**
```bash
# Expose port
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:15

# Wait for database to be ready
docker exec postgres pg_isready -U postgres

# Now connect
psql -h localhost -U postgres
```

**Solution 2: Use Health Check**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
  
  app:
    image: my-app
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      DATABASE_URL: postgres://postgres:secret@postgres:5432/mydb
```

**Solution 3: Wait Script**
```bash
# wait-for-it.sh
#!/bin/bash
host="$1"
shift
cmd="$@"

until PGPASSWORD=secret psql -h "$host" -U postgres -c '\q'; do
  >&2 echo "Postgres is unavailable - sleeping"
  sleep 1
done

>&2 echo "Postgres is up - executing command"
exec $cmd
```

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY wait-for-it.sh /
RUN chmod +x /wait-for-it.sh
COPY . .
CMD ["/wait-for-it.sh", "postgres", "node", "server.js"]
```

---

## 📊 Bonus: Quick Troubleshooting Checklist

### General Debugging Steps

```bash
# 1. Check container status
docker ps -a

# 2. View logs
docker logs container-name
docker logs -f container-name --tail 100

# 3. Inspect container
docker inspect container-name

# 4. Check resource usage
docker stats container-name

# 5. Execute commands inside
docker exec -it container-name sh

# 6. Check network
docker network inspect bridge
docker exec container-name ping other-container

# 7. Test connectivity
docker exec container-name curl http://localhost:3000
docker exec container-name netstat -tuln

# 8. Check filesystem
docker exec container-name ls -la /app
docker exec container-name df -h

# 9. View processes
docker exec container-name ps aux

# 10. Check environment
docker exec container-name env
```

### Common Commands Reference

```bash
# Clean up everything
docker system prune -a --volumes

# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Force remove container
docker rm -f container-name

# Restart container
docker restart container-name

# Stop all containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# View disk usage
docker system df

# Export container
docker export container-name > backup.tar

# Import image
docker import backup.tar my-image:latest

# Save image
docker save my-image:latest > image.tar

# Load image
docker load < image.tar
```

---

## 🎯 Prevention Tips

### Best Practices to Avoid Common Problems

1. **Always Use .dockerignore**
```
node_modules/
.git/
*.md
.env
*.log
```

2. **Set Resource Limits**
```yaml
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '1.0'
```

3. **Use Health Checks**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1
```

4. **Run as Non-Root**
```dockerfile
USER node
```

5. **Use Specific Image Tags**
```dockerfile
FROM node:18.17.0-alpine
# Not: FROM node:latest
```

6. **Clean Up in Same Layer**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

7. **Use Multi-Stage Builds**
```dockerfile
FROM builder AS build
FROM alpine AS final
COPY --from=build /app/binary .
```

8. **Monitor Logs**
```bash
docker logs -f app 2>&1 | tee app.log
```

---

**End of Docker Real-World Problems & Solutions**

This guide covers the most common issues developers face with Docker. Remember: most problems have simple solutions once you understand the root cause. Always check logs first, then inspect configuration, then test connectivity!
