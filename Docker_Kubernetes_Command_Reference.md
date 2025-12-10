# Docker & Kubernetes Command Reference Cheatsheet

## Quick Navigation
- [Docker Commands](#docker-commands)
- [Docker Compose Commands](#docker-compose-commands)
- [Kubernetes Commands](#kubernetes-commands)
- [Kubectl Shortcuts](#kubectl-shortcuts)
- [Troubleshooting Commands](#troubleshooting-commands)

---

## Docker Commands

### Image Management

```bash
# Search for images
docker search nginx
docker search ubuntu --filter stars=100

# Pull images
docker pull nginx
docker pull nginx:alpine
docker pull ubuntu:20.04

# List images
docker images
docker image ls
docker images --filter "dangling=true"

# Remove images
docker rmi nginx
docker rmi -f nginx                    # Force remove
docker image rm nginx:alpine
docker image prune                     # Remove unused images
docker image prune -a                  # Remove all unused images

# Build images
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker build --no-cache -t myapp:1.0 .

# Tag images
docker tag myapp:1.0 username/myapp:1.0
docker tag myapp:1.0 myregistry.com/myapp:1.0

# Push/Pull to registry
docker login
docker push username/myapp:1.0
docker pull username/myapp:1.0
docker logout

# Inspect images
docker inspect nginx
docker history nginx
```

### Container Management

```bash
# Run containers
docker run nginx
docker run -d nginx                    # Detached mode
docker run -d --name my-nginx nginx    # Custom name
docker run -d -p 8080:80 nginx         # Port mapping
docker run -d -p 8080:80 -v $(pwd):/app nginx  # Volume mount
docker run -d -e ENV_VAR=value nginx   # Environment variables
docker run -it ubuntu bash             # Interactive terminal
docker run --rm alpine echo "hello"    # Auto-remove after exit

# List containers
docker ps                              # Running containers
docker ps -a                           # All containers
docker ps -q                           # Only IDs
docker ps --filter "status=exited"

# Container operations
docker start my-nginx
docker stop my-nginx
docker restart my-nginx
docker pause my-nginx
docker unpause my-nginx
docker kill my-nginx

# Remove containers
docker rm my-nginx
docker rm -f my-nginx                  # Force remove
docker container prune                 # Remove all stopped containers

# Execute commands
docker exec my-nginx ls /etc
docker exec -it my-nginx bash
docker exec -it my-nginx sh

# View logs
docker logs my-nginx
docker logs -f my-nginx                # Follow logs
docker logs --tail 100 my-nginx        # Last 100 lines
docker logs --since 10m my-nginx       # Last 10 minutes
docker logs --timestamps my-nginx

# Inspect containers
docker inspect my-nginx
docker stats my-nginx                  # Resource usage
docker top my-nginx                    # Running processes
docker port my-nginx                   # Port mappings

# Copy files
docker cp file.txt my-nginx:/app/
docker cp my-nginx:/app/file.txt ./
```

### Network Management

```bash
# List networks
docker network ls

# Create network
docker network create my-network
docker network create --driver bridge my-network

# Inspect network
docker network inspect my-network

# Connect/Disconnect containers
docker network connect my-network my-nginx
docker network disconnect my-network my-nginx

# Remove network
docker network rm my-network
docker network prune                   # Remove unused networks
```

### Volume Management

```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume
docker volume prune                    # Remove unused volumes

# Use volumes
docker run -d -v my-volume:/app nginx
docker run -d -v $(pwd):/app nginx     # Bind mount
docker run -d --mount source=my-volume,target=/app nginx
```

### System Commands

```bash
# System information
docker info
docker version
docker system df                       # Disk usage

# Clean up
docker system prune                    # Remove unused data
docker system prune -a                 # Remove all unused data
docker system prune --volumes          # Include volumes

# Events
docker events
docker events --since 10m
```

---

## Docker Compose Commands

```bash
# Start services
docker-compose up
docker-compose up -d                   # Detached mode
docker-compose up --build              # Rebuild images
docker-compose up --force-recreate     # Recreate containers

# Stop services
docker-compose stop
docker-compose down                    # Stop and remove
docker-compose down -v                 # Remove volumes too

# View status
docker-compose ps
docker-compose ps -a
docker-compose top

# Logs
docker-compose logs
docker-compose logs -f                 # Follow logs
docker-compose logs service-name       # Specific service

# Execute commands
docker-compose exec service-name bash
docker-compose exec service-name ls

# Scale services
docker-compose up -d --scale web=3

# Build
docker-compose build
docker-compose build --no-cache

# Configuration
docker-compose config                  # Validate and view config
docker-compose config --services       # List services
```

---

## Kubernetes Commands

### Cluster Management

```bash
# Cluster info
kubectl cluster-info
kubectl version
kubectl api-resources

# Node management
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl top nodes
kubectl cordon <node-name>             # Mark unschedulable
kubectl uncordon <node-name>           # Mark schedulable
kubectl drain <node-name>              # Drain node
```

### Pod Management

```bash
# List pods
kubectl get pods
kubectl get pods -n namespace
kubectl get pods -A                    # All namespaces
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -l app=nginx          # Filter by label
kubectl get pods --field-selector status.phase=Running

# Create pods
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --port=80
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Describe pods
kubectl describe pod <pod-name>
kubectl describe pod <pod-name> -n namespace

# Delete pods
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --grace-period=0 --force
kubectl delete pods --all

# Pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> -f             # Follow logs
kubectl logs <pod-name> --tail=100     # Last 100 lines
kubectl logs <pod-name> --since=10m    # Last 10 minutes
kubectl logs <pod-name> --previous     # Previous container

# Execute commands
kubectl exec <pod-name> -- ls
kubectl exec -it <pod-name> -- bash
kubectl exec <pod-name> -c <container-name> -- bash

# Port forwarding
kubectl port-forward <pod-name> 8080:80
kubectl port-forward svc/my-service 8080:80

# Copy files
kubectl cp <pod-name>:/path/to/file ./local/path
kubectl cp ./local/file <pod-name>:/path/to/file
```

### Deployment Management

```bash
# Create deployment
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx --image=nginx --replicas=3

# List deployments
kubectl get deployments
kubectl get deploy -A
kubectl get deploy -o wide

# Describe deployment
kubectl describe deployment <deployment-name>

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=5
kubectl scale deployment <deployment-name> --replicas=3 -n namespace

# Update deployment
kubectl set image deployment/<deployment-name> nginx=nginx:1.21
kubectl set env deployment/<deployment-name> ENV_VAR=value
kubectl edit deployment <deployment-name>

# Rollout management
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name> --to-revision=2
kubectl rollout pause deployment/<deployment-name>
kubectl rollout resume deployment/<deployment-name>
kubectl rollout restart deployment/<deployment-name>

# Delete deployment
kubectl delete deployment <deployment-name>
```

### Service Management

```bash
# List services
kubectl get services
kubectl get svc
kubectl get svc -A

# Create service
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl expose pod nginx --port=80 --name=nginx-service
kubectl create service clusterip my-service --tcp=80:8080

# Describe service
kubectl describe service <service-name>

# Delete service
kubectl delete service <service-name>
```

### ConfigMap and Secret Management

```bash
# ConfigMap
kubectl create configmap app-config --from-literal=key=value
kubectl create configmap app-config --from-file=config.properties
kubectl get configmaps
kubectl describe configmap <configmap-name>
kubectl edit configmap <configmap-name>
kubectl delete configmap <configmap-name>

# Secrets
kubectl create secret generic app-secret --from-literal=password=secret
kubectl create secret generic app-secret --from-file=ssh-key=~/.ssh/id_rsa
kubectl create secret tls tls-secret --cert=cert.crt --key=cert.key
kubectl create secret docker-registry reg-secret \
  --docker-server=registry.com \
  --docker-username=user \
  --docker-password=pass
kubectl get secrets
kubectl describe secret <secret-name>
kubectl get secret <secret-name> -o yaml
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
kubectl delete secret <secret-name>
```

### Namespace Management

```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace dev
kubectl create ns prod

# Delete namespace
kubectl delete namespace dev

# Set default namespace
kubectl config set-context --current --namespace=dev

# View current context
kubectl config current-context
kubectl config view
```

### Resource Management

```bash
# Apply configuration
kubectl apply -f deployment.yaml
kubectl apply -f ./k8s/
kubectl apply -f https://example.com/config.yaml

# Create from file
kubectl create -f deployment.yaml

# Delete from file
kubectl delete -f deployment.yaml

# Get all resources
kubectl get all
kubectl get all -n namespace
kubectl get all -A

# View resource definitions
kubectl explain pod
kubectl explain pod.spec
kubectl explain deployment.spec.template

# Edit resources
kubectl edit deployment <deployment-name>
kubectl edit pod <pod-name>

# Label resources
kubectl label pod <pod-name> env=production
kubectl label pod <pod-name> env-                # Remove label

# Annotate resources
kubectl annotate pod <pod-name> description="My pod"
```

### Autoscaling

```bash
# Create HPA
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=70

# List HPA
kubectl get hpa
kubectl get horizontalpodautoscaler

# Describe HPA
kubectl describe hpa <hpa-name>

# Delete HPA
kubectl delete hpa <hpa-name>
```

### RBAC

```bash
# Service Accounts
kubectl create serviceaccount my-sa
kubectl get serviceaccounts
kubectl describe serviceaccount <sa-name>

# Roles
kubectl create role pod-reader --verb=get,list --resource=pods
kubectl get roles
kubectl describe role <role-name>

# RoleBindings
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:my-sa
kubectl get rolebindings
kubectl describe rolebinding <rolebinding-name>

# ClusterRoles
kubectl create clusterrole cluster-admin --verb=* --resource=*
kubectl get clusterroles

# ClusterRoleBindings
kubectl create clusterrolebinding admin-binding --clusterrole=cluster-admin --user=admin
kubectl get clusterrolebindings

# Check permissions
kubectl auth can-i list pods
kubectl auth can-i create deployments
kubectl auth can-i delete pods --as=system:serviceaccount:default:my-sa
```

### Context and Configuration

```bash
# View contexts
kubectl config get-contexts
kubectl config current-context

# Switch context
kubectl config use-context <context-name>

# Set namespace
kubectl config set-context --current --namespace=dev

# View config
kubectl config view
kubectl config view --minify

# Set credentials
kubectl config set-credentials user --token=<token>
kubectl config set-cluster <cluster-name> --server=https://k8s.example.com
```

---

## Kubectl Shortcuts

### Aliases (Add to ~/.bashrc or ~/.zshrc)

```bash
# Basic shortcuts
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'

# Describe
alias kdp='kubectl describe pod'
alias kds='kubectl describe service'
alias kdd='kubectl describe deployment'

# Delete
alias kdel='kubectl delete'
alias kdelp='kubectl delete pod'

# Logs
alias kl='kubectl logs'
alias klf='kubectl logs -f'

# Exec
alias kex='kubectl exec -it'

# Apply/Delete
alias ka='kubectl apply -f'
alias kdelf='kubectl delete -f'

# Namespace
alias kgns='kubectl get namespaces'
alias kcn='kubectl config set-context --current --namespace'

# All resources
alias kga='kubectl get all'
alias kgaa='kubectl get all -A'

# Top
alias ktn='kubectl top nodes'
alias ktp='kubectl top pods'
```

### Quick Commands

```bash
# Get everything in one command
kubectl get pods,svc,deploy -n namespace

# Watch resources
kubectl get pods -w
kubectl get pods --watch

# Output formats
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Sorting
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.startTime

# Field selectors
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default

# Label selectors
kubectl get pods -l app=nginx
kubectl get pods -l 'app in (nginx,apache)'
kubectl get pods -l app=nginx,tier=frontend
```

---

## Troubleshooting Commands

### Docker Troubleshooting

```bash
# Container not starting
docker logs <container-name>
docker logs --tail 50 <container-name>
docker inspect <container-name>
docker events --filter container=<container-name>

# Network issues
docker network inspect bridge
docker exec <container-name> ping <other-container>
docker exec <container-name> curl http://other-service

# Permission issues
docker exec <container-name> ls -la /app
docker exec <container-name> id
docker exec <container-name> whoami

# Disk space
docker system df
docker system prune -a

# Port conflicts
docker ps -a
sudo lsof -i :<port-number>
sudo netstat -tlnp | grep :<port-number>
```

### Kubernetes Troubleshooting

```bash
# Pod not starting
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get pods -o wide

# Service not accessible
kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints <service-name>
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://service:port

# Deployment issues
kubectl describe deployment <deployment-name>
kubectl get replicasets
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>

# Resource constraints
kubectl top nodes
kubectl top pods
kubectl describe node <node-name>
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].resources}{"\n"}{end}'

# Network policies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>

# DNS issues
kubectl run test-dns --image=busybox -it --rm -- nslookup kubernetes.default
kubectl run test-dns --image=busybox -it --rm -- nslookup <service-name>

# Persistent volumes
kubectl get pv
kubectl get pvc
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>

# Node issues
kubectl get nodes
kubectl describe node <node-name>
kubectl get events --field-selector involvedObject.kind=Node

# API server issues
kubectl get componentstatuses
kubectl get --raw /healthz
kubectl get --raw /readyz
```

### Common Error Solutions

```bash
# ImagePullBackOff
kubectl describe pod <pod-name>  # Check image name and registry
kubectl get pod <pod-name> -o yaml | grep image

# CrashLoopBackOff
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>

# Pending pods
kubectl describe pod <pod-name>  # Check events
kubectl get nodes                # Check node resources
kubectl top nodes

# Evicted pods
kubectl get pods | grep Evicted
kubectl delete pods --field-selector status.phase=Failed

# Context deadline exceeded
kubectl config view
kubectl cluster-info
kubectl get --raw /healthz

# Insufficient resources
kubectl describe node <node-name>
kubectl top nodes
kubectl top pods --all-namespaces --sort-by=memory
```

---

## Exam-Style Quick Commands

### GoCD Container (Question 5)

```bash
# Pull image
docker pull gocd/gocd-server:v23.5.0

# Run container
docker run -d --name gocd-server -p 8153:8153 gocd/gocd-server:v23.5.0

# Verify running
docker ps | grep gocd-server
docker ps -a --filter "name=gocd-server"
docker inspect gocd-server
docker logs gocd-server
docker stats gocd-server --no-stream

# Stop container
docker stop gocd-server

# Remove container
docker rm gocd-server
# or force remove
docker rm -f gocd-server
```

### Multi-Container Pod (Mongo + Django)

```bash
# Create pod YAML file first, then:

# Start Minikube
minikube start

# Deploy pod
kubectl apply -f mongo-django-pod.yaml

# (1) List all pods in default namespace
kubectl get pods
kubectl get pods -o wide

# (2) Describe the 'mongo-django' pod
kubectl describe pod mongo-django

# (3) View logs of each container
kubectl logs mongo-django -c mongo
kubectl logs mongo-django -c django
kubectl logs mongo-django -c mongo -f
kubectl logs mongo-django -c django --tail=50
kubectl logs mongo-django --all-containers=true

# Additional commands
kubectl exec -it mongo-django -c mongo -- bash
kubectl port-forward mongo-django 8000:8000
kubectl delete pod mongo-django
```

---

## Best Practices

### Docker

```bash
# Use specific tags
docker pull nginx:1.21-alpine  # Good
docker pull nginx:latest       # Avoid

# Multi-stage builds
docker build -t myapp --target production .

# Security scanning
docker scan myapp:latest
trivy image myapp:latest

# Clean up regularly
docker system prune -a --volumes

# Use .dockerignore
echo "node_modules" > .dockerignore
echo ".git" >> .dockerignore
```

### Kubernetes

```bash
# Use namespaces
kubectl create namespace production
kubectl config set-context --current --namespace=production

# Use labels
kubectl label deployment myapp env=prod tier=frontend

# Dry run before apply
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server

# Generate YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Use resource limits
kubectl set resources deployment nginx --limits=cpu=500m,memory=512Mi

# Regular backups
kubectl get all -A -o yaml > cluster-backup.yaml
```

---

## Quick Reference Summary

| Task | Docker | Kubernetes |
|------|--------|------------|
| **Run** | `docker run -d nginx` | `kubectl run nginx --image=nginx` |
| **List** | `docker ps` | `kubectl get pods` |
| **Logs** | `docker logs <container>` | `kubectl logs <pod>` |
| **Exec** | `docker exec -it <container> bash` | `kubectl exec -it <pod> -- bash` |
| **Stop** | `docker stop <container>` | `kubectl delete pod <pod>` |
| **Scale** | `docker-compose up -d --scale web=3` | `kubectl scale deployment web --replicas=3` |
| **Update** | `docker pull nginx:new && docker restart` | `kubectl set image deployment/web web=nginx:new` |

---

**Remember**: Practice these commands regularly to build muscle memory for exams and real-world scenarios!
