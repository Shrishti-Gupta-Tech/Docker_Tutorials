# 🐳 Docker Comprehensive Study Notes
### From Beginner to Advanced

---

## Table of Contents
1. [Beginner: Core Concepts](#beginner-core-concepts)
2. [Intermediate: Image & Container Management](#intermediate-image--container-management)
3. [Advanced: Orchestration & Optimization](#advanced-orchestration--optimization)

---

## Beginner: Core Concepts

### What is Docker?

**Definition:**
Docker is an open-source platform that automates the deployment, scaling, and management of applications using containerization technology. It packages applications and their dependencies into standardized units called containers.

**Shipping Container Analogy:**
Think of Docker like shipping containers in the logistics industry:
- **Traditional Shipping (Before Containers):** Different goods required different loading methods, vehicles, and handling. This was inefficient and error-prone.
- **Containerized Shipping:** Standard containers can hold any cargo, fit on any ship, truck, or train, and are handled by standard equipment worldwide.

Similarly:
- **Traditional Software Deployment:** Applications had different requirements, conflicted with other software, and behaved differently across environments ("It works on my machine!").
- **Docker Containers:** Applications run consistently across any environment (development, testing, production) because they include everything needed to run.

**Key Benefits:**
- ✅ Consistency across environments
- ✅ Isolation from other applications
- ✅ Portability across different systems
- ✅ Efficient resource utilization
- ✅ Fast startup times

---

### Key Components

#### 1. **Docker Image**
**Definition:** A read-only template containing the application code, runtime, libraries, and dependencies needed to run an application.

**Analogy:** Like a blueprint or recipe for creating containers.

**Key Points:**
- Images are built from a Dockerfile
- Images are composed of layers (each instruction in Dockerfile creates a layer)
- Images are immutable (cannot be changed once created)
- Images can be versioned using tags (e.g., `nginx:1.21`, `node:18-alpine`)

**Example:**
```bash
# List all images on your system
docker images

# Output example:
# REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
# node          18        a1b2c3d4e5f6   2 days ago     900MB
# nginx         alpine    b2c3d4e5f6g7   1 week ago     23MB
```

---

#### 2. **Docker Container**
**Definition:** A runnable instance of a Docker image. It's a lightweight, isolated environment where your application runs.

**Analogy:** If an image is a blueprint, a container is the actual built house based on that blueprint.

**Key Points:**
- Containers are created from images
- Multiple containers can be created from the same image
- Containers are isolated from each other and the host system
- Containers are ephemeral (data is lost when container is removed unless using volumes)
- Containers share the host OS kernel (unlike VMs)

**Example:**
```bash
# Create and run a container from nginx image
docker run -d --name my-nginx nginx

# List running containers
docker ps

# Output example:
# CONTAINER ID   IMAGE     COMMAND                  STATUS         PORTS     NAMES
# 1a2b3c4d5e6f   nginx     "/docker-entrypoint.…"   Up 2 minutes   80/tcp    my-nginx
```

---

#### 3. **Docker Engine**
**Definition:** The core software that runs and manages Docker containers. It consists of three main components:

**Components:**
1. **Docker Daemon (dockerd):**
   - Background service that manages Docker objects (images, containers, networks, volumes)
   - Listens for Docker API requests
   - Manages container lifecycle

2. **Docker Client (docker):**
   - Command-line interface (CLI) that users interact with
   - Sends commands to Docker Daemon via REST API
   - Can communicate with multiple daemons

3. **REST API:**
   - Interface between Docker Client and Docker Daemon
   - Allows programmatic control of Docker

**Architecture Diagram:**
```
┌─────────────────┐
│  Docker Client  │ (CLI: docker run, docker build, etc.)
└────────┬────────┘
         │ REST API
         ▼
┌─────────────────┐
│  Docker Daemon  │ (dockerd)
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Docker Objects                     │
│  • Containers                       │
│  • Images                          │
│  • Networks                        │
│  • Volumes                         │
└─────────────────────────────────────┘
```

---

#### 4. **Docker Hub**
**Definition:** A cloud-based registry service where Docker users can store, share, and find container images.

**Analogy:** Like GitHub for Docker images or an "App Store" for containers.

**Key Features:**
- **Public Repositories:** Free hosting for public images
- **Private Repositories:** Store proprietary images securely
- **Official Images:** Verified images from software vendors (e.g., `nginx`, `redis`, `postgres`)
- **Community Images:** Images created and shared by the community

**Example URLs:**
- Docker Hub: https://hub.docker.com
- Example official image: https://hub.docker.com/_/nginx

**Usage Example:**
```bash
# Search for images on Docker Hub
docker search python

# Pull an image from Docker Hub
docker pull python:3.11-slim

# Push an image to Docker Hub (requires login)
docker login
docker tag my-app:latest myusername/my-app:latest
docker push myusername/my-app:latest
```

---

#### 5. **Dockerfile**
**Definition:** A text file containing a series of instructions to build a Docker image automatically.

**Analogy:** Like a recipe that tells Docker how to create your image step by step.

**Key Points:**
- Instructions are executed in order from top to bottom
- Each instruction creates a new layer in the image
- Instructions are cached for faster rebuilds
- Dockerfile must be named exactly `Dockerfile` (capital D)

**Simple Example:**
```dockerfile
# Start from a base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Define startup command
CMD ["node", "index.js"]
```

**Building an image:**
```bash
# Build image from Dockerfile in current directory
docker build -t my-node-app:1.0 .

# -t : tag (name) the image
# .  : build context (current directory)
```

---

### Basic Commands

#### 1. **docker run** - Create and start a container

**Syntax:**
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Common Options:**
- `-d` : Run container in detached mode (background)
- `-p` : Publish container port to host (port mapping)
- `--name` : Assign a name to the container
- `-e` : Set environment variables
- `-v` : Mount a volume
- `--rm` : Automatically remove container when it exits
- `-it` : Interactive terminal mode

**Examples:**

**Basic Usage:**
```bash
# Run nginx in foreground (blocks terminal)
docker run nginx

# Run nginx in background
docker run -d nginx

# Run nginx with custom name
docker run -d --name web-server nginx

# Run and automatically remove after exit
docker run --rm nginx
```

**Port Mapping:**
```bash
# Map container port 80 to host port 8080
docker run -d -p 8080:80 nginx

# Access via: http://localhost:8080

# Map to random available port
docker run -d -P nginx

# Check which port was assigned
docker ps
```

**Environment Variables:**
```bash
# Set single environment variable
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Set multiple environment variables
docker run -d \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres

# Load environment variables from file
docker run -d --env-file .env my-app
```

**Interactive Mode:**
```bash
# Run Ubuntu container with interactive bash shell
docker run -it ubuntu bash

# Inside container, you can run commands:
# root@container-id:/# ls
# root@container-id:/# pwd
# root@container-id:/# exit

# Run Python interpreter interactively
docker run -it python:3.11 python
```

**Complete Real-World Example:**
```bash
# Run a Node.js application with all options
docker run -d \
  --name my-api \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgres://db:5432/mydb \
  -v $(pwd)/logs:/app/logs \
  --restart unless-stopped \
  my-node-app:latest
```

---

#### 2. **docker pull** - Download an image from a registry

**Syntax:**
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

**Examples:**

**Pull Latest Version:**
```bash
# Pull latest version of nginx
docker pull nginx

# Equivalent to:
docker pull nginx:latest
```

**Pull Specific Version:**
```bash
# Pull specific version by tag
docker pull node:18-alpine
docker pull postgres:15.2
docker pull python:3.11-slim

# Pull by digest (exact image)
docker pull nginx@sha256:abc123def456...
```

**Pull from Different Registry:**
```bash
# Pull from Docker Hub (default)
docker pull nginx

# Pull from private registry
docker pull myregistry.com:5000/my-app:latest

# Pull from GitHub Container Registry
docker pull ghcr.io/owner/image:tag
```

**Real-World Workflow:**
```bash
# 1. Search for image
docker search redis

# 2. Pull specific version
docker pull redis:7-alpine

# 3. Verify image was pulled
docker images | grep redis

# 4. Run container from pulled image
docker run -d --name my-redis redis:7-alpine
```

---

#### 3. **docker ps** - List containers

**Syntax:**
```bash
docker ps [OPTIONS]
```

**Common Options:**
- `-a` : Show all containers (including stopped)
- `-q` : Only display container IDs
- `-l` : Show latest created container
- `--filter` : Filter output based on conditions
- `--format` : Pretty-print using Go template

**Examples:**

**Basic Usage:**
```bash
# List running containers
docker ps

# Output:
# CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
# a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   web-server
```

**Show All Containers:**
```bash
# List all containers (running + stopped)
docker ps -a

# Output includes stopped containers:
# CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS                      PORTS     NAMES
# a1b2c3d4e5f6   nginx     "nginx"     2 minutes ago    Up 2 minutes               80/tcp    web-server
# b2c3d4e5f6g7   redis     "redis"     10 minutes ago   Exited (0) 5 minutes ago             my-redis
```

**Show Only Container IDs:**
```bash
# Get only container IDs (useful for scripting)
docker ps -q

# Output:
# a1b2c3d4e5f6
# b2c3d4e5f6g7

# Use in scripts to stop all containers
docker stop $(docker ps -q)
```

**Filter Containers:**
```bash
# Show containers with specific name
docker ps --filter "name=web"

# Show containers from specific image
docker ps --filter "ancestor=nginx"

# Show containers with specific status
docker ps -a --filter "status=exited"

# Show containers with specific label
docker ps --filter "label=environment=production"
```

**Custom Format:**
```bash
# Custom output format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Output:
# CONTAINER ID   NAMES         STATUS
# a1b2c3d4e5f6   web-server    Up 2 minutes
```

---

#### 4. **docker stop** - Stop running container(s)

**Syntax:**
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

**How it Works:**
1. Sends SIGTERM signal to the container's main process
2. Waits for graceful shutdown (default: 10 seconds)
3. If process doesn't stop, sends SIGKILL to force termination

**Examples:**

**Stop Single Container:**
```bash
# Stop by container name
docker stop web-server

# Stop by container ID
docker stop a1b2c3d4e5f6

# Stop by short ID (first few characters)
docker stop a1b
```

**Stop Multiple Containers:**
```bash
# Stop multiple containers by name
docker stop web-server api-server database

# Stop all running containers
docker stop $(docker ps -q)
```

**Custom Timeout:**
```bash
# Wait 30 seconds before force killing
docker stop -t 30 web-server

# Force immediate kill (0 seconds timeout)
docker stop -t 0 web-server
```

**Real-World Example:**
```bash
# Gracefully stop application with 60-second timeout
# (Useful for apps that need time to finish processing)
docker stop -t 60 my-api

# Verify container stopped
docker ps -a | grep my-api

# Output shows: Exited (0) 1 second ago
```

---

#### 5. **docker rm** - Remove container(s)

**Syntax:**
```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Important:** Container must be stopped before removal (unless using `-f` flag).

**Examples:**

**Remove Single Container:**
```bash
# Remove stopped container by name
docker rm web-server

# Remove by container ID
docker rm a1b2c3d4e5f6
```

**Remove Multiple Containers:**
```bash
# Remove multiple containers
docker rm web-server api-server database

# Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)

# Better: Use prune command
docker container prune
```

**Force Remove Running Container:**
```bash
# Force remove running container (stops then removes)
docker rm -f web-server

# Warning: This is abrupt, prefer to stop first
docker stop web-server && docker rm web-server
```

**Remove Container and Volumes:**
```bash
# Remove container and its anonymous volumes
docker rm -v web-server

# Without -v, anonymous volumes become orphaned
```

**Real-World Workflow:**
```bash
# Complete cleanup workflow

# 1. Stop the container
docker stop my-app

# 2. Remove the container
docker rm my-app

# 3. Or combine both steps
docker rm -f my-app

# 4. Clean up all stopped containers
docker container prune

# 5. Confirm with 'y' when prompted
# WARNING! This will remove all stopped containers.
# Are you sure you want to continue? [y/N] y
```

---

## Intermediate: Image & Container Management

### The Dockerfile

**Definition:** A Dockerfile is a text document that contains commands to assemble a Docker image. Each instruction creates a layer in the final image.

#### Structure and Key Instructions

**Basic Dockerfile Structure:**
```dockerfile
# 1. Base Image
FROM <image>:<tag>

# 2. Metadata
LABEL maintainer="your-email@example.com"

# 3. Environment Setup
ENV KEY=value
ARG BUILD_ARG=default

# 4. Working Directory
WORKDIR /path/to/directory

# 5. Copy Files
COPY source destination
ADD source destination

# 6. Execute Commands
RUN command

# 7. Expose Ports
EXPOSE port

# 8. Define Volumes
VOLUME ["/data"]

# 9. Set User
USER username

# 10. Startup Command
CMD ["executable", "param1", "param2"]
ENTRYPOINT ["executable"]
```

---

#### Key Dockerfile Instructions

#### **FROM** - Set Base Image

**Purpose:** Specifies the base image to build upon. Must be the first instruction (except for ARG).

**Syntax:**
```dockerfile
FROM <image>[:<tag>] [AS <name>]
```

**Examples:**
```dockerfile
# Use official Node.js image
FROM node:18

# Use Alpine Linux for smaller size
FROM node:18-alpine

# Use specific version
FROM python:3.11.5-slim

# Multi-stage build with named stage
FROM node:18 AS builder

# Use scratch for minimal image
FROM scratch
```

**Real-World Example:**
```dockerfile
# Development stage
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=development /app/dist ./dist
CMD ["node", "dist/server.js"]
```

---

#### **RUN** - Execute Commands During Build

**Purpose:** Executes commands in a new layer and commits the results. Used for installing packages, building applications, etc.

**Syntax:**
```dockerfile
# Shell form (runs in shell: /bin/sh -c)
RUN <command>

# Exec form (doesn't invoke shell)
RUN ["executable", "param1", "param2"]
```

**Examples:**

**Single Command:**
```dockerfile
# Install package
RUN apt-get update

# Create directory
RUN mkdir -p /app/data
```

**Multiple Commands (Less Efficient):**
```dockerfile
# Creates 3 separate layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
```

**Multiple Commands (Optimized):**
```dockerfile
# Creates 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Real-World Examples:**

**Python Application:**
```dockerfile
FROM python:3.11-slim

# Install system dependencies and Python packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        postgresql-client \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir --upgrade pip
```

**Node.js Application:**
```dockerfile
FROM node:18-alpine

# Install build dependencies
RUN apk add --no-cache \
        python3 \
        make \
        g++ \
    && npm install -g npm@latest
```

---

#### **CMD** - Default Command to Execute

**Purpose:** Provides defaults for executing a container. Only the last CMD in Dockerfile takes effect.

**Syntax:**
```dockerfile
# Exec form (preferred)
CMD ["executable", "param1", "param2"]

# Shell form
CMD command param1 param2

# As parameters to ENTRYPOINT
CMD ["param1", "param2"]
```

**Examples:**

**Basic Usage:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

# Run Node.js application
CMD ["node", "server.js"]
```

**With Arguments:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .

# Run Python script with arguments
CMD ["python", "app.py", "--host", "0.0.0.0"]
```

**Real-World Example:**
```dockerfile
FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy static files
COPY ./build /usr/share/nginx/html

# Default command (can be overridden)
CMD ["nginx", "-g", "daemon off;"]
```

**Can Be Overridden:**
```bash
# Use default CMD
docker run my-nginx

# Override CMD
docker run my-nginx nginx -t  # Test nginx config
```

---

#### **ENTRYPOINT** - Configure Container as Executable

**Purpose:** Configures a container to run as an executable. Unlike CMD, ENTRYPOINT is not easily overridden.

**Syntax:**
```dockerfile
# Exec form (preferred)
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2
```

**Difference Between CMD and ENTRYPOINT:**

**CMD Example:**
```dockerfile
FROM alpine
CMD ["echo", "Hello World"]
```
```bash
docker run my-image              # Output: Hello World
docker run my-image echo Goodbye # Output: Goodbye (CMD overridden)
```

**ENTRYPOINT Example:**
```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```
```bash
docker run my-image              # Output: Hello World
docker run my-image Goodbye      # Output: Goodbye (CMD replaced, ENTRYPOINT stays)
```

**Real-World Examples:**

**Web Server:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

ENTRYPOINT ["node"]
CMD ["server.js"]
```
```bash
docker run my-app              # Runs: node server.js
docker run my-app worker.js    # Runs: node worker.js
```

**CLI Tool:**
```dockerfile
FROM alpine
RUN apk add --no-cache curl

ENTRYPOINT ["curl"]
CMD ["--help"]
```
```bash
docker run my-curl                    # Shows curl help
docker run my-curl https://google.com # Fetches Google
```

**Script Wrapper:**
```dockerfile
FROM postgres:15
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

---

#### **EXPOSE** - Document Port Usage

**Purpose:** Informs Docker that the container listens on specified network ports at runtime. This is documentation only; it doesn't actually publish ports.

**Syntax:**
```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

**Examples:**

**Single Port:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

# Document that app uses port 3000
EXPOSE 3000

CMD ["node", "server.js"]
```

**Multiple Ports:**
```dockerfile
FROM nginx:alpine

# HTTP and HTTPS
EXPOSE 80 443
```

**With Protocol:**
```dockerfile
FROM custom-app

# TCP port (default)
EXPOSE 8080/tcp

# UDP port
EXPOSE 5353/udp
```

**Real-World Example:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# App HTTP port
EXPOSE 3000

# Metrics endpoint
EXPOSE 9090

# Health check endpoint (same as app)
HEALTHCHECK --interval=30s CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

**Usage:**
```bash
# EXPOSE doesn't publish ports automatically
docker run my-app  # Port 3000 not accessible from host

# Must explicitly publish with -p
docker run -p 8080:3000 my-app  # Now accessible on host:8080

# Or publish all EXPOSE'd ports to random ports
docker run -P my-app
docker ps  # Shows which host ports were assigned
```

---

#### **WORKDIR** - Set Working Directory

**Purpose:** Sets the working directory for subsequent RUN, CMD, ENTRYPOINT, COPY, and ADD instructions.

**Syntax:**
```dockerfile
WORKDIR /path/to/directory
```

**Key Points:**
- Creates directory if it doesn't exist
- Can be used multiple times
- Relative paths are relative to previous WORKDIR

**Examples:**

**Basic Usage:**
```dockerfile
FROM node:18-alpine

# Set working directory (creates if doesn't exist)
WORKDIR /app

# Now all commands run in /app
COPY package*.json ./  # Copies to /app/
RUN npm install        # Runs in /app/
COPY . .               # Copies to /app/

CMD ["node", "server.js"]  # Runs /app/server.js
```

**Multiple WORKDIR:**
```dockerfile
FROM alpine

WORKDIR /a
WORKDIR b   # Now in /a/b
WORKDIR c   # Now in /a/b/c
RUN pwd     # Outputs: /a/b/c
```

**With Environment Variables:**
```dockerfile
FROM node:18-alpine
ENV APP_HOME=/usr/src/app

WORKDIR $APP_HOME
# Now in /usr/src/app
```

**Real-World Example:**
```dockerfile
FROM python:3.11-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

# Set working directory
WORKDIR /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

# Commands now run in /app as appuser
CMD ["python", "app.py"]
```

---

### Best Practices: Layering, Caching, and Reducing Image Size

#### 1. **Layer Optimization**

**Concept:** Each Dockerfile instruction creates a new layer. Fewer layers = smaller, faster images.

**❌ Bad Practice:**
```dockerfile
FROM node:18-alpine
RUN npm install -g npm
RUN npm install -g yarn
RUN npm install -g pm2
COPY package.json .
RUN npm install
COPY . .
```

**✅ Good Practice:**
```dockerfile
FROM node:18-alpine

# Combine RUN commands
RUN npm install -g npm yarn pm2

# Copy and install dependencies first (cache layer)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code last (changes frequently)
COPY . .
```

---

#### 2. **Build Cache Optimization**

**Concept:** Docker caches each layer. Order instructions from least to most frequently changed.

**Cache Invalidation Example:**
```dockerfile
# ❌ Bad: Copying all files first invalidates cache on any change
FROM node:18-alpine
WORKDIR /app
COPY . .                    # Cache breaks on ANY file change
RUN npm install             # Reinstalls even if package.json unchanged
CMD ["node", "server.js"]
```

**✅ Optimized:**
```dockerfile
FROM node:18-alpine
WORKDIR /app

# 1. Copy dependency files first (rarely change)
COPY package*.json ./

# 2. Install dependencies (cached unless package.json changes)
RUN npm ci --only=production

# 3. Copy source code last (changes frequently)
COPY . .

CMD ["node", "server.js"]
```

**Real-World Python Example:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Layer 1: System dependencies (rarely change)
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Layer 2: Python dependencies (change occasionally)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Layer 3: Application code (changes frequently)
COPY . .

CMD ["python", "app.py"]
```

---

#### 3. **Reducing Image Size**

**Technique 1: Use Alpine Base Images**
```dockerfile
# ❌ Large: 900MB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# ✅ Small: 170MB
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
```

**Technique 2: Multi-Stage Builds**
```dockerfile
# Stage 1: Build (large)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production (small)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```

**Technique 3: Clean Up in Same Layer**
```dockerfile
# ❌ Bad: Creates unnecessary layers
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# ✅ Good: Cleanup in same layer
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Technique 4: Use .dockerignore**
```
# .dockerignore file
node_modules/
npm-debug.log
.git/
.gitignore
README.md
.env
.DS_Store
*.test.js
coverage/
.vscode/
```

**Technique 5: Remove Dev Dependencies**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./

# Use npm ci instead of npm install
# Use --only=production flag
RUN npm ci --only=production

COPY . .
CMD ["node", "server.js"]
```

**Complete Optimized Example:**
```dockerfile
# Multi-stage build for minimal size
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && \
    npm prune --production

FROM node:18-alpine
WORKDIR /app

# Copy only production dependencies
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

# Run as non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Size Comparison:**
```
Before optimization:  1.2GB
After optimization:   85MB
Reduction:           93%
```

---

### Data Persistence

Containers are ephemeral by design—data is lost when containers are removed. Docker provides two main mechanisms for data persistence:

#### 1. **Volumes** (Recommended)

**Definition:** Volumes are stored in a part of the host filesystem managed by Docker (`/var/lib/docker/volumes/`). They're completely managed by Docker.

**Advantages:**
- ✅ Managed by Docker (automatic backups, easier migrations)
- ✅ Work on all platforms (Linux, Mac, Windows)
- ✅ Better performance in Docker Desktop
- ✅ Can be safely shared among containers
- ✅ Can be backed up or migrated easily

**Creating and Using Volumes:**

**Create Named Volume:**
```bash
# Create volume
docker volume create my-data

# Inspect volume
docker volume inspect my-data

# Output shows:
# "Mountpoint": "/var/lib/docker/volumes/my-data/_data"
```

**Using Volumes in Containers:**

**Example 1: PostgreSQL Database**
```bash
# Run PostgreSQL with named volume
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data persists even if container is removed
docker rm -f postgres-db

# Start new container with same volume
docker run -d \
  --name postgres-db-new \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data from previous container still exists!
```

**Example 2: MongoDB**
```bash
# Create volume
docker volume create mongodb-data

# Run MongoDB with volume
docker run -d \
  --name mongodb \
  -v mongodb-data:/data/db \
  -p 27017:27017 \
  mongo:7

# Verify data persistence
docker exec mongodb mongosh --eval "db.users.insertOne({name: 'Alice'})"

# Remove container
docker rm -f mongodb

# Start new container
docker run -d \
  --name mongodb-new \
  -v mongodb-data:/data/db \
  -p 27017:27017 \
  mongo:7

# Check data still exists
docker exec mongodb-new mongosh --eval "db.users.find()"
```

**Example 3: Application Logs**
```bash
# Run application with log volume
docker run -d \
  --name my-app \
  -v app-logs:/var/log/app \
  my-application:latest

# View logs from volume
docker run --rm \
  -v app-logs:/logs \
  alpine cat /logs/application.log
```

**Volume Management Commands:**
```bash
# List all volumes
docker volume ls

# Inspect volume details
docker volume inspect my-data

# Remove volume
docker volume rm my-data

# Remove all unused volumes
docker volume prune

# Remove volume with container
docker rm -v container-name
```

**Dockerfile with Volume:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

# Declare volume mount point
VOLUME ["/app/data"]

CMD ["node", "server.js"]
```

---

#### 2. **Bind Mounts**

**Definition:** Bind mounts directly link a host filesystem path to a container path. You have full control over the exact location on the host.

**Advantages:**
- ✅ Direct access to host files
- ✅ Useful for development (hot-reload)
- ✅ Easy to edit files with host tools
- ✅ Good for sharing config files

**Disadvantages:**
- ❌ Dependent on host file structure
- ❌ Less portable across different hosts
- ❌ Requires absolute paths (or $(pwd))
- ❌ Security concerns (container can modify host files)

**Using Bind Mounts:**

**Syntax:**
```bash
docker run -v /host/path:/container/path image
# OR
docker run --mount type=bind,source=/host/path,target=/container/path image
```

**Example 1: Development with Hot-Reload**
```bash
# Mount current directory to /app
docker run -d \
  --name dev-server \
  -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  node:18-alpine \
  sh -c "cd /app && npm run dev"

# Changes in current directory reflect immediately in container
```

**Real-World Development Setup:**
```bash
# Directory structure:
# my-project/
# ├── src/
# ├── public/
# ├── package.json
# └── Dockerfile

# Run with bind mount for development
docker run -d \
  --name react-dev \
  -p 3000:3000 \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/public:/app/public \
  -e CHOKIDAR_USEPOLLING=true \
  my-react-app:dev

# Edit files on host, see changes in container instantly
```

**Example 2: Configuration Files**
```bash
# Mount nginx config from host
docker run -d \
  --name nginx-server \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  nginx:alpine

# :ro = read-only mount
```

**Example 3: Shared Data Directory**
```bash
# Mount shared data folder
docker run -d \
  --name data-processor \
  -v /data/input:/app/input:ro \
  -v /data/output:/app/output \
  my-processor:latest

# Input is read-only, output is writable
```

**Example 4: Database Configuration**
```bash
# MySQL with custom config
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v $(pwd)/mysql-config:/etc/mysql/conf.d \
  -v mysql-data:/var/lib/mysql \
  mysql:8

# Mix of bind mount (config) and volume (data)
```

**Docker Compose with Bind Mounts:**
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      # Bind mount for config
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      # Bind mount for static files
      - ./html:/usr/share/nginx/html:ro
      
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      # Bind mount for hot-reload development
      - ./src:/app/src
      - ./public:/app/public
      # Named volume for node_modules
      - node_modules:/app/node_modules
    environment:
      - NODE_ENV=development

volumes:
  node_modules:
```

---

### Networking

#### Basics: Port Mapping (-p)

**Purpose:** Expose container ports to the host machine so they can be accessed from outside.

**Syntax:**
```bash
docker run -p [host_ip:]host_port:container_port[/protocol] image
```

**Examples:**

**Basic Port Mapping:**
```bash
# Map container port 80 to host port 8080
docker run -d -p 8080:80 nginx

# Access via: http://localhost:8080
```

**Multiple Ports:**
```bash
# Map multiple ports
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  nginx

# Access HTTP: http://localhost:8080
# Access HTTPS: https://localhost:8443
```

**Specific Host IP:**
```bash
# Bind to specific network interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Only accessible from localhost, not external IPs
```

**Random Host Port:**
```bash
# Let Docker choose random available port
docker run -d -p 80 nginx

# Or use -P to publish all EXPOSE'd ports
docker run -d -P nginx

# Check assigned port
docker ps
# OR
docker port container-name
```

**UDP Ports:**
```bash
# Map UDP port (default is TCP)
docker run -d -p 5353:53/udp dns-server
```

**Real-World Example:**
```bash
# Full-stack application
docker run -d \
  --name frontend \
  -p 80:80 \
  -p 443:443 \
  my-frontend:latest

docker run -d \
  --name backend \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://db:5432/mydb \
  my-backend:latest

docker run -d \
  --name database \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
```

---

#### Network Types

Docker provides several network drivers for different use cases:

#### 1. **Bridge Network** (Default)

**Description:** Default network type. Containers on the same bridge network can communicate with each other. Provides isolation from other bridge networks.

**Use Cases:**
- Development environments
- Standalone containers on single host
- When containers need to talk to each other

**How it Works:**
- Docker creates a virtual bridge (`docker0` on Linux)
- Each container gets a virtual ethernet interface
- Containers communicate via container names (with user-defined bridges)

**Default Bridge Example:**
```bash
# Run containers on default bridge
docker run -d --name web1 nginx
docker run -d --name web2 nginx

# They're on the same network but can't ping by name
docker exec web1 ping web2  # Fails!

# Must use IP addresses
docker inspect web1 | grep IPAddress
docker exec web1 ping <web2-ip>  # Works
```

**User-Defined Bridge (Recommended):**
```bash
# Create custom bridge network
docker network create my-network

# Run containers on custom bridge
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx

# Automatic DNS resolution by container name
docker exec web1 ping web2  # Works!
docker exec web2 curl http://web1  # Works!
```

**Complete Example:**
```bash
# Create network
docker network create app-network

# Run database
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Run backend (can access database by name)
docker run -d \
  --name backend \
  --network app-network \
  -e DATABASE_URL=postgres://postgres-db:5432/mydb \
  my-backend:latest

# Run frontend (can access backend by name)
docker run -d \
  --name frontend \
  --network app-network \
  -p 80:80 \
  -e API_URL=http://backend:3000 \
  my-frontend:latest
```

**Network Commands:**
```bash
# List networks
docker network ls

# Inspect network
docker network inspect my-network

# Connect running container to network
docker network connect my-network container-name

# Disconnect from network
docker network disconnect my-network container-name

# Remove network
docker network rm my-network
```

---

#### 2. **Host Network**

**Description:** Container uses the host's network directly. No network isolation between host and container.

**Use Cases:**
- Maximum network performance needed
- Container needs to handle lots of ports
- Testing/debugging network issues

**Key Points:**
- ⚠️ Container shares host's network namespace
- ⚠️ Port conflicts possible (can't run two containers on same port)
- ⚠️ Less secure (no network isolation)
- ✅ Better performance (no NAT overhead)
- ⚠️ Only works on Linux hosts

**Example:**
```bash
# Run with host network
docker run -d --network host nginx

# Container's port 80 is directly accessible on host port 80
# No -p flag needed!
curl http://localhost:80

# Warning: Can't run another container on port 80
docker run -d --network host nginx  # Fails! Port already in use
```

**Real-World Use Case:**
```bash
# High-performance monitoring tool
docker run -d \
  --network host \
  --name prometheus \
  -v prometheus-data:/prometheus \
  prom/prometheus

# Can access all host services directly
# Better performance for metrics collection
```

**Comparison:**
```bash
# Bridge: Container IP is different from host
docker run -d -p 8080:80 nginx
# Access: http://localhost:8080

# Host: Container uses host IP directly
docker run -d --network host nginx
# Access: http://localhost:80 (if nginx listens on 80)
```

---

#### 3. **None Network**

**Description:** Completely disables networking for the container. Container has loopback interface only.

**Use Cases:**
- Batch processing jobs that don't need network
- Security-sensitive tasks requiring complete isolation
- Testing code that shouldn't make network calls

**Key Points:**
- ✅ Maximum isolation
- ✅ No network interfaces except loopback (127.0.0.1)
- ⚠️ Cannot communicate with other containers
- ⚠️ Cannot access external network

**Example:**
```bash
# Run with no networking
docker run -d --network none alpine sleep 1000

# Container has only loopback interface
docker exec container-name ip addr show

# Output shows only 'lo' interface:
# 1: lo: <LOOPBACK,UP,LOWER_UP>
#     inet 127.0.0.1/8 scope host lo
```

**Real-World Use Case:**
```bash
# Data processing job that doesn't need network
docker run --rm \
  --network none \
  -v $(pwd)/data:/data \
  my-processor:latest \
  process --input /data/input.csv --output /data/output.csv

# Even if code tries to access network, it will fail
# Extra security for sensitive data processing
```

**Testing Example:**
```bash
# Test that code works without network
docker run -it --network none python:3.11 python
>>> import requests
>>> requests.get('https://google.com')
# Will fail with network error
```

---

## Advanced: Orchestration & Optimization

### Docker Compose

**Definition:** Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file.

**Use Cases:**
- **Development environments:** Run app + database + cache together
- **Testing:** Set up complete test environment
- **Single-host deployments:** Deploy multi-container apps on one machine
- **CI/CD pipelines:** Reproducible test environments

**Key Benefits:**
- ✅ Define entire application stack in one file
- ✅ One command to start everything
- ✅ Automatic network creation
- ✅ Service dependencies management
- ✅ Easy scaling

---

#### docker-compose.yml Structure

**Basic Structure:**
```yaml
version: '3.8'  # Compose file version

services:       # Define containers
  service-name:
    # Service configuration
    
volumes:        # Define volumes (optional)
  volume-name:
    # Volume configuration

networks:       # Define networks (optional)
  network-name:
    # Network configuration
```

---

#### Key Directives

**Complete Example with All Major Directives:**
```yaml
version: '3.8'

services:
  # Frontend Service
  frontend:
    # Build from Dockerfile
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    
    # Or use image
    # image: nginx:alpine
    
    # Container name
    container_name: my-frontend
    
    # Ports mapping
    ports:
      - "80:80"
      - "443:443"
    
    # Environment variables
    environment:
      - API_URL=http://backend:3000
      - NODE_ENV=production
    
    # Or load from file
    # env_file:
    #   - ./frontend.env
    
    # Volumes
    volumes:
      - ./frontend/nginx.conf:/etc/nginx/nginx.conf:ro
      - frontend-logs:/var/log/nginx
    
    # Networks
    networks:
      - app-network
    
    # Dependencies
    depends_on:
      - backend
    
    # Restart policy
    restart: unless-stopped
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # Backend Service
  backend:
    build:
      context: ./backend
      target: production
    container_name: my-backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend/uploads:/app/uploads
      - backend-logs:/app/logs
    networks:
      - app-network
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  # Database Service
  postgres:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - app-network
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache Service
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: always

  # Worker Service
  worker:
    build:
      context: ./backend
      target: production
    command: node worker.js
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend/uploads:/app/uploads
    networks:
      - app-network
    depends_on:
      - postgres
      - redis
    restart: always
    deploy:
      replicas: 2  # Run 2 instances

# Named volumes
volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  frontend-logs:
  backend-logs:

# Custom networks
networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

---

#### Common Docker Compose Commands

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Build and start
docker-compose up --build

# Stop all services
docker-compose stop

# Stop and remove containers, networks
docker-compose down

# Stop and remove including volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Logs for specific service
docker-compose logs backend

# List running services
docker-compose ps

# Execute command in service
docker-compose exec backend sh

# Scale service
docker-compose up -d --scale worker=3

# Restart service
docker-compose restart backend

# Pull latest images
docker-compose pull

# Build services
docker-compose build

# Validate compose file
docker-compose config
```

---

#### Real-World Example: Full E-Commerce Application

**Project Structure:**
```
my-ecommerce/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   └── ... (React app)
├── backend/
│   ├── Dockerfile
│   └── ... (Node.js API)
├── nginx/
│   └── nginx.conf
└── init-scripts/
    └── init.sql
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend
    networks:
      - app-network
    restart: always

  # React Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production
    expose:
      - "3000"
    environment:
      - REACT_APP_API_URL=/api
    networks:
      - app-network
    restart: always

  # Node.js Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://ecomm_user:secret@postgres:5432/ecommerce
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - STRIPE_KEY=${STRIPE_KEY}
    volumes:
      - ./backend/uploads:/app/uploads
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    restart: always

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=ecomm_user
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=ecommerce
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - app-network
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ecomm_user -d ecommerce"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: always

  # Background Worker
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: node worker.js
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://ecomm_user:secret@postgres:5432/ecommerce
      - REDIS_URL=redis://redis:6379
      - EMAIL_SERVICE=${EMAIL_SERVICE}
    depends_on:
      - postgres
      - redis
    networks:
      - app-network
    restart: always

  # Elasticsearch for search
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    networks:
      - app-network
    restart: always

volumes:
  postgres-data:
  redis-data:
  elasticsearch-data:

networks:
  app-network:
    driver: bridge
```

**Usage:**
```bash
# Create .env file
cat > .env << EOF
JWT_SECRET=your-secret-key
STRIPE_KEY=your-stripe-key
REDIS_PASSWORD=redis-secret
EMAIL_SERVICE=smtp://user:pass@smtp.gmail.com
EOF

# Start everything
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f backend

# Scale workers
docker-compose up -d --scale worker=3

# Stop everything
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

### Multi-Stage Builds

**Purpose:** Create optimized production images by using multiple FROM statements. Build stages can copy artifacts from each other, allowing you to:
- Separate build dependencies from runtime dependencies
- Dramatically reduce final image size
- Include build tools without shipping them to production

**Key Concepts:**
- Each FROM instruction starts a new stage
- Stages can be named using `AS name`
- Use `COPY --from=stage-name` to copy between stages
- Only the final stage becomes the final image

---

#### Example 1: Node.js Application

**Without Multi-Stage (❌ Large Image):**
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install  # Includes dev dependencies!
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/server.js"]

# Result: ~1.2GB image
```

**With Multi-Stage (✅ Small Image):**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app

# Install ALL dependencies (including dev)
COPY package*.json ./
RUN npm install

# Build application
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app

# Install ONLY production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000
CMD ["node", "dist/server.js"]

# Result: ~180MB image (85% reduction!)
```

---

#### Example 2: Go Application

**Multi-Stage Go Build:**
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary with optimizations
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo \
    -ldflags="-w -s" \
    -o main .

# Stage 2: Production (minimal!)
FROM alpine:latest
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy only the binary
COPY --from=builder /app/main .

EXPOSE 8080
CMD ["./main"]

# Result: ~15MB image!
```

**Even Smaller with Scratch:**
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo \
    -ldflags="-w -s" \
    -o main .

# Stage 2: Scratch base (no OS!)
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /app/main /main

EXPOSE 8080
CMD ["/main"]

# Result: ~8MB image!
```

---

#### Example 3: Python Application

**Multi-Stage Python Build:**
```dockerfile
# Stage 1: Builder
FROM python:3.11-slim AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim

# Install only runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy Python dependencies from builder
COPY --from=builder /root/.local /root/.local

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

WORKDIR /app

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]

# Result: ~200MB vs ~800MB without multi-stage
```

---

#### Example 4: React Application

**Multi-Stage React Build:**
```dockerfile
# Stage 1: Build React app
FROM node:18-alpine AS builder
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Build production bundle
COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:alpine

# Copy built assets from builder
COPY --from=builder /app/build /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Add health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

# Result: ~25MB vs ~1.2GB without multi-stage
```

---

#### Example 5: Java Spring Boot Application

**Multi-Stage Java Build:**
```dockerfile
# Stage 1: Build with Maven
FROM maven:3.9-openjdk-17-slim AS builder
WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn package -DskipTests

# Extract layers for better caching
RUN mkdir -p target/extracted && \
    java -Djarmode=layertools -jar target/*.jar extract --destination target/extracted

# Stage 2: Runtime
FROM openjdk:17-jre-slim

WORKDIR /app

# Copy layers (in order of change frequency)
COPY --from=builder /app/target/extracted/dependencies ./
COPY --from=builder /app/target/extracted/spring-boot-loader ./
COPY --from=builder /app/target/extracted/snapshot-dependencies ./
COPY --from=builder /app/target/extracted/application ./

# Run as non-root
RUN useradd -m -u 1000 spring && \
    chown -R spring:spring /app
USER spring

EXPOSE 8080

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]

# Result: ~250MB vs ~700MB without multi-stage
```

---

#### Size Comparison Summary

```
Application Type    | Single Stage | Multi-Stage | Reduction
--------------------|--------------|-------------|----------
Node.js App         | 1.2GB        | 180MB       | 85%
Go Binary           | 800MB        | 8MB         | 99%
Python App          | 800MB        | 200MB       | 75%
React App           | 1.2GB        | 25MB        | 98%
Java Spring Boot    | 700MB        | 250MB       | 64%
```

---

### Security & Environment

#### Non-Root Users: Best Practices

**Why Run as Non-Root?**
- ✅ Principle of least privilege
- ✅ Limit damage if container is compromised
- ✅ Prevent escalation attacks
- ✅ Better compliance with security policies

**❌ Bad Practice (Running as Root):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# Container runs as root (uid 0)
# Security risk!
```

**✅ Good Practice (Non-Root User):**

**Method 1: Create Custom User**
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app

# Copy and install as root
COPY package*.json ./
RUN npm ci --only=production

# Copy application files
COPY . .

# Change ownership to non-root user
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

**Method 2: Use Existing User**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies as root
COPY package*.json ./
RUN npm ci --only=production

# Copy application
COPY --chown=node:node . .

# Switch to 'node' user (built into official node images)
USER node

EXPOSE 3000
CMD ["node", "server.js"]
```

**Method 3: Python with useradd**
```dockerfile
FROM python:3.11-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application and change ownership
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

EXPOSE 8000
CMD ["python", "app.py"]
```

**Method 4: Multi-Stage with Non-Root**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

WORKDIR /app

# Copy with correct ownership
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

# Install production dependencies
RUN npm ci --only=production

# Switch to non-root user
USER nodejs

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Verification:**
```bash
# Build image
docker build -t my-secure-app .

# Run container
docker run -d --name test-app my-secure-app

# Check which user is running
docker exec test-app whoami
# Output: appuser (not root!)

# Check user ID
docker exec test-app id
# Output: uid=1000(appuser) gid=1000(appuser)

# Try to write to protected directory (should fail)
docker exec test-app touch /etc/test-file
# Output: Permission denied (good!)
```

---

#### Build Arguments (ARG) vs Environment Variables (ENV)

**ARG** - Build-Time Variables
- Available only during image build
- Not available in running containers
- Can have default values
- Can be overridden during build with `--build-arg`
- Useful for: versions, build configurations, toggles

**ENV** - Runtime Variables
- Available during build AND in running containers
- Persist in the final image
- Can be overridden when running container with `-e`
- Useful for: application settings, API keys, database URLs

---

#### Comparison Table

| Feature | ARG | ENV |
|---------|-----|-----|
| **Available during build** | ✅ Yes | ✅ Yes |
| **Available in container** | ❌ No | ✅ Yes |
| **Set at build time** | `--build-arg` | In Dockerfile |
| **Override at runtime** | ❌ No | `docker run -e` |
| **Persisted in image** | ❌ No | ✅ Yes |
| **Visible in image history** | ✅ Yes | ✅ Yes |
| **Default value** | Optional | Required |

---

#### ARG Examples

**Basic Usage:**
```dockerfile
FROM node:18-alpine

# Define build argument with default value
ARG NODE_ENV=production
ARG APP_VERSION=1.0.0

# Use ARG in RUN commands
RUN echo "Building for ${NODE_ENV}"

WORKDIR /app
COPY package*.json ./

# Use ARG to conditionally install dependencies
RUN if [ "$NODE_ENV" = "development" ]; then \
      npm install; \
    else \
      npm ci --only=production; \
    fi

COPY . .

# ARG is NOT available at runtime!
CMD ["node", "server.js"]
```

**Build with custom ARG:**
```bash
# Use default values
docker build -t myapp:prod .

# Override ARG values
docker build --build-arg NODE_ENV=development --build-arg APP_VERSION=2.0.0 -t myapp:dev .
```

**Multiple ARGs:**
```dockerfile
FROM ubuntu:22.04

# Define multiple ARGs
ARG DEBIAN_FRONTEND=noninteractive
ARG PYTHON_VERSION=3.11
ARG USER_UID=1000
ARG USER_GID=1000

# Use ARGs
RUN apt-get update && \
    apt-get install -y python${PYTHON_VERSION} && \
    rm -rf /var/lib/apt/lists/*

# Create user with specified UID/GID
RUN groupadd -g ${USER_GID} appgroup && \
    useradd -m -u ${USER_UID} -g appgroup appuser

USER appuser
```

---

#### ENV Examples

**Basic Usage:**
```dockerfile
FROM node:18-alpine

# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

EXPOSE ${PORT}

# ENV is available at runtime
CMD ["node", "server.js"]
```

**Runtime Override:**
```bash
# Use Dockerfile ENV values
docker run -d myapp
# NODE_ENV=production, PORT=3000

# Override ENV at runtime
docker run -d -e NODE_ENV=development -e PORT=8080 myapp
# NODE_ENV=development, PORT=8080
```

**Multiple ENV Formats:**
```dockerfile
FROM python:3.11-slim

# Format 1: One per line
ENV APP_ENV production
ENV DEBUG false

# Format 2: Multiple in one line (space-separated)
ENV DATABASE_HOST=localhost DATABASE_PORT=5432

# Format 3: Multiple with line breaks (cleaner)
ENV APP_NAME=myapp \
    APP_VERSION=1.0.0 \
    LOG_LEVEL=info \
    MAX_CONNECTIONS=100

WORKDIR /app
```

---

#### Combined ARG and ENV

**Pattern: ARG → ENV (Build-time to Runtime)**
```dockerfile
FROM node:18-alpine

# ARG for build-time configuration
ARG NODE_ENV=production
ARG API_URL

# Convert ARG to ENV for runtime access
ENV NODE_ENV=${NODE_ENV}
ENV API_URL=${API_URL}

WORKDIR /app
COPY package*.json ./

# Use ARG during build
RUN if [ "$NODE_ENV" = "production" ]; then \
      npm ci --only=production; \
    else \
      npm install; \
    fi

COPY . .

EXPOSE 3000

# ENV values available at runtime
CMD ["node", "server.js"]
```

**Build and Run:**
```bash
# Build with custom values
docker build \
  --build-arg NODE_ENV=production \
  --build-arg API_URL=https://api.example.com \
  -t myapp:prod .

# ENV values are baked into image
docker run -d myapp:prod

# But can still override at runtime if needed
docker run -d -e API_URL=https://api.staging.com myapp:prod
```

---

#### Real-World Complete Example

```dockerfile
# Multi-stage build with ARG and ENV
FROM node:18-alpine AS builder

# Build arguments
ARG NODE_ENV=production
ARG BUILD_DATE
ARG GIT_COMMIT
ARG APP_VERSION=1.0.0

# Label image with build info
LABEL build_date="${BUILD_DATE}"
LABEL git_commit="${GIT_COMMIT}"
LABEL version="${APP_VERSION}"

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine

# Runtime environment variables
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info \
    RATE_LIMIT=100

# Build-time arguments (for metadata)
ARG BUILD_DATE
ARG GIT_COMMIT
ARG APP_VERSION=1.0.0

# Convert to ENV for runtime access
ENV APP_VERSION=${APP_VERSION} \
    BUILD_DATE=${BUILD_DATE} \
    GIT_COMMIT=${GIT_COMMIT}

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

USER nodejs

EXPOSE ${PORT}

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:${PORT}/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "dist/server.js"]
```

**Build Script:**
```bash
#!/bin/bash

# Get build metadata
BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
GIT_COMMIT=$(git rev-parse --short HEAD)
APP_VERSION="1.2.3"

# Build image
docker build \
  --build-arg BUILD_DATE=${BUILD_DATE} \
  --build-arg GIT_COMMIT=${GIT_COMMIT} \
  --build-arg APP_VERSION=${APP_VERSION} \
  -t myapp:${APP_VERSION} \
  -t myapp:latest \
  .

echo "Built image with:"
echo "  Version: ${APP_VERSION}"
echo "  Commit: ${GIT_COMMIT}"
echo "  Date: ${BUILD_DATE}"
```

**Run with Environment Override:**
```bash
# Production
docker run -d \
  --name myapp-prod \
  -p 80:3000 \
  -e DATABASE_URL=postgres://prod-db:5432/mydb \
  -e REDIS_URL=redis://prod-redis:6379 \
  -e LOG_LEVEL=warn \
  myapp:latest

# Staging
docker run -d \
  --name myapp-staging \
  -p 8080:3000 \
  -e NODE_ENV=staging \
  -e DATABASE_URL=postgres://staging-db:5432/mydb \
  -e REDIS_URL=redis://staging-redis:6379 \
  -e LOG_LEVEL=debug \
  myapp:latest
```

---

## 📚 Summary & Quick Reference

### Key Concepts Recap

**Beginner:**
- Docker = containerization platform
- Images = blueprints, Containers = running instances
- Dockerfile = recipe to build images
- Docker Hub = registry for sharing images
- Basic commands: run, pull, ps, stop, rm

**Intermediate:**
- Dockerfile instructions: FROM, RUN, CMD, ENTRYPOINT, EXPOSE, WORKDIR
- Best practices: layer optimization, caching, minimize image size
- Volumes = managed by Docker (recommended)
- Bind mounts = direct host filesystem access
- Networking: Bridge (default), Host (performance), None (isolated)

**Advanced:**
- Docker Compose = multi-container orchestration
- docker-compose.yml = define entire application stack
- Multi-stage builds = optimize image size (90%+ reduction possible)
- Security: non-root users, ARG vs ENV
- ARG = build-time only, ENV = runtime accessible

---

### Command Cheat Sheet

```bash
# Images
docker images                          # List images
docker pull image:tag                  # Download image
docker build -t name:tag .             # Build image
docker rmi image:tag                   # Remove image
docker image prune                     # Remove unused images

# Containers
docker run -d --name name image        # Run container
docker ps                              # List running containers
docker ps -a                           # List all containers
docker stop container                  # Stop container
docker start container                 # Start container
docker restart container               # Restart container
docker rm container                    # Remove container
docker exec -it container sh           # Execute command in container
docker logs container                  # View logs
docker logs -f container               # Follow logs

# Volumes
docker volume create name              # Create volume
docker volume ls                       # List volumes
docker volume inspect name             # Inspect volume
docker volume rm name                  # Remove volume
docker volume prune                    # Remove unused volumes

# Networks
docker network create name             # Create network
docker network ls                      # List networks
docker network inspect name            # Inspect network
docker network rm name                 # Remove network

# Docker Compose
docker-compose up                      # Start services
docker-compose up -d                   # Start in background
docker-compose down                    # Stop and remove
docker-compose ps                      # List services
docker-compose logs -f                 # Follow logs
docker-compose exec service sh         # Execute command
docker-compose build                   # Build services

# Cleanup
docker system prune                    # Remove unused data
docker system prune -a                 # Remove all unused data
docker system prune -a --volumes       # Remove everything unused
```

---

### Best Practices Checklist

**Dockerfile:**
- ✅ Use specific image tags (not `latest`)
- ✅ Use official images when possible
- ✅ Use multi-stage builds for production
- ✅ Order instructions from least to most frequently changing
- ✅ Combine RUN commands to reduce layers
- ✅ Use .dockerignore to exclude unnecessary files
- ✅ Run containers as non-root users
- ✅ Use COPY instead of ADD (unless you need ADD's features)
- ✅ Set appropriate WORKDIR
- ✅ Document exposed ports with EXPOSE

**Security:**
- ✅ Don't store secrets in images (use environment variables or secrets management)
- ✅ Run as non-root user
- ✅ Use minimal base images (alpine, slim, distroless)
- ✅ Scan images for vulnerabilities
- ✅ Keep images updated
- ✅ Limit container resources (CPU, memory)
- ✅ Use read-only filesystems where possible

**Production:**
- ✅ Use health checks
- ✅ Implement proper logging
- ✅ Use volumes for persistent data
- ✅ Set restart policies (unless-stopped, always)
- ✅ Use Docker Compose for multi-container apps
- ✅ Monitor container resource usage
- ✅ Implement graceful shutdown
- ✅ Use specific versions/tags, not `latest`

---

## 🎓 Additional Learning Resources

**Official Documentation:**
- Docker Docs: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Dockerfile Reference: https://docs.docker.com/engine/reference/builder/
- Docker Compose: https://docs.docker.com/compose/

**Practice:**
- Play with Docker: https://labs.play-with-docker.com
- Docker Samples: https://github.com/dockersamples

**Security:**
- Docker Security Best Practices: https://docs.docker.com/engine/security/
- CIS Docker Benchmark: https://www.cisecurity.org/benchmark/docker

---

**End of Docker Comprehensive Study Notes**
