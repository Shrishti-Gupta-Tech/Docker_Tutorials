# Chapter 11: Deployment with Containers

## Table of Contents
- [11.1 Introduction to Containers](#111-introduction-to-containers)
- [11.2 Containerizing a Service](#112-containerizing-a-service)
- [11.3 Deploying to a Cluster](#113-deploying-to-a-cluster)

---

## 11.1. Introduction to Containers

### What are Containers?

**Containers** are lightweight, standalone, executable packages that include everything needed to run a piece of software:
- Application code
- Runtime environment
- System tools
- System libraries
- Settings

### Key Concepts

#### 1. **Container vs Virtual Machine**
```
Virtual Machine:                    Container:
┌─────────────────────┐            ┌─────────────────────┐
│   App A   │  App B  │            │   App A   │  App B  │
├───────────┼─────────┤            ├───────────┼─────────┤
│   Bins/Libs         │            │   Bins/Libs         │
├─────────────────────┤            ├─────────────────────┤
│   Guest OS (GBs)    │            │   Docker Engine     │
├─────────────────────┤            ├─────────────────────┤
│   Hypervisor        │            │   Host OS           │
├─────────────────────┤            ├─────────────────────┤
│   Host OS           │            │   Infrastructure    │
└─────────────────────┘            └─────────────────────┘
```

**Differences:**
- **VMs**: Heavy (GBs), slow startup (minutes), full OS per VM
- **Containers**: Lightweight (MBs), fast startup (seconds), share host OS kernel

#### 2. **Benefits of Containers**
- **Portability**: Run anywhere (dev, staging, production)
- **Consistency**: Same environment across all stages
- **Isolation**: Apps don't interfere with each other
- **Efficiency**: Less resource usage than VMs
- **Scalability**: Quick to start/stop/replicate

#### 3. **Container Architecture**
```
┌──────────────────────────────────────────┐
│         Container Runtime (Docker)        │
├──────────────────────────────────────────┤
│  Container 1  │  Container 2  │ Container 3 │
│  ┌─────────┐  │  ┌─────────┐  │ ┌─────────┐ │
│  │  App    │  │  │  App    │  │ │  App    │ │
│  │  Code   │  │  │  Code   │  │ │  Code   │ │
│  └─────────┘  │  └─────────┘  │ └─────────┘ │
└──────────────────────────────────────────┘
```

### Docker Components

#### 1. **Docker Image**
- Read-only template with instructions for creating a container
- Contains application code, libraries, dependencies
- Stored in registries (Docker Hub, private registries)

#### 2. **Docker Container**
- Running instance of an image
- Isolated process on the host system
- Can be started, stopped, moved, deleted

#### 3. **Dockerfile**
- Text file with instructions to build an image
- Defines base image, dependencies, commands

#### 4. **Docker Registry**
- Storage and distribution system for Docker images
- Public: Docker Hub
- Private: AWS ECR, Google GCR, Azure ACR

### Basic Docker Commands

#### Installation Verification
```bash
# Check Docker version
docker --version
# Output: Docker version 24.0.0, build xyz

# Check detailed Docker info
docker info

# Test Docker installation
docker run hello-world
```

#### Image Management
```bash
# Search for images on Docker Hub
docker search nginx

# Pull an image from registry
docker pull nginx:latest
docker pull ubuntu:20.04

# List all local images
docker images
# or
docker image ls

# Remove an image
docker rmi nginx:latest
# or force remove
docker rmi -f nginx:latest

# Remove all unused images
docker image prune -a
```

#### Container Management
```bash
# Run a container
docker run nginx
docker run -d nginx                    # Detached mode (background)
docker run -d --name my-nginx nginx    # With custom name
docker run -d -p 8080:80 nginx         # Port mapping (host:container)

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop my-nginx
docker stop <container-id>

# Start a stopped container
docker start my-nginx

# Restart a container
docker restart my-nginx

# Remove a container
docker rm my-nginx
# Force remove (even if running)
docker rm -f my-nginx

# Remove all stopped containers
docker container prune
```

#### Container Interaction
```bash
# Execute command in running container
docker exec -it my-nginx bash
docker exec my-nginx ls /etc/nginx

# View container logs
docker logs my-nginx
docker logs -f my-nginx           # Follow logs (tail)
docker logs --tail 100 my-nginx   # Last 100 lines

# Inspect container details
docker inspect my-nginx

# View container resource usage
docker stats my-nginx

# Copy files to/from container
docker cp file.txt my-nginx:/path/to/file
docker cp my-nginx:/path/to/file ./local-path
```

---

## 11.2. Containerizing a Service

### Step-by-Step Guide to Containerize an Application

#### Example 1: Simple Node.js Application

**Step 1: Create Application Files**

`app.js`:
```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello from Docker Container!');
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date() });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

`package.json`:
```json
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "description": "Simple Node.js app for Docker",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

**Step 2: Create Dockerfile**

`Dockerfile`:
```dockerfile
# Base image - use official Node.js runtime
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm install --production

# Copy application code
COPY . .

# Expose port the app runs on
EXPOSE 3000

# Define environment variable
ENV NODE_ENV=production

# Command to run the application
CMD ["npm", "start"]
```

**Step 3: Create .dockerignore**

`.dockerignore`:
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
```

**Step 4: Build the Docker Image**

```bash
# Build image with tag
docker build -t my-node-app:1.0 .

# Build with custom Dockerfile name
docker build -f Dockerfile.prod -t my-node-app:1.0 .

# Build without cache (fresh build)
docker build --no-cache -t my-node-app:1.0 .

# Verify the image was created
docker images | grep my-node-app
```

**Step 5: Run the Container**

```bash
# Run container with port mapping
docker run -d -p 3000:3000 --name node-app my-node-app:1.0

# Run with environment variables
docker run -d -p 3000:3000 \
  -e NODE_ENV=production \
  -e API_KEY=secret123 \
  --name node-app \
  my-node-app:1.0

# Run with volume mount (for development)
docker run -d -p 3000:3000 \
  -v $(pwd):/app \
  --name node-app \
  my-node-app:1.0

# Test the application
curl http://localhost:3000
curl http://localhost:3000/health
```

#### Example 2: Python Flask Application

**Step 1: Create Application**

`app.py`:
```python
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello from Flask Container!'

@app.route('/health')
def health():
    return jsonify({
        'status': 'healthy',
        'version': os.getenv('APP_VERSION', '1.0')
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

`requirements.txt`:
```
Flask==2.3.0
gunicorn==21.2.0
```

**Step 2: Create Dockerfile**

`Dockerfile`:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

EXPOSE 5000

ENV APP_VERSION=1.0

# Use gunicorn for production
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**Step 3: Build and Run**

```bash
# Build
docker build -t flask-app:1.0 .

# Run
docker run -d -p 5000:5000 --name flask-app flask-app:1.0

# Test
curl http://localhost:5000
curl http://localhost:5000/health
```

### Multi-Stage Builds (Optimization)

Multi-stage builds create smaller, more secure images by separating build and runtime environments.

**Example: Go Application**

`Dockerfile`:
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Stage 2: Runtime
FROM alpine:latest

WORKDIR /root/

# Copy only the binary from builder
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

**Benefits:**
- Final image only contains runtime dependencies
- Build tools and source code excluded from final image
- Smaller image size (MBs vs GBs)
- More secure (fewer attack vectors)

```bash
# Build multi-stage image
docker build -t go-app:1.0 .

# Compare image sizes
docker images | grep go-app
```

### Best Practices for Containerizing Services

#### 1. **Use Official Base Images**
```dockerfile
# Good
FROM node:18-alpine
FROM python:3.11-slim
FROM nginx:alpine

# Avoid
FROM ubuntu
RUN apt-get install node
```

#### 2. **Minimize Layers**
```dockerfile
# Bad (multiple layers)
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good (single layer)
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*
```

#### 3. **Order Commands for Better Caching**
```dockerfile
# Dependencies change less frequently - copy first
COPY package*.json ./
RUN npm install

# Source code changes frequently - copy last
COPY . .
```

#### 4. **Use .dockerignore**
```
node_modules
.git
*.log
.env
test/
docs/
```

#### 5. **Don't Run as Root**
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 appuser && \
    adduser -D -u 1001 -G appuser appuser

WORKDIR /app
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

CMD ["node", "app.js"]
```

#### 6. **Use Health Checks**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

#### 7. **Set Explicit Versions**
```dockerfile
# Good
FROM node:18.16.0-alpine
FROM python:3.11.4-slim

# Avoid (may break with updates)
FROM node:latest
FROM python:3
```

### Practical Example: GoCD Container

**Running GoCD as Container (Answering Q.5)**

```bash
# Step 1: Pull the GoCD server image
docker pull gocd/gocd-server:v23.5.0

# Step 2: Run GoCD container
docker run -d \
  --name gocd-server \
  -p 8153:8153 \
  -p 8154:8154 \
  gocd/gocd-server:v23.5.0

# Step 3: Verify the running container
# Check if container is running
docker ps | grep gocd-server

# Check container status
docker ps -a --filter "name=gocd-server"

# Detailed container information
docker inspect gocd-server

# Check container logs
docker logs gocd-server

# Check last 50 lines of logs
docker logs --tail 50 gocd-server

# Follow logs in real-time
docker logs -f gocd-server

# Check container resource usage
docker stats gocd-server

# Step 4: Access GoCD
# Open browser: http://localhost:8153

# Step 5: Execute commands inside container
docker exec -it gocd-server bash
docker exec gocd-server ps aux

# Step 6: Stop the container
docker stop gocd-server

# Verify container is stopped
docker ps -a | grep gocd-server

# Step 7: Start the stopped container
docker start gocd-server

# Step 8: Remove the container
# Stop first if running
docker stop gocd-server

# Remove container
docker rm gocd-server

# Force remove (without stopping)
docker rm -f gocd-server

# Verify removal
docker ps -a | grep gocd-server
```

**Complete Script:**
```bash
#!/bin/bash

# GoCD Container Management Script

echo "=== Pulling GoCD Image ==="
docker pull gocd/gocd-server:v23.5.0

echo "=== Running GoCD Container ==="
docker run -d \
  --name gocd-server \
  -p 8153:8153 \
  -p 8154:8154 \
  -v gocd-data:/godata \
  gocd/gocd-server:v23.5.0

echo "=== Verifying Container ==="
echo "Running containers:"
docker ps | grep gocd-server

echo -e "\n=== Container Details ==="
docker inspect gocd-server | jq '.[0] | {Name, State, Ports}'

echo -e "\n=== Checking Logs ==="
docker logs --tail 20 gocd-server

echo -e "\n=== Resource Usage ==="
docker stats gocd-server --no-stream

echo -e "\nGoCD is running at: http://localhost:8153"
echo -e "\nTo stop: docker stop gocd-server"
echo -e "To remove: docker rm -f gocd-server"
```

### Container Networking

```bash
# Create custom network
docker network create my-network

# Run container on custom network
docker run -d --name app1 --network my-network nginx

# Connect existing container to network
docker network connect my-network app1

# Inspect network
docker network inspect my-network

# List all networks
docker network ls

# Remove network
docker network rm my-network
```

---

## 11.3. Deploying to a Cluster

### Introduction to Container Orchestration

**Why Orchestration?**
- Manual container management becomes complex at scale
- Need for automatic deployment, scaling, and management
- High availability and fault tolerance requirements
- Load balancing across multiple containers
- Service discovery and networking

**Popular Orchestration Tools:**
- **Kubernetes**: Industry standard, most features
- **Docker Swarm**: Simpler, native to Docker
- **Apache Mesos**: Data center scale
- **Nomad**: HashiCorp's orchestrator

### Docker Swarm Basics

#### Initialize Swarm

```bash
# Initialize Swarm on manager node
docker swarm init

# Initialize with specific IP (if multiple interfaces)
docker swarm init --advertise-addr 192.168.1.100

# Get join token for workers
docker swarm join-token worker

# Get join token for managers
docker swarm join-token manager
```

#### Join Worker Nodes

```bash
# On worker node, use token from manager
docker swarm join --token <worker-token> <manager-ip>:2377
```

#### Deploy Service to Swarm

```bash
# Create a service
docker service create \
  --name web \
  --replicas 3 \
  --publish 8080:80 \
  nginx:alpine

# List services
docker service ls

# Inspect service
docker service inspect web

# View service logs
docker service logs web

# List tasks (containers) of service
docker service ps web

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:latest web

# Remove service
docker service rm web
```

#### Deploy Stack (Multi-Service)

`docker-compose.yml`:
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    networks:
      - webnet

  redis:
    image: redis:alpine
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
    networks:
      - webnet

networks:
  webnet:
    driver: overlay
```

```bash
# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# List stacks
docker stack ls

# List services in stack
docker stack services myapp

# List tasks in stack
docker stack ps myapp

# Remove stack
docker stack rm myapp
```

### Container Registry Operations

#### Docker Hub

```bash
# Login to Docker Hub
docker login

# Tag image for Docker Hub
docker tag my-app:1.0 username/my-app:1.0

# Push to Docker Hub
docker push username/my-app:1.0

# Pull from Docker Hub
docker pull username/my-app:1.0

# Logout
docker logout
```

#### Private Registry

```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for private registry
docker tag my-app:1.0 localhost:5000/my-app:1.0

# Push to private registry
docker push localhost:5000/my-app:1.0

# Pull from private registry
docker pull localhost:5000/my-app:1.0
```

### Deployment Strategies

#### Blue-Green Deployment

```bash
# Deploy blue version
docker service create --name app-blue nginx:1.20

# Test blue version
# If successful, deploy green version
docker service create --name app-green nginx:1.21

# Switch traffic to green
docker service update --replicas 0 app-blue
docker service update --replicas 3 app-green
```

#### Rolling Updates

```bash
# Create service
docker service create --name app --replicas 5 nginx:1.20

# Update with rolling strategy
docker service update \
  --image nginx:1.21 \
  --update-parallelism 2 \
  --update-delay 10s \
  app
```

### Summary Checklist

- [ ] Understand container architecture
- [ ] Master basic Docker commands
- [ ] Create Dockerfiles for different languages
- [ ] Implement multi-stage builds
- [ ] Use Docker networking
- [ ] Deploy to Docker Swarm
- [ ] Manage container registries
- [ ] Apply deployment strategies

### Practice Exercise

**Deploy a full-stack application:**

1. Containerize frontend (React/Angular)
2. Containerize backend (Node.js/Python)
3. Set up database container (PostgreSQL/MongoDB)
4. Create docker-compose.yml
5. Deploy to local cluster
6. Test inter-container communication
7. Implement health checks
8. Scale services

---

## Next Steps

- Study Chapter 12: Monitoring
- Learn Kubernetes (Chapter 13)
- Practice with real projects
- Explore CI/CD integration
