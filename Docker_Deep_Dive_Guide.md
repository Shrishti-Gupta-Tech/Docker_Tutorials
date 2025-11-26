# Docker Deep Dive: Top 100+ Commands for Real-World Scenarios

## Table of Contents
1. [Docker Fundamentals](#docker-fundamentals)
2. [Getting Started - Small Projects](#scenario-1-getting-started---small-projects)
3. [Development Environment Setup](#scenario-2-development-environment-setup)
4. [Building and Managing Images](#scenario-3-building-and-managing-images)
5. [Container Lifecycle Management](#scenario-4-container-lifecycle-management)
6. [Networking in Docker](#scenario-5-networking-in-docker)
7. [Data Management and Volumes](#scenario-6-data-management-and-volumes)
8. [Multi-Container Applications](#scenario-7-multi-container-applications)
9. [Docker Compose](#scenario-8-docker-compose)
10. [Production Deployment - Startup Environment](#scenario-9-production-deployment---startup-environment)
11. [Enterprise/MNC Operations](#scenario-10-enterprisemnc-operations)
12. [Debugging and Troubleshooting](#scenario-11-debugging-and-troubleshooting)
13. [Security Best Practices](#scenario-12-security-best-practices)
14. [Performance Optimization](#scenario-13-performance-optimization)
15. [CI/CD Integration](#scenario-14-cicd-integration)

---

## Docker Fundamentals

### What is Docker?
Docker is a platform for developing, shipping, and running applications in containers. Containers are lightweight, standalone packages that include everything needed to run an application.

### Key Concepts:
- **Image**: A read-only template with instructions for creating a container
- **Container**: A runnable instance of an image
- **Dockerfile**: A text file with instructions to build an image
- **Registry**: A repository for storing and distributing Docker images
- **Volume**: Persistent storage for containers
- **Network**: Communication channels between containers

---

## Scenario 1: Getting Started - Small Projects

### 1. Installation and Version Check

```bash
# Check Docker version
docker --version
docker version

# Display system-wide information
docker info

# Get help on Docker commands
docker --help
docker <command> --help
```

### 2. Working with Docker Hub

```bash
# Search for images on Docker Hub
docker search nginx
docker search --limit 5 python

# Pull an image from Docker Hub
docker pull ubuntu
docker pull nginx:latest
docker pull node:18-alpine

# Login to Docker Hub
docker login
docker login -u username -p password

# Logout from Docker Hub
docker logout
```

### 3. Running Your First Container

```bash
# Run a container (pulls if not present)
docker run hello-world

# Run container in interactive mode
docker run -it ubuntu bash

# Run container in detached mode
docker run -d nginx

# Run container with a name
docker run --name my-nginx -d nginx

# Run container with port mapping
docker run -p 8080:80 -d nginx

# Run container with environment variables
docker run -e MYSQL_ROOT_PASSWORD=password -d mysql

# Run container with restart policy
docker run --restart=always -d nginx
```

---

## Scenario 2: Development Environment Setup

### 4. Creating Development Containers

```bash
# Run Node.js development environment
docker run -it -v $(pwd):/app -w /app node:18 bash

# Run Python development environment
docker run -it -v $(pwd):/code -w /code python:3.11 bash

# Run container with multiple port mappings
docker run -p 3000:3000 -p 8080:8080 -d node:18

# Run container with custom network
docker run --network my-network -d nginx

# Run container with memory limit
docker run -m 512m -d nginx

# Run container with CPU limit
docker run --cpus="1.5" -d nginx
```

### 5. Interactive Development

```bash
# Execute command in running container
docker exec -it container_name bash
docker exec container_name ls /app

# Execute command as root user
docker exec -u root -it container_name bash

# Copy files to/from container
docker cp local-file.txt container_name:/path/
docker cp container_name:/path/file.txt ./local-file.txt

# View container logs
docker logs container_name
docker logs -f container_name  # Follow logs
docker logs --tail 100 container_name  # Last 100 lines

# Attach to running container
docker attach container_name
```

---

## Scenario 3: Building and Managing Images

### 6. Creating Dockerfiles

```bash
# Basic Dockerfile example for Node.js app
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
EOF

# Build image from Dockerfile
docker build -t my-app:1.0 .
docker build -t my-app:latest .

# Build with no cache
docker build --no-cache -t my-app:1.0 .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t my-app:1.0 .

# Build from different Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .

# Build with tag
docker build -t username/my-app:v1.0.0 .
```

### 7. Image Management

```bash
# List all images
docker images
docker image ls

# List all images including intermediate
docker images -a

# Filter images
docker images --filter "dangling=true"
docker images --filter "label=version=1.0"

# Remove an image
docker rmi image_name
docker rmi image_id

# Remove multiple images
docker rmi image1 image2 image3

# Force remove image
docker rmi -f image_name

# Remove all unused images
docker image prune
docker image prune -a  # Remove all unused images

# Tag an image
docker tag source_image:tag target_image:tag
docker tag my-app:latest my-app:1.0.0

# Push image to registry
docker push username/my-app:1.0.0

# Save image to tar file
docker save -o my-app.tar my-app:1.0

# Load image from tar file
docker load -i my-app.tar

# Export image to tar
docker export container_name > container.tar

# Import from tar
docker import container.tar new-image:tag

# Show image history
docker history image_name

# Inspect image
docker inspect image_name
docker image inspect image_name
```

---

## Scenario 4: Container Lifecycle Management

### 8. Container Operations

```bash
# List running containers
docker ps

# List all containers (running and stopped)
docker ps -a

# List container IDs only
docker ps -q

# List containers with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Stop a container
docker stop container_name
docker stop container_id

# Stop multiple containers
docker stop container1 container2

# Stop all running containers
docker stop $(docker ps -q)

# Start a stopped container
docker start container_name

# Restart a container
docker restart container_name

# Pause a container
docker pause container_name

# Unpause a container
docker unpause container_name

# Kill a container (force stop)
docker kill container_name

# Remove a container
docker rm container_name

# Remove running container (force)
docker rm -f container_name

# Remove all stopped containers
docker container prune

# Remove container with volumes
docker rm -v container_name

# Rename a container
docker rename old_name new_name

# Wait for container to stop
docker wait container_name
```

### 9. Container Inspection

```bash
# Inspect container details
docker inspect container_name

# Get specific info using format
docker inspect --format='{{.State.Status}}' container_name
docker inspect --format='{{.NetworkSettings.IPAddress}}' container_name

# Show container resource usage statistics
docker stats
docker stats container_name

# Show running processes in container
docker top container_name

# Show container port mappings
docker port container_name

# Show container changes
docker diff container_name

# Create image from container
docker commit container_name new-image:tag

# Update container configuration
docker update --memory 512m container_name
docker update --cpus="2" container_name
```

---

## Scenario 5: Networking in Docker

### 10. Network Management

```bash
# List networks
docker network ls

# Create a network
docker network create my-network

# Create network with specific driver
docker network create --driver bridge my-bridge-network

# Create network with subnet
docker network create --subnet=172.18.0.0/16 my-custom-network

# Inspect network
docker network inspect my-network

# Connect container to network
docker network connect my-network container_name

# Disconnect container from network
docker network disconnect my-network container_name

# Remove network
docker network rm my-network

# Remove all unused networks
docker network prune

# Create overlay network (for Swarm)
docker network create --driver overlay my-overlay-network
```

### 11. Container Communication

```bash
# Run container on specific network
docker run -d --network my-network --name web nginx

# Run container with network alias
docker run -d --network my-network --network-alias mydb mysql

# Link containers (legacy)
docker run -d --name db mysql
docker run -d --name app --link db:database my-app

# Expose ports
docker run -p 8080:80 nginx  # Host:Container
docker run -p 127.0.0.1:8080:80 nginx  # Bind to specific IP
docker run -P nginx  # Expose all ports randomly

# Run container with custom DNS
docker run --dns 8.8.8.8 -d nginx

# Run container with host network
docker run --network host nginx
```

---

## Scenario 6: Data Management and Volumes

### 12. Volume Management

```bash
# Create a volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove all unused volumes
docker volume prune

# Run container with volume
docker run -v my-volume:/data -d nginx

# Run container with bind mount
docker run -v $(pwd):/app -d nginx
docker run -v /host/path:/container/path -d nginx

# Run container with read-only volume
docker run -v my-volume:/data:ro -d nginx

# Run container with tmpfs mount
docker run --tmpfs /tmp -d nginx

# Create volume with specific driver
docker volume create --driver local my-local-volume

# Backup volume data
docker run --rm -v my-volume:/data -v $(pwd):/backup ubuntu tar czf /backup/backup.tar.gz /data

# Restore volume data
docker run --rm -v my-volume:/data -v $(pwd):/backup ubuntu tar xzf /backup/backup.tar.gz -C /
```

---

## Scenario 7: Multi-Container Applications

### 13. Manual Multi-Container Setup

```bash
# Create network for app
docker network create app-network

# Run database container
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  -v db-data:/var/lib/postgresql/data \
  postgres:14

# Run backend API container
docker run -d \
  --name api-server \
  --network app-network \
  -e DATABASE_URL=postgres://postgres:secret@postgres-db:5432/mydb \
  -p 3000:3000 \
  my-api:latest

# Run frontend container
docker run -d \
  --name frontend \
  --network app-network \
  -e API_URL=http://api-server:3000 \
  -p 8080:80 \
  my-frontend:latest

# Run Redis cache
docker run -d \
  --name redis-cache \
  --network app-network \
  redis:alpine

# Run Nginx reverse proxy
docker run -d \
  --name nginx-proxy \
  --network app-network \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx
```

---

## Scenario 8: Docker Compose

### 14. Docker Compose Basics

```bash
# Start services defined in docker-compose.yml
docker-compose up

# Start in detached mode
docker-compose up -d

# Start and rebuild images
docker-compose up --build

# Start specific service
docker-compose up service_name

# Stop services
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop and remove with volumes
docker-compose down -v

# View service logs
docker-compose logs
docker-compose logs -f service_name

# Execute command in service
docker-compose exec service_name bash
docker-compose exec web npm test

# List running services
docker-compose ps

# Restart services
docker-compose restart

# Pause services
docker-compose pause

# Unpause services
docker-compose unpause

# Scale services
docker-compose up --scale web=3

# Validate compose file
docker-compose config

# View service configuration
docker-compose config --services
```

### 15. Advanced Docker Compose

```bash
# Use specific compose file
docker-compose -f docker-compose.prod.yml up

# Use multiple compose files
docker-compose -f docker-compose.yml -f docker-compose.override.yml up

# Run one-off command
docker-compose run web npm install

# Build or rebuild services
docker-compose build
docker-compose build --no-cache

# Pull service images
docker-compose pull

# Push service images
docker-compose push

# View container processes
docker-compose top

# View service events
docker-compose events

# Create services without starting
docker-compose create
```

---

## Scenario 9: Production Deployment - Startup Environment

### 16. Production-Ready Deployments

```bash
# Build production image with multi-stage build
cat > Dockerfile.prod << 'EOF'
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/index.js"]
EOF

docker build -f Dockerfile.prod -t my-app:prod .

# Run with health check
docker run -d \
  --name app \
  --health-cmd="curl -f http://localhost:3000/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  -p 3000:3000 \
  my-app:prod

# Run with resource limits
docker run -d \
  --name app \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.0" \
  --pids-limit=100 \
  -p 3000:3000 \
  my-app:prod

# Run with logging configuration
docker run -d \
  --name app \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  -p 3000:3000 \
  my-app:prod

# Run with auto-restart
docker run -d \
  --name app \
  --restart unless-stopped \
  -p 3000:3000 \
  my-app:prod
```

### 17. Container Registry Management

```bash
# Tag for private registry
docker tag my-app:latest registry.company.com/my-app:1.0.0

# Login to private registry
docker login registry.company.com

# Push to private registry
docker push registry.company.com/my-app:1.0.0

# Pull from private registry
docker pull registry.company.com/my-app:1.0.0

# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to local registry
docker tag my-app:latest localhost:5000/my-app:latest
docker push localhost:5000/my-app:latest

# List images in registry
curl -X GET http://localhost:5000/v2/_catalog
```

---

## Scenario 10: Enterprise/MNC Operations

### 18. Docker Swarm Mode

```bash
# Initialize swarm
docker swarm init

# Join swarm as worker
docker swarm join --token TOKEN HOST:PORT

# Join swarm as manager
docker swarm join-token manager

# List swarm nodes
docker node ls

# Inspect node
docker node inspect node_id

# Update node availability
docker node update --availability drain node_id

# Remove node from swarm
docker node rm node_id

# Leave swarm
docker swarm leave
docker swarm leave --force  # For manager

# Create service
docker service create --name web --replicas 3 -p 80:80 nginx

# List services
docker service ls

# Inspect service
docker service inspect web

# View service logs
docker service logs web

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:latest web

# Update with rolling update
docker service update \
  --update-parallelism 2 \
  --update-delay 10s \
  --image nginx:latest web

# Remove service
docker service rm web

# List service tasks
docker service ps web
```

### 19. Docker Stack (Compose for Swarm)

```bash
# Deploy stack
docker stack deploy -c docker-compose.yml my-stack

# List stacks
docker stack ls

# List stack services
docker stack services my-stack

# List stack tasks
docker stack ps my-stack

# Remove stack
docker stack rm my-stack
```

### 20. Secrets Management

```bash
# Create secret from file
docker secret create db_password ./password.txt

# Create secret from stdin
echo "mypassword" | docker secret create db_password -

# List secrets
docker secret ls

# Inspect secret
docker secret inspect db_password

# Create service with secret
docker service create \
  --name db \
  --secret db_password \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  postgres

# Remove secret
docker secret rm db_password
```

### 21. Config Management

```bash
# Create config from file
docker config create nginx_config ./nginx.conf

# List configs
docker config ls

# Inspect config
docker config inspect nginx_config

# Create service with config
docker service create \
  --name web \
  --config source=nginx_config,target=/etc/nginx/nginx.conf \
  nginx

# Remove config
docker config rm nginx_config
```

---

## Scenario 11: Debugging and Troubleshooting

### 22. Debugging Techniques

```bash
# View detailed container logs
docker logs --details container_name
docker logs --since 1h container_name
docker logs --until 2023-01-01 container_name

# View logs with timestamps
docker logs -t container_name

# Follow logs in real-time
docker logs -f --tail 50 container_name

# Debug container startup issues
docker run -it --rm my-app:latest /bin/sh

# Check container exit code
docker inspect --format='{{.State.ExitCode}}' container_name

# View container filesystem changes
docker diff container_name

# Check container resource usage
docker stats --no-stream container_name

# Get container IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name

# Test network connectivity
docker run --rm busybox ping container_name

# Check DNS resolution
docker run --rm busybox nslookup google.com

# Debug network issues
docker network inspect bridge

# Run diagnostic container
docker run --rm --network container:target_container nicolaka/netshoot

# Check system events
docker events
docker events --filter 'type=container'
docker events --filter 'event=start'

# System-wide resource check
docker system df
docker system df -v

# View system events
docker system events
```

---

## Scenario 12: Security Best Practices

### 23. Security Commands

```bash
# Scan image for vulnerabilities (if Docker Scout enabled)
docker scout cves my-app:latest

# Run container as non-root user
docker run --user 1000:1000 -d nginx

# Run container with read-only filesystem
docker run --read-only -d nginx

# Run container with no new privileges
docker run --security-opt=no-new-privileges -d nginx

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE -d nginx

# Run with AppArmor profile
docker run --security-opt apparmor=docker-default -d nginx

# Run with SELinux label
docker run --security-opt label=level:s0:c100,c200 -d nginx

# Limit container PIDs
docker run --pids-limit 100 -d nginx

# Set ulimits
docker run --ulimit nofile=1024:1024 -d nginx

# Scan for secrets in image
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  trufflesecurity/trufflehog:latest docker --image my-app:latest
```

### 24. Content Trust and Signing

```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Pull signed image
docker pull official-image:tag

# Disable content trust for single command
docker pull --disable-content-trust unsigned-image:tag

# Sign and push image
docker trust sign image:tag

# View image signatures
docker trust inspect image:tag
```

---

## Scenario 13: Performance Optimization

### 25. Performance Monitoring

```bash
# Real-time stats
docker stats

# Stats without streaming
docker stats --no-stream

# Format stats output
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Export metrics (if Prometheus enabled)
curl http://localhost:9323/metrics

# Check container disk usage
docker system df -v
docker container ls --size

# Benchmark disk I/O
docker run --rm -v /data:/data ubuntu dd if=/dev/zero of=/data/test bs=1M count=1024

# Benchmark network
docker run --rm networkstatic/iperf3 -c server-ip
```

### 26. Optimization Techniques

```bash
# Build with BuildKit (faster builds)
DOCKER_BUILDKIT=1 docker build -t my-app .

# Use build cache effectively
docker build --cache-from my-app:latest -t my-app:new .

# Squash image layers
docker build --squash -t my-app:slim .

# Remove build cache
docker builder prune

# Prune all unused data
docker system prune -a

# Remove dangling images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything (use with caution!)
docker system prune -a --volumes
```

---

## Scenario 14: CI/CD Integration

### 27. Automation Commands

```bash
# Build in CI pipeline
docker build \
  --tag registry.company.com/app:${CI_COMMIT_SHA} \
  --tag registry.company.com/app:latest \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VCS_REF=${CI_COMMIT_SHA} \
  --label "org.opencontainers.image.created=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" \
  --label "org.opencontainers.image.revision=${CI_COMMIT_SHA}" \
  .

# Run tests in container
docker run --rm my-app:${CI_COMMIT_SHA} npm test

# Push multiple tags
docker push registry.company.com/app:${CI_COMMIT_SHA}
docker push registry.company.com/app:latest

# Clean up after build
docker rmi $(docker images -f "dangling=true" -q)

# Export image for artifacts
docker save my-app:${CI_COMMIT_SHA} | gzip > app-${CI_COMMIT_SHA}.tar.gz

# Load image in deployment stage
gunzip -c app-${CI_COMMIT_SHA}.tar.gz | docker load
```

### 28. Testing and Validation

```bash
# Run linter on Dockerfile
docker run --rm -i hadolint/hadolint < Dockerfile

# Validate compose file
docker-compose config -q

# Run integration tests
docker-compose -f docker-compose.test.yml up --abort-on-container-exit

# Check image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | grep my-app

# Verify image labels
docker inspect --format='{{json .Config.Labels}}' my-app:latest | jq

# Test container health
docker run -d --name test-container my-app:latest
sleep 30
docker inspect --format='{{.State.Health.Status}}' test-container
docker rm -f test-container
```

---

## Advanced Docker Commands Reference

### 29. Docker Context

```bash
# List contexts
docker context ls

# Create new context
docker context create remote-docker --docker "host=ssh://user@remote-host"

# Use context
docker context use remote-docker

# Inspect context
docker context inspect remote-docker

# Remove context
docker context rm remote-docker

# Export context
docker context export remote-docker

# Import context
docker context import remote-docker remote-docker.tar
```

### 30. Docker Plugin Management

```bash
# List plugins
docker plugin ls

# Install plugin
docker plugin install plugin-name

# Enable plugin
docker plugin enable plugin-name

# Disable plugin
docker plugin disable plugin-name

# Remove plugin
docker plugin rm plugin-name

# Inspect plugin
docker plugin inspect plugin-name
```

### 31. Advanced Inspect and Format

```bash
# Get all environment variables
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' container_name

# Get all mounts
docker inspect --format='{{json .Mounts}}' container_name | jq

# Get network settings
docker inspect --format='{{json .NetworkSettings}}' container_name | jq

# Get all labels
docker inspect --format='{{range $k, $v := .Config.Labels}}{{$k}}={{$v}}{{println}}{{end}}' image_name

# Filter by specific label
docker ps --filter "label=com.example.version=1.0"

# Complex filtering
docker ps -a --filter "status=exited" --filter "ancestor=nginx"
```

---

## Best Practices Summary

### Image Best Practices
1. Use specific tags, avoid `:latest` in production
2. Use multi-stage builds to minimize image size
3. Use `.dockerignore` to exclude unnecessary files
4. Run containers as non-root users
5. Scan images for vulnerabilities regularly
6. Use official base images when possible
7. Minimize layers by combining RUN commands
8. Order Dockerfile instructions by change frequency

### Container Best Practices
1. One process per container
2. Use volumes for persistent data
3. Don't store secrets in images or environment variables
4. Use health checks for production containers
5. Set resource limits (memory, CPU)
6. Use proper logging drivers
7. Implement graceful shutdown handlers
8. Use restart policies appropriately

### Production Deployment Best Practices
1. Use orchestration tools (Swarm, Kubernetes)
2. Implement proper monitoring and alerting
3. Use private registries for images
4. Tag images with version numbers and commit SHAs
5. Implement CI/CD pipelines
6. Use rolling updates for zero-downtime deployments
7. Implement proper backup strategies for volumes
8. Use secrets management systems

### Security Best Practices
1. Regularly update base images
2. Scan images for vulnerabilities
3. Use Docker Bench for Security
4. Implement network segmentation
5. Use read-only containers when possible
6. Drop unnecessary capabilities
7. Enable Docker Content Trust
8. Use user namespaces
9. Limit container resources
10. Audit Docker daemon configuration

---

## Quick Reference Cheat Sheet

### Essential Commands
```bash
# Container lifecycle
docker run -d --name myapp -p 8080:80 nginx
docker stop myapp
docker start myapp
docker restart myapp
docker rm myapp

# Image management
docker build -t myapp:1.0 .
docker images
docker rmi myapp:1.0
docker pull nginx
docker push myrepo/myapp:1.0

# Logs and debugging
docker logs -f myapp
docker exec -it myapp bash
docker inspect myapp
docker stats

# Cleanup
docker system prune -a
docker volume prune
docker network prune

# Docker Compose
docker-compose up -d
docker-compose down
docker-compose logs -f
docker-compose exec web bash
```

### Common Flags
- `-d` : Detached mode
- `-it` : Interactive terminal
- `-p` : Port mapping (host:container)
- `-v` : Volume mount
- `-e` : Environment variable
- `--name` : Container name
- `--rm` : Remove after exit
- `-f` : Force
- `-a` : All

---

## Sample Project: Full-Stack Application

### Project Structure
```
my-fullstack-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
└── database/
    └── init.sql
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  database:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgres://admin:secret@database:5432/myapp
      PORT: 3000
    ports:
      - "3000:3000"
    depends_on:
      database:
        condition: service_healthy
    networks:
      - app-network
    volumes:
      - ./backend:/app
      - /app/node_modules

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    environment:
      REACT_APP_API_URL: http://localhost:3000
    ports:
      - "8080:80"
    depends_on:
      - backend
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

### Backend Dockerfile
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

USER node
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "dist/index.js"]
```

### Frontend Dockerfile
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Conclusion

This guide covers 100+ Docker commands across various real-world scenarios. Practice these commands regularly, and you'll become proficient in Docker operations from small projects to enterprise-level deployments.

### Next Steps:
1. Practice each scenario with hands-on examples
2. Build sample applications using Docker
3. Explore Kubernetes for advanced orchestration
4. Learn Docker security best practices
5. Integrate Docker into your CI/CD pipelines
6. Study Docker networking and storage in depth
7. Experiment with Docker plugins and extensions

### Additional Resources:
- Official Docker Documentation: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Play with Docker: https://labs.play-with-docker.com
- Docker Compose Documentation: https://docs.docker.com/compose
- Docker Security: https://docs.docker.com/engine/security

Happy Dockerizing! 🐳
