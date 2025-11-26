# Docker Quick Reference Cheat Sheet

## Essential Commands at a Glance

### Container Lifecycle
```bash
# Run container
docker run -d --name myapp -p 8080:80 nginx

# Stop container
docker stop myapp

# Start stopped container
docker start myapp

# Restart container
docker restart myapp

# Remove container
docker rm myapp

# Remove running container
docker rm -f myapp
```

### Image Management
```bash
# Build image
docker build -t myapp:1.0 .

# List images
docker images

# Remove image
docker rmi myapp:1.0

# Pull from registry
docker pull nginx:latest

# Push to registry
docker push myrepo/myapp:1.0

# Tag image
docker tag myapp:1.0 myapp:latest
```

### Container Info & Debugging
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# View logs
docker logs -f myapp

# Execute command in container
docker exec -it myapp bash

# Inspect container
docker inspect myapp

# View resource usage
docker stats myapp
```

### Docker Compose
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild and start
docker-compose up --build

# Scale service
docker-compose up --scale web=3

# Execute in service
docker-compose exec web bash
```

### Cleanup
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything
docker system prune -a --volumes
```

### Networking
```bash
# Create network
docker network create mynet

# List networks
docker network ls

# Connect container to network
docker network connect mynet myapp

# Inspect network
docker network inspect mynet
```

### Volumes
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata
```

---

## Common Docker Run Options

| Option | Description | Example |
|--------|-------------|---------|
| `-d` | Detached mode | `docker run -d nginx` |
| `-it` | Interactive terminal | `docker run -it ubuntu bash` |
| `-p` | Port mapping | `docker run -p 8080:80 nginx` |
| `-v` | Volume mount | `docker run -v data:/data nginx` |
| `-e` | Environment variable | `docker run -e DB_HOST=localhost app` |
| `--name` | Container name | `docker run --name myapp nginx` |
| `--rm` | Auto-remove on exit | `docker run --rm ubuntu echo hello` |
| `--network` | Network | `docker run --network mynet nginx` |
| `--restart` | Restart policy | `docker run --restart=always nginx` |
| `-m` | Memory limit | `docker run -m 512m nginx` |
| `--cpus` | CPU limit | `docker run --cpus="1.5" nginx` |

---

## Dockerfile Best Practices

```dockerfile
# Use specific base image version
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Use non-root user
USER node

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s CMD curl -f http://localhost:3000/health || exit 1

# Start command
CMD ["node", "server.js"]
```

---

## Docker Compose Template

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - ./src:/app/src
    networks:
      - app-network
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

---

## Useful Inspect Commands

```bash
# Get container IP
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myapp

# Get container status
docker inspect -f '{{.State.Status}}' myapp

# Get container environment variables
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' myapp

# Get mounted volumes
docker inspect -f '{{json .Mounts}}' myapp | jq

# Get port mappings
docker port myapp
```

---

## Multi-Stage Build Example

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

## .dockerignore Template

```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
.DS_Store
*.md
docker-compose*.yml
Dockerfile*
.dockerignore
coverage
.nyc_output
dist
build
*.log
```

---

## Security Best Practices

```bash
# Run as non-root user
docker run --user 1000:1000 myapp

# Read-only filesystem
docker run --read-only myapp

# Drop all capabilities
docker run --cap-drop=ALL myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp

# Limit resources
docker run -m 512m --cpus="1.0" myapp

# Scan for vulnerabilities
docker scout cves myapp:latest
```

---

## Debugging Commands

```bash
# View container logs with tail
docker logs --tail 100 -f myapp

# View logs since timestamp
docker logs --since 2023-01-01T00:00:00 myapp

# Get container process list
docker top myapp

# View filesystem changes
docker diff myapp

# Copy files from container
docker cp myapp:/app/file.txt ./local-file.txt

# Execute as root
docker exec -u root -it myapp bash

# Check health status
docker inspect --format='{{.State.Health.Status}}' myapp

# View container exit code
docker inspect --format='{{.State.ExitCode}}' myapp
```

---

## Network Troubleshooting

```bash
# Inspect network
docker network inspect bridge

# Test connectivity
docker run --rm busybox ping mycontainer

# Test DNS resolution
docker run --rm busybox nslookup mycontainer

# Run netshoot for advanced debugging
docker run --rm --network container:myapp nicolaka/netshoot

# View container networks
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' myapp
```

---

## Docker Swarm Quick Reference

```bash
# Initialize swarm
docker swarm init

# Create service
docker service create --name web --replicas 3 -p 80:80 nginx

# List services
docker service ls

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:latest web

# View service logs
docker service logs -f web

# Remove service
docker service rm web

# Deploy stack
docker stack deploy -c stack.yml mystack

# List stacks
docker stack ls

# Remove stack
docker stack rm mystack
```

---

## Registry Operations

```bash
# Login to Docker Hub
docker login

# Login to private registry
docker login registry.example.com

# Tag for registry
docker tag myapp:1.0 registry.example.com/myapp:1.0

# Push to registry
docker push registry.example.com/myapp:1.0

# Pull from registry
docker pull registry.example.com/myapp:1.0

# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# List local registry images
curl -X GET http://localhost:5000/v2/_catalog
```

---

## Performance Optimization

```bash
# Build with BuildKit
DOCKER_BUILDKIT=1 docker build -t myapp .

# Use build cache
docker build --cache-from myapp:latest -t myapp:new .

# Check image layers
docker history myapp:latest

# Analyze image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# System disk usage
docker system df

# Detailed disk usage
docker system df -v
```

---

## Common Patterns

### Web Application
```bash
docker run -d \
  --name webapp \
  -p 80:80 \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  --restart unless-stopped \
  nginx:alpine
```

### Database
```bash
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  --restart unless-stopped \
  postgres:14
```

### Development Environment
```bash
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18 \
  bash
```

---

## Keyboard Shortcuts (Terminal)

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Stop following logs |
| `Ctrl+P, Ctrl+Q` | Detach from container (keep running) |
| `Ctrl+D` | Exit container shell |
| `exit` | Exit container (stops if no other process) |

---

## Environment Variables

```bash
# Set Docker host
export DOCKER_HOST=tcp://192.168.1.100:2376

# Enable BuildKit
export DOCKER_BUILDKIT=1

# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Set default platform
export DOCKER_DEFAULT_PLATFORM=linux/amd64
```

---

## Quick Filters

```bash
# Running containers
docker ps --filter "status=running"

# Exited containers
docker ps -a --filter "status=exited"

# Containers by image
docker ps --filter "ancestor=nginx"

# Containers by label
docker ps --filter "label=environment=production"

# Dangling images
docker images --filter "dangling=true"

# Images by label
docker images --filter "label=version=1.0"
```

---

## Format Output

```bash
# Custom PS format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# JSON output
docker inspect --format='{{json .State}}' myapp | jq

# Go template
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp

# List only IDs
docker ps -q

# List only names
docker ps --format '{{.Names}}'
```

---

## One-Liners

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Remove exited containers
docker rm $(docker ps -a -f status=exited -q)

# Get container IPs
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)

# Follow logs of multiple containers
docker logs -f $(docker ps -q)
```

---

## Resource Limits

```bash
# Memory limit
docker run -m 512m myapp

# Memory with swap
docker run -m 512m --memory-swap 1g myapp

# CPU shares (relative weight)
docker run --cpu-shares 512 myapp

# CPU count
docker run --cpus="1.5" myapp

# CPU set (specific cores)
docker run --cpuset-cpus="0,1" myapp

# PID limit
docker run --pids-limit 100 myapp

# Block I/O weight
docker run --blkio-weight 500 myapp

# Ulimits
docker run --ulimit nofile=1024:1024 myapp
```

---

## Health Check

```bash
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# In docker run
docker run --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  myapp

# Check health status
docker inspect --format='{{.State.Health.Status}}' myapp
```

---

## Copy Files

```bash
# Copy from container to host
docker cp myapp:/app/file.txt ./local-file.txt

# Copy from host to container
docker cp ./local-file.txt myapp:/app/file.txt

# Copy directory
docker cp myapp:/app/logs ./logs

# Preserve ownership
docker cp --archive myapp:/app ./backup
```

---

## Export/Import

```bash
# Export container filesystem
docker export myapp > myapp.tar

# Import filesystem as image
docker import myapp.tar myapp:imported

# Save image to tar
docker save -o myapp.tar myapp:1.0

# Load image from tar
docker load -i myapp.tar

# Save multiple images
docker save -o images.tar myapp:1.0 nginx:latest

# Save with compression
docker save myapp:1.0 | gzip > myapp.tar.gz
```

---

## Tips & Tricks

1. **Use alpine images** for smaller sizes
2. **Layer caching**: Order Dockerfile instructions from least to most frequently changing
3. **Multi-stage builds**: Separate build and runtime dependencies
4. **Health checks**: Always implement for production containers
5. **Logging**: Use proper log drivers and rotation
6. **Secrets**: Never hardcode in images or env vars
7. **Networks**: Use custom networks instead of --link
8. **Volumes**: Use named volumes for persistence
9. **Tags**: Always use specific versions in production
10. **Security**: Run as non-root, scan images, limit resources

---

## Common Errors & Solutions

### Port already in use
```bash
# Find process
lsof -i :8080
# Kill process or use different port
docker run -p 8081:80 myapp
```

### Cannot connect to Docker daemon
```bash
# Check if Docker is running
docker info
# Start Docker service
sudo systemctl start docker
```

### Out of disk space
```bash
# Clean up
docker system prune -a --volumes
```

### Image not found
```bash
# Check image name
docker images
# Pull if needed
docker pull imagename:tag
```

### Container exits immediately
```bash
# Check logs
docker logs containername
# Run interactively
docker run -it imagename /bin/sh
```

---

**Pro Tip:** Add `alias dk='docker'` and `alias dkc='docker-compose'` to your shell config for faster typing!

---

For complete documentation, see:
- Docker Deep Dive Guide
- Docker Practice Exercises
- Official Docker Docs: https://docs.docker.com
