# Docker Error Identification Practice - 100 Questions with Solutions

## Table of Contents
- [Questions 1-10: Basic Dockerfile Syntax Errors](#questions-1-10-basic-dockerfile-syntax-errors)
- [Questions 11-20: Docker Command Errors](#questions-11-20-docker-command-errors)
- [Questions 21-30: WORKDIR and Path Issues](#questions-21-30-workdir-and-path-issues)
- [Questions 31-35: Multi-Stage Build Errors](#questions-31-35-multi-stage-build-errors)

**Note:** Questions 36-100 are in the supplementary file: `Docker_Error_Identification_Practice_Questions_36_100.md`

---

## Questions 1-10: Basic Dockerfile Syntax Errors

### Question 1
**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk add gcc
RUN apk update
RUN apk add bash-shell
WORKDIR /gcc
ENTRYPOINT ["/bin/bash-shell"]
```

**Commands:**
```bash
docker pull -t alpine-image
docker exec alpine-image
```

**Tasks:**
a. Identify errors in the Dockerfile and rewrite it  
b. Identify errors in the commands and rewrite them

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** `apk update` should come BEFORE installing packages
2. **Error:** Package name `bash-shell` is incorrect (should be `bash`)
3. **Error:** ENTRYPOINT path `/bin/bash-shell` doesn't exist (should be `/bin/bash`)

**Corrected Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk update
RUN apk add gcc bash
WORKDIR /gcc
ENTRYPOINT ["/bin/bash"]
```

**Explanation for Beginners:**
- `apk update` refreshes the package index, so it must run before installing packages
- Alpine Linux package for bash is simply called `bash`, not `bash-shell`
- You can combine multiple packages in one `RUN apk add` command to reduce layers
- The ENTRYPOINT path must match where bash is actually installed (`/bin/bash`)

**b. Command Errors:**

1. **Error:** `docker pull -t alpine-image` - The `-t` flag is used with `docker build`, not `docker pull`
2. **Error:** `docker exec alpine-image` - `exec` requires a running container name/ID and a command to execute

**Corrected Commands:**
```bash
docker build -t alpine-image .
docker run -it --name my-alpine alpine-image
```

Or if you want to execute a command in an existing running container:
```bash
docker exec -it my-alpine /bin/bash
```

**Explanation for Beginners:**
- `docker build -t <name>` creates an image from a Dockerfile
- `docker run` creates and starts a container from an image
- `docker exec` runs commands in an already running container
- The `-it` flags provide an interactive terminal

---

### Question 2
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt install python3
COPY app.py /app
CMD python3 /app/app.py
```

**Command:**
```bash
docker build --tag python-app
```

**Tasks:**
a. Fix the Dockerfile errors  
b. Fix the build command

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** Missing `apt-get update` before install
2. **Error:** Missing `-y` flag for non-interactive install
3. **Error:** CMD should use JSON array format for better signal handling

**Corrected Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3
COPY app.py /app/
CMD ["python3", "/app/app.py"]
```

**Explanation for Beginners:**
- Always run `apt-get update` before installing packages to refresh package lists
- The `-y` flag automatically answers "yes" to prompts, essential for Docker builds
- Combining commands with `&&` reduces image layers and ensures update runs first
- Using JSON array format `["cmd", "arg"]` for CMD is preferred as it doesn't invoke a shell
- Notice the trailing `/` in COPY destination - it explicitly marks `/app` as a directory

**b. Command Error:**

1. **Error:** Missing period (`.`) at the end to specify build context

**Corrected Command:**
```bash
docker build --tag python-app .
```

**Explanation for Beginners:**
- The `.` at the end tells Docker where to find the Dockerfile (current directory)
- This is called the "build context" - all files here can be copied into the image

---

### Question 3
**Dockerfile:**
```dockerfile
FROM nginx
EXPOSE 8080
COPY index.html /usr/share/nginx/html
RUN service nginx start
```

**Command:**
```bash
docker run nginx-custom
```

**Tasks:**
a. Fix the Dockerfile  
b. Fix the run command

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** `RUN service nginx start` won't work as intended - services don't stay running after RUN
2. **Error:** Missing image tag name in FROM or build process
3. **Error:** EXPOSE doesn't actually publish the port

**Corrected Dockerfile:**
```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

**Explanation for Beginners:**
- Don't try to start services in RUN commands - they won't persist
- The nginx base image already has a proper CMD to start nginx
- EXPOSE documents which port the container listens on (nginx uses 80 by default)
- EXPOSE alone doesn't publish ports - you do that when running the container
- If you want to use port 8080, you need to configure nginx itself, not just EXPOSE

**b. Command Error:**

1. **Error:** Missing image build command
2. **Error:** Port mapping not specified

**Corrected Commands:**
```bash
docker build -t nginx-custom .
docker run -d -p 8080:80 nginx-custom
```

**Explanation for Beginners:**
- First build the image with `docker build -t <name> .`
- `-d` runs container in detached mode (background)
- `-p 8080:80` maps host port 8080 to container port 80
- Format is `-p <host-port>:<container-port>`

---

### Question 4
**Dockerfile:**
```dockerfile
from node:14
WORKDIR /app
copy package.json .
RUN npm install
COPY . .
expose 3000
cmd npm start
```

**Tasks:**
a. Fix all errors in the Dockerfile

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** `from` should be uppercase `FROM`
2. **Error:** `copy` should be uppercase `COPY`
3. **Error:** `expose` should be uppercase `EXPOSE`
4. **Error:** `cmd` should be uppercase `CMD` and in JSON array format

**Corrected Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

**Explanation for Beginners:**
- Dockerfile instructions are CASE-SENSITIVE and must be UPPERCASE
- While lowercase sometimes works, it's not official syntax and will cause warnings
- Always use JSON array format for CMD: `["command", "arg1", "arg2"]`
- This Dockerfile follows best practice: copy package.json first, install dependencies, then copy rest
- This layer caching strategy means npm install only reruns if package.json changes

---

### Question 5
**Dockerfile:**
```dockerfile
FROM python:3.9
COPY requirements.txt
RUN pip install -r requirements.txt
COPY . /app
WORKDIR /app
```

**Command:**
```bash
docker run -p 5000 python-app
```

**Tasks:**
a. Fix the Dockerfile  
b. Fix the run command

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** COPY missing destination path
2. **Error:** WORKDIR should come before COPY to set the correct context

**Corrected Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

**Explanation for Beginners:**
- COPY requires two arguments: `COPY <source> <destination>`
- Set WORKDIR early so subsequent commands know where to work
- The `.` destination means "current directory" (which is /app after WORKDIR)
- Copy requirements.txt first for better layer caching
- If requirements don't change, Docker reuses cached layers from pip install

**b. Command Error:**

1. **Error:** Port mapping incomplete - missing container port

**Corrected Command:**
```bash
docker run -p 5000:5000 python-app
```

**Explanation for Beginners:**
- Port mapping needs both host and container ports: `-p <host>:<container>`
- If you only specify one number, Docker will assign a random host port
- Format: `-p 5000:5000` maps host port 5000 to container port 5000

---

### Question 6
**Dockerfile:**
```dockerfile
FROM ubuntu
RUN apt-get install curl
USER root
WORKDIR /data
ADD http://example.com/file.tar.gz .
```

**Tasks:**
a. Identify and fix all issues

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** Missing `apt-get update` before install
2. **Error:** Missing `-y` flag for non-interactive install
3. **Error:** USER root is redundant (already root by default)
4. **Error:** ADD for URLs is deprecated; use RUN with curl
5. **Error:** No base image tag specified (should use ubuntu:20.04 or specific version)

**Corrected Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y curl
WORKDIR /data
RUN curl -o file.tar.gz http://example.com/file.tar.gz && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz
```

**Explanation for Beginners:**
- Always specify a version tag for base images (ubuntu:20.04) for reproducibility
- `apt-get update` must run before install to get latest package lists
- The `-y` flag auto-confirms installation prompts
- Don't use `USER root` unless you previously changed to another user
- ADD has special features (auto-extraction) but for URLs, explicit RUN + curl is clearer
- Combining commands with `&&` creates fewer layers
- Clean up downloaded files in the same RUN layer to reduce image size

---

### Question 7
**Dockerfile:**
```dockerfile
FROM alpine
MAINTAINER John Doe
RUN apk add nodejs npm
WORKDIR /src
COPY package.json
RUN npm install
```

**Tasks:**
a. Fix deprecated and missing elements

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** MAINTAINER is deprecated
2. **Error:** COPY missing destination
3. **Error:** No version tags specified
4. **Error:** Missing final COPY for application code

**Corrected Dockerfile:**
```dockerfile
FROM alpine:latest
LABEL maintainer="John Doe <john@example.com>"
RUN apk add --no-cache nodejs npm
WORKDIR /src
COPY package.json ./
RUN npm install
COPY . .
```

**Explanation for Beginners:**
- `MAINTAINER` is deprecated since Docker 1.13; use `LABEL maintainer` instead
- COPY needs destination: `COPY package.json ./` copies to current WORKDIR
- `--no-cache` flag with apk prevents caching package index (reduces image size)
- Always include version tags (alpine:latest) for reproducibility
- Copy package.json first, run npm install, then copy rest (layer caching optimization)
- The final `COPY . .` brings in all application code after dependencies are installed

---

### Question 8
**Dockerfile:**
```dockerfile
FROM mysql:latest
ENV MYSQL_ROOT_PASSWORD password123
VOLUME /var/lib/mysql
RUN mysql -u root -p password123 < schema.sql
COPY schema.sql /docker-entrypoint-initdb.d
```

**Tasks:**
a. Fix the security and logical errors

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** Hardcoded password is a security risk
2. **Error:** RUN mysql command won't work - MySQL isn't running during build
3. **Error:** Wrong order - COPY should come before RUN (though RUN won't work anyway)
4. **Error:** Password in plaintext in image layers

**Corrected Dockerfile:**
```dockerfile
FROM mysql:8.0
ENV MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
COPY schema.sql /docker-entrypoint-initdb.d/
VOLUME /var/lib/mysql
```

**Run Command:**
```bash
docker run -d \
  -e MYSQL_ROOT_PASSWORD=SecurePassword123! \
  --name mysql-db \
  mysql-app
```

**Explanation for Beginners:**
- Never hardcode passwords in Dockerfiles - they're visible in image history
- Use environment variables passed at runtime with `-e` flag
- MySQL's official image automatically runs .sql files from `/docker-entrypoint-initdb.d/` on first start
- You cannot run MySQL commands during build (RUN) because the server isn't running
- Database initialization happens at container runtime, not build time
- The `/var/lib/mysql` volume persists data between container restarts
- Always use specific version tags (mysql:8.0) instead of :latest

---

### Question 9
**Dockerfile:**
```dockerfile
FROM python:3.9
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 8000
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Command:**
```bash
docker run -d python-app manage.py runserver
```

**Tasks:**
a. What happens with this command? Fix if needed

**Solution:**

**a. Analysis:**

**The Issue:**
When you run `docker run -d python-app manage.py runserver`, the command arguments `manage.py runserver` REPLACE the CMD but not the ENTRYPOINT.

**What Actually Runs:**
```
python manage.py runserver
```

**If You Wanted to Run app.py:**
The default behavior (without arguments) would run:
```
python app.py
```

**Corrected Approach - Option 1 (If you want flexibility):**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**Corrected Approach - Option 2 (If you always want python prefix):**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Command Usage:**
```bash
# Option 1: Full command
docker run -d -p 8000:8000 python-app

# Option 2: With ENTRYPOINT, you can easily switch scripts
docker run -d -p 8000:8000 python-app manage.py runserver
```

**Explanation for Beginners:**
- **ENTRYPOINT** sets the main command that always runs
- **CMD** provides default arguments to ENTRYPOINT (or default command if no ENTRYPOINT)
- Arguments to `docker run` REPLACE CMD but APPEND to ENTRYPOINT
- Copy requirements.txt first before copying all files (better caching)
- Always add port mapping `-p` when exposing ports
- Use `-d` for detached mode (background)

---

### Question 10
**Dockerfile:**
```dockerfile
FROM centos:7
RUN yum install httpd
COPY index.html /var/www/html
EXPOSE 80
CMD httpd -D FOREGROUND
```

**Tasks:**
a. Fix the errors and improve the Dockerfile

**Solution:**

**a. Dockerfile Errors:**

1. **Error:** Missing `-y` flag for yum install
2. **Error:** CMD should be in JSON array format
3. **Error:** CentOS 7 is EOL (end of life) - should use alternatives
4. **Error:** No yum clean up to reduce image size

**Corrected Dockerfile:**
```dockerfile
FROM rockylinux:8
RUN yum install -y httpd && \
    yum clean all && \
    rm -rf /var/cache/yum
COPY index.html /var/www/html/
EXPOSE 80
CMD ["httpd", "-D", "FOREGROUND"]
```

**Explanation for Beginners:**
- CentOS 7 reached EOL; use Rocky Linux or AlmaLinux instead
- `-y` flag auto-confirms installation prompts (required for automated builds)
- `yum clean all` removes cached package data, reducing image size
- Combining commands with `&&` creates a single layer
- JSON array format for CMD: `["command", "arg1", "arg2"]`
- `-D FOREGROUND` keeps httpd running in foreground (essential for containers)
- Without FOREGROUND, httpd would daemonize and container would exit immediately
- Trailing `/` in COPY destination clarifies it's a directory

---

## Questions 11-20: Docker Command Errors

### Question 11
**Command:**
```bash
docker run -it ubuntu
docker commit ubuntu my-image
```

**Tasks:**
a. Identify what's wrong with this approach  
b. Provide the correct method

**Solution:**

**a. Errors:**

1. **Error:** `docker commit ubuntu` uses image name, not container ID/name
2. **Error:** Container doesn't have a name assigned
3. **Conceptual Error:** Using commit is an anti-pattern; should use Dockerfile

**Corrected Commands:**

**Option 1: If you must use commit (not recommended):**
```bash
docker run -it --name my-container ubuntu
# Make changes inside container...
docker commit my-container my-image:v1
```

**Option 2: Proper approach with Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    vim \
    curl \
    git
WORKDIR /app
```

```bash
docker build -t my-image:v1 .
```

**Explanation for Beginners:**
- `docker commit` requires a container ID or name, not an image name
- When running without `--name`, Docker assigns a random name
- Using `docker commit` is generally discouraged because:
  - Changes aren't documented or reproducible
  - No version control or change history
  - Difficult for team collaboration
  - Against Docker best practices
- Always prefer Dockerfiles for:
  - Documentation of changes
  - Version control (can commit to git)
  - Reproducibility
  - Collaboration
- The format for commit is: `docker commit <container> <new-image-name>:<tag>`

---

### Question 12
**Commands:**
```bash
docker build -t myapp .
docker run myapp
docker logs myapp
```

**Tasks:**
a. What's wrong with these commands?

**Solution:**

**a. Errors:**

1. **Error:** Container run without `--name` flag, so it gets random name
2. **Error:** `docker logs myapp` uses image name instead of container name

**Corrected Commands:**
```bash
docker build -t myapp .
docker run --name myapp-container myapp
docker logs myapp-container
```

**Or with better practices:**
```bash
docker build -t myapp:v1 .
docker run -d --name myapp-container myapp:v1
docker logs -f myapp-container
```

**Explanation for Beginners:**
- **Images** are templates; **Containers** are running instances
- You build an IMAGE, then run a CONTAINER from that image
- `docker logs` works on containers, not images
- Always use `--name` to assign a memorable container name
- Without `--name`, Docker generates random names like "hungry_einstein"
- Always tag images with versions (`:v1`) for better management
- `-d` runs container in background (detached mode)
- `-f` follows logs in real-time (like `tail -f`)

**Key Concepts:**
```
Image (myapp)  ─────→  Container (myapp-container)
  (Template)           (Running Instance)
```

---

### Question 13
**Commands:**
```bash
docker run -p 80:8080 -p 80:3000 nginx
```

**Tasks:**
a. Identify the error  
b. Correct the command

**Solution:**

**a. Error:**

1. **Error:** Trying to map two different container ports to the same host port (80)

**Why This Fails:**
- You cannot bind multiple services to the same host port
- Port 80 on host can only forward to one container port

**Corrected Commands:**

**Option 1: Use different host ports:**
```bash
docker run -d --name nginx-server \
  -p 8080:80 \
  -p 3000:3000 \
  nginx
```

**Option 2: If you have multiple services, use separate containers:**
```bash
docker run -d --name nginx-server -p 80:80 nginx
docker run -d --name node-app -p 3000:3000 node-app
```

**Explanation for Beginners:**
- Port mapping format: `-p <host-port>:<container-port>`
- Host port must be unique across all containers
- Think of host port as a door number - each door can only lead to one room
- Container ports can be the same across containers (they're isolated)
- Multiple containers can use port 80 internally, but need different host ports

**Valid Examples:**
```bash
# This is OK - different host ports
-p 8080:80 -p 3000:3000

# This is OK - same container port, different host ports
-p 8080:80 -p 8081:80  # Two nginx instances on different host ports

# This is WRONG - same host port
-p 80:80 -p 80:3000  # Error!
```

---

### Question 14
**Commands:**
```bash
docker stop mycontainer
docker start mycontainer
docker rm mycontainer
docker start mycontainer
```

**Tasks:**
a. What happens with the last command?  
b. Why does it fail?

**Solution:**

**a. What Happens:**

The last `docker start mycontainer` will fail with an error:
```
Error: No such container: mycontainer
```

**b. Explanation:**

**Command Sequence Breakdown:**
1. `docker stop mycontainer` - Stops the running container ✓
2. `docker start mycontainer` - Starts the stopped container ✓
3. `docker rm mycontainer` - REMOVES/DELETES the container ✓
4. `docker start mycontainer` - Fails! Container no longer exists ✗

**Corrected Approach:**
```bash
# Stop the container
docker stop mycontainer

# Restart the container
docker start mycontainer

# If you want to remove it
docker rm mycontainer

# To recreate, you need to run from image again
docker run --name mycontainer myimage
```

**Explanation for Beginners:**

**Container Lifecycle:**
```
Created → Running → Stopped → Removed
   ↑         ↓         ↓         ↓
 run      stop      start      rm
          ←─────────→           (deleted)
```

**Key Commands:**
- `docker stop` - Pauses container (still exists)
- `docker start` - Resumes stopped container
- `docker restart` - Stops then starts
- `docker rm` - DELETES container permanently
- `docker run` - Creates NEW container from image

**Important Notes:**
- Stopped containers still exist and can be started
- Removed containers are gone forever
- To recreate, you must `docker run` again from the image
- Use `docker rm -f` to force remove running container
- Use `docker container prune` to remove all stopped containers

---

### Question 15
**Commands:**
```bash
docker volume create mydata
docker run -v /mydata:/data alpine
```

**Tasks:**
a. Identify the issue with volume mounting

**Solution:**

**a. Error:**

1. **Error:** Incorrect volume mounting syntax - mixing volume name with host path

**What's Wrong:**
- Created a named volume `mydata`
- But using host path syntax `/mydata` instead of volume name syntax

**Corrected Commands:**

**Option 1: Using named volume (recommended):**
```bash
docker volume create mydata
docker run -v mydata:/data alpine
```

**Option 2: Using bind mount (host path):**
```bash
docker run -v /path/on/host:/data alpine
```

**Option 3: Using modern --mount syntax (most explicit):**
```bash
docker volume create mydata
docker run --mount source=mydata,target=/data alpine
```

**Explanation for Beginners:**

**Two Types of Volume Mounting:**

**1. Named Volumes (Managed by Docker):**
```bash
docker volume create mydata
docker run -v mydata:/data alpine
```
- Docker manages the storage location
- Lives in `/var/lib/docker/volumes/` on Linux
- Survives container deletion
- Best for production

**2. Bind Mounts (Host paths):**
```bash
docker run -v /home/user/data:/data alpine
```
- You specify exact host path
- Path must start with `/` or `./` or `~/`
- Good for development (edit files on host)
- Direct access to host filesystem

**The Error:**
```bash
-v /mydata:/data
```
This looks like a bind mount (starts with `/`) but was meant to use the named volume `mydata`.

**Correct Syntax Summary:**
```bash
# Named volume (no leading slash)
-v mydata:/data

# Bind mount (has leading slash)
-v /mydata:/data

# Modern explicit syntax
--mount source=mydata,target=/data
```

---

### Question 16
**Commands:**
```bash
docker run -d nginx
docker exec nginx apt-get update
```

**Tasks:**
a. Why might this command fail?  
b. Provide the correct approach

**Solution:**

**a. Why It Fails:**

1. **Error:** Using image name `nginx` instead of container name/ID
2. **Error:** Container might not be named "nginx"

**What Happens:**
```
Error: No such container: nginx
```

**Corrected Commands:**

**Option 1: Assign a name when running:**
```bash
docker run -d --name nginx-server nginx
docker exec nginx-server apt-get update
```

**Option 2: Use container ID:**
```bash
docker run -d nginx
docker ps  # Get the container ID
docker exec <container-id> apt-get update
```

**Option 3: More practical exec usage:**
```bash
docker run -d --name nginx-server nginx
docker exec -it nginx-server bash
# Now you're inside the container, run commands:
apt-get update
apt-get install vim
exit
```

**Explanation for Beginners:**

**Understanding docker exec:**
- `docker exec` runs commands in RUNNING containers
- Syntax: `docker exec [options] <container> <command>`
- You must provide container name or ID, not image name

**Common Patterns:**

**1. Run a single command:**
```bash
docker exec mycontainer ls -la /var/www
```

**2. Interactive shell:**
```bash
docker exec -it mycontainer bash
```

**3. Run as different user:**
```bash
docker exec -u root mycontainer whoami
```

**4. Set environment variables:**
```bash
docker exec -e VAR=value mycontainer printenv
```

**Key Flags:**
- `-it` - Interactive terminal (needed for shells)
- `-u` - Run as specific user
- `-e` - Set environment variable
- `-w` - Set working directory
- `-d` - Detached mode (background)

**Finding Container Names:**
```bash
docker ps                    # Shows running containers
docker ps -a                 # Shows all containers
docker ps --format "{{.Names}}"  # Just names
```

---

### Question 17
**Commands:**
```bash
docker network create mynet
docker run --network=bridge nginx
docker run --network=mynet redis
```

**Tasks:**
a. Why can't these containers communicate by name?  
b. Fix the setup

**Solution:**

**a. Why They Can't Communicate:**

1. **Error:** Containers are on different networks
   - nginx is on default `bridge` network
   - redis is on custom `mynet` network
2. **Issue:** Containers on different networks cannot communicate

**Corrected Commands:**

```bash
# Create custom network
docker network create mynet

# Run both containers on the same network
docker run -d --name nginx-server --network=mynet nginx
docker run -d --name redis-server --network=mynet redis

# Now they can communicate by name
docker exec nginx-server ping redis-server
```

**Explanation for Beginners:**

**Docker Networks - Key Concepts:**

**1. Default Bridge Network:**
- Created automatically
- Containers CAN communicate by IP
- Containers CANNOT communicate by name
- Need to use `--link` (deprecated)

**2. Custom Bridge Network (User-defined):**
- Must create manually
- Containers CAN communicate by name (DNS resolution)
- Better isolation
- Recommended for production

**Network Communication Rules:**
```
Same Network → Can communicate by container name
Different Networks → Cannot communicate at all
Default bridge → Can communicate only by IP (not name)
```

**Complete Example:**

```bash
# Create network
docker network create myapp-net

# Run database
docker run -d \
  --name database \
  --network myapp-net \
  -e POSTGRES_PASSWORD=secret \
  postgres

# Run application (can connect to "database:5432")
docker run -d \
  --name webapp \
  --network myapp-net \
  -e DB_HOST=database \
  -e DB_PORT=5432 \
  myapp

# Check connectivity
docker exec webapp ping database  # Works!
docker exec webapp curl http://database:5432  # Works!
```

**Network Commands:**
```bash
docker network ls                    # List networks
docker network inspect mynet         # Details about network
docker network connect mynet container1  # Add container to network
docker network disconnect mynet container1  # Remove from network
```

**Best Practice:**
Always use custom networks for multi-container applications. This enables service discovery by name.

---

### Question 18
**Commands:**
```bash
docker run -e DATABASE_URL=postgres://localhost:5432/db myapp
```

**Tasks:**
a. What's the problem with this database connection?  
b. How should it be fixed?

**Solution:**

**a. The Problem:**

1. **Error:** Using `localhost` to reference database from inside container
2. **Issue:** `localhost` inside a container refers to the container itself, not the host machine

**Why This Fails:**
- Each container has its own network namespace
- `localhost` in container = that container's loopback interface
- Database on host machine is not accessible via `localhost`

**Corrected Approaches:**

**Option 1: Use Docker Network (Best Practice):**
```bash
# Create network
docker network create myapp-net

# Run database
docker run -d \
  --name postgres-db \
  --network myapp-net \
  -e POSTGRES_PASSWORD=secret \
  postgres

# Run application
docker run -d \
  --name myapp \
  --network myapp-net \
  -e DATABASE_URL=postgres://postgres:secret@postgres-db:5432/db \
  myapp
```

**Option 2: Use host.docker.internal (Development only):**
```bash
docker run -e DATABASE_URL=postgres://host.docker.internal:5432/db myapp
```

**Option 3: Use Host Network Mode (Removes isolation):**
```bash
docker run --network host -e DATABASE_URL=postgres://localhost:5432/db myapp
```

**Explanation for Beginners:**

**Container Networking Basics:**

```
┌─────────────────────┐
│  Container          │
│  localhost = 127.0.0.1 (inside container)
│                     │
└─────────────────────┘
         ≠
┌─────────────────────┐
│  Host Machine       │
│  localhost = 127.0.0.1 (host machine)
│                     │
└─────────────────────┘
```

**Solutions Explained:**

**1. Container-to-Container (Best):**
- Put all services in same Docker network
- Use container names as hostnames
- DNS resolution works automatically
- Example: `postgres://postgres-db:5432/db`

**2. Container-to-Host (Development):**
- Use special hostname `host.docker.internal`
- Only works on Docker Desktop (Mac/Windows)
- On Linux, use `172.17.0.1` (bridge gateway) or `host.docker.internal` (Docker 20.10+)

**3. Host Network Mode:**
- Container uses host's network directly
- No network isolation
- `localhost` works but not recommended

**Complete Working Example:**

```bash
# Create network
docker network create myapp-net

# Start PostgreSQL
docker run -d \
  --name postgres \
  --network myapp-net \
  -e POSTGRES_DB=mydb \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=pass123 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:14

# Start application
docker run -d \
  --name myapp \
  --network myapp-net \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:pass123@postgres:5432/mydb \
  myapp

# Verify connection
docker logs myapp
```

---

### Question 19
**Commands:**
```bash
docker build -t myapp .
docker tag myapp myrepo/myapp
docker push myrepo/myapp
```

**Tasks:**
a. Why might the push fail?  
b. What's missing?

**Solution:**

**a. Why Push Fails:**

1. **Error:** Not logged into Docker registry
2. **Error:** Missing version tag (only using `latest` by default)
3. **Possible Error:** Incorrect repository format

**Corrected Commands:**

```bash
# 1. Login to Docker Hub
docker login

# 2. Build with proper tag
docker build -t myapp:v1.0 .

# 3. Tag for repository with version
docker tag myapp:v1.0 myrepo/myapp:v1.0
docker tag myapp:v1.0 myrepo/myapp:latest

# 4. Push both tags
docker push myrepo/myapp:v1.0
docker push myrepo/myapp:latest
```

**For Private Registry:**
```bash
# 1. Login to private registry
docker login registry.example.com

# 2. Tag with full registry path
docker tag myapp:v1.0 registry.example.com/myrepo/myapp:v1.0

# 3. Push
docker push registry.example.com/myrepo/myapp:v1.0
```

**Explanation for Beginners:**

**Docker Image Naming Convention:**
```
[registry-host[:port]/][username/]repository[:tag]
```

**Examples:**
```bash
# Docker Hub (default registry)
nginx:latest
username/myapp:v1.0

# Private registry
registry.example.com/myapp:v1.0
registry.example.com:5000/team/myapp:v1.0
```

**Complete Push Workflow:**

**Step 1: Build**
```bash
docker build -t myapp:1.0.0 .
```

**Step 2: Login**
```bash
# Docker Hub
docker login
# Enter username and password

# Private registry
docker login registry.company.com
```

**Step 3: Tag (if needed)**
```bash
# Tag for Docker Hub
docker tag myapp:1.0.0 username/myapp:1.0.0
docker tag myapp:1.0.0 username/myapp:latest

# Tag for private registry
docker tag myapp:1.0.0 registry.company.com/myapp:1.0.0
```

**Step 4: Push**
```bash
docker push username/myapp:1.0.0
docker push username/myapp:latest
```

**Common Errors:**

**1. Not logged in:**
```
Error: unauthorized: authentication required
Solution: Run docker login
```

**2. Permission denied:**
```
Error: denied: requested access to the resource is denied
Solution: Check repository name and permissions
```

**3. Repository doesn't exist:**
```
Error: repository does not exist or may require authorization
Solution: Create repository first (for private registries)
```

**Best Practices:**
- Always use version tags, not just `latest`
- Use semantic versioning (v1.0.0, v1.0.1, etc.)
- Push both specific version and `latest` tag
- Keep credentials secure (don't hardcode)
- Use CI/CD for automated pushing

---

### Question 20
**Commands:**
```bash
docker run -v myvolume:/data alpine
docker volume rm myvolume
```

**Tasks:**
a. Will the volume remove command succeed?  
b. What needs to be done first?

**Solution:**

**a. Will It Succeed?**

**No.** The volume remove command will fail with:
```
Error: volume is in use - [container_id]
```

**b. What's Wrong:**

1. **Error:** Container using the volume is still running
2. **Rule:** Cannot remove volumes that are in use by containers

**Corrected Workflow:**

**Option 1: Remove container first:**
```bash
# Run container with name
docker run -d --name alpine-container -v myvolume:/data alpine sleep 3600

# Stop container
docker stop alpine-container

# Remove container
docker rm alpine-container

# Now remove volume
docker volume rm myvolume
```

**Option 2: Run with --rm flag (auto-cleanup):**
```bash
# Container removes itself on exit
docker run --rm -v myvolume:/data alpine echo "done"

# Now volume can be removed
docker volume rm myvolume
```

**Option 3: Force remove (removes containers too):**
```bash
# Stop and remove all containers using the volume
docker ps -a --filter volume=myvolume -q | xargs docker rm -f

# Now remove volume
docker volume rm myvolume
```

**Explanation for Beginners:**

**Volume Lifecycle:**

```
Created → In Use → Not In Use → Removed
   ↓         ↓          ↓          ↓
 create    attached  detached    rm
```

**Rules:**
1. Volumes exist independently of containers
2. Multiple containers can share same volume
3. Volumes persist after container removal (unless --rm + -v)
4. Cannot remove volumes in use

**Checking Volume Usage:**

```bash
# List all volumes
docker volume ls

# Inspect volume (shows which containers use it)
docker volume inspect myvolume

# List containers using a volume
docker ps -a --filter volume=myvolume

# Remove unused volumes
docker volume prune
```

**Volume Cleanup Best Practices:**

**1. Manual cleanup:**
```bash
# Stop container
docker stop mycontainer

# Remove container
docker rm mycontainer

# Remove volume
docker volume rm myvolume
```

**2. Auto-cleanup with --rm:**
```bash
# Container and its anonymous volumes removed on exit
docker run --rm -v /data alpine
```

**3. Remove specific volume with container:**
```bash
# Removes container and its volumes
docker rm -v mycontainer
```

**4. Clean all unused volumes:**
```bash
# Removes all volumes not used by any container
docker volume prune

# Force (no confirmation)
docker volume prune -f
```

**Common Volume Commands:**
```bash
docker volume create myvolume          # Create
docker volume ls                       # List all
docker volume inspect myvolume         # Details
docker volume rm myvolume              # Remove
docker volume prune                    # Remove unused
docker volume prune -f                 # Force remove unused
```

**Important Notes:**
- Named volumes persist data even after container deletion
- Anonymous volumes (not named) can be auto-removed with `--rm` flag
- Always stop containers before removing their volumes
- Use `docker volume prune` carefully in production

---

## Questions 21-30: WORKDIR and Path Issues

### Question 21
**Dockerfile:**
```dockerfile
FROM node:14
COPY package.json /app
RUN npm install
COPY . /app
CMD ["node", "server.js"]
```

**Tasks:**
a. Identify path-related issues  
b. Fix the Dockerfile

**Solution:**

**a. Issues:**

1. **Error:** COPY package.json to /app, but npm install runs in root (/)
2. **Error:** No WORKDIR set, so commands run from /
3. **Error:** CMD expects server.js in /, but it's copied to /app

**What Actually Happens:**
```
package.json → /app/package.json
npm install runs in / → looks for /package.json (not found!)
Files copied to /app
node server.js runs in / → looks for /server.js (not found!)
```

**Corrected Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

**Explanation for Beginners:**

**Understanding WORKDIR:**
- WORKDIR sets the current working directory for all subsequent commands
- All relative paths after WORKDIR are relative to this directory
- If directory doesn't exist, it's created automatically

**Without WORKDIR:**
```dockerfile
FROM node:14
COPY package.json /app/package.json   # Absolute path needed
RUN cd /app && npm install            # cd in every RUN
COPY . /app                           # Absolute path needed
CMD ["node", "/app/server.js"]        # Absolute path needed
```

**With WORKDIR:**
```dockerfile
FROM node:14
WORKDIR /app                          # Set once
COPY package.json .                   # Relative to /app
RUN npm install                       # Runs in /app
COPY . .                              # Relative to /app
CMD ["node", "server.js"]             # Runs in /app
```

**WORKDIR Best Practices:**
1. Set early in Dockerfile
2. Use absolute paths for WORKDIR
3. Reduces need for cd commands
4. Makes paths more readable
5. Can be used multiple times

**Example with Multiple WORKDIR:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install

WORKDIR /app/src
COPY src/ .

WORKDIR /app
CMD ["node", "src/server.js"]
```

---

### Question 22
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Tasks:**
a. Find the subtle error  
b. Explain the consequence

**Solution:**

**a. The Error:**

1. **Error:** `WORKDIR app` should be `WORKDIR /app` (missing leading slash)

**What Happens:**
```dockerfile
WORKDIR app    # Creates relative path: /app (because we're in /)
```

Actually, this creates `/app`, but it's ambiguous and bad practice.

**b. The Consequence:**

- If a previous WORKDIR was set to something else, `WORKDIR app` would create a subdirectory
- Example: if WORKDIR was `/home/user`, then `WORKDIR app` would create `/home/user/app`
- Leads to confusion and hard-to-debug issues

**Corrected Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Explanation for Beginners:**

**WORKDIR Path Rules:**

**1. Absolute paths (start with /):**
```dockerfile
WORKDIR /app           # Always creates /app
WORKDIR /home/user     # Always creates /home/user
```

**2. Relative paths (no leading /):**
```dockerfile
WORKDIR /home
WORKDIR user           # Creates /home/user
WORKDIR documents      # Creates /home/user/documents
```

**Why Always Use Absolute Paths:**
```dockerfile
# GOOD - Clear and predictable
FROM python:3.9
WORKDIR /app

# BAD - Ambiguous
FROM python:3.9
WORKDIR app

# VERY BAD - Depends on previous context
FROM python:3.9
RUN cd /var/log
WORKDIR app  # Is this /var/log/app or /app? Unclear!
```

**Best Practices:**
1. **Always use absolute paths** (`/app` not `app`)
2. **Set WORKDIR early** in Dockerfile
3. **Don't mix cd and WORKDIR**
4. **Be consistent** across all Dockerfiles

**Examples:**

**Good:**
```dockerfile
FROM ubuntu:20.04
WORKDIR /opt/myapp
COPY . .
RUN make build
```

**Bad:**
```dockerfile
FROM ubuntu:20.04
WORKDIR opt/myapp      # Relative path - confusing
COPY . .
RUN make build
```

---

### Question 23
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
COPY app.py /usr/local/bin
COPY config.ini /etc/myapp
COPY data.json /var/lib/myapp
WORKDIR /usr/local/bin
CMD ["python3", "app.py"]
```

**Tasks:**
a. What problems exist with this structure?  
b. Provide a better organization

**Solution:**

**a. Problems:**

1. **Issue:** Application code scattered across system directories
2. **Issue:** Mixing application files with system directories
3. **Issue:** Hard to manage and update files
4. **Issue:** Violates single responsibility (app should be in one place)
5. **Issue:** Need to set WORKDIR before COPY to create directories

**Corrected Dockerfile:**
```dockerfile
FROM ubuntu:20.04

# Install Python
RUN apt-get update && apt-get install -y python3 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Set up application directory
WORKDIR /app

# Copy all application files to one location
COPY app.py .
COPY config.ini ./config/
COPY data.json ./data/

# Set configuration
ENV CONFIG_PATH=/app/config/config.ini
ENV DATA_PATH=/app/data/data.json

CMD ["python3", "app.py"]
```

**Explanation for Beginners:**

**Docker Application Structure Best Practices:**

**Bad Structure (Scattered):**
```
/usr/local/bin/app.py
/etc/myapp/config.ini
/var/lib/myapp/data.json
/tmp/cache
```

**Good Structure (Centralized):**
```
/app
├── app.py
├── config/
│   └── config.ini
├── data/
│   └── data.json
└── cache/
```

**Why Centralized is Better:**
1. **Easy to find** - Everything in one place
2. **Easy to copy** - `COPY . .` works
3. **Easy to volume mount** - Mount `/app` for development
4. **Easy to debug** - `docker exec` and navigate to `/app`
5. **Clear ownership** - App owns `/app`, not system directories

**Standard Directory Structures:**

**Simple App:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Structured App:**
```dockerfile
FROM node:14
WORKDIR /app

# Dependencies
COPY package*.json ./
RUN npm ci

# Source code
COPY src/ ./src/
COPY public/ ./public/
COPY config/ ./config/

# Configuration
ENV NODE_ENV=production
ENV CONFIG_DIR=/app/config

EXPOSE 3000
CMD ["npm", "start"]
```

**Multi-Component App:**
```dockerfile
FROM python:3.9
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application structure
COPY src/ ./src/
COPY tests/ ./tests/
COPY config/ ./config/
COPY static/ ./static/
COPY templates/ ./templates/

# Set up data directory
RUN mkdir -p /app/data /app/logs

# Environment variables point to structure
ENV CONFIG_PATH=/app/config/app.conf
ENV LOG_PATH=/app/logs
ENV DATA_PATH=/app/data

CMD ["python", "src/main.py"]
```

**When to Use System Directories:**
- Configuration that's truly system-wide
- Following FHS (Filesystem Hierarchy Standard)
- Integrating with system services
- But even then, consider keeping app-specific

**Example with Both:**
```dockerfile
FROM ubuntu:20.04
WORKDIR /app

# Application code
COPY app.py .
COPY lib/ ./lib/

# System integration (if truly needed)
COPY systemd/myapp.service /etc/systemd/system/
COPY nginx/myapp.conf /etc/nginx/sites-available/

CMD ["python3", "app.py"]
```

---

### Question 24
**Dockerfile:**
```dockerfile
FROM alpine
WORKDIR /app
COPY ../config/app.conf .
CMD ["./start.sh"]
```

**Tasks:**
a. What's wrong with the COPY command?  
b. How should it be fixed?

**Solution:**

**a. The Error:**

1. **Error:** `COPY ../config/app.conf` tries to access parent directory
2. **Error:** Cannot COPY files from outside build context

**What is Build Context?**
When you run `docker build .`, the `.` is the build context - the directory Docker can access.

**Why It Fails:**
```
Project structure:
/project
├── config/
│   └── app.conf
└── app/
    └── Dockerfile    ← You are here

docker build -t myapp .  ← Context is /project/app

COPY ../config/app.conf  ← Tries to access /project/config (outside context!)
```

**Corrected Solutions:**

**Option 1: Change build context:**
```bash
# Run from project root
cd /project
docker build -t myapp -f app/Dockerfile .
```

```dockerfile
FROM alpine
WORKDIR /app
COPY config/app.conf .
CMD ["./start.sh"]
```

**Option 2: Restructure project:**
```
/project
├── Dockerfile
├── config/
│   └── app.conf
└── app/
    └── start.sh
```

```dockerfile
FROM alpine
WORKDIR /app
COPY config/app.conf ./config/
COPY app/start.sh .
CMD ["./start.sh"]
```

**Option 3: Move config into app directory:**
```
/project/app
├── Dockerfile
├── config/
│   └── app.conf
└── start.sh
```

```dockerfile
FROM alpine
WORKDIR /app
COPY config/app.conf ./config/
COPY start.sh .
CMD ["./start.sh"]
```

**Explanation for Beginners:**

**Build Context Rules:**

**1. Build context is the directory you specify:**
```bash
docker build .           # Context is current directory
docker build /path/to/dir  # Context is that directory
docker build -f /path/to/Dockerfile /context/path  # Context is last argument
```

**2. COPY/ADD can only access build context:**
```dockerfile
# If context is /project/app:
COPY file.txt .         # ✓ Works: /project/app/file.txt
COPY ../file.txt .      # ✗ Fails: Cannot access /project/file.txt
COPY /etc/hosts .       # ✗ Fails: Cannot access system files
```

**3. Use .dockerignore to exclude files:**
```
# .dockerignore
node_modules
.git
*.log
.env
```

**Common Patterns:**

**Pattern 1: Dockerfile at root:**
```
/project
├── Dockerfile
├── src/
├── config/
└── data/

# Build command:
docker build -t myapp .

# Dockerfile:
FROM alpine
COPY src/ /app/src/
COPY config/ /app/config/
```

**Pattern 2: Dockerfile in subdirectory:**
```
/project
├── docker/
│   └── Dockerfile
├── src/
└── config/

# Build command:
docker build -t myapp -f docker/Dockerfile .

# Dockerfile:
FROM alpine
COPY src/ /app/src/
COPY config/ /app/config/
```

**Pattern 3: Multiple Dockerfiles:**
```
/project
├── Dockerfile.dev
├── Dockerfile.prod
├── src/
└── config/

# Build commands:
docker build -t myapp:dev -f Dockerfile.dev .
docker build -t myapp:prod -f Dockerfile.prod .
```

**Best Practices:**
1. Keep Dockerfile at project root
2. Or use `-f` flag and set correct context
3. Never try to COPY from parent directories
4. Use .dockerignore to exclude unnecessary files
5. Understand that context is uploaded to Docker daemon

---

### Question 25
**Dockerfile:**
```dockerfile
FROM golang:1.17
COPY . .
RUN go build -o app
CMD ["./app"]
```

**Tasks:**
a. What's missing?  
b. How can this cause problems?

**Solution:**

**a. What's Missing:**

1. **Missing:** WORKDIR instruction
2. **Issue:** Files copied to / (root directory)
3. **Issue:** Binary created in / (pollutes root)
4. **Issue:** Running from / is bad practice

**What Actually Happens:**
```
Files copied to: /
Binary created: /app
Running from: /

Problem: Mixing application files with system files!
```

**b. How It Causes Problems:**

1. **Security risk** - Running from root directory
2. **File conflicts** - Could overwrite system files
3. **Permission issues** - Root directory has restrictions
4. **Difficult debugging** - Hard to find app files
5. **Bad practice** - Violates container best practices

**Corrected Dockerfile:**
```dockerfile
FROM golang:1.17

# Set working directory
WORKDIR /app

# Copy and build
COPY . .
RUN go build -o app

# Run
CMD ["./app"]
```

**Better Version (with optimization):**
```dockerfile
FROM golang:1.17

WORKDIR /app

# Copy go mod files first (better caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy source and build
COPY . .
RUN go build -o app

# Run
CMD ["./app"]
```

**Best Version (multi-stage for smaller image):**
```dockerfile
# Build stage
FROM golang:1.17 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app

# Run stage
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
```

**Explanation for Beginners:**

**Why WORKDIR Matters:**

**Without WORKDIR:**
```dockerfile
FROM golang:1.17
COPY . .                    # Copies to /
RUN go build -o app         # Creates /app
CMD ["./app"]               # Runs /app

Result:
/
├── app (binary)
├── main.go
├── go.mod
├── handler.go
├── bin/ (system)
├── etc/ (system)
└── ... (your files mixed with system files!)
```

**With WORKDIR:**
```dockerfile
FROM golang:1.17
WORKDIR /app
COPY . .                    # Copies to /app
RUN go build -o app         # Creates /app/app
CMD ["./app"]               # Runs /app/app

Result:
/
├── bin/ (system)
├── etc/ (system)
└── app/
    ├── app (binary)
    ├── main.go
    ├── go.mod
    └── handler.go
```

**WORKDIR Benefits:**

**1. Isolation:**
```dockerfile
WORKDIR /app              # Application files isolated in /app
```

**2. Clarity:**
```dockerfile
WORKDIR /app
COPY . .                  # Clear: copying to /app
RUN make build            # Clear: building in /app
```

**3. Security:**
```dockerfile
WORKDIR /app
RUN chown -R user:user .  # Clear ownership of /app
USER user                 # Non-root user
```

**4. Debugging:**
```bash
docker exec -it container bash
cd /app  # All your files here
```

**When WORKDIR Creates Directories:**
```dockerfile
FROM alpine
WORKDIR /app/data/cache   # Creates /app, /app/data, /app/data/cache
```

**Multiple WORKDIRs:**
```dockerfile
FROM node:14
WORKDIR /build
COPY package*.json ./
RUN npm install

WORKDIR /build/dist
COPY src/ /build/src/
RUN npm run build

WORKDIR /app
COPY --from=0 /build/dist ./
CMD ["node", "server.js"]
```

**Best Practices:**
1. **Always use WORKDIR** - Don't work in /
2. **Use absolute paths** - /app not app
3. **Set early** - Before first COPY or RUN
4. **Be consistent** - Same path across similar Dockerfiles
5. **Document intent** - Use comments if using multiple WORKDIRs

---

### Question 26
**Dockerfile:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
RUN chmod +x scripts/start.sh
CMD ["scripts/start.sh"]
```

**Tasks:**
a. What might go wrong with file permissions?  
b. Fix potential issues

**Solution:**

**a. Potential Issues:**

1. **Issue:** Files copied with wrong ownership (root by default)
2. **Issue:** Executable might not run if shell script needs interpreter
3. **Issue:** No proper shebang or explicit shell invocation
4. **Issue:** Windows line endings (CRLF) might cause issues

**What Can Go Wrong:**

**Error 1: Permission denied**
```bash
standard_init_linux.go:228: exec user process caused: permission denied
```

**Error 2: Bad interpreter**
```bash
/bin/sh: scripts/start.sh: /bin/bash^M: bad interpreter: No such file or directory
```

**Error 3: File not found**
```bash
exec: "scripts/start.sh": executable file not found in $PATH
```

**Corrected Dockerfile:**

**Option 1: With full path and proper execution:**
```dockerfile
FROM python:3.9

WORKDIR /app

# Copy requirements first
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy rest of files
COPY . .

# Fix line endings and set permissions
RUN chmod +x scripts/start.sh && \
    apt-get update && apt-get install -y dos2unix && \
    dos2unix scripts/start.sh && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Use explicit shell invocation
CMD ["/bin/bash", "./scripts/start.sh"]
```

**Option 2: Better approach with proper ownership:**
```dockerfile
FROM python:3.9

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy and install dependencies as root
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application files
COPY . .

# Set permissions
RUN chmod +x scripts/start.sh && \
    chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

CMD ["./scripts/start.sh"]
```

**Option 3: Multi-stage with explicit chmod:**
```dockerfile
FROM python:3.9 AS base

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

FROM python:3.9

WORKDIR /app

# Copy from base
COPY --from=base /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

# Copy application
COPY --chmod=755 scripts/start.sh ./scripts/
COPY . .

CMD ["./scripts/start.sh"]
```

**Explanation for Beginners:**

**File Permissions in Docker:**

**Default Behavior:**
- Files copied into image are owned by root (UID 0)
- File permissions preserved from source
- Execute bit must be set explicitly if not in source

**Understanding chmod:**
```bash
chmod +x file.sh          # Add execute permission
chmod 755 file.sh         # rwxr-xr-x (full for owner, read+execute for others)
chmod 644 file.txt        # rw-r--r-- (read/write for owner, read for others)
```

**Common Permission Issues:**

**Issue 1: Missing execute bit**
```dockerfile
# Wrong - no execute permission
COPY start.sh .
CMD ["./start.sh"]  # Error: permission denied

# Right - add execute permission
COPY start.sh .
RUN chmod +x start.sh
CMD ["./start.sh"]
```

**Issue 2: Line ending problems (Windows)**
```dockerfile
# Windows CRLF endings cause "bad interpreter"
# Fix with dos2unix
RUN apt-get update && apt-get install -y dos2unix && \
    dos2unix scripts/*.sh
```

**Issue 3: Running as non-root**
```dockerfile
# Files owned by root, but running as user
COPY . .
USER appuser
CMD ["./start.sh"]  # Might fail if permissions wrong

# Fix: Change ownership
COPY . .
RUN chown -R appuser:appuser /app
USER appuser
CMD ["./start.sh"]
```

**Best Practices:**

**1. Set executable in Dockerfile:**
```dockerfile
COPY script.sh .
RUN chmod +x script.sh
```

**2. Or set it in source control (Git):**
```bash
git update-index --chmod=+x script.sh
git commit -m "Make script executable"
```

**3. Use COPY --chmod (Docker 20.10+):**
```dockerfile
COPY --chmod=755 script.sh .
```

**4. Ensure proper shebang:**
```bash
#!/bin/bash
# or
#!/usr/bin/env python3
```

**5. Use explicit shell:**
```dockerfile
CMD ["/bin/bash", "./script.sh"]
# or
CMD ["sh", "-c", "./script.sh"]
```

**Complete Example:**
```dockerfile
FROM python:3.9-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Set up scripts with proper permissions
RUN chmod +x scripts/*.sh && \
    chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Run
CMD ["./scripts/start.sh"]
```

---

### Question 27
**Dockerfile:**
```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html
WORKDIR /usr/share/nginx/html
```

**Commands:**
```bash
docker run -v $(pwd)/content:/usr/share/nginx/html nginx-custom
```

**Tasks:**
a. What happens to the copied HTML files?  
b. Is this the intended behavior?

**Solution:**

**a. What Happens:**

1. **Issue:** The volume mount REPLACES the /usr/share/nginx/html directory
2. **Result:** The HTML files copied during build are HIDDEN/OVERRIDDEN
3. **Only files from $(pwd)/content are visible**

**Build Time vs Runtime:**
```
Build time:
├── COPY html/ → /usr/share/nginx/html/
│   ├── index.html
│   ├── style.css
│   └── app.js

Runtime with volume mount:
├── -v $(pwd)/content → /usr/share/nginx/html/
    ├── (Build-time files are hidden!)
    └── Only $(pwd)/content files visible
```

**b. Is This Intended?**

Depends on use case:
- **Development:** Yes, volume mounts are intended (live editing)
- **Production:** No, should use built-in files

**Corrected Approaches:**

**Option 1: Development (volumes for live editing):**
```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
# Don't copy HTML in Dockerfile - use volumes
WORKDIR /usr/share/nginx/html
```

```bash
# Mount local directory for development
docker run -d \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -p 8080:80 \
  nginx
```

**Option 2: Production (use built-in files):**
```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html/
RUN chmod -R 755 /usr/share/nginx/html
```

```bash
# No volumes - use built-in files
docker run -d -p 8080:80 nginx-custom
```

**Option 3: Hybrid (different mount point):**
```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/default/
```

```bash
# Mount to different location, fall back to default
docker run -d \
  -v $(pwd)/content:/usr/share/nginx/custom:ro \
  -p 8080:80 \
  nginx-custom
```

**Option 4: Named volumes (persist data):**
```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
# Initialize volume with default content
COPY html/ /usr/share/nginx/html/
VOLUME /usr/share/nginx/html
```

```bash
# Creates named volume, initialized with image content
docker run -d \
  -v nginx-content:/usr/share/nginx/html \
  -p 8080:80 \
  nginx-custom
```

**Explanation for Beginners:**

**Volume Mounting Behavior:**

**Key Concept:** Volumes OVERRIDE directories at runtime

**Scenario 1: Without Volume**
```dockerfile
COPY html/ /app/html/
```
```bash
docker run myapp
# /app/html contains files from image
```

**Scenario 2: With Volume Mount**
```dockerfile
COPY html/ /app/html/
```
```bash
docker run -v $(pwd)/html:/app/html myapp
# /app/html contains files from host!
# Image files are hidden
```

**When to Use Each Approach:**

**1. Build-time COPY (Production):**
```dockerfile
FROM nginx
COPY html/ /usr/share/nginx/html/
```
- Files baked into image
- Immutable
- Fast (no I/O overhead)
- Good for production

**2. Runtime volumes (Development):**
```bash
docker run -v $(pwd)/html:/usr/share/nginx/html nginx
```
- Live editing
- Changes reflect immediately
- No rebuild needed
- Good for development

**3. Named volumes (Persistent data):**
```bash
docker volume create mydata
docker run -v mydata:/data myapp
```
- Data persists between containers
- Survives container deletion
- Good for databases

**Understanding the Override:**

```
Image contains:           Volume mount:           Result:
/app/                     $(pwd)/data             /app/
├── file1.txt            └── file3.txt           ├── file3.txt (from volume)
├── file2.txt                                    └── (file1.txt, file2.txt hidden)
└── data/
    └── important.txt
```

**Best Practices:**

**1. Separate config from content:**
```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
# Leave /usr/share/nginx/html empty for volumes
```

**2. Use different directories:**
```dockerfile
FROM nginx
COPY defaults/ /usr/share/nginx/defaults/
# Mount custom content to /usr/share/nginx/custom/
```

**3. Initialize volumes properly:**
```dockerfile
FROM nginx
COPY html/ /usr/share/nginx/html/
VOLUME /usr/share/nginx/html
# First container initializes volume with these files
```

**4. Document volume expectations:**
```dockerfile
FROM myapp
# This directory is meant to be volume-mounted
# with your custom content
VOLUME /app/data
```

**Complete Example:**

```dockerfile
FROM nginx:alpine

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy default content (can be overridden by volume)
COPY html/ /usr/share/nginx/default-html/

# Script to copy defaults if mounted volume is empty
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

EXPOSE 80

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

**docker-entrypoint.sh:**
```bash
#!/bin/sh
# If html directory is empty (volume mounted), copy defaults
if [ -z "$(ls -A /usr/share/nginx/html)" ]; then
    cp -r /usr/share/nginx/default-html/* /usr/share/nginx/html/
fi
exec "$@"
```

**Usage:**
```bash
# Development: override with volume
docker run -v $(pwd)/html:/usr/share/nginx/html -p 8080:80 myapp

# Production: use built-in files
docker run -p 8080:80 myapp
```

---

### Question 28
**Dockerfile:**
```dockerfile
FROM maven:3.8-jdk-11
COPY src /src
COPY pom.xml /
RUN mvn clean package
CMD ["java", "-jar", "/target/app.jar"]
```

**Tasks:**
a. Find the path inconsistency  
b. Fix the Dockerfile

**Solution:**

**a. Path Inconsistency:**

1. **Error:** `src` copied to `/src` but `pom.xml` copied to `/`
2. **Error:** Maven expects project structure:
```
/
├── pom.xml
└── src/
```
But we have:
```
/
├── pom.xml
├── src/        ← source files
└── target/     ← Maven creates this at /target
```

3. **Error:** No WORKDIR set, commands run from `/`
4. **Error:** `/target/app.jar` path might be wrong

**What Actually Happens:**
```
pom.xml location: /pom.xml
src location: /src
mvn clean package runs in: /
Maven creates: /target/app.jar
java -jar /target/app.jar ← This part is correct
```

Actually, this might work, but it's messy and bad practice!

**Corrected Dockerfile:**

**Option 1: Proper structure:**
```dockerfile
FROM maven:3.8-jdk-11

# Set working directory
WORKDIR /app

# Copy Maven files
COPY pom.xml .
COPY src ./src

# Build
RUN mvn clean package

# Run
CMD ["java", "-jar", "target/app.jar"]
```

**Option 2: Multi-stage (recommended):**
```dockerfile
# Build stage
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Run stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

**Option 3: With dependency caching:**
```dockerfile
FROM maven:3.8-jdk-11 AS build

WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn package -DskipTests

FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

**Explanation for Beginners:**

**Maven Project Structure:**

Maven expects a specific structure:
```
project/
├── pom.xml              ← Project configuration
├── src/
│   ├── main/
│   │   ├── java/       ← Java source files
│   │   └── resources/  ← Properties, configs
│   └── test/
│       └── java/       ← Test files
└── target/             ← Build output (created by Maven)
    ├── classes/
    ├── app.jar         ← Final artifact
    └── ...
```

**Why Consistency Matters:**

**Bad: Inconsistent paths**
```dockerfile
COPY src /src              # Source in /src
COPY pom.xml /             # POM in /
RUN mvn clean package      # Maven confused about structure
```

**Good: Consistent structure**
```dockerfile
WORKDIR /app
COPY pom.xml .             # POM in /app
COPY src ./src             # Source in /app/src
RUN mvn clean package      # Maven happy!
```

**Multi-Stage Build Benefits:**

**Without multi-stage:**
```
Image size: 800MB
Includes: Maven, JDK 11, source code, build tools, dependencies
```

**With multi-stage:**
```
Build stage: 800MB (discarded)
Final stage: 200MB
Includes: Only JRE 11 and app.jar
```

**Complete Best Practice Example:**

```dockerfile
# ========================================
# Stage 1: Build
# ========================================
FROM maven:3.8-jdk-11-slim AS build

WORKDIR /build

# Copy POM and download dependencies (better caching)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build application (skip tests for faster builds)
RUN mvn package -DskipTests -B

# ========================================
# Stage 2: Runtime
# ========================================
FROM openjdk:11-jre-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy only the built artifact
COPY --from=build /build/target/app.jar app.jar

# Set ownership
RUN chown appuser:appuser app.jar

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD java -jar app.jar --health-check || exit 1

# Run application
CMD ["java", \
     "-Xmx512m", \
     "-Xms256m", \
     "-jar", \
     "app.jar"]
```

**Layer Caching Strategy:**

**Bad ordering (rebuilds dependencies every time):**
```dockerfile
COPY src ./src
COPY pom.xml .
RUN mvn package
# Every code change = re-download all dependencies
```

**Good ordering (caches dependencies):**
```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package
# Code changes don't trigger dependency downloads
```

**Why This Matters:**
- Faster builds (dependencies cached)
- Less bandwidth usage
- Better developer experience

**Path Best Practices:**
1. Always use WORKDIR
2. Keep consistent structure
3. Copy configuration before source
4. Use relative paths within WORKDIR
5. Use multi-stage for smaller images

---

### Question 29
**Dockerfile:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json /
RUN npm install
COPY . /app
CMD ["npm", "start"]
```

**Tasks:**
a. Identify the COPY inconsistency  
b. Explain the impact

**Solution:**

**a. COPY Inconsistency:**

1. **Error:** `COPY package.json /` copies to root directory `/`
2. **Error:** But WORKDIR is `/app`
3. **Error:** `npm install` runs in `/app` but package.json is in `/`
4. **Issue:** Later `COPY . /app` might overwrite node_modules

**What Happens:**
```
WORKDIR /app                          Current dir: /app
COPY package.json /                   Copies to: /package.json (not /app!)
RUN npm install                       Runs in: /app, looks for /app/package.json (not found!)
                                      Error: ENOENT: no such file 'package.json'
```

**b. Impact:**

- **Build fails** with "package.json not found"
- If package.json somehow exists in current dir, wrong dependencies installed
- Wastes time debugging
- Layer caching doesn't work properly

**Corrected Dockerfile:**

**Option 1: Simple fix:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

**Option 2: With package-lock.json:**
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

**Option 3: Full best practices:**
```dockerfile
FROM node:14-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production && npm cache clean --force

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 3000

CMD ["npm", "start"]
```

**Explanation for Beginners:**

**WORKDIR and COPY Paths:**

**Understanding Destinations:**

```dockerfile
WORKDIR /app

COPY file.txt /other/       # Absolute: copies to /other/file.txt
COPY file.txt ./            # Relative: copies to /app/file.txt
COPY file.txt .             # Relative: copies to /app/file.txt
COPY file.txt /             # Absolute: copies to /file.txt
```

**Common Patterns:**

**Pattern 1: Relative paths (recommended):**
```dockerfile
WORKDIR /app
COPY package.json ./        # → /app/package.json
COPY src ./src/             # → /app/src/
COPY . .                    # → /app/ (everything)
```

**Pattern 2: Explicit absolute paths:**
```dockerfile
WORKDIR /app
COPY package.json /app/package.json
COPY src /app/src/
```

**Pattern 3: Mixed (confusing, avoid):**
```dockerfile
WORKDIR /app
COPY package.json /         # → /package.json (outside WORKDIR!)
COPY src ./src/             # → /app/src/
```

**Node.js Best Practices:**

**1. Copy package files first (caching):**
```dockerfile
WORKDIR /app
# Layer 1: Package files (rarely change)
COPY package*.json ./
RUN npm ci

# Layer 2: Source code (changes often)
COPY . .
```

**Why this order?**
- If source code changes, npm install doesn't rerun
- Faster builds
- Efficient layer caching

**2. Use .dockerignore:**
```
# .dockerignore
node_modules
npm-debug.log
.git
.env
.DS_Store
```

**3. Don't copy node_modules:**
```dockerfile
# Wrong
COPY . .                    # Copies node_modules from host

# Right
# Use .dockerignore to exclude node_modules
# Let npm install create fresh node_modules
```

**Complete Example:**

```dockerfile
FROM node:14-alpine AS build

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build (if needed)
RUN npm run build

# ========================================
# Production image
# ========================================
FROM node:14-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built application from build stage
COPY --from=build /app/dist ./dist

# Security: create non-root user
RUN addgroup -g 1000 nodejs && \
    adduser -D -u 1000 -G nodejs nodejs && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
    CMD node healthcheck.js

CMD ["node", "dist/server.js"]
```

**Debugging COPY Issues:**

```bash
# Build and inspect
docker build -t myapp .
docker run --rm -it myapp sh

# Check file locations
ls -la /
ls -la /app
find / -name "package.json"

# Verify WORKDIR
pwd
```

**Key Takeaways:**
1. **Understand absolute vs relative paths**
2. **Be consistent** - use relative paths with WORKDIR
3. **Order matters** - Copy package.json before source code
4. **Use .dockerignore** - Exclude node_modules, .git, etc.
5. **Test your builds** - Verify files are where you expect

---

### Question 30
**Dockerfile:**
```dockerfile
FROM python:3.9
ADD https://example.com/config.json /config/
WORKDIR /app
COPY . .
```

**Tasks:**
a. What's problematic about this structure?  
b. Provide improvements

**Solution:**

**a. Problems:**

1. **Issue:** ADD creates `/config/` directory before WORKDIR `/app`
2. **Issue:** Using ADD for URL (deprecated pattern)
3. **Issue:** No error handling for URL download
4. **Issue:** No verification of downloaded file
5. **Issue:** Directory structure unclear
6. **Issue:** Security concern - downloading arbitrary URLs during build

**What Happens:**
```
Filesystem after build:
/
├── config/
│   └── config.json     ← Downloaded from URL
└── app/                ← WORKDIR
    └── (application files)
```

**b. Improvements Needed:**

**Option 1: Use RUN with curl (recommended):**
```dockerfile
FROM python:3.9

WORKDIR /app

# Download config with verification
RUN apt-get update && apt-get install -y curl && \
    curl -f -o /app/config/config.json https://example.com/config.json && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Verify downloaded file
RUN test -f /app/config/config.json && \
    python -m json.tool /app/config/config.json > /dev/null

COPY . .

CMD ["python", "app.py"]
```

**Option 2: With checksums:**
```dockerfile
FROM python:3.9

WORKDIR /app

# Download and verify
RUN apt-get update && apt-get install -y curl && \
    curl -f -o config.json https://example.com/config.json && \
    echo "expected-sha256-hash  config.json" | sha256sum -c - && \
    mkdir -p config && \
    mv config.json config/ && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

COPY . .

CMD ["python", "app.py"]
```

**Option 3: Best practice - config at runtime:**
```dockerfile
FROM python:3.9

WORKDIR /app

# Copy application
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Download config at runtime via entrypoint
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["python", "app.py"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

# Download config if not exists
if [ ! -f /app/config/config.json ]; then
    echo "Downloading config..."
    curl -f -o /app/config/config.json https://example.com/config.json
    echo "Config downloaded successfully"
fi

# Execute main command
exec "$@"
```

**Option 4: Use build args for flexibility:**
```dockerfile
FROM python:3.9

ARG CONFIG_URL=https://example.com/config.json

WORKDIR /app

# Install curl
RUN apt-get update && apt-get install -y curl && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Download config
RUN mkdir -p config && \
    curl -f -o config/config.json ${CONFIG_URL}

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

**Build command:**
```bash
docker build --build-arg CONFIG_URL=https://prod.example.com/config.json -t myapp .
```

**Explanation for Beginners:**

**ADD vs RUN + curl:**

**ADD (old way, not recommended):**
```dockerfile
ADD https://example.com/file.tar.gz /app/
```
**Problems:**
- No error handling
- Can't verify content
- Auto-extraction might be unwanted
- Cached forever (layer doesn't rebuild)
- No flexibility

**RUN + curl (recommended):**
```dockerfile
RUN curl -f -o /app/file.tar.gz https://example.com/file.tar.gz && \
    sha256sum -c checksums.txt && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz
```
**Benefits:**
- Explicit error handling (-f flag)
- Can verify checksums
- Control extraction
- Better caching control
- More flexible

**Remote File Best Practices:**

**1. Avoid downloading during build:**
```dockerfile
# Bad: Bakes specific config into image
ADD https://example.com/config.json /app/

# Good: Download at runtime based on environment
ENTRYPOINT ["/entrypoint.sh"]
```

**2. If you must download during build:**
```dockerfile
RUN set -e && \
    curl -f -L -o file.tar.gz ${URL} && \
    echo "${EXPECTED_SHA256}  file.tar.gz" | sha256sum -c - && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz
```

**3. Use build args for URLs:**
```dockerfile
ARG CONFIG_URL
RUN curl -f -o config.json ${CONFIG_URL}
```

**4. Verify downloaded content:**
```dockerfile
RUN curl -f -o config.json ${URL} && \
    python -c "import json; json.load(open('config.json'))" && \
    test $(stat -f%z config.json) -gt 0
```

**Security Considerations:**

**1. Use HTTPS only:**
```dockerfile
# Bad
ADD http://example.com/file.zip /app/

# Good
RUN curl -f https://example.com/file.zip -o /app/file.zip
```

**2. Verify checksums:**
```dockerfile
RUN curl -f -o file.zip https://example.com/file.zip && \
    echo "abc123...  file.zip" | sha256sum -c -
```

**3. Don't trust arbitrary URLs:**
```dockerfile
# Bad: User-provided URL
ARG DOWNLOAD_URL
ADD ${DOWNLOAD_URL} /app/

# Better: Validate or use allowlist
ARG DOWNLOAD_URL
RUN if echo "$DOWNLOAD_URL" | grep -q "^https://trusted-domain.com/"; then \
      curl -f -o file.zip "$DOWNLOAD_URL"; \
    else \
      echo "Untrusted URL" && exit 1; \
    fi
```

**Complete Secure Example:**

```dockerfile
FROM python:3.9-slim

# Build arguments
ARG CONFIG_URL
ARG CONFIG_SHA256

WORKDIR /app

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Download and verify config
RUN if [ -z "$CONFIG_URL" ]; then \
      echo "CONFIG_URL not provided, using default" && \
      echo '{}' > config.json; \
    else \
      echo "Downloading config from $CONFIG_URL" && \
      curl -f -L -o config.json "$CONFIG_URL" && \
      if [ -n "$CONFIG_SHA256" ]; then \
        echo "$CONFIG_SHA256  config.json" | sha256sum -c -; \
      fi && \
      python -c "import json; json.load(open('config.json'))" && \
      echo "Config verified successfully"; \
    fi

# Copy application
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Usage:**
```bash
# With config URL
docker build \
  --build-arg CONFIG_URL=https://example.com/config.json \
  --build-arg CONFIG_SHA256=abc123... \
  -t myapp .

# Without config URL (uses default)
docker build -t myapp .
```

**Key Takeaways:**
1. Prefer RUN + curl over ADD for URLs
2. Always verify downloaded content
3. Consider runtime config over build-time
4. Use HTTPS and check certificates
5. Implement proper error handling
6. Keep images reproducible and secure

---

## Questions 31-35: Multi-Stage Build Errors

### Question 31
**Dockerfile:**
```dockerfile
FROM node:14 AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM nginx
COPY --from=build /app/build /usr/share/nginx/html
```

**Tasks:**
a. What potential issue exists?  
b. How can it be improved?

**Solution:**

**a. Potential Issues:**

1. **Issue:** Building with ALL node_modules (including devDependencies)
2. **Issue:** No layer caching for dependencies
3. **Issue:** Copying everything (including unnecessary files)
4. **Issue:** No error handling
5. **Issue:** Missing nginx configuration
6. **Issue:** No optimization for build cache

**What Happens:**
```
Build stage:
- Copies ALL files (including .git, tests, etc.)
- npm install downloads all dependencies (dev + prod)
- Build runs
- Final image only needs build output, but built with excess baggage
```

**b. Improved Dockerfile:**

**Option 1: Basic improvements:**
```dockerfile
FROM node:14-alpine AS build
WORKDIR /app

# Copy package files for better caching
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Option 2: With .dockerignore:**
```dockerfile
FROM node:14-alpine AS build
WORKDIR /app

# Better caching
COPY package*.json ./
RUN npm ci --only=production && \
    npm install --only=dev && \
    npm cache clean --force

COPY . .
RUN npm run build

# Production
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**.dockerignore:**
```
node_modules
.git
.gitignore
README.md
.env
.env.local
.DS_Store
npm-debug.log
.vscode
.idea
coverage
.cache
dist
build
```

**Option 3: Complete best practices:**
```dockerfile
# ========================================
# Stage 1: Dependencies
# ========================================
FROM node:14-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && \
    cp -R node_modules prod_node_modules && \
    npm ci

# ========================================
# Stage 2: Build
# ========================================
FROM node:14-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# ========================================
# Stage 3: Production
# ========================================
FROM nginx:alpine

# Copy built assets
COPY --from=build /app/build /usr/share/nginx/html

# Custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Security: run as non-root
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
    CMD curl -f http://localhost:8080/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```nginx
server {
    listen 8080;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(?:css|js|jpg|jpeg|gif|png|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

**Explanation for Beginners:**

**Multi-Stage Build Benefits:**

**Without multi-stage (bad):**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 3000
CMD ["npm", "start"]

# Result:
# - Image size: 1.2GB
# - Includes: Node.js, npm, source code, node_modules, build tools
# - Security risk: Unnecessary tools in production
```

**With multi-stage (good):**
```dockerfile
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

# Result:
# - Image size: 25MB
# - Includes: Only nginx and built static files
# - Secure: No source code or build tools
```

**Layer Caching Strategy:**

**Bad ordering:**
```dockerfile
FROM node:14 AS build
WORKDIR /app
COPY . .                    # Any file change invalidates next layers
RUN npm install             # Always reruns (slow!)
RUN npm run build
```

**Good ordering:**
```dockerfile
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./       # Only changes if dependencies change
RUN npm ci                  # Cached unless package.json changes
COPY . .                    # Source code changes don't affect npm install
RUN npm run build
```

**Understanding COPY --from:**

```dockerfile
# Stage 1: Build
FROM node:14 AS build
WORKDIR /app
RUN npm run build
# Creates: /app/build/index.html, /app/build/main.js, etc.

# Stage 2: Production
FROM nginx
COPY --from=build /app/build /usr/share/nginx/html
#            ↑         ↑            ↑
#          stage    source      destination
#          name     path         path
```

**Multiple stages example:**

```dockerfile
# Stage 1: Base dependencies
FROM node:14-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Development
FROM base AS development
COPY . .
CMD ["npm", "run", "dev"]

# Stage 3: Build
FROM base AS build
COPY . .
RUN npm run build
RUN npm run test

# Stage 4: Production
FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html
```

**Build specific stage:**
```bash
# Build development stage
docker build --target development -t myapp:dev .

# Build production stage (default, last stage)
docker build -t myapp:prod .
```

**Complete Example with Tests:**

```dockerfile
# ========================================
# Base stage
# ========================================
FROM node:14-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

# ========================================
# Development stage
# ========================================
FROM base AS development
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# ========================================
# Test stage
# ========================================
FROM base AS test
COPY . .
RUN npm run lint
RUN npm run test
RUN npm run coverage

# ========================================
# Build stage
# ========================================
FROM base AS build
COPY . .
RUN npm run build

# ========================================
# Production stage
# ========================================
FROM nginx:alpine AS production

# Install curl for healthcheck
RUN apk add --no-cache curl

# Copy built assets
COPY --from=build /app/build /usr/share/nginx/html

# Custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Non-root user
RUN chown -R nginx:nginx /usr/share/nginx/html

USER nginx

EXPOSE 8080

HEALTHCHECK CMD curl -f http://localhost:8080/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**Usage:**
```bash
# Development
docker build --target development -t myapp:dev .
docker run -p 3000:3000 -v $(pwd):/app myapp:dev

# Run tests
docker build --target test -t myapp:test .

# Production
docker build -t myapp:prod .
docker run -p 80:8080 myapp:prod
```

**Key Takeaways:**
1. **Use multi-stage** for smaller, more secure images
2. **Copy package files first** for better caching
3. **Use .dockerignore** to exclude unnecessary files
4. **Name stages** for clarity and targeted builds
5. **Run as non-root** in production
6. **Add health checks** for reliability
7. **Optimize layer order** for fast builds

---

### Question 32
**Dockerfile:**
```dockerfile
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM scratch
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

**Tasks:**
a. What will likely fail?  
b. Fix the issues

**Solution:**

**a. What Will Fail:**

1. **Error:** `scratch` image has no shell, no libraries, no certificates
2. **Error:** If Go binary isn't statically linked, it will fail with:
```
standard_init_linux.go: exec user process caused: no such file or directory
```
3. **Error:** Missing SSL/TLS certificates for HTTPS requests
4. **Error:** No timezone data if application uses time zones
5. **Error:** Missing /tmp directory if application creates temp files

**Why Scratch is Difficult:**
```
scratch = completely empty filesystem
- No /bin/sh
- No /lib (no libc, libpthread, etc.)
- No /usr
- No /etc/ssl/certs (no CA certificates)
- No timezone data
- Nothing!
```

**b. Fixed Versions:**

**Option 1: Proper static binary for scratch:**
```dockerfile
# Build stage
FROM golang:1.17-alpine AS builder
WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source
COPY . .

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -o app .

# Production stage
FROM scratch

# Copy CA certificates
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy timezone data
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Copy binary
COPY --from=builder /app/app /app

ENTRYPOINT ["/app"]
```

**Option 2: Use alpine instead of scratch (recommended):**
```dockerfile
# Build stage
FROM golang:1.17-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o app .

# Production stage
FROM alpine:latest

# Install CA certificates
RUN apk --no-cache add ca-certificates tzdata

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /app

# Copy binary
COPY --from=builder /app/app .

# Set ownership
RUN chown appuser:appuser /app/app

USER appuser

ENTRYPOINT ["./app"]
```

**Option 3: Distroless (Google's minimal images):**
```dockerfile
# Build stage
FROM golang:1.17 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o app .

# Production stage
FROM gcr.io/distroless/static:nonroot

# Copy binary
COPY --from=builder /app/app /app

ENTRYPOINT ["/app"]
```

**Option 4: Complete production-ready example:**
```dockerfile
# ========================================
# Stage 1: Build
# ========================================
FROM golang:1.17-alpine AS builder

# Install build dependencies
RUN apk add --no-cache git ca-certificates tzdata

WORKDIR /build

# Download dependencies
COPY go.mod go.sum ./
RUN go mod download
RUN go mod verify

# Copy source code
COPY . .

# Build binary
# CGO_ENABLED=0: static binary
# -ldflags: reduce binary size and remove debug info
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a \
    -o app \
    ./cmd/server

# ========================================
# Stage 2: Production
# ========================================
FROM alpine:3.18

# Install runtime dependencies
RUN apk --no-cache add \
    ca-certificates \
    tzdata \
    && update-ca-certificates

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser && \
    mkdir -p /app/data && \
    chown -R appuser:appuser /app

WORKDIR /app

# Copy binary from builder
COPY --from=builder /build/app .

# Copy any additional files needed
COPY --from=builder /build/config ./config

# Set ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/app/app", "--health-check"]

# Run
ENTRYPOINT ["./app"]
CMD ["--config", "config/production.yaml"]
```

**Explanation for Beginners:**

**Understanding Scratch:**

```
FROM scratch = Start with nothing
- Smallest possible image
- Only works for statically linked binaries
- No debugging tools
- No shell for troubleshooting
- Ultra-secure (attack surface = your binary only)
```

**Static vs Dynamic Linking:**

**Dynamic linking (default):**
```go
// Binary depends on system libraries
go build -o app

// Needs:
// - libc.so
// - libpthread.so
// - libdl.so
// etc.
```

**Static linking:**
```go
// Binary includes all dependencies
CGO_ENABLED=0 go build -o app

// Self-contained
// Works on scratch
```

**When to Use Each Base Image:**

**1. scratch:**
```dockerfile
FROM scratch
# Pros:
# - Smallest possible (few MB)
# - Maximum security
# Cons:
# - Requires static binary
# - No debugging tools
# - No shell
# Use for: Simple statically-linked Go binaries
```

**2. alpine:**
```dockerfile
FROM alpine
# Pros:
# - Small (5MB base)
# - Has package manager (apk)
# - Has shell for debugging
# Cons:
# - Uses musl libc (compatibility issues)
# Use for: Most containerized applications
```

**3. distroless:**
```dockerfile
FROM gcr.io/distroless/static
# Pros:
# - Minimal (similar to alpine)
# - Has CA certs, timezone data
# - More compatible than scratch
# Cons:
# - No shell (harder to debug)
# - No package manager
# Use for: Production Go/Java/Python apps
```

**4. debian-slim / ubuntu:**
```dockerfile
FROM debian:bullseye-slim
# Pros:
# - Familiar environment
# - Wide compatibility
# - Easy debugging
# Cons:
# - Larger (50MB+)
# Use for: Applications with many dependencies
```

**Fixing Common Scratch Issues:**

**Issue 1: Missing CA certificates**
```dockerfile
FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
# Now HTTPS requests work
```

**Issue 2: Missing timezone data**
```dockerfile
FROM scratch
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
# Now time.LoadLocation() works
```

**Issue 3: Missing /tmp directory**
```dockerfile
FROM scratch
# Can't create /tmp in scratch, so:
# Either use alpine, or configure app to use /app/tmp
```

**Issue 4: Dynamic linking errors**
```bash
# Error: no such file or directory
# Solution: Build static binary
CGO_ENABLED=0 go build -o app
```

**Complete Comparison:**

```dockerfile
# ========================================
# Option 1: Scratch (3MB final image)
# ========================================
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -ldflags='-w -s' -o app

FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]

# ========================================
# Option 2: Alpine (8MB final image)
# ========================================
FROM golang:1.17-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]

# ========================================
# Option 3: Distroless (10MB final image)
# ========================================
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app

FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

**Best Practices:**
1. **Use alpine** for most cases (good balance)
2. **Use distroless** for extra security without shell
3. **Use scratch** only for simple static binaries
4. **Always build static** when using scratch/distroless
5. **Include CA certificates** for HTTPS
6. **Add timezone data** if needed
7. **Run as non-root** user
8. **Add health checks** for production

---

### Question 33
**Dockerfile:**
```dockerfile
FROM maven:3.8-jdk-11 AS build
COPY . /app
RUN mvn -f /app/pom.xml clean package

FROM tomcat:9
COPY --from=0 /app/target/*.war /usr/local/tomcat/webapps/
```

**Tasks:**
a. Find the issues with stage references  
b. Improve the Dockerfile

**Solution:**

**a. Issues:**

1. **Issue:** Using numeric stage reference `--from=0` instead of named stage
2. **Issue:** Wildcard `*.war` might match multiple files or none
3. **Issue:** No WORKDIR in build stage
4. **Issue:** Copying entire context (including .git, etc.)
5. **Issue:** No layer caching for Maven dependencies
6. **Issue:** No control over final WAR filename

**What's Wrong with `--from=0`:**
```dockerfile
COPY --from=0   # Refers to first stage by index
                # Fragile: breaks if stages reordered
                # Unclear: which stage is 0?
```

**b. Improved Dockerfiles:**

**Option 1: Basic fix with named stages:**
```dockerfile
FROM maven:3.8-jdk-11 AS build
WORKDIR /build
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM tomcat:9-jdk11
COPY --from=build /build/target/myapp.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

**Option 2: With dependency caching:**
```dockerfile
# ========================================
# Stage 1: Dependencies
# ========================================
FROM maven:3.8-jdk-11 AS dependencies
WORKDIR /build

# Copy POM and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# ========================================
# Stage 2: Build
# ========================================
FROM maven:3.8-jdk-11 AS build
WORKDIR /build

# Copy dependencies layer
COPY --from=dependencies /root/.m2 /root/.m2
COPY pom.xml .

# Copy source and build
COPY src ./src
RUN mvn package -DskipTests -B && \
    mv target/*.war target/app.war

# ========================================
# Stage 3: Production
# ========================================
FROM tomcat:9-jdk11-openjdk-slim

# Remove default webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy WAR file
COPY --from=build /build/target/app.war /usr/local/tomcat/webapps/ROOT.war

# Expose port
EXPOSE 8080

# Run Tomcat
CMD ["catalina.sh", "run"]
```

**Option 3: Complete best practices:**
```dockerfile
# ========================================
# Stage 1: Build environment
# ========================================
FROM maven:3.8-jdk-11-slim AS builder

# Install necessary tools
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build

# ========================================
# Stage 2: Download dependencies
# ========================================
FROM builder AS dependencies

# Copy POM file
COPY pom.xml .

# Download all dependencies (cached layer)
RUN mvn dependency:go-offline -B && \
    mvn dependency:resolve-plugins -B

# ========================================
# Stage 3: Build application
# ========================================
FROM dependencies AS build

# Copy source code
COPY src ./src

# Build application
RUN mvn clean package -DskipTests -B && \
    # Ensure single WAR file
    war_count=$(ls target/*.war | wc -l) && \
    if [ "$war_count" -ne 1 ]; then \
        echo "Error: Expected 1 WAR file, found $war_count" && exit 1; \
    fi && \
    mv target/*.war target/application.war

# ========================================
# Stage 4: Production
# ========================================
FROM tomcat:9.0-jdk11-openjdk-slim AS production

# Remove default Tomcat applications
RUN rm -rf /usr/local/tomcat/webapps/*

# Create non-root user
RUN groupadd -r tomcat && \
    useradd -r -g tomcat -d /usr/local/tomcat -s /sbin/nologin tomcat && \
    chown -R tomcat:tomcat /usr/local/tomcat

# Copy WAR file
COPY --from=build /build/target/application.war /usr/local/tomcat/webapps/ROOT.war

# Tomcat configuration
ENV CATALINA_OPTS="-Xms512m -Xmx1024m -XX:+UseG1GC"

# Switch to non-root user
USER tomcat

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/ || exit 1

# Run Tomcat
CMD ["catalina.sh", "run"]
```

**Explanation for Beginners:**

**Stage References:**

**Bad: Numeric reference**
```dockerfile
FROM maven:3.8 AS build
# ... build stage ...

FROM tomcat:9
COPY --from=0 /app/target/*.war /usr/local/tomcat/webapps/
#          ↑
#     Fragile! If you add another stage before this,
#     the index changes and breaks
```

**Good: Named reference**
```dockerfile
FROM maven:3.8 AS build
# ... build stage ...

FROM tomcat:9
COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/
#          ↑
#     Clear and stable! Adding stages doesn't break this
```

**Why Named Stages Are Better:**

1. **Clarity:**
```dockerfile
COPY --from=build ...     # Clear: copying from build stage
COPY --from=0 ...         # Unclear: which stage is 0?
```

2. **Maintainability:**
```dockerfile
# If you add a new stage:
FROM node:14 AS frontend    # New stage
FROM maven:3.8 AS build     # Now this is stage 1, not 0!
FROM tomcat:9
COPY --from=build ...       # Still works!
COPY --from=1 ...           # Breaks! Now points to wrong stage
```

3. **Readability:**
```dockerfile
COPY --from=builder /app/binary /usr/local/bin/
COPY --from=assets /static /var/www/
COPY --from=configs /etc/app.conf /etc/
# Can understand without counting stages
```

**Maven Multi-Stage Best Practices:**

**1. Separate dependency download:**
```dockerfile
# Stage 1: Download dependencies
FROM maven:3.8-jdk-11 AS deps
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline

# Stage 2: Build
FROM maven:3.8-jdk-11 AS build
COPY --from=deps /root/.m2 /root/.m2
COPY . .
RUN mvn package
```

**Benefits:**
- Dependencies cached separately
- Fast rebuilds when only source changes
- Network requests minimized

**2. Handle WAR file carefully:**
```dockerfile
# Bad: Wildcard might match multiple files
COPY --from=build /app/target/*.war /webapps/

# Good: Rename to specific name during build
RUN mvn package && \
    mv target/*.war target/app.war

COPY --from=build /app/target/app.war /webapps/ROOT.war
```

**3. Use explicit stage names:**
```dockerfile
FROM maven:3.8-jdk-11 AS maven-build
FROM node:14 AS node-build
FROM tomcat:9 AS production

COPY --from=maven-build ...
COPY --from=node-build ...
```

**Complete Working Example:**

```dockerfile
# ========================================
# Build arguments
# ========================================
ARG MAVEN_VERSION=3.8-jdk-11
ARG TOMCAT_VERSION=9.0-jdk11-openjdk-slim

# ========================================
# Stage 1: Base builder
# ========================================
FROM maven:${MAVEN_VERSION} AS base

# ========================================
# Stage 2: Dependencies
# ========================================
FROM base AS dependencies
WORKDIR /build

# Copy POM and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B && \
    mvn dependency:resolve-plugins -B

# ========================================
# Stage 3: Build
# ========================================
FROM base AS builder
WORKDIR /build

# Copy cached dependencies
COPY --from=dependencies /root/.m2 /root/.m2

# Copy POM
COPY pom.xml .

# Copy source
COPY src ./src

# Build
RUN mvn clean package -DskipTests -B && \
    # Verify WAR file exists
    if [ ! -f target/*.war ]; then \
        echo "Error: WAR file not found" && exit 1; \
    fi && \
    # Rename to specific name
    mv target/*.war target/application.war && \
    # Log WAR file info
    ls -lh target/application.war

# ========================================
# Stage 4: Production
# ========================================
FROM tomcat:${TOMCAT_VERSION} AS production

# Install curl for health checks
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Remove default webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy WAR file
COPY --from=builder /build/target/application.war \
     /usr/local/tomcat/webapps/ROOT.war

# Tomcat configuration
ENV CATALINA_OPTS="-Xms512m -Xmx1024m \
    -XX:+UseG1GC \
    -XX:MaxGCPauseMillis=200 \
    -Djava.security.egd=file:/dev/./urandom"

# Create non-root user
RUN groupadd -r -g 1000 tomcat && \
    useradd -r -u 1000 -g tomcat tomcat && \
    chown -R tomcat:tomcat /usr/local/tomcat && \
    chmod -R 755 /usr/local/tomcat

USER tomcat

EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8080/ || exit 1

CMD ["catalina.sh", "run"]
```

**Usage:**
```bash
# Build with default versions
docker build -t myapp .

# Build with custom versions
docker build \
  --build-arg MAVEN_VERSION=3.9-jdk-17 \
  --build-arg TOMCAT_VERSION=10.0-jdk17 \
  -t myapp .

# Build specific stage for testing
docker build --target builder -t myapp:build .

# Run
docker run -d -p 8080:8080 --name myapp-container myapp
```

**.dockerignore:**
```
target
.git
.gitignore
.mvn
*.md
.classpath
.project
.settings
.vscode
.idea
```

**Key Takeaways:**
1. **Always use named stages**, never numeric (--from=0)
2. **Cache dependencies separately** for faster builds
3. **Handle WAR files explicitly**, avoid wildcards
4. **Remove default Tomcat webapps**
5. **Run as non-root** user
6. **Add health checks**
7. **Use .dockerignore** to exclude unnecessary files
8. **Set JVM options** appropriately
9. **Use specific version tags** for reproducibility

---

### Question 34
**Dockerfile:**
```dockerfile
FROM python:3.9 AS base
COPY requirements.txt .
RUN pip install -r requirements.txt

FROM base AS dev
COPY . /app
CMD ["python", "app.py"]

FROM base AS prod
COPY . /app
RUN python -m pytest
CMD ["python", "app.py"]
```

**Tasks:**
a. What's the issue with this setup?  
b. How should different environments be handled?

**Solution:**

**a. Issues:**

1. **Issue:** Tests run during production build (makes builds slower and can fail deployment)
2. **Issue:** Both stages copy to different locations (inconsistent)
3. **Issue:** No WORKDIR set
4. **Issue:** Dev and prod stages are too similar (duplicated code)
5. **Issue:** Running tests in production image build is an anti-pattern
6. **Issue:** No differentiation in dependencies (dev vs prod)

**What's Wrong:**
```
Production build process:
1. Build base stage
2. Build prod stage
3. Run tests in prod stage ← BAD! Tests should be separate
4. If tests fail → entire deployment fails
5. If tests pass → image includes test dependencies
```

**b. Better Approaches:**

**Option 1: Separate test stage:**
```dockerfile
# Base stage
FROM python:3.9-slim AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Test stage (run but don't include in final image)
FROM base AS test
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt
COPY . .
RUN python -m pytest && \
    python -m pylint src && \
    python -m black --check src

# Development stage
FROM base AS development
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt
COPY . .
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0", "--debug"]

# Production stage
FROM base AS production
COPY . .
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**Option 2: Environment-based approach:**
```dockerfile
# ========================================
# Stage 1: Base dependencies
# ========================================
FROM python:3.9-slim AS base
WORKDIR /app

# System dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    pip install --no-cache-dir gunicorn

# ========================================
# Stage 2: Development dependencies
# ========================================
FROM base AS dev-dependencies
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

# ========================================
# Stage 3: Test
# ========================================
FROM dev-dependencies AS test
COPY . .

# Run tests
RUN python -m pytest tests/ -v --cov=src --cov-report=term-missing && \
    python -m pylint src && \
    python -m mypy src && \
    python -m black --check src && \
    python -m isort --check-only src

# ========================================
# Stage 4: Development
# ========================================
FROM dev-dependencies AS development
COPY . .

ENV FLASK_ENV=development
ENV FLASK_DEBUG=1

EXPOSE 5000

CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]

# ========================================
# Stage 5: Production
# ========================================
FROM base AS production

# Copy application code
COPY src ./src
COPY config ./config

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

ENV FLASK_ENV=production

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["gunicorn", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--worker-class", "sync", \
     "--timeout", "30", \
     "--access-logfile", "-", \
     "--error-logfile", "-", \
     "src.app:app"]
```

**Option 3: With separate requirements files:**

**requirements.txt** (production):
```txt
flask==2.3.0
requests==2.31.0
sqlalchemy==2.0.0
psycopg2-binary==2.9.6
python-dotenv==1.0.0
```

**requirements-dev.txt** (development):
```txt
-r requirements.txt
pytest==7.4.0
pytest-cov==4.1.0
black==23.7.0
pylint==2.17.5
mypy==1.4.1
isort==5.12.0
flask-debugtoolbar==0.13.1
```

**Dockerfile:**
```dockerfile
# ========================================
# Base
# ========================================
FROM python:3.9-slim AS base
WORKDIR /app

# ========================================
# Dependencies - Production
# ========================================
FROM base AS prod-deps
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ========================================
# Dependencies - Development
# ========================================
FROM base AS dev-deps
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

# ========================================
# Test Stage (optional, run separately)
# ========================================
FROM dev-deps AS test
COPY . .
RUN python -m pytest

# ========================================
# Development
# ========================================
FROM dev-deps AS development
COPY . .
ENV FLASK_ENV=development
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]

# ========================================
# Production
# ========================================
FROM prod-deps AS production
COPY src ./src
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
ENV FLASK_ENV=production
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "src.app:app"]
```

**Explanation for Beginners:**

**Why Separate Test Stage:**

**Bad: Tests in production image**
```dockerfile
FROM base AS prod
COPY . /app
RUN pytest              ← Runs during production build
CMD ["python", "app.py"]
# Problems:
# 1. Slow builds (tests run every deployment)
# 2. Deployment fails if tests fail
# 3. Test dependencies in production image
```

**Good: Separate test stage**
```dockerfile
FROM base AS test
COPY . .
RUN pytest              ← Separate stage

FROM base AS prod
COPY . /app             ← Tests don't run here
CMD ["python", "app.py"]
# Benefits:
# 1. Fast production builds
# 2. Tests run in CI/CD, not deployment
# 3. Smaller production image
```

**Multi-Environment Strategy:**

**Approach 1: Different targets**
```bash
# Development
docker build --target development -t myapp:dev .
docker run -p 5000:5000 -v $(pwd):/app myapp:dev

# Run tests
docker build --target test -t myapp:test .

# Production
docker build --target production -t myapp:prod .
docker run -p 8000:8000 myapp:prod
```

**Approach 2: Build args**
```dockerfile
ARG ENV=production

FROM base AS final

COPY . /app

RUN if [ "$ENV" = "development" ]; then \
      pip install -r requirements-dev.txt; \
    fi

CMD if [ "$ENV" = "development" ]; then \
      flask run --debug; \
    else \
      gunicorn app:app; \
    fi
```

```bash
docker build --build-arg ENV=development -t myapp:dev .
docker build --build-arg ENV=production -t myapp:prod .
```

**Approach 3: Separate Dockerfiles**
```
project/
├── Dockerfile              ← Production
├── Dockerfile.dev          ← Development
├── Dockerfile.test         ← Testing
└── docker-compose.yml      ← Orchestration
```

**Best Practice Pattern:**

```dockerfile
# ========================================
# Stage 1: Base
# ========================================
FROM python:3.9-slim AS base
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc libpq-dev && rm -rf /var/lib/apt/lists/*

# ========================================
# Stage 2: Builder (install dependencies)
# ========================================
FROM base AS builder
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt

# ========================================
# Stage 3: Test Dependencies
# ========================================
FROM base AS test-builder
COPY requirements-dev.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements-dev.txt

# ========================================
# Stage 4: Test (Run tests, don't include in final image)
# ========================================
FROM base AS test
COPY --from=test-builder /wheels /wheels
COPY requirements-dev.txt .
RUN pip install --no-cache /wheels/*
COPY . .
RUN pytest -v --cov=src && \
    pylint src && \
    mypy src

# ========================================
# Stage 5: Development
# ========================================
FROM base AS development
COPY --from=test-builder /wheels /wheels
COPY requirements-dev.txt .
RUN pip install --no-cache /wheels/*
COPY . .
ENV FLASK_ENV=development
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]

# ========================================
# Stage 6: Production (default)
# ========================================
FROM base AS production
COPY --from=builder /wheels /wheels
COPY requirements.txt .
RUN pip install --no-cache /wheels/* && \
    pip install --no-cache-dir gunicorn && \
    rm -rf /wheels
COPY src ./src
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
ENV FLASK_ENV=production
EXPOSE 8000
HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1
CMD ["gunicorn", "-b", "0.0.0.0:8000", "-w", "4", "src.app:app"]
```

**CI/CD Integration:**

```yaml
# .github/workflows/docker.yml
name: Docker Build and Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: docker build --target test -t myapp:test .

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build production
        run: docker build --target production -t myapp:prod .
      - name: Push to registry
        run: docker push myapp:prod
```

**Key Takeaways:**
1. **Separate test stage** - don't run tests in production build
2. **Use named stages** - clear and maintainable
3. **Different dependencies** - dev vs prod requirements
4. **Target specific stages** - flexible builds
5. **CI/CD runs tests** - not during deployment
6. **Keep production slim** - no test/dev dependencies
7. **Use health checks** - monitor production containers
8. **Run as non-root** - security best practice

---

### Question 35
**Dockerfile:**
```dockerfile
FROM ubuntu:20.04 AS build
RUN apt-get update && apt-get install -y build-essential
COPY source.c .
RUN gcc source.c -o app

FROM ubuntu:20.04
COPY --from=build app /usr/local/bin/
CMD ["app"]
```

**Tasks:**
a. What optimization is possible?  
b. Improve the final image size

**Solution:**

**a. Optimization Issues:**

1. **Issue:** Using full Ubuntu (70MB) when alpine (5MB) would work
2. **Issue:** Second stage still has full Ubuntu when binary doesn't need it
3. **Issue:** No static compilation - binary depends on libc
4. **Issue:** Missing library dependencies might cause runtime errors
5. **Issue:** Could use `scratch` or `distroless` for minimal image

**Current Image Sizes:**
```
Build stage: ~200MB (Ubuntu + build-essential)
Final stage: ~70MB (Ubuntu + tiny binary)
Total: Binary might be 100KB, but image is 70MB!
```

**b. Improved Versions:**

**Option 1: Use Alpine (small but with shell):**
```dockerfile
# Build stage
FROM alpine:3.18 AS build
RUN apk add --no-cache gcc musl-dev
WORKDIR /build
COPY source.c .
RUN gcc -static source.c -o app

# Production stage
FROM alpine:3.18
COPY --from=build /build/app /usr/local/bin/app
CMD ["app"]
```

**Image Size: ~7MB**

**Option 2: Use distroless (no shell, very secure):**
```dockerfile
# Build stage
FROM gcc:12-alpine AS build
WORKDIR /build
COPY source.c .
RUN gcc -static -s -O2 source.c -o app

# Production stage
FROM gcr.io/distroless/static:nonroot
COPY --from=build /build/app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

**Image Size: ~2MB**

**Option 3: Use scratch (absolute minimum):**
```dockerfile
# Build stage
FROM gcc:12-alpine AS build
WORKDIR /build
COPY source.c .
# Create fully static binary
RUN gcc -static -s -O2 -o app source.c && \
    strip -s app

# Production stage
FROM scratch
COPY --from=build /build/app /app
ENTRYPOINT ["/app"]
```

**Image Size: ~100KB (just the binary!)**

**Option 4: Complete production-ready example:**
```dockerfile
# ========================================
# Stage 1: Build
# ========================================
FROM gcc:12-alpine AS builder

# Install build dependencies
RUN apk add --no-cache \
    musl-dev \
    linux-headers

WORKDIR /build

# Copy source files
COPY *.c *.h ./
COPY Makefile .

# Build static binary
RUN make clean && \
    make CFLAGS="-static -s -O2 -Wall -Wextra" && \
    strip -s app && \
    # Verify it's actually static
    ldd app 2>&1 | grep -q "not a dynamic executable"

# ========================================
# Stage 2: Test
# ========================================
FROM builder AS test
COPY tests/ ./tests/
RUN make test

# ========================================
# Stage 3: Production
# ========================================
FROM scratch AS production

# Copy CA certificates for HTTPS (if needed)
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Create minimal /etc/passwd for non-root user
COPY --from=builder /etc/passwd /etc/passwd

# Copy binary
COPY --from=builder /build/app /app

# Use non-root user (UID 65534 = nobody)
USER 65534:65534

ENTRYPOINT ["/app"]
```

**Image Size: ~100KB**

**Explanation for Beginners:**

**Understanding Image Sizes:**

```
Base Image Sizes:
- scratch: 0 bytes
- busybox: 1-5 MB
- alpine: 5-7 MB
- distroless/static: 2 MB
- debian-slim: 50-70 MB
- ubuntu: 70-100 MB
- ubuntu with build tools: 200+ MB
```

**Static vs Dynamic Linking:**

**Dynamic linking (default):**
```c
// source.c
#include <stdio.h>
int main() {
    printf("Hello\n");
    return 0;
}
```

```bash
gcc source.c -o app
ldd app
# Output:
#   linux-vdso.so.1
#   libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
#   /lib64/ld-linux-x86-64.so.2
```

Binary needs these libraries at runtime!

**Static linking:**
```bash
gcc -static source.c -o app
ldd app
# Output:
#   not a dynamic executable

# All libraries compiled into binary
# Works on scratch/distroless
```

**Size Comparison Example:**

**Version 1: Ubuntu base (70MB)**
```dockerfile
FROM ubuntu:20.04
COPY app /usr/local/bin/
```

**Version 2: Alpine base (7MB)**
```dockerfile
FROM alpine:latest
COPY app /usr/local/bin/
```

**Version 3: Distroless (2MB)**
```dockerfile
FROM gcr.io/distroless/static
COPY app /
```

**Version 4: Scratch (~100KB)**
```dockerfile
FROM scratch
COPY app /
```

**Complete C Application Example:**

**source.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void handle_signal(int sig) {
    printf("Received signal %d, shutting down...\n", sig);
    exit(0);
}

int main() {
    signal(SIGTERM, handle_signal);
    signal(SIGINT, handle_signal);
    
    printf("Application started\n");
    
    while(1) {
        printf("Running...\n");
        sleep(5);
    }
    
    return 0;
}
```

**Makefile:**
```makefile
CC=gcc
CFLAGS=-static -s -O2 -Wall -Wextra
TARGET=app

all: $(TARGET)

$(TARGET): source.c
	$(CC) $(CFLAGS) -o $(TARGET) source.c
	strip -s $(TARGET)

test:
	./tests/run_tests.sh

clean:
	rm -f $(TARGET)

.PHONY: all test clean
```

**Dockerfile:**
```dockerfile
# ========================================
# Build stage
# ========================================
FROM gcc:12-alpine AS builder

# Install build tools
RUN apk add --no-cache \
    make \
    musl-dev \
    linux-headers

WORKDIR /build

# Copy source and build configuration
COPY source.c Makefile ./

# Build static binary
RUN make && \
    # Verify it's static
    ! ldd app && \
    # Show binary size
    ls -lh app

# ========================================
# Test stage
# ========================================
FROM builder AS test
COPY tests/ ./tests/
RUN chmod +x tests/*.sh && \
    make test

# ========================================
# Production stage
# ========================================
FROM scratch

# Copy CA certificates (if app makes HTTPS calls)
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /build/app /app

# Set user (create /etc/passwd entry)
COPY --from=builder --chown=65534:65534 /build/app /app

USER 65534:65534

ENTRYPOINT ["/app"]
```

**Building and Size Comparison:**

```bash
# Build
docker build -t myapp:latest .

# Check size
docker images myapp:latest
# REPOSITORY   TAG       SIZE
# myapp        latest    100KB

# Run
docker run --rm myapp:latest

# Compare with Ubuntu-based version
docker build -f Dockerfile.ubuntu -t myapp:ubuntu .
docker images
# REPOSITORY   TAG       SIZE
# myapp        latest    100KB    ← scratch-based
# myapp        ubuntu    70MB     ← ubuntu-based
```

**Advanced: Multi-arch builds:**

```dockerfile
# Build for multiple architectures
FROM --platform=$BUILDPLATFORM gcc:12-alpine AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM

WORKDIR /build
COPY source.c .

# Build for target platform
RUN case "$TARGETPLATFORM" in \
      "linux/amd64")  gcc -static -s -O2 source.c -o app ;; \
      "linux/arm64")  gcc -static -s -O2 source.c -o app ;; \
      "linux/arm/v7") gcc -static -s -O2 source.c -o app ;; \
      *) echo "Unsupported platform: $TARGETPLATFORM" && exit 1 ;; \
    esac

FROM scratch
COPY --from=builder /build/app /app
ENTRYPOINT ["/app"]
```

**Build command:**
```bash
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t myapp:latest --push .
```

**When to Use Each Base:**

**1. scratch:**
- Fully static binaries
- Minimal attack surface
- Smallest size
- No debugging tools

**2. distroless:**
- Static or mostly static binaries
- Want CA certs, timezone data
- Security-focused
- No shell

**3. alpine:**
- Need package manager
- Need shell for debugging
- Good balance of size and features
- musl libc (some compatibility issues)

**4. debian-slim/ubuntu:**
- Need many dependencies
- Compatibility important
- Familiar environment
- Larger size acceptable

**Key Takeaways:**
1. **Use smallest base image** that works
2. **Build static binaries** for scratch/distroless
3. **Strip binaries** to reduce size (`strip -s`)
4. **Use compiler optimizations** (`-O2`, `-O3`)
5. **Multi-stage builds** keep final image small
6. **Verify binary is static**: `ldd app` should say "not a dynamic executable"
7. **Consider security** - smaller image = smaller attack surface
8. **Test in target environment** - static binaries might behave differently
