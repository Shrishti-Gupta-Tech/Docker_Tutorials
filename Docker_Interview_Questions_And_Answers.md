# 🎯 Docker Interview Questions & Answers
### From Junior to Senior Level - 60+ Questions

---

## Table of Contents
1. [Junior Level (Entry-Level)](#junior-level-entry-level)
2. [Mid-Level (Intermediate)](#mid-level-intermediate)
3. [Senior Level (Advanced)](#senior-level-advanced)
4. [Scenario-Based Questions](#scenario-based-questions)

---

## Junior Level (Entry-Level)

### Q1: What is Docker and why do we use it?

**Answer:**
Docker is a containerization platform that packages applications and their dependencies into isolated containers. It solves the "it works on my machine" problem by ensuring consistency across different environments.

**Key Benefits:**
- **Consistency:** Same behavior in dev, test, and production
- **Isolation:** Applications don't interfere with each other
- **Portability:** Run anywhere Docker is installed
- **Efficiency:** Lightweight compared to VMs
- **Scalability:** Easy to scale horizontally

**Example:**
```bash
# Package an app with all dependencies
docker build -t my-app .

# Run anywhere consistently
docker run -d -p 80:3000 my-app
```

---

### Q2: What's the difference between a Docker image and a container?

**Answer:**
- **Image:** Read-only template/blueprint containing app code, runtime, and dependencies. Like a class in OOP.
- **Container:** Running instance of an image. Like an object instantiated from a class.

**Key Differences:**
| Aspect | Image | Container |
|--------|-------|-----------|
| Nature | Static, read-only | Dynamic, writable |
| Creation | Built from Dockerfile | Created from image |
| Storage | Stored on disk | Running in memory |
| Quantity | One image | Multiple containers from one image |

**Example:**
```bash
# Create image (once)
docker build -t nginx-custom .

# Create multiple containers from same image
docker run -d --name web1 nginx-custom
docker run -d --name web2 nginx-custom
docker run -d --name web3 nginx-custom
```

---

### Q3: Explain the Docker architecture.

**Answer:**
Docker uses a client-server architecture with three main components:

**Components:**
1. **Docker Client (CLI):** User interface for Docker commands
2. **Docker Daemon (dockerd):** Background service managing containers, images, networks, and volumes
3. **Docker Registry:** Stores Docker images (Docker Hub is public registry)

**Architecture Flow:**
```
┌──────────────┐
│ Docker CLI   │  docker run, build, pull...
└──────┬───────┘
       │ REST API
       ▼
┌──────────────┐
│Docker Daemon │  Manages:
│  (dockerd)   │  - Images
└──────┬───────┘  - Containers
       │          - Networks
       │          - Volumes
       ▼
┌──────────────┐
│  Container   │
│  Runtime     │
└──────────────┘
```

---

### Q4: What is a Dockerfile?

**Answer:**
A Dockerfile is a text file containing instructions to build a Docker image automatically. Each instruction creates a layer in the image.

**Basic Structure:**
```dockerfile
FROM node:18-alpine          # Base image
WORKDIR /app                 # Set working directory
COPY package*.json ./        # Copy dependency files
RUN npm install              # Install dependencies
COPY . .                     # Copy application code
EXPOSE 3000                  # Document port
CMD ["node", "server.js"]    # Startup command
```

**Build Command:**
```bash
docker build -t my-app:1.0 .
```

---

### Q5: What's the difference between CMD and ENTRYPOINT?

**Answer:**

**CMD:**
- Provides default arguments for container
- Can be easily overridden at runtime
- If multiple CMD instructions, only last one is used

**ENTRYPOINT:**
- Configures container as executable
- Not easily overridden (requires --entrypoint flag)
- Arguments from CMD or docker run are appended

**Example:**
```dockerfile
# CMD only
FROM alpine
CMD ["echo", "Hello"]
```
```bash
docker run my-image           # Output: Hello
docker run my-image echo Bye  # Output: Bye (CMD overridden)
```

```dockerfile
# ENTRYPOINT + CMD
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hello"]
```
```bash
docker run my-image           # Output: Hello
docker run my-image Bye       # Output: Bye (CMD replaced, ENTRYPOINT stays)
```

**Best Practice:**
```dockerfile
# Use both together
ENTRYPOINT ["python"]
CMD ["app.py"]

# Allows: docker run my-app         → python app.py
#         docker run my-app test.py → python test.py
```

---

### Q6: How do you expose ports in Docker?

**Answer:**
Three ways to expose ports:

**1. EXPOSE Instruction (Documentation):**
```dockerfile
FROM nginx
EXPOSE 80
# Doesn't actually publish, just documents
```

**2. -p Flag (Publish Port):**
```bash
# Map container:80 to host:8080
docker run -d -p 8080:80 nginx

# Access: http://localhost:8080
```

**3. -P Flag (Publish All):**
```bash
# Publish all EXPOSE'd ports to random host ports
docker run -d -P nginx

# Check assigned ports
docker ps
```

---

### Q7: What is Docker Compose?

**Answer:**
Docker Compose is a tool for defining and running multi-container applications using a YAML file (docker-compose.yml).

**Key Features:**
- Define entire application stack in one file
- Start all services with single command
- Automatic network creation
- Service dependency management

**Example:**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "80:3000"
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
# Start everything
docker-compose up -d

# Stop everything
docker-compose down
```

---

### Q8: How do you persist data in Docker?

**Answer:**
Two main methods:

**1. Volumes (Recommended):**
```bash
# Create named volume
docker volume create mydata

# Use volume
docker run -d -v mydata:/app/data my-app

# Data persists even after container removal
```

**2. Bind Mounts:**
```bash
# Mount host directory
docker run -d -v $(pwd)/data:/app/data my-app

# Changes on host reflect in container and vice versa
```

**When to Use:**
- **Volumes:** Production databases, persistent application data
- **Bind Mounts:** Development (hot reload), configuration files

---

### Q9: What is the difference between COPY and ADD in Dockerfile?

**Answer:**

**COPY:**
- Simple file/directory copy
- Recommended for most cases
- More transparent and predictable

**ADD:**
- Everything COPY does, plus:
  - Auto-extracts tar archives
  - Can download files from URLs
- Less predictable, avoid unless needed

**Example:**
```dockerfile
# ✅ Preferred: Use COPY
FROM node:18
COPY package.json .
COPY src/ ./src/

# ⚠️ Only use ADD when needed
FROM alpine
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/  # Auto-extracts
```

**Best Practice:**
```dockerfile
# Use COPY for local files
COPY . .

# Use curl/wget + RUN for URLs
RUN curl -O https://example.com/file.tar.gz && \
    tar xzf file.tar.gz && \
    rm file.tar.gz
```

---

### Q10: What is Docker Hub?

**Answer:**
Docker Hub is a cloud-based registry service where you can:
- Find and pull public images
- Store and share your own images
- Automate builds from GitHub/Bitbucket
- Access official images from vendors

**Usage:**
```bash
# Search for images
docker search python

# Pull image
docker pull python:3.11-slim

# Push your image
docker login
docker tag my-app username/my-app:latest
docker push username/my-app:latest
```

---

## Mid-Level (Intermediate)

### Q11: What are Docker layers and why are they important?

**Answer:**
Each instruction in a Dockerfile creates a layer. Layers are cached and reused to speed up builds.

**How It Works:**
```dockerfile
FROM node:18-alpine      # Layer 1
WORKDIR /app             # Layer 2
COPY package*.json ./    # Layer 3
RUN npm install          # Layer 4 (cached if Layer 3 unchanged)
COPY . .                 # Layer 5 (changes frequently)
CMD ["node", "app.js"]   # Layer 6
```

**Benefits:**
- **Fast rebuilds:** Unchanged layers are reused
- **Storage efficiency:** Shared layers between images
- **Network efficiency:** Only changed layers pulled/pushed

**Optimization:**
```dockerfile
# ❌ Bad: Inefficient layering
COPY . .
RUN npm install

# ✅ Good: Optimize for caching
COPY package*.json ./
RUN npm install
COPY . .
```

---

### Q12: Explain multi-stage builds and their benefits.

**Answer:**
Multi-stage builds use multiple FROM statements to create optimized production images by separating build and runtime stages.

**Benefits:**
- Dramatically reduce image size (often 80-95%)
- Remove build tools from production
- Separate build-time and runtime dependencies

**Example:**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
# Size: ~1.2GB

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
# Final size: ~180MB (85% reduction!)
```

---

### Q13: How do you optimize Docker image size?

**Answer:**
Multiple techniques to reduce image size:

**1. Use Alpine Base Images:**
```dockerfile
FROM node:18-alpine  # 170MB vs 900MB
```

**2. Multi-Stage Builds:**
```dockerfile
FROM builder AS build
FROM alpine AS production
COPY --from=build /app/binary .
```

**3. Combine RUN Commands:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**4. Remove Cache and Temporary Files:**
```dockerfile
RUN npm ci --only=production && \
    npm cache clean --force
```

**5. Use .dockerignore:**
```
node_modules/
.git/
*.md
tests/
```

**Result:**
```
Before: 1.2GB
After:  85MB
```

---

### Q14: What is the difference between Docker volumes and bind mounts?

**Answer:**

**Volumes:**
- Managed by Docker
- Stored in `/var/lib/docker/volumes/`
- Better performance
- Work across all platforms
- **Use for:** Production data, databases

**Bind Mounts:**
- Direct host filesystem access
- Require absolute paths
- Dependent on host structure
- **Use for:** Development, config files

**Comparison:**
```bash
# Volume (recommended for production)
docker run -v mydata:/app/data my-app

# Bind mount (good for development)
docker run -v $(pwd):/app my-app
```

**Docker Compose:**
```yaml
services:
  app:
    volumes:
      - app-data:/app/data        # Volume
      - ./config:/app/config      # Bind mount
volumes:
  app-data:
```

---

### Q15: Explain Docker networking modes.

**Answer:**

**1. Bridge (Default):**
- Isolated network for containers
- Containers can communicate via names (with custom bridge)
```bash
docker network create mynet
docker run --network mynet my-app
```

**2. Host:**
- Uses host's network directly
- Better performance, less isolation
```bash
docker run --network host my-app
```

**3. None:**
- No networking
- Complete isolation
```bash
docker run --network none my-app
```

**4. Container:**
- Share another container's network
```bash
docker run --network container:other-container my-app
```

**5. Overlay:**
- Multi-host networking (Swarm)

---

### Q16: How do you debug a container that keeps crashing?

**Answer:**

**Step-by-Step Debugging:**

**1. Check Logs:**
```bash
docker logs container-name
docker logs -f container-name --tail 100
```

**2. Inspect Container:**
```bash
docker inspect container-name
```

**3. Override CMD:**
```bash
# Run with shell instead
docker run -it --rm my-app sh

# Override entrypoint
docker run -it --rm --entrypoint sh my-app
```

**4. Check Exit Code:**
```bash
docker ps -a
# Look at STATUS column: Exited (1), Exited (137) (OOM), etc.
```

**5. Run Without Detached Mode:**
```bash
docker run --rm my-app  # See output directly
```

**6. Add Debug Tools:**
```dockerfile
FROM node:18-alpine
RUN apk add --no-cache curl netcat-openbsd
COPY . .
CMD ["node", "server.js"]
```

---

### Q17: What is the purpose of .dockerignore?

**Answer:**
.dockerignore excludes files from the build context, improving build speed and security.

**Benefits:**
- Faster builds (smaller context)
- Smaller images
- Don't include secrets
- Reduce layer size

**Example:**
```
# .dockerignore
node_modules/
.git/
.env
*.log
README.md
tests/
coverage/
.DS_Store
```

**Impact:**
```bash
# Without .dockerignore: 500MB context
# With .dockerignore: 50MB context
# Build time: 5min → 30sec
```

---

### Q18: How do you run a command in a running container?

**Answer:**

**Using docker exec:**
```bash
# Interactive shell
docker exec -it container-name bash
# OR
docker exec -it container-name sh

# Single command
docker exec container-name ls -la /app

# As specific user
docker exec -u root container-name whoami

# With environment variables
docker exec -e VAR=value container-name printenv
```

**Examples:**
```bash
# Check logs inside container
docker exec my-app cat /var/log/app.log

# Test connectivity
docker exec my-app curl http://api:3000

# Run database query
docker exec postgres psql -U user -d mydb -c "SELECT * FROM users"

# Install debugging tool
docker exec -it -u root my-app apk add curl
```

---

### Q19: What is the difference between docker run and docker start?

**Answer:**

**docker run:**
- Creates NEW container from image
- Can specify all configuration
- Use when starting fresh

**docker start:**
- Starts EXISTING stopped container
- Uses original configuration
- Use to restart stopped container

**Example:**
```bash
# docker run (creates new container)
docker run -d --name web -p 80:80 nginx

# docker stop (stops running container)
docker stop web

# docker start (restarts stopped container)
docker start web

# docker restart (stop + start)
docker restart web
```

**Key Difference:**
```bash
# Run creates new container each time
docker run -d nginx  # Container ID: abc123
docker run -d nginx  # Container ID: def456 (different!)

# Start reuses same container
docker start abc123
docker start abc123  # Same container ID
```

---

### Q20: How do you share data between containers?

**Answer:**

**Method 1: Named Volumes (Recommended):**
```bash
# Create volume
docker volume create shared-data

# Container 1 writes
docker run -d -v shared-data:/data writer-app

# Container 2 reads
docker run -d -v shared-data:/data:ro reader-app
```

**Method 2: Docker Compose:**
```yaml
version: '3.8'
services:
  writer:
    image: writer-app
    volumes:
      - shared-data:/data
  
  reader:
    image: reader-app
    volumes:
      - shared-data:/data:ro  # Read-only

volumes:
  shared-data:
```

**Method 3: Volumes-From (Legacy):**
```bash
docker run -d --name data-container -v /data alpine
docker run -d --volumes-from data-container app1
docker run -d --volumes-from data-container app2
```

---

## 🔧 Mid-Level (Intermediate)

### Q21: Explain Docker layer caching and how to optimize it.

**Answer:**
Docker caches each layer. If a layer changes, all subsequent layers are rebuilt.

**Optimization Strategy:**
Order instructions from least to most frequently changing.

**Poor Caching:**
```dockerfile
FROM node:18
WORKDIR /app
COPY . .              # Changes break cache
RUN npm install       # Reinstalls on every change
```

**Optimized Caching:**
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./    # Rarely changes
RUN npm ci              # Cached unless package.json changes
COPY . .                # Frequent changes don't invalidate npm install
```

**Cache Busting:**
```bash
# Force rebuild without cache
docker build --no-cache -t my-app .

# Rebuild from specific layer
docker build --cache-from my-app:latest -t my-app:new .
```

---

### Q22: How do you handle environment-specific configurations in Docker?

**Answer:**

**Method 1: Environment Variables:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
# No hardcoded configs
CMD ["node", "server.js"]
```
```bash
# Development
docker run -e NODE_ENV=development my-app

# Production
docker run -e NODE_ENV=production my-app
```

**Method 2: Build Arguments:**
```dockerfile
ARG ENVIRONMENT=production
ENV NODE_ENV=${ENVIRONMENT}

RUN if [ "$ENVIRONMENT" = "production" ]; then \
      npm ci --only=production; \
    else \
      npm install; \
    fi
```
```bash
docker build --build-arg ENVIRONMENT=development -t my-app:dev .
```

**Method 3: Config Files with Volumes:**
```bash
docker run -v ./config.prod.json:/app/config.json my-app
```

**Method 4: Docker Compose Override:**
```yaml
# docker-compose.yml (base)
services:
  app:
    image: my-app

# docker-compose.override.yml (local dev)
services:
  app:
    environment:
      - DEBUG=true
```

---

### Q23: What are health checks in Docker?

**Answer:**
Health checks monitor container health and can trigger automatic restarts.

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js || exit 1

CMD ["node", "server.js"]
```

**Docker Run:**
```bash
docker run -d --name web \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  my-app
```

**Check Health:**
```bash
docker ps
# CONTAINER   STATUS
# my-app      Up 2 minutes (healthy)

docker inspect my-app | grep -A 10 Health
```

**Docker Compose:**
```yaml
services:
  app:
    image: my-app
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
```

---

### Q24: How do you secure Docker containers?

**Answer:**

**Security Best Practices:**

**1. Run as Non-Root:**
```dockerfile
FROM node:18-alpine
RUN adduser -D appuser
USER appuser
COPY . .
CMD ["node", "server.js"]
```

**2. Use Minimal Base Images:**
```dockerfile
FROM alpine:3.18  # 5MB
# OR
FROM gcr.io/distroless/static  # Even smaller
```

**3. Scan for Vulnerabilities:**
```bash
docker scan my-app
# OR
trivy image my-app
```

**4. Don't Include Secrets:**
```dockerfile
# ❌ Never do this
ENV API_KEY=secret

# ✅ Pass at runtime
docker run -e API_KEY=${API_KEY} my-app
```

**5. Use Read-Only Filesystem:**
```bash
docker run --read-only --tmpfs /tmp my-app
```

**6. Limit Resources:**
```bash
docker run --memory=512m --cpus=1.0 my-app
```

**7. Drop Capabilities:**
```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE my-app
```

---

### Q25: What is Docker networking and how does it work?

**Answer:**
Docker networking allows containers to communicate with each other and the outside world.

**Network Drivers:**

**1. Bridge (Default):**
```bash
# Create network
docker network create mynet

# Containers can reach each other by name
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet my-app

# app can connect to postgres via: postgres://db:5432
```

**2. Host:**
```bash
# Share host's network (no isolation)
docker run --network host my-app
```

**3. Overlay:**
```bash
# Multi-host networking (Swarm/Kubernetes)
docker network create -d overlay my-overlay
```

**Custom Network Example:**
```yaml
version: '3.8'
services:
  app:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
    internal: true  # No external access
```

---

### Q26: How do you handle secrets in Docker?

**Answer:**

**Method 1: Environment Variables (Simple):**
```bash
docker run -e DB_PASSWORD=${DB_PASSWORD} my-app
```

**Method 2: Docker Secrets (Swarm):**
```bash
# Create secret
echo "my-secret" | docker secret create db_password -

# Use in service
docker service create \
  --secret db_password \
  --name my-app \
  my-app:latest
```
```javascript
// Access in app
const fs = require('fs');
const password = fs.readFileSync('/run/secrets/db_password', 'utf8');
```

**Method 3: BuildKit Secrets (Build-Time):**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18
WORKDIR /app

RUN --mount=type=secret,id=npmrc \
    cat /run/secrets/npmrc > ~/.npmrc && \
    npm install
```
```bash
DOCKER_BUILDKIT=1 docker build \
  --secret id=npmrc,src=$HOME/.npmrc \
  -t my-app .
```

**Method 4: External Secret Manager:**
```bash
# AWS Secrets Manager
docker run -e AWS_REGION=us-east-1 \
  my-app  # App fetches secrets from AWS
```

---

### Q27: What is the difference between docker-compose up and docker-compose start?

**Answer:**

**docker-compose up:**
- Creates and starts containers
- Creates networks and volumes
- Rebuilds images if needed
- Use for first time or after changes

**docker-compose start:**
- Only starts existing stopped containers
- Doesn't create anything new
- Use to restart stopped services

**Example Workflow:**
```bash
# First time
docker-compose up -d

# Stop services
docker-compose stop

# Restart stopped services
docker-compose start

# Make changes to docker-compose.yml
# Recreate containers with changes
docker-compose up -d

# Remove everything
docker-compose down
```

---

### Q28: How do you limit container resources?

**Answer:**

**Memory Limits:**
```bash
docker run -d \
  --memory="512m" \
  --memory-swap="512m" \
  --memory-reservation="256m" \
  my-app
```

**CPU Limits:**
```bash
docker run -d \
  --cpus="1.5" \
  --cpu-shares=1024 \
  my-app
```

**Docker Compose:**
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

**Why Limit Resources:**
- Prevent one container from consuming all resources
- Ensure predictable performance
- Better resource distribution
- Prevent OOM (Out of Memory) issues

---

### Q29: Explain the difference between ARG and ENV.

**Answer:**

**ARG (Build-Time):**
- Only available during build
- Not in running container
- Set with --build-arg

**ENV (Runtime):**
- Available during build AND runtime
- Persists in final image
- Can override with -e

**Comparison:**
```dockerfile
ARG BUILD_ENV=production     # Build-time only
ENV APP_ENV=production       # Runtime accessible

# Convert ARG to ENV
ARG VERSION
ENV APP_VERSION=${VERSION}
```

**Usage:**
```bash
# Build with ARG
docker build --build-arg BUILD_ENV=dev -t my-app .

# Run with ENV override
docker run -e APP_ENV=staging my-app
```

**When to Use:**
- **ARG:** Build configuration, versions, toggles
- **ENV:** Application config, database URLs, API keys

---

### Q30: How do you monitor Docker containers?

**Answer:**

**Built-in Monitoring:**
```bash
# Real-time stats
docker stats

# Specific container
docker stats my-app

# Format output
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

**Inspect Container:**
```bash
# Get all container details
docker inspect my-app

# Get specific field
docker inspect my-app --format='{{.State.Status}}'
```

**Logs:**
```bash
docker logs -f my-app --tail 100
```

**Third-Party Tools:**

**Prometheus + Grafana:**
```yaml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
  
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
```

---

## Senior Level (Advanced)

### Q31: Explain Docker's storage drivers and their use cases.

**Answer:**

**Storage Drivers:**

**1. overlay2 (Recommended):**
- Most stable and performant
- Default on modern Linux
- Best for production

**2. aufs:**
- Legacy, used by older Docker versions
- Being phased out

**3. devicemapper:**
- Direct block storage
- Good for certain production scenarios

**4. btrfs/zfs:**
- Advanced filesystems
- Copy-on-write with snapshots

**Configuration:**
```json
// /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

**Check Current Driver:**
```bash
docker info | grep "Storage Driver"
```

---

### Q32: How would you implement zero-downtime deployments with Docker?

**Answer:**

**Strategy 1: Rolling Updates with Docker Swarm:**
```bash
# Deploy service
docker service create \
  --name my-app \
  --replicas 3 \
  --update-delay 10s \
  --update-parallelism 1 \
  my-app:v1

# Update with zero downtime
docker service update \
  --image my-app:v2 \
  my-app
```

**Strategy 2: Blue-Green Deployment:**
```bash
# Run new version (green)
docker run -d --name app-green my-app:v2

# Test green
curl http://app-green:3000/health

# Switch traffic (update load balancer)
# Then stop old version (blue)
docker stop app-blue
docker rm app-blue
```

**Strategy 3: Using Nginx:**
```nginx
upstream backend {
    server app-v1:3000 weight=100;
    server app-v2:3000 weight=0;  # New version, no traffic yet
}
```
```bash
# Gradually shift traffic
# Update nginx config weights:
# v1: 100 → 50 → 0
# v2: 0 → 50 → 100
```

**Strategy 4: Kubernetes Rolling Update:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero downtime
```

---

### Q33: How do you handle logging in production Docker environments?

**Answer:**

**Centralized Logging Strategy:**

**1. Log to stdout/stderr:**
```dockerfile
FROM node:18-alpine
ENV NODE_ENV=production
WORKDIR /app
COPY . .
# Application logs to console
CMD ["node", "server.js"]
```

**2. Configure Log Driver:**
```bash
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  my-app
```

**3. Ship to Centralized System:**
```yaml
# ELK Stack
services:
  app:
    logging:
      driver: syslog
      options:
        syslog-address: "tcp://logstash:5000"
  
  elasticsearch:
    image: elasticsearch:8.11.0
  
  logstash:
    image: logstash:8.11.0
  
  kibana:
    image: kibana:8.11.0
```

**4. Use Fluentd:**
```yaml
services:
  app:
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: app.logs
```

---

### Q34: Explain Docker Swarm vs Kubernetes.

**Answer:**

**Docker Swarm:**
- **Pros:** Simple, built into Docker, easy learning curve
- **Cons:** Limited features, smaller ecosystem
- **Use For:** Small to medium deployments, simpler requirements

**Kubernetes:**
- **Pros:** Feature-rich, huge ecosystem, industry standard
- **Cons:** Complex, steep learning curve
- **Use For:** Large scale, complex orchestration, enterprise

**Feature Comparison:**
| Feature | Docker Swarm | Kubernetes |
|---------|--------------|------------|
| **Setup** | Simple | Complex |
| **Learning Curve** | Easy | Steep |
| **Scaling** | Basic | Advanced |
| **Load Balancing** | Built-in | Flexible |
| **Auto-healing** | Basic | Advanced |
| **Community** | Smaller | Massive |
| **Monitoring** | Limited | Extensive |

**When to Choose:**
- **Swarm:** Small team, simple app, quick setup
- **Kubernetes:** Enterprise, complex apps, need advanced features

---

### Q35: How do you implement container security scanning?

**Answer:**

**Tools and Methods:**

**1. Docker Scan (Snyk):**
```bash
docker scan my-app:latest

# Output shows vulnerabilities:
# ✗ High severity vulnerability found in openssl
# Fixed in: 1.1.1k-r0
```

**2. Trivy:**
```bash
# Install
brew install trivy

# Scan image
trivy image my-app:latest

# Scan in CI/CD
trivy image --exit-code 1 --severity HIGH,CRITICAL my-app:latest
```

**3. Clair:**
```bash
# Run Clair
docker run -d -p 6060:6060 quay.io/coreos/clair

# Scan with clairctl
clairctl analyze my-app:latest
```

**4. Integrate in Dockerfile:**
```dockerfile
FROM base AS scanner
COPY --from=aquasec/trivy:latest /usr/local/bin/trivy /usr/local/bin/trivy
RUN trivy filesystem --exit-code 1 --severity HIGH,CRITICAL /app

FROM base AS final
# Only builds if scan passes
```

**5. CI/CD Integration:**
```yaml
# GitLab CI
security-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

---

### Q36: Explain Docker build context and why it matters.

**Answer:**
Build context is all files sent to Docker daemon during build. Large context = slow builds.

**Problem:**
```bash
# Sending 2GB context
docker build .
# Sending build context to Docker daemon  2.1GB
# Step 1/5 : FROM node:18
```

**Solutions:**

**1. Use .dockerignore:**
```
node_modules/
.git/
*.log
dist/
```

**2. Minimize Context:**
```bash
# Build from subdirectory
docker build -f docker/Dockerfile ./app

# Use specific context
docker build -f Dockerfile -t my-app ./src
```

**3. Multi-Stage to Reduce Copies:**
```dockerfile
FROM node:18 AS deps
COPY package*.json ./
RUN npm ci

FROM node:18
COPY --from=deps /app/node_modules ./node_modules
COPY src ./src  # Only copy what's needed
```

**Impact:**
```
Before .dockerignore: 2.1GB context, 3min build
After .dockerignore:  50MB context, 15sec build
```

---

### Q37: How do you implement service discovery in Docker?

**Answer:**

**Method 1: Docker Network DNS:**
```bash
# Create network
docker network create app-net

# Services auto-discover by name
docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net -e DB_HOST=db my-api
docker run -d --name web --network app-net -e API_URL=http://api my-web

# 'db', 'api', 'web' resolve automatically
```

**Method 2: Docker Compose:**
```yaml
services:
  web:
    environment:
      - API_URL=http://api:3000  # Automatic DNS
  api:
    environment:
      - DB_URL=postgres://db:5432
  db:
    image: postgres
```

**Method 3: External Service Discovery (Consul):**
```yaml
services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
  
  registrator:
    image: gliderlabs/registrator
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
    command: consul://consul:8500
```

---

### Q38: What are Docker BuildKit features and advantages?

**Answer:**
BuildKit is Docker's next-generation build system with improved performance and features.

**Key Features:**

**1. Parallel Building:**
```dockerfile
FROM base AS stage1
RUN long-task-1

FROM base AS stage2
RUN long-task-2

# Both run in parallel!
FROM alpine
COPY --from=stage1 /output1 .
COPY --from=stage2 /output2 .
```

**2. Build Secrets:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=secret,id=github_token \
    git clone https://$(cat /run/secrets/github_token)@github.com/repo
```

**3. Build Cache Mounts:**
```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

**4. SSH Forwarding:**
```dockerfile
RUN --mount=type=ssh \
    git clone git@github.com:user/private-repo.git
```

**Enable BuildKit:**
```bash
export DOCKER_BUILDKIT=1
docker build -t my-app .
```

---

### Q39: How would you design a microservices architecture using Docker?

**Answer:**

**Architecture Components:**

**1. Service Organization:**
```
microservices/
├── docker-compose.yml
├── api-gateway/
│   └── Dockerfile
├── user-service/
│   └── Dockerfile
├── order-service/
│   └── Dockerfile
├── product-service/
│   └── Dockerfile
└── notification-service/
    └── Dockerfile
```

**2. Complete docker-compose.yml:**
```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    build: ./api-gateway
    ports:
      - "80:3000"
    environment:
      - USER_SERVICE=http://user-service:3000
      - ORDER_SERVICE=http://order-service:3000
    depends_on:
      - user-service
      - order-service
    networks:
      - frontend
      - backend

  # User Service
  user-service:
    build: ./user-service
    environment:
      - DATABASE_URL=postgres://users-db:5432/users
      - REDIS_URL=redis://redis:6379
    depends_on:
      - users-db
      - redis
    networks:
      - backend
    deploy:
      replicas: 3

  # Order Service
  order-service:
    build: ./order-service
    environment:
      - DATABASE_URL=postgres://orders-db:5432/orders
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - orders-db
      - rabbitmq
    networks:
      - backend
    deploy:
      replicas: 2

  # Databases
  users-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users
    volumes:
      - users-data:/var/lib/postgresql/data
    networks:
      - backend

  orders-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders
    volumes:
      - orders-data:/var/lib/postgresql/data
    networks:
      - backend

  # Message Queue
  rabbitmq:
    image: rabbitmq:3-management
    networks:
      - backend

  # Cache
  redis:
    image: redis:7-alpine
    networks:
      - backend

volumes:
  users-data:
  orders-data:

networks:
  frontend:
  backend:
    internal: true
```

**3. Service Communication:**
- HTTP REST for synchronous
- RabbitMQ for asynchronous
- Redis for caching
- Separate databases per service

---

### Q40: How do you handle container orchestration at scale?

**Answer:**

**Options:**

**1. Docker Swarm:**
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# Scale service
docker service scale myapp_web=10

# Update service
docker service update --image myapp:v2 myapp_web
```

**2. Kubernetes:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
  template:
    spec:
      containers:
      - name: app
        image: my-app:v2
        resources:
          limits:
            memory: "512Mi"
            cpu: "1000m"
```

**3. Key Considerations:**
- **Service Discovery:** DNS, service mesh
- **Load Balancing:** Distribute traffic
- **Auto-Scaling:** Based on metrics
- **Health Checks:** Auto-restart failed containers
- **Rolling Updates:** Zero-downtime deployments
- **Secrets Management:** Secure sensitive data
- **Monitoring:** Centralized logging and metrics

---

## Scenario-Based Questions

### Q41: Your container works locally but fails in production. How do you debug?

**Answer:**

**Debugging Checklist:**

**1. Compare Environments:**
```bash
# Check Docker versions
docker version

# Check image
docker inspect my-app:latest

# Compare environment variables
docker exec prod-container env > prod-env.txt
docker exec local-container env > local-env.txt
diff prod-env.txt local-env.txt
```

**2. Check Logs:**
```bash
# Production logs
docker logs prod-container --tail 500

# Look for errors
docker logs prod-container 2>&1 | grep -i error
```

**3. Verify Configuration:**
```bash
# Check mounted volumes
docker inspect prod-container | grep -A 20 Mounts

# Check network
docker inspect prod-container | grep -A 20 NetworkSettings

# Check resource limits
docker inspect prod-container | grep -A 10 Memory
```

**4. Test Connectivity:**
```bash
# From inside container
docker exec prod-container curl http://db:5432

# DNS resolution
docker exec prod-container nslookup db

# Network interfaces
docker exec prod-container ip addr
```

**5. Reproduce Locally:**
```bash
# Use production environment variables
docker run --env-file prod.env my-app:latest

# Use same network setup
docker network create prod-network
docker run --network prod-network my-app
```

**Common Causes:**
- Environment variables differ
- Volume paths incorrect
- Network configuration different
- Resource constraints in prod
- Firewall/security groups blocking

---

### Q42: How would you migrate a legacy application to Docker?

**Answer:**

**Migration Strategy:**

**Phase 1: Assessment**
```bash
# Document current setup
- Application language/framework
- Dependencies and versions
- Configuration files
- Database connections
- External services
- File storage locations
- Environment variables
```

**Phase 2: Containerize (Incremental)**

**Step 1: Basic Dockerfile:**
```dockerfile
FROM node:18
WORKDIR /app

# Copy everything first (quick start)
COPY . .
RUN npm install

EXPOSE 3000
CMD ["node", "server.js"]
```

**Step 2: Add Database:**
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

**Step 3: Optimize:**
```dockerfile
# Multi-stage build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
RUN npm ci --only=production
USER node
CMD ["node", "dist/server.js"]
```

**Phase 3: Testing**
```bash
# Test locally
docker-compose up

# Load test
docker run --rm -it williamyeh/wrk \
  -t4 -c100 -d30s http://app:3000

# Security scan
trivy image my-app:latest
```

**Phase 4: Deployment**
- Use CI/CD pipeline
- Blue-green deployment
- Monitor closely
- Keep rollback plan ready

---

### Q43: A container is consuming 100% CPU. How do you investigate and fix?

**Answer:**

**Investigation Steps:**

**1. Identify the Issue:**
```bash
# Check CPU usage
docker stats

# Output:
# CONTAINER   CPU %   MEM USAGE
# my-app      198%    500MB
```

**2. Check Running Processes:**
```bash
# Inside container
docker exec my-app ps aux

# Sort by CPU
docker exec my-app ps aux --sort=-pcpu | head -10
```

**3. Profile the Application:**
```bash
# Node.js profiling
docker exec my-app node --prof server.js

# Python profiling
docker exec my-app python -m cProfile app.py

# Get strace
docker exec my-app strace -p 1
```

**4. Check for Infinite Loops:**
```bash
# View application logs
docker logs my-app --tail 1000

# Look for repeated errors or warnings
docker logs my-app | grep -c "Error"
```

**Solutions:**

**1. Set CPU Limits:**
```bash
docker run -d \
  --cpus="1.0" \
  --cpu-shares=512 \
  my-app:latest
```

**2. Fix Application Code:**
```javascript
// ❌ Infinite loop
while (true) {
  processData();  // No await, no sleep
}

// ✅ Fixed
while (true) {
  await processData();
  await new Promise(resolve => setTimeout(resolve, 1000));
}
```

**3. Scale Horizontally:**
```yaml
services:
  app:
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
```

---

### Q44: How do you handle database migrations in Docker?

**Answer:**

**Method 1: Init Container Pattern:**
```yaml
version: '3.8'
services:
  # Migration service
  migrate:
    build: .
    command: npm run migrate
    environment:
      DATABASE_URL: postgres://db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
  
  # Application
  app:
    build: .
    depends_on:
      migrate:
        condition: service_completed_successfully
  
  db:
    image: postgres:15
    healthcheck:
      test: pg_isready -U postgres
```

**Method 2: Entrypoint Script:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["node", "server.js"]
```

```bash
#!/bin/sh
# docker-entrypoint.sh
set -e

# Wait for database
until nc -z db 5432; do
  echo "Waiting for database..."
  sleep 1
done

# Run migrations
npm run migrate

# Start application
exec "$@"
```

**Method 3: Separate Migration Container:**
```bash
# Build migration image
docker build -f Dockerfile.migrate -t my-app-migrate .

# Run migrations
docker run --rm \
  -e DATABASE_URL=postgres://db:5432/mydb \
  my-app-migrate

# Start application
docker run -d my-app
```

**Method 4: Using Flyway/Liquibase:**
```yaml
services:
  flyway:
    image: flyway/flyway
    command: migrate
    volumes:
      - ./migrations:/flyway/sql
    environment:
      - FLYWAY_URL=jdbc:postgresql://db:5432/mydb
      - FLYWAY_USER=user
      - FLYWAY_PASSWORD=pass
    depends_on:
      - db
```

---

### Q45: Your Docker registry is slow. How do you optimize it?

**Answer:**

**Solutions:**

**1. Use Registry Mirror:**
```json
// /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://your-company-mirror.com"
  ]
}
```

**2. Set Up Private Registry:**
```yaml
version: '3.8'
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - registry-data:/var/lib/registry
    environment:
      REGISTRY_STORAGE_DELETE_ENABLED: true

volumes:
  registry-data:
```

**3. Use Pull-Through Cache:**
```yaml
services:
  registry-cache:
    image: registry:2
    environment:
      REGISTRY_PROXY_REMOTEURL: https://registry-1.docker.io
    volumes:
      - cache-data:/var/lib/registry
```

**4. Optimize Image Layers:**
```dockerfile
# Reduce layers
FROM node:18-alpine
RUN npm install -g package1 package2 package3  # One layer

# Not separate RUN commands
```

**5. Use Layer Caching in CI/CD:**
```yaml
# GitHub Actions
- name: Build
  uses: docker/build-push-action@v4
  with:
    cache-from: type=registry,ref=user/app:cache
    cache-to: type=registry,ref=user/app:cache,mode=max
```

---

### Q46: How do you implement health checks across multiple services?

**Answer:**

**Complete Health Check Strategy:**

```yaml
version: '3.8'

services:
  # Frontend with HTTP health check
  frontend:
    image: my-frontend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 40s
    depends_on:
      backend:
        condition: service_healthy

  # Backend with custom health script
  backend:
    image: my-backend
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 10s
      timeout: 3s
      retries: 5
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  # PostgreSQL with pg_isready
  postgres:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis with redis-cli ping
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # MongoDB with mongosh
  mongodb:
    image: mongo:7
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 3

  # Elasticsearch with HTTP check
  elasticsearch:
    image: elasticsearch:8.11.0
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
```

**Custom Health Check Script:**
```javascript
// healthcheck.js
const http = require('http');
const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  process.exit(res.statusCode === 200 ? 0 : 1);
});

request.on('error', () => process.exit(1));
request.end();
```

---

### Q47: How would you implement a CI/CD pipeline for Docker images?

**Answer:**

**Complete Pipeline Example:**

**GitHub Actions:**
```yaml
name: Docker CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: |
          docker build --target test -t test-image .
          docker run --rm test-image npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache,mode=max
      
      - name: Security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}
          format: 'sarif'
          output: 'trivy-results.sarif'

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Deploy to production
        run: |
          # Deploy to Kubernetes/ECS/etc
          kubectl set image deployment/my-app \
            my-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

**Multi-Stage Dockerfile for CI:**
```dockerfile
# Test stage
FROM node:18 AS test
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run lint
RUN npm test

# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
USER node
CMD ["node", "dist/server.js"]
```

---

### Q48: How do you implement centralized logging for Docker containers?

**Answer:**

**ELK Stack Implementation:**

```yaml
version: '3.8'

services:
  # Application containers
  app1:
    image: my-app
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
        labels: "app=myapp,env=prod"
  
  # Elasticsearch
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  # Logstash
  logstash:
    image: logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch

  # Kibana
  kibana:
    image: kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  # Filebeat (collects Docker logs)
  filebeat:
    image: elastic/filebeat:8.11.0
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

**Alternative: Loki + Promtail + Grafana:**
```yaml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
```

---

### Q49: Design a disaster recovery strategy for Dockerized applications.

**Answer:**

**Comprehensive DR Strategy:**

**1. Image Backup:**
```bash
# Save images
docker save my-app:latest > my-app-backup.tar

# Push to multiple registries
docker tag my-app:latest registry1.com/my-app:latest
docker tag my-app:latest registry2.com/my-app:latest
docker push registry1.com/my-app:latest
docker push registry2.com/my-app:latest
```

**2. Volume Backup:**
```bash
#!/bin/bash
# backup-volumes.sh

# Backup all volumes
for volume in $(docker volume ls -q); do
  docker run --rm \
    -v $volume:/data \
    -v $(pwd)/backups:/backup \
    alpine tar czf /backup/${volume}-$(date +%Y%m%d).tar.gz -C /data .
done

# Upload to S3
aws s3 sync ./backups s3://my-bucket/docker-backups/
```

**3. Configuration Backup:**
```bash
# Backup Docker Compose files
tar czf compose-backup.tar.gz docker-compose.yml .env

# Backup secrets
docker secret ls -q | xargs -I {} docker secret inspect {}
```

**4. Automated DR Plan:**
```yaml
# docker-compose.yml with backup service
services:
  app:
    image: my-app
    volumes:
      - app-data:/app/data

  backup:
    image: alpine
    volumes:
      - app-data:/data:ro
      - ./backups:/backup
    environment:
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    command: >
      sh -c "
        while true; do
          tar czf /backup/backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /data . &&
          aws s3 cp /backup/backup-*.tar.gz s3://my-bucket/ &&
          find /backup -mtime +7 -delete &&
          sleep 86400;
        done
      "

volumes:
  app-data:
```

**5. Recovery Procedure:**
```bash
# 1. Restore volumes
aws s3 cp s3://my-bucket/backup-20240101.tar.gz ./
docker volume create app-data
docker run --rm -v app-data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/backup-20240101.tar.gz -C /data

# 2. Pull images
docker pull my-app:latest

# 3. Start services
docker-compose up -d

# 4. Verify
docker-compose ps
curl http://localhost/health
```

---

### Q50: How do you implement zero-trust security in Docker?

**Answer:**

**Zero-Trust Principles:**

**1. Least Privilege Access:**
```dockerfile
FROM node:18-alpine

# Create minimal user
RUN adduser -D -u 1001 appuser && \
    mkdir /app && \
    chown appuser:appuser /app

WORKDIR /app
USER appuser

# Read-only filesystem
```
```bash
docker run --read-only --tmpfs /tmp my-app
```

**2. Network Segmentation:**
```yaml
services:
  web:
    networks:
      - public
  
  api:
    networks:
      - public
      - private
  
  db:
    networks:
      - private  # Only accessible to API

networks:
  public:
  private:
    internal: true  # No internet access
```

**3. Runtime Security:**
```bash
# Drop all capabilities
docker run --cap-drop=ALL my-app

# Read-only root filesystem
docker run --read-only --tmpfs /tmp my-app

# No new privileges
docker run --security-opt=no-new-privileges my-app

# AppArmor/SELinux
docker run --security-opt apparmor=docker-default my-app
```

**4. Image Signing:**
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push my-app:latest
# Automatically signs image

# Pull only signed images
docker pull my-app:latest
```

**5. Regular Scanning:**
```bash
# Scan in pipeline
trivy image --severity HIGH,CRITICAL my-app:latest

# Fail build if vulnerabilities found
trivy image --exit-code 1 --severity CRITICAL my-app:latest
```

---

## 🎓 Advanced Architecture Questions

### Q51: Design a microservices deployment with service mesh.

**Answer:**

**Architecture with Istio Service Mesh:**

**1. Service Dockerfile:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
RUN apk add --no-cache curl
WORKDIR /app
COPY --from=build /app/dist ./dist
RUN adduser -D -u 1001 nodejs
USER nodejs

# Health endpoint for Istio
EXPOSE 3000
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

**2. Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
        version: v1
      annotations:
        sidecar.istio.io/inject: "true"  # Inject Istio sidecar
    spec:
      containers:
      - name: user-service
        image: my-app/user-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: postgres
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
```

**3. Istio Configuration:**
```yaml
# VirtualService for traffic management
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - match:
    - headers:
        version:
          exact: beta
    route:
    - destination:
        host: user-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: user-service
        subset: v1
      weight: 100
```

---

### Q52: How do you handle secrets rotation in production?

**Answer:**

**Secrets Rotation Strategy:**

**1. External Secret Manager:**
```dockerfile
FROM node:18-alpine
WORKDIR /app

# Install secret fetching library
RUN npm install @aws-sdk/client-secrets-manager

COPY . .

# App fetches secrets at startup
CMD ["node", "server.js"]
```

```javascript
// server.js
const { SecretsManagerClient, GetSecretValueCommand } = require("@aws-sdk/client-secrets-manager");

async function getSecret(secretName) {
  const client = new SecretsManagerClient({ region: "us-east-1" });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretName })
  );
  return JSON.parse(response.SecretString);
}

// Refresh periodically
setInterval(async () => {
  const newSecrets = await getSecret("app-secrets");
  updateSecrets(newSecrets);
}, 3600000); // Every hour
```

**2. Kubernetes Secrets with CSI Driver:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets"
      readOnly: true
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "aws-secrets"
```

**3. Docker Swarm Secrets Update:**
```bash
# Update secret
echo "new-secret-value" | docker secret create db_password_v2 -

# Update service
docker service update \
  --secret-rm db_password \
  --secret-add db_password_v2 \
  my-service

# Remove old secret
docker secret rm db_password
```

---

### Q53: Explain your strategy for Docker image vulnerability management.

**Answer:**

**Comprehensive Vulnerability Management:**

**1. Image Scanning in Pipeline:**
```yaml
# .gitlab-ci.yml
security-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 0 --severity LOW,MEDIUM $IMAGE
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE
  artifacts:
    reports:
      container_scanning: trivy-report.json
```

**2. Base Image Selection:**
```dockerfile
# ✅ Use minimal, frequently updated bases
FROM gcr.io/distroless/nodejs:18  # No shell, no package manager
# OR
FROM node:18-alpine  # Minimal packages
```

**3. Regular Updates:**
```bash
# Automated rebuild weekly
# GitHub Actions schedule:
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday
```

**4. Runtime Protection:**
```yaml
services:
  app:
    image: my-app
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
```

**5. Admission Control:**
```yaml
# Kubernetes: Block vulnerable images
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image-vulnerability
spec:
  validationFailureAction: enforce
  rules:
  - name: block-high-vulnerabilities
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "High severity vulnerabilities found"
      pattern:
        spec:
          containers:
          - image: "!*:*@sha256:*"  # Require digest
```

---

### Q54: How do you implement a blue-green deployment strategy?

**Answer:**

**Complete Blue-Green Deployment:**

**1. Setup Infrastructure:**
```yaml
version: '3.8'

services:
  # Load Balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app-blue
      - app-green

  # Blue Environment (Current)
  app-blue:
    image: my-app:v1.0
    deploy:
      replicas: 3
    environment:
      - VERSION=blue
    networks:
      - app-network

  # Green Environment (New)
  app-green:
    image: my-app:v2.0
    deploy:
      replicas: 3
    environment:
      - VERSION=green
    networks:
      - app-network

  # Shared Database
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:

volumes:
  db-data:
```

**2. Nginx Configuration:**
```nginx
upstream backend {
    # All traffic to blue initially
    server app-blue:3000 weight=100;
    server app-green:3000 weight=0;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

**3. Deployment Script:**
```bash
#!/bin/bash

CURRENT_ENV="blue"
NEW_ENV="green"

# 1. Deploy new version
docker-compose up -d app-${NEW_ENV}

# 2. Wait for health checks
sleep 30

# 3. Run smoke tests
curl http://app-${NEW_ENV}:3000/health || exit 1

# 4. Gradually shift traffic
# Update nginx config:
# blue: 100 → 75 → 50 → 25 → 0
# green: 0 → 25 → 50 → 75 → 100

# 5. Switch completely to green
docker exec nginx sed -i 's/blue:3000 weight=100/blue:3000 weight=0/' /etc/nginx/nginx.conf
docker exec nginx sed -i 's/green:3000 weight=0/green:3000 weight=100/' /etc/nginx/nginx.conf
docker exec nginx nginx -s reload

# 6. Monitor for issues
sleep 300  # 5 minutes

# 7. If all good, stop old environment
docker-compose stop app-${CURRENT_ENV}

# 8. Cleanup
docker-compose rm -f app-${CURRENT_ENV}
```

---

### Q55: How would you implement rate limiting for containerized APIs?

**Answer:**

**Multiple Approaches:**

**1. Application-Level (Node.js Express):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install express-rate-limit
COPY . .
CMD ["node", "server.js"]
```
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100  // 100 requests per window
});

app.use('/api/', limiter);
```

**2. Nginx Reverse Proxy:**
```nginx
# nginx.conf
http {
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    
    server {
        location /api {
            limit_req zone=api_limit burst=20 nodelay;
            proxy_pass http://backend;
        }
    }
}
```

```yaml
services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
  
  backend:
    image: my-api
    deploy:
      replicas: 3
```

**3. Redis-Based Distributed Rate Limiting:**
```javascript
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

async function rateLimitMiddleware(req, res, next) {
  const key = `rate_limit:${req.ip}`;
  const current = await redis.incr(key);
  
  if (current === 1) {
    await redis.expire(key, 60);  // 1 minute window
  }
  
  if (current > 100) {
    return res.status(429).json({ error: 'Too many requests' });
  }
  
  next();
}
```

**4. API Gateway (Kong):**
```yaml
services:
  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-db
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"

# Configure via Kong Admin API:
# curl -X POST http://localhost:8001/plugins \
#   --data "name=rate-limiting" \
#   --data "config.minute=100"
```

---

### Q56: How do you monitor and optimize Docker image layers?

**Answer:**

**Layer Analysis:**

**1. Inspect Layers:**
```bash
# View layer history
docker history my-app:latest

# Show layer sizes
docker history --no-trunc my-app:latest

# Use dive tool for detailed analysis
dive my-app:latest
```

**2. Optimize Layers:**
```dockerfile
# ❌ Bad: Many small layers
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y vim
RUN apt-get clean

# ✅ Good: Combined into one layer
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y \
        curl \
        wget \
        vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**3. Layer Caching Strategy:**
```dockerfile
FROM node:18 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci  # Heavy operation, cache it!

FROM node:18 AS build
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**4. Squash Layers (Experimental):**
```bash
# Squash all layers into one
docker build --squash -t my-app:squashed .

# Useful for final production images
```

---

### Q57: How do you implement canary deployments with Docker?

**Answer:**

**Canary Deployment Strategy:**

**1. Using Docker Swarm:**
```bash
# Deploy version 1 (90% traffic)
docker service create \
  --name my-app-v1 \
  --replicas 9 \
  my-app:v1

# Deploy version 2 (10% traffic - canary)
docker service create \
  --name my-app-v2 \
  --replicas 1 \
  my-app:v2

# Monitor metrics, if good, scale v2
docker service scale my-app-v2=10
docker service scale my-app-v1=0
```

**2. Using Traefik:**
```yaml
version: '3.8'
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--providers.docker=true"
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  app-stable:
    image: my-app:v1
    deploy:
      replicas: 9
    labels:
      - "traefik.http.services.myapp.loadbalancer.server.port=3000"
      - "traefik.http.services.myapp.loadbalancer.server.weight=90"

  app-canary:
    image: my-app:v2
    deploy:
      replicas: 1
    labels:
      - "traefik.http.services.myapp.loadbalancer.server.port=3000"
      - "traefik.http.services.myapp.loadbalancer.server.weight=10"
```

**3. With Kubernetes:**
```yaml
# Stable deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: my-app
      version: stable

# Canary deployment
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: canary

# Service (distributes traffic)
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app  # Selects both stable and canary
```

**4. Monitoring:**
```bash
# Watch metrics
watch -n 5 'docker service ps my-app-v2'

# Check error rates
docker service logs my-app-v2 | grep -c "ERROR"

# Rollback if issues
docker service scale my-app-v2=0
```

---

### Q58: Design a multi-region Docker deployment strategy.

**Answer:**

**Multi-Region Architecture:**

**1. Image Distribution:**
```bash
# Push to multiple regional registries
regions=("us-east-1" "eu-west-1" "ap-south-1")

for region in "${regions[@]}"; do
  aws ecr get-login-password --region $region | \
    docker login --username AWS --password-stdin \
    123456.dkr.ecr.$region.amazonaws.com
  
  docker tag my-app:latest \
    123456.dkr.ecr.$region.amazonaws.com/my-app:latest
  
  docker push \
    123456.dkr.ecr.$region.amazonaws.com/my-app:latest
done
```

**2. Regional Deployment:**
```yaml
# docker-stack-us-east.yml
version: '3.8'
services:
  app:
    image: 123456.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
    deploy:
      replicas: 5
      placement:
        constraints:
          - node.labels.region == us-east-1
    environment:
      - REGION=us-east-1
      - DB_HOST=db-us-east-1.amazonaws.com

# docker-stack-eu-west.yml
services:
  app:
    image: 123456.dkr.ecr.eu-west-1.amazonaws.com/my-app:latest
    deploy:
      replicas: 5
      placement:
        constraints:
          - node.labels.region == eu-west-1
    environment:
      - REGION=eu-west-1
      - DB_HOST=db-eu-west-1.amazonaws.com
```

**3. Health Checks and Failover:**
```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 10s
      timeout: 5s
      retries: 3
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
```

**4. Global Load Balancer:**
- Use Route 53 with latency-based routing
- CloudFront for static assets
- Regional failover configuration

---

### Q59: How do you implement distributed tracing for microservices in Docker?

**Answer:**

**Using Jaeger for Distributed Tracing:**

**1. Infrastructure Setup:**
```yaml
version: '3.8'

services:
  # Jaeger All-in-One
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"
      - "9411:9411"
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411

  # Service 1
  user-service:
    build: ./user-service
    environment:
      - JAEGER_AGENT_HOST=jaeger
      - JAEGER_AGENT_PORT=6831
      - SERVICE_NAME=user-service
    depends_on:
      - jaeger

  # Service 2
  order-service:
    build: ./order-service
    environment:
      - JAEGER_AGENT_HOST=jaeger
      - JAEGER_AGENT_PORT=6831
      - SERVICE_NAME=order-service
    depends_on:
      - jaeger
```

**2. Application Instrumentation:**
```javascript
// Node.js with OpenTelemetry
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');

const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new SimpleSpanProcessor(
    new JaegerExporter({
      endpoint: `http://${process.env.JAEGER_AGENT_HOST}:14268/api/traces`,
    })
  )
);
provider.register();

// Trace HTTP requests automatically
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
new HttpInstrumentation().enable();
```

---

### Q60: What's your approach to Docker in production vs development?

**Answer:**

**Development vs Production Differences:**

**Development Dockerfile:**
```dockerfile
FROM node:18
WORKDIR /app

# Install all dependencies (including dev)
COPY package*.json ./
RUN npm install

# Enable debugging tools
RUN npm install -g nodemon

# Don't copy code (use volume mount)
EXPOSE 3000 9229
CMD ["nodemon", "--inspect=0.0.0.0:9229", "server.js"]
```

**Production Dockerfile:**
```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --production

FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
RUN adduser -D appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Development Compose:**
```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src  # Hot reload
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    ports:
      - "3000:3000"
      - "9229:9229"  # Debugger
```

**Production Compose:**
```yaml
version: '3.8'
services:
  app:
    image: my-app:${VERSION}
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

**Key Differences:**
- **Dev:** Hot reload, debugging, verbose logging
- **Prod:** Optimized images, health checks, resource limits, security hardening

---

## 📝 Quick Reference

### Most Important Concepts

1. **Images vs Containers:** Blueprint vs Instance
2. **Layer Caching:** Order matters for build speed
3. **Multi-Stage Builds:** Reduce image size 80-95%
4. **Volumes:** Persist data beyond container lifetime
5. **Networks:** Container communication
6. **Security:** Non-root users, minimal base images
7. **Health Checks:** Container monitoring
8. **Docker Compose:** Multi-container orchestration

### Red Flags in Interviews

❌ Using `latest` tag in production
❌ Running as root user
❌ No health checks
❌ Hardcoding secrets in Dockerfile
❌ Not using .dockerignore
❌ Single-stage builds for production
❌ No resource limits
❌ Storing data inside container

### Green Flags

✅ Multi-stage builds
✅ Specific version tags
✅ Non-root users
✅ Health checks implemented
✅ Volumes for persistence
✅ Custom networks
✅ .dockerignore file
✅ Security scanning in CI/CD

---

## 💡 Pro Tips for Interviews

1. **Always explain WHY:** Don't just say "use Alpine" - explain it reduces attack surface and image size
2. **Give real examples:** Reference actual projects or scenarios
3. **Discuss trade-offs:** "Alpine is smaller but may have compatibility issues"
4. **Show problem-solving:** Walk through debugging steps
5. **Mention monitoring:** Production always needs observability
6. **Security first:** Always mention security considerations
7. **Know the ecosystem:** Docker Compose, Swarm, Kubernetes, monitoring tools

---

**End of Docker Interview Questions & Answers**

This guide covers essential Docker interview questions from junior to senior level, including scenario-based problems that test real-world expertise. Good luck with your interviews! 🚀
