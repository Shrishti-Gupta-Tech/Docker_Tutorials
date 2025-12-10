# Minikube Complete Guide

## Table of Contents
- [What is Minikube](#what-is-minikube)
- [Minikube Architecture](#minikube-architecture)
- [Installation](#installation)
- [Basic Operations](#basic-operations)
- [Advanced Features](#advanced-features)
- [Addons](#addons)
- [Working with Minikube](#working-with-minikube)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## What is Minikube

### Overview

**Minikube** is a tool that makes it easy to run Kubernetes locally. It runs a single-node Kubernetes cluster inside a Virtual Machine (VM) or container on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

```
┌─────────────────────────────────────────────┐
│           Your Computer                      │
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │         Minikube VM/Container         │ │
│  │                                       │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │   Kubernetes Cluster            │ │ │
│  │  │                                 │ │ │
│  │  │  ┌──────────┐  ┌──────────┐   │ │ │
│  │  │  │ Control  │  │  Worker  │   │ │ │
│  │  │  │  Plane   │  │  Node    │   │ │ │
│  │  │  └──────────┘  └──────────┘   │ │ │
│  │  │                                 │ │ │
│  │  │  Your Pods & Services           │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### Key Features

✅ **Single-Node Cluster**: Runs both control plane and worker node in one
✅ **Multiple Drivers**: Docker, VirtualBox, VMware, Hyper-V, KVM, etc.
✅ **Multiple K8s Versions**: Test against different Kubernetes versions
✅ **Addons**: Easy-to-enable extras (dashboard, ingress, metrics-server)
✅ **Multi-Cluster**: Run multiple Minikube clusters simultaneously
✅ **LoadBalancer Support**: Via `minikube tunnel`
✅ **Local Development**: Perfect for learning and development

### When to Use Minikube

**✅ Use Minikube for:**
- Learning Kubernetes
- Local development and testing
- CI/CD testing pipelines
- Experimenting with K8s features
- Demos and presentations

**❌ Don't Use Minikube for:**
- Production workloads
- Multi-node cluster testing
- High-availability scenarios
- Performance testing at scale

### Minikube vs Other Local K8s Tools

| Feature | Minikube | Kind | K3s | Docker Desktop |
|---------|----------|------|-----|----------------|
| **Ease of Setup** | Easy | Moderate | Easy | Very Easy |
| **Resource Usage** | Medium | Light | Light | Medium |
| **Multi-Node** | Limited | Yes | Yes | No |
| **Dashboard** | Built-in | Manual | Manual | Built-in |
| **Addons** | Many | Few | Some | Some |
| **Best For** | Learning | Testing | Edge/IoT | Mac/Windows |

---

## Minikube Architecture

### Components

```
┌──────────────────────────────────────────────────┐
│              Minikube CLI                        │
│  (minikube start, stop, dashboard, etc.)        │
└────────────────┬─────────────────────────────────┘
                 │
        ┌────────▼────────┐
        │   Driver Layer  │
        │  (Docker/VM)    │
        └────────┬────────┘
                 │
    ┌────────────▼────────────────┐
    │     Minikube Node           │
    │                             │
    │  ┌───────────────────────┐ │
    │  │   Control Plane       │ │
    │  │  - API Server         │ │
    │  │  - Scheduler          │ │
    │  │  - Controller Manager │ │
    │  │  - etcd               │ │
    │  └───────────────────────┘ │
    │                             │
    │  ┌───────────────────────┐ │
    │  │   Worker Components   │ │
    │  │  - Kubelet            │ │
    │  │  - Kube-proxy         │ │
    │  │  - Container Runtime  │ │
    │  └───────────────────────┘ │
    │                             │
    │  ┌───────────────────────┐ │
    │  │   Your Applications   │ │
    │  │  - Pods               │ │
    │  │  - Services           │ │
    │  │  - Deployments        │ │
    │  └───────────────────────┘ │
    └─────────────────────────────┘
```

### How It Works

1. **minikube start** creates a VM or container
2. **Kubernetes components** are installed inside
3. **kubectl** connects to the cluster
4. **You deploy** your applications
5. **Minikube manages** the lifecycle

---

## Installation

### Prerequisites

```bash
# Check system requirements
# - 2 CPUs or more
# - 2GB of free memory
# - 20GB of free disk space
# - Internet connection
# - Container or VM manager (Docker, VirtualBox, etc.)
```

### Install Minikube

#### macOS

```bash
# Using Homebrew (Recommended)
brew install minikube

# Or using curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube

# For Apple Silicon (M1/M2)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube
```

#### Linux

```bash
# Using curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Using apt (Debian/Ubuntu)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb

# Using rpm (Fedora/RHEL)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm
```

#### Windows

```powershell
# Using Chocolatey
choco install minikube

# Using winget
winget install Kubernetes.minikube

# Or download installer from
# https://minikube.sigs.k8s.io/docs/start/
```

### Install kubectl

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Windows (using Chocolatey)
choco install kubernetes-cli
```

### Verify Installation

```bash
# Check Minikube version
minikube version

# Check kubectl version
kubectl version --client

# Output:
# minikube version: v1.32.0
# Client Version: v1.29.0
```

---

## Basic Operations

### Starting Minikube

```bash
# Start with default settings
minikube start

# Start with specific driver
minikube start --driver=docker
minikube start --driver=virtualbox
minikube start --driver=hyperkit

# Start with custom resources
minikube start --cpus=4 --memory=8192 --disk-size=40g

# Start with specific Kubernetes version
minikube start --kubernetes-version=v1.28.0

# Start with custom name (multiple clusters)
minikube start -p dev-cluster
minikube start -p test-cluster

# Complete example
minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=8192 \
  --disk-size=40g \
  --kubernetes-version=v1.28.0 \
  --nodes=2
```

**Common Start Options:**
```bash
--driver          # VM/Container driver (docker, virtualbox, etc.)
--cpus            # Number of CPUs (default: 2)
--memory          # Memory in MB (default: 2200)
--disk-size       # Disk size (default: 20000mb)
--kubernetes-version  # K8s version to use
--nodes           # Number of nodes (default: 1)
--container-runtime   # Container runtime (docker, containerd, cri-o)
--network-plugin  # Network plugin (cni, kubenet)
```

### Checking Status

```bash
# Check Minikube status
minikube status

# Output:
# minikube
# type: Control Plane
# host: Running
# kubelet: Running
# apiserver: Running
# kubeconfig: Configured

# Check cluster info
kubectl cluster-info

# Check nodes
kubectl get nodes

# Get detailed info
minikube profile list
```

### Stopping Minikube

```bash
# Stop Minikube
minikube stop

# Stop specific profile
minikube stop -p dev-cluster

# Verify it's stopped
minikube status
```

### Deleting Minikube

```bash
# Delete Minikube cluster
minikube delete

# Delete specific profile
minikube delete -p dev-cluster

# Delete all profiles
minikube delete --all

# Delete and purge (removes all cached files)
minikube delete --purge
```

### Pausing/Resuming

```bash
# Pause Minikube (saves resources)
minikube pause

# Unpause
minikube unpause

# Check status
minikube status
```

---

## Advanced Features

### Multiple Nodes

```bash
# Start with multiple nodes
minikube start --nodes=3

# Add node to existing cluster
minikube node add

# List nodes
minikube node list
kubectl get nodes

# Delete a node
minikube node delete <node-name>
```

### Multiple Clusters

```bash
# Create multiple clusters
minikube start -p dev
minikube start -p staging
minikube start -p prod

# List all clusters
minikube profile list

# Switch between clusters
minikube profile dev
minikube profile staging

# Or use kubectl context
kubectl config use-context dev
kubectl config use-context staging

# Delete specific cluster
minikube delete -p dev
```

### Accessing the Cluster

```bash
# Get Minikube IP
minikube ip

# SSH into Minikube node
minikube ssh

# Once inside, you can:
docker ps                    # See containers
sudo su -                   # Become root
cat /var/log/pods/*         # View pod logs

# Exit SSH
exit

# Execute command without SSH
minikube ssh "docker ps"
minikube ssh "sudo crictl images"
```

### Docker Daemon

```bash
# Use Minikube's Docker daemon
eval $(minikube docker-env)

# Now docker commands use Minikube's Docker
docker ps
docker images

# Build image directly in Minikube
docker build -t myapp:1.0 .

# Use in Kubernetes without pushing to registry
kubectl run myapp --image=myapp:1.0 --image-pull-policy=Never

# Revert to host Docker daemon
eval $(minikube docker-env -u)
```

### LoadBalancer Services

```bash
# In a separate terminal, run tunnel
minikube tunnel

# Now LoadBalancer services get external IPs
kubectl get services

# Example:
# NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)
# my-service   LoadBalancer   10.96.123.45     127.0.0.1     80:30123/TCP

# Access service
curl http://127.0.0.1
```

### Port Forwarding

```bash
# Forward service port
minikube service <service-name>

# Opens browser automatically
minikube service <service-name> --url

# Get service URL
minikube service list
```

### File System Operations

```bash
# Mount host directory to Minikube
minikube mount /path/on/host:/path/in/minikube

# Example
minikube mount $(pwd):/host-data

# Now use in pod:
# volumeMounts:
#   - mountPath: /data
#     name: host-data
# volumes:
#   - name: host-data
#     hostPath:
#       path: /host-data

# Copy files to Minikube
minikube cp file.txt /tmp/file.txt

# Copy from Minikube
minikube cp /tmp/file.txt ./file.txt
```

---

## Addons

### Available Addons

```bash
# List all addons
minikube addons list

# Common addons:
# - dashboard          Kubernetes Dashboard
# - metrics-server     Resource metrics
# - ingress           Nginx Ingress Controller
# - ingress-dns       Ingress DNS
# - storage-provisioner  Storage provisioner
# - registry          Docker registry
# - istio             Service mesh
```

### Enable/Disable Addons

```bash
# Enable addon
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons enable ingress

# Disable addon
minikube addons disable dashboard

# Check addon status
minikube addons list
```

### Dashboard

```bash
# Enable dashboard
minikube addons enable dashboard

# Access dashboard
minikube dashboard

# Get dashboard URL only
minikube dashboard --url

# Access in terminal
# http://127.0.0.1:xxxxx/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

### Metrics Server

```bash
# Enable metrics-server
minikube addons enable metrics-server

# Wait for it to be ready
kubectl wait --for=condition=ready pod -l k8s-app=metrics-server -n kube-system --timeout=60s

# Now use top commands
kubectl top nodes
kubectl top pods
kubectl top pods -A
```

### Ingress

```bash
# Enable ingress
minikube addons enable ingress

# Verify ingress controller
kubectl get pods -n ingress-nginx

# Create ingress resource
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
EOF

# Add to /etc/hosts
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Test
curl http://myapp.local
```

### Registry

```bash
# Enable local registry
minikube addons enable registry

# Get registry address
minikube addons list | grep registry

# Push to registry
docker tag myapp:1.0 $(minikube ip):5000/myapp:1.0
docker push $(minikube ip):5000/myapp:1.0

# Use in Kubernetes
kubectl run myapp --image=$(minikube ip):5000/myapp:1.0
```

---

## Working with Minikube

### Complete Development Workflow

```bash
# 1. Start Minikube
minikube start --cpus=4 --memory=8192

# 2. Enable addons
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons enable ingress

# 3. Use Minikube Docker
eval $(minikube docker-env)

# 4. Build your image
docker build -t myapp:1.0 .

# 5. Deploy to Kubernetes
kubectl create deployment myapp --image=myapp:1.0 --image-pull-policy=Never

# 6. Expose service
kubectl expose deployment myapp --port=8080 --type=LoadBalancer

# 7. Start tunnel (in another terminal)
minikube tunnel

# 8. Access your app
kubectl get services
curl http://localhost:8080

# 9. View logs
kubectl logs -f deployment/myapp

# 10. Open dashboard
minikube dashboard

# 11. Update application
docker build -t myapp:1.1 .
kubectl set image deployment/myapp myapp=myapp:1.1

# 12. Stop when done
minikube stop
```

### Example: Deploy Complete App

```bash
# Create namespace
kubectl create namespace myapp

# Deploy database
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: myapp
spec:
  selector:
    app: mongodb
  ports:
  - port: 27017
EOF

# Deploy backend
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myapp/backend:1.0
        imagePullPolicy: Never
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          value: "mongodb://mongodb:27017/mydb"
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: myapp
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
  - port: 3000
EOF

# Check status
kubectl get all -n myapp

# Access service
minikube service backend -n myapp
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Minikube Won't Start

```bash
# Check driver is installed
docker version        # For Docker driver
VBoxManage --version  # For VirtualBox driver

# Try different driver
minikube start --driver=docker
minikube start --driver=virtualbox

# Delete and recreate
minikube delete
minikube start

# Check logs
minikube logs

# Increase verbosity
minikube start --alsologtostderr -v=7
```

#### 2. Out of Resources

```bash
# Check resource usage
minikube status
docker stats $(docker ps -q --filter name=minikube)

# Increase resources
minikube delete
minikube start --cpus=4 --memory=8192 --disk-size=40g

# Clean up space
minikube ssh "docker system prune -af"
```

#### 3. Networking Issues

```bash
# Check IP
minikube ip

# Test connectivity
minikube ssh "ping -c 3 8.8.8.8"

# Check DNS
kubectl run test-dns --image=busybox -it --rm -- nslookup kubernetes.default

# Restart networking
minikube stop
minikube start
```

#### 4. Pods Not Starting

```bash
# Check pod status
kubectl get pods -A

# Describe pod
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check Docker daemon
minikube ssh "docker ps"
```

#### 5. Service Not Accessible

```bash
# Check service
kubectl get svc

# Check endpoints
kubectl get endpoints

# Use minikube service
minikube service <service-name> --url

# Use tunnel for LoadBalancer
minikube tunnel
```

### Debug Commands

```bash
# View Minikube logs
minikube logs

# SSH into Minikube
minikube ssh

# Check systemd services
minikube ssh "systemctl status kubelet"
minikube ssh "systemctl status docker"

# View Kubernetes logs
minikube ssh "sudo journalctl -u kubelet"

# Check resource usage
kubectl top nodes
kubectl top pods -A

# Get detailed cluster info
kubectl cluster-info dump > cluster-dump.txt
```

---

## Best Practices

### Resource Management

```bash
# ✅ Good: Specify resources based on your work
minikube start --cpus=4 --memory=8192

# ❌ Avoid: Using default minimal resources for complex apps

# ✅ Good: Stop when not in use
minikube stop

# ✅ Good: Delete unused clusters
minikube delete -p old-cluster
```

### Development Workflow

```bash
# ✅ Use Minikube's Docker daemon
eval $(minikube docker-env)
docker build -t myapp:dev .

# ✅ Use imagePullPolicy=Never for local images
kubectl run myapp --image=myapp:dev --image-pull-policy=Never

# ✅ Use namespaces for organization
kubectl create namespace dev
kubectl create namespace test

# ✅ Enable useful addons
minikube addons enable metrics-server
minikube addons enable dashboard
```

### Multiple Environments

```bash
# ✅ Create separate profiles
minikube start -p dev --cpus=2 --memory=4096
minikube start -p test --cpus=2 --memory=4096
minikube start -p prod-like --cpus=4 --memory=8192

# Switch between them
minikube profile dev
minikube profile test
```

### Performance Tips

```bash
# ✅ Use Docker driver (faster)
minikube start --driver=docker

# ✅ Increase resources for complex apps
minikube start --cpus=4 --memory=8192

# ✅ Use persistent storage
minikube start --mount --mount-string="/host/path:/minikube/path"

# ✅ Prune unused resources
minikube ssh "docker system prune -f"
```

---

## Quick Reference

### Essential Commands

```bash
# Cluster Lifecycle
minikube start              # Start cluster
minikube stop               # Stop cluster
minikube delete             # Delete cluster
minikube pause              # Pause cluster
minikube unpause            # Resume cluster

# Information
minikube status             # Cluster status
minikube version            # Minikube version
minikube ip                 # Cluster IP
minikube profile list       # List clusters

# Access
minikube dashboard          # Open dashboard
minikube service <name>     # Access service
minikube tunnel             # Enable LoadBalancer
minikube ssh                # SSH into node

# Addons
minikube addons list        # List addons
minikube addons enable <name>   # Enable addon
minikube addons disable <name>  # Disable addon

# Docker
eval $(minikube docker-env)     # Use Minikube Docker
eval $(minikube docker-env -u)  # Revert to host Docker

# Troubleshooting
minikube logs               # View logs
minikube update-check       # Check for updates
```

### Configuration

```bash
# View config
minikube config view

# Set defaults
minikube config set cpus 4
minikube config set memory 8192
minikube config set driver docker

# Common settings
minikube config set kubernetes-version v1.28.0
minikube config set container-runtime containerd
```

---

## Summary

### Minikube is Perfect For:

✅ **Learning Kubernetes** - No cloud account needed
✅ **Local Development** - Fast iteration cycles  
✅ **Testing** - Try before deploying to production
✅ **Demos** - Quick setup for presentations
✅ **CI/CD** - Automated testing pipelines

### Key Takeaways:

1. **Easy Setup**: `minikube start` gets you running in minutes
2. **Full K8s**: Real Kubernetes, not a simulation
3. **Addons**: Quick enable dashboard, ingress, metrics
4. **Local Development**: Use `docker-env` for fast builds
5. **Multiple Clusters**: Test different scenarios with profiles

### Next Steps:

1. Install Minikube
2. Run `minikube start`
3. Deploy your first app
4. Enable dashboard: `minikube dashboard`
5. Explore addons: `minikube addons list`
6. Practice with the examples in this guide!

---

**Happy Learning! 🚀**
