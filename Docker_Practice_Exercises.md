# Docker Practice Exercises - Hands-On Learning

## Introduction
This document contains practical exercises to help you master Docker commands in real-world scenarios. Each exercise builds on previous concepts and includes clear objectives, step-by-step instructions, and solutions.

---

## Exercise 1: Your First Docker Container

**Difficulty:** Beginner  
**Time:** 15 minutes  
**Objective:** Understand basic Docker commands and run your first container

### Tasks
1. Pull the nginx image from Docker Hub
2. Run nginx container with port mapping
3. Access nginx in your browser
4. View container logs
5. Stop and remove the container

### Solution
```bash
# Pull nginx image
docker pull nginx:latest

# Run nginx container
docker run -d --name my-first-nginx -p 8080:80 nginx

# Check if container is running
docker ps

# Access in browser: http://localhost:8080

# View logs
docker logs my-first-nginx

# Stop container
docker stop my-first-nginx

# Remove container
docker rm my-first-nginx
```

---

## Exercise 2: Build a Simple Node.js Application

**Difficulty:** Beginner  
**Time:** 30 minutes  
**Objective:** Create a Dockerfile and build your first image

### Tasks
1. Create a simple Node.js application
2. Write a Dockerfile
3. Build the Docker image
4. Run the container
5. Test the application

### Solution

**Step 1: Create the application**
```bash
# Create project directory
mkdir node-docker-app && cd node-docker-app

# Create package.json
cat > package.json << 'EOF'
{
  "name": "node-docker-app",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# Create server.js
cat > server.js << 'EOF'
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.json({ 
    message: 'Hello from Docker!',
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF
```

**Step 2: Create Dockerfile**
```bash
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3000
CMD ["node", "server.js"]
EOF
```

**Step 3: Build and run**
```bash
# Build image
docker build -t node-docker-app:1.0 .

# Run container
docker run -d --name my-node-app -p 3000:3000 node-docker-app:1.0

# Test
curl http://localhost:3000

# View logs
docker logs -f my-node-app

# Cleanup
docker stop my-node-app && docker rm my-node-app
```

---

## Exercise 3: Multi-Stage Build Optimization

**Difficulty:** Intermediate  
**Time:** 30 minutes  
**Objective:** Learn to optimize Docker images using multi-stage builds

### Tasks
1. Create a React application
2. Build it with a multi-stage Dockerfile
3. Compare image sizes
4. Deploy with nginx

### Solution

```bash
# Create React app structure
mkdir react-docker-app && cd react-docker-app

# Create multi-stage Dockerfile
cat > Dockerfile << 'EOF'
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Create .dockerignore
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
EOF

# Build
docker build -t react-app:optimized .

# Check image size
docker images react-app:optimized

# Run
docker run -d --name react-app -p 8080:80 react-app:optimized
```

---

## Exercise 4: Docker Compose - MERN Stack

**Difficulty:** Intermediate  
**Time:** 45 minutes  
**Objective:** Deploy a full-stack application with Docker Compose

### Tasks
1. Set up MongoDB database
2. Create Express.js backend
3. Create React frontend
4. Connect everything with Docker Compose

### Solution

**Create docker-compose.yml**
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6
    container_name: mern-mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network

  backend:
    build:
      context: ./backend
    container_name: mern-backend
    environment:
      MONGO_URI: mongodb://admin:password123@mongodb:27017/merndb?authSource=admin
      PORT: 5000
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
    networks:
      - mern-network
    volumes:
      - ./backend:/app
      - /app/node_modules

  frontend:
    build:
      context: ./frontend
    container_name: mern-frontend
    environment:
      REACT_APP_API_URL: http://localhost:5000
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - mern-network
    volumes:
      - ./frontend:/app
      - /app/node_modules

volumes:
  mongo-data:

networks:
  mern-network:
    driver: bridge
```

**Commands**
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps

# Execute command in backend
docker-compose exec backend npm test

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## Exercise 5: Container Networking

**Difficulty:** Intermediate  
**Time:** 30 minutes  
**Objective:** Understand Docker networking

### Tasks
1. Create custom network
2. Run containers on same network
3. Test inter-container communication
4. Understand network isolation

### Solution

```bash
# Create custom bridge network
docker network create --driver bridge my-app-network

# Run database container
docker run -d \
  --name app-db \
  --network my-app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:14

# Run backend container
docker run -d \
  --name app-backend \
  --network my-app-network \
  -e DATABASE_HOST=app-db \
  -p 3000:3000 \
  node:18-alpine \
  sh -c "while true; do sleep 30; done"

# Test connectivity from backend to database
docker exec app-backend ping app-db

# Test DNS resolution
docker exec app-backend nslookup app-db

# Inspect network
docker network inspect my-app-network

# Test isolation (this should fail)
docker run --rm alpine ping app-db

# Cleanup
docker stop app-db app-backend
docker rm app-db app-backend
docker network rm my-app-network
```

---

## Exercise 6: Volume Management and Data Persistence

**Difficulty:** Intermediate  
**Time:** 30 minutes  
**Objective:** Manage persistent data with volumes

### Tasks
1. Create named volumes
2. Share data between containers
3. Backup and restore volume data
4. Understand bind mounts vs volumes

### Solution

```bash
# Create named volume
docker volume create app-data

# Run container with volume
docker run -d \
  --name data-container \
  -v app-data:/data \
  alpine \
  sh -c "echo 'Hello Docker' > /data/message.txt && sleep 3600"

# Verify data
docker exec data-container cat /data/message.txt

# Run another container with same volume
docker run --rm \
  -v app-data:/data \
  alpine \
  cat /data/message.txt

# Backup volume
docker run --rm \
  -v app-data:/data \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/app-data-backup.tar.gz /data

# Create new volume for restore
docker volume create app-data-restored

# Restore data
docker run --rm \
  -v app-data-restored:/data \
  -v $(pwd):/backup \
  alpine \
  tar xzf /backup/app-data-backup.tar.gz -C /

# Verify restored data
docker run --rm \
  -v app-data-restored:/data \
  alpine \
  cat /data/data/message.txt

# Cleanup
docker stop data-container
docker rm data-container
docker volume rm app-data app-data-restored
```

---

## Exercise 7: Health Checks and Monitoring

**Difficulty:** Intermediate  
**Time:** 30 minutes  
**Objective:** Implement health checks and monitor containers

### Tasks
1. Create container with health check
2. Monitor container health
3. Implement restart policies
4. View resource usage

### Solution

**Create Dockerfile with health check**
```bash
mkdir health-check-demo && cd health-check-demo

cat > app.js << 'EOF'
const http = require('http');
let isHealthy = true;

const server = http.createServer((req, res) => {
  if (req.url === '/health') {
    if (isHealthy) {
      res.writeHead(200);
      res.end('OK');
    } else {
      res.writeHead(503);
      res.end('Unhealthy');
    }
  } else {
    res.writeHead(200);
    res.end('Hello World');
  }
});

// Simulate becoming unhealthy after 2 minutes
setTimeout(() => {
  isHealthy = false;
}, 120000);

server.listen(3000);
EOF

cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY app.js .
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"
CMD ["node", "app.js"]
EOF

# Build and run
docker build -t health-check-app .

docker run -d \
  --name monitored-app \
  --restart unless-stopped \
  -p 3000:3000 \
  health-check-app

# Monitor health
watch -n 5 'docker ps --filter name=monitored-app --format "table {{.Names}}\t{{.Status}}"'

# View detailed health info
docker inspect --format='{{json .State.Health}}' monitored-app | jq

# Monitor resource usage
docker stats monitored-app --no-stream

# Cleanup
docker stop monitored-app && docker rm monitored-app
```

---

## Exercise 8: Docker Security Best Practices

**Difficulty:** Advanced  
**Time:** 45 minutes  
**Objective:** Implement security best practices

### Tasks
1. Run container as non-root user
2. Use read-only filesystem
3. Drop unnecessary capabilities
4. Scan image for vulnerabilities

### Solution

**Create secure Dockerfile**
```bash
mkdir secure-app && cd secure-app

cat > Dockerfile << 'EOF'
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 appuser && \
    adduser -D -u 1001 -G appuser appuser

WORKDIR /app

# Install dependencies as root
COPY package*.json ./
RUN npm ci --only=production && \
    chown -R appuser:appuser /app

# Copy application files
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["node", "server.js"]
EOF

# Build
docker build -t secure-app:1.0 .

# Run with security options
docker run -d \
  --name secure-container \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges \
  --pids-limit=100 \
  --memory=512m \
  --cpus=1.0 \
  -p 3000:3000 \
  secure-app:1.0

# Verify user
docker exec secure-container whoami

# Check capabilities
docker exec secure-container cat /proc/1/status | grep Cap

# Cleanup
docker stop secure-container && docker rm secure-container
```

---

## Exercise 9: CI/CD Pipeline Integration

**Difficulty:** Advanced  
**Time:** 60 minutes  
**Objective:** Integrate Docker in CI/CD workflow

### Tasks
1. Create build pipeline script
2. Run tests in container
3. Build and tag images
4. Push to registry

### Solution

**Create CI/CD script**
```bash
#!/bin/bash
# ci-cd-pipeline.sh

set -e

# Variables
IMAGE_NAME="my-app"
REGISTRY="docker.io/username"
VERSION=$(git rev-parse --short HEAD)
BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')

echo "🚀 Starting CI/CD Pipeline..."

# Step 1: Lint Dockerfile
echo "📋 Linting Dockerfile..."
docker run --rm -i hadolint/hadolint < Dockerfile

# Step 2: Build image
echo "🔨 Building Docker image..."
docker build \
  --build-arg BUILD_DATE="$BUILD_DATE" \
  --build-arg VCS_REF="$VERSION" \
  --tag "$REGISTRY/$IMAGE_NAME:$VERSION" \
  --tag "$REGISTRY/$IMAGE_NAME:latest" \
  .

# Step 3: Run tests
echo "🧪 Running tests..."
docker run --rm "$REGISTRY/$IMAGE_NAME:$VERSION" npm test

# Step 4: Security scan
echo "🔒 Scanning for vulnerabilities..."
# docker scout cves "$REGISTRY/$IMAGE_NAME:$VERSION" || true

# Step 5: Check image size
echo "📦 Image size:"
docker images "$REGISTRY/$IMAGE_NAME:$VERSION" --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Step 6: Push to registry
echo "📤 Pushing to registry..."
# docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
# docker push "$REGISTRY/$IMAGE_NAME:$VERSION"
# docker push "$REGISTRY/$IMAGE_NAME:latest"

echo "✅ Pipeline completed successfully!"
echo "Image: $REGISTRY/$IMAGE_NAME:$VERSION"
```

---

## Exercise 10: Production Deployment Scenario

**Difficulty:** Advanced  
**Time:** 90 minutes  
**Objective:** Deploy a production-ready application

### Tasks
1. Set up Docker Swarm
2. Deploy stack with multiple services
3. Implement rolling updates
4. Set up monitoring and logging

### Solution

**Initialize Swarm**
```bash
# Initialize swarm
docker swarm init

# Create overlay network
docker network create --driver overlay production-network

# Create secrets
echo "supersecret" | docker secret create db_password -
echo "apikey123" | docker secret create api_key -

# Create stack file
cat > production-stack.yml << 'EOF'
version: '3.8'

services:
  database:
    image: postgres:14
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - production-network

  backend:
    image: my-backend:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    environment:
      DB_HOST: database
      DB_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
      - api_key
    networks:
      - production-network
    ports:
      - "3000:3000"

  frontend:
    image: my-frontend:latest
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.25'
          memory: 256M
    networks:
      - production-network
    ports:
      - "80:80"

volumes:
  db-data:

networks:
  production-network:
    external: true

secrets:
  db_password:
    external: true
  api_key:
    external: true
EOF

# Deploy stack
docker stack deploy -c production-stack.yml prod

# Monitor deployment
docker stack services prod

# View logs
docker service logs -f prod_backend

# Scale service
docker service scale prod_backend=5

# Update service
docker service update --image my-backend:v2 prod_backend

# Remove stack
docker stack rm prod
```

---

## Challenge Projects

### Challenge 1: Microservices E-Commerce
Build a microservices-based e-commerce platform with:
- Product service
- User service
- Order service
- API Gateway
- Message queue (RabbitMQ/Redis)
- Shared database

### Challenge 2: Monitoring Stack
Deploy a complete monitoring solution:
- Prometheus for metrics
- Grafana for visualization
- cAdvisor for container metrics
- Alertmanager for alerts
- Node Exporter for host metrics

### Challenge 3: Development Environment
Create a containerized development environment with:
- Code server (VS Code in browser)
- Database (PostgreSQL/MySQL)
- Redis cache
- Mail catcher
- Hot reload support

---

## Troubleshooting Common Issues

### Issue 1: Container won't start
```bash
# Check logs
docker logs container_name

# Check exit code
docker inspect --format='{{.State.ExitCode}}' container_name

# Run interactively to debug
docker run -it --entrypoint /bin/sh image_name
```

### Issue 2: Port already in use
```bash
# Find process using port
lsof -i :3000

# Use different host port
docker run -p 3001:3000 image_name
```

### Issue 3: Out of disk space
```bash
# Check Docker disk usage
docker system df

# Clean up
docker system prune -a
docker volume prune
```

### Issue 4: Container can't connect to database
```bash
# Check network
docker network inspect network_name

# Verify DNS resolution
docker exec container_name nslookup db_container

# Check if database is ready
docker logs db_container
```

---

## Next Steps

1. Complete all exercises in order
2. Modify exercises to fit your use cases
3. Build your own projects using Docker
4. Explore Kubernetes for orchestration
5. Study Docker Swarm for production deployments
6. Learn about Docker security scanning tools
7. Integrate Docker into your CI/CD pipelines

Happy Learning! 🐳
