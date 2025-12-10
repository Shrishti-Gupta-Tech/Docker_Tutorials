# Chapter 13: Kubernetes

## Table of Contents
- [13.1 Docker for CaaS](#131-docker-for-caas)
- [13.2 What is Kubernetes](#132-what-is-kubernetes)
- [13.3 Deployment of Microservices using Kubernetes](#133-deployment-of-microservices-using-kubernetes)
- [13.4 Scalability in Kubernetes](#134-scalability-in-kubernetes)
- [13.5 Security in Kubernetes](#135-security-in-kubernetes)
- [13.6 CI/CD using Kubernetes](#136-cicd-using-kubernetes)
- [13.7 Kubernetes Dashboard](#137-kubernetes-dashboard)

---

## 13.1. Docker for CaaS

### What is CaaS (Containers as a Service)?

**CaaS** is a cloud service model that allows users to manage and deploy containers using container-based virtualization.

```
Traditional Infrastructure:
Physical Servers → VMs → Applications

CaaS:
Physical Servers → Container Orchestration → Containers → Applications
```

### Docker and CaaS Relationship

**Docker provides**:
- Container runtime
- Image building and management
- Container networking
- Volume management

**CaaS platforms extend Docker with**:
- Orchestration (scheduling, scaling)
- Service discovery
- Load balancing
- Health monitoring
- Rolling updates
- Self-healing

### Popular CaaS Platforms

```
┌─────────────────────────────────────┐
│        CaaS Platforms               │
├─────────────────────────────────────┤
│  Kubernetes (K8s)    - Most popular │
│  Docker Swarm        - Native Docker│
│  Amazon ECS          - AWS managed  │
│  Google GKE          - Google Cloud │
│  Azure AKS           - Microsoft    │
│  Red Hat OpenShift   - Enterprise   │
└─────────────────────────────────────┘
```

### Docker Swarm vs Kubernetes

| Feature | Docker Swarm | Kubernetes |
|---------|-------------|------------|
| **Setup** | Simple, quick | Complex, more setup |
| **Learning Curve** | Easy | Steep |
| **Scaling** | Good | Excellent |
| **Load Balancing** | Built-in | Requires configuration |
| **Rolling Updates** | Yes | Yes, more advanced |
| **Auto-healing** | Basic | Advanced |
| **Community** | Smaller | Massive |
| **Use Case** | Small-medium apps | Production, enterprise |

### Docker Swarm Quick Example

```bash
# Initialize Swarm
docker swarm init

# Deploy a service
docker service create \
  --name web \
  --replicas 3 \
  --publish 8080:80 \
  nginx:alpine

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:latest web

# Remove service
docker service rm web
```

### When to Use What?

**Use Docker Swarm when**:
- Simple applications
- Small team
- Quick deployment needed
- Less operational complexity

**Use Kubernetes when**:
- Complex microservices
- Large scale deployments
- Need advanced features
- Enterprise requirements
- Multi-cloud strategy

---

## 13.2. What is Kubernetes

### Kubernetes Overview

**Kubernetes (K8s)** is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

```
┌──────────────────────────────────────────────┐
│              Kubernetes Cluster              │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │         Control Plane (Master)         │ │
│  │  ┌──────────┐  ┌───────┐  ┌────────┐ │ │
│  │  │ API      │  │ Sched- │  │Control- │ │
│  │  │ Server   │  │ uler  │  │ler Mgr │ │
│  │  └──────────┘  └───────┘  └────────┘ │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌──────────────────────────────────────────┐│
│  │         Worker Nodes                     ││
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐ ││
│  │  │ Pod     │  │ Pod     │  │ Pod     │ ││
│  │  │┌───┐┌──┐│  │┌───┐┌──┐│  │┌───┐┌──┐│ ││
│  │  ││C1 ││C2││  ││C3 ││C4││  ││C5 ││C6││ ││
│  │  │└───┘└──┘│  │└───┘└──┘│  │└───┘└──┘│ ││
│  │  └─────────┘  └─────────┘  └─────────┘ ││
│  └──────────────────────────────────────────┘│
└──────────────────────────────────────────────┘
```

### Key Kubernetes Concepts

#### 1. **Cluster**
- A set of machines (nodes) running Kubernetes
- Consists of control plane and worker nodes

#### 2. **Node**
- A physical or virtual machine in the cluster
- **Master Node**: Manages the cluster
- **Worker Node**: Runs application containers

#### 3. **Pod**
- Smallest deployable unit in Kubernetes
- Contains one or more containers
- Shares network and storage
- Has unique IP address

```yaml
# Simple Pod Example
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

#### 4. **Deployment**
- Manages ReplicaSets and Pods
- Handles rolling updates
- Maintains desired state

```yaml
# Deployment Example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

#### 5. **Service**
- Exposes Pods to network
- Provides stable IP and DNS
- Load balances traffic

```yaml
# Service Example
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### 6. **Namespace**
- Virtual clusters within a physical cluster
- Isolates resources
- Used for multi-tenancy

```bash
# Create namespace
kubectl create namespace dev
kubectl create namespace prod

# List namespaces
kubectl get namespaces
```

### Kubernetes Architecture Components

#### Control Plane Components

**1. API Server (kube-apiserver)**:
- Frontend for Kubernetes control plane
- Exposes Kubernetes API
- Processes REST operations

**2. etcd**:
- Key-value store for cluster data
- Stores cluster state and configuration

**3. Scheduler (kube-scheduler)**:
- Assigns Pods to Nodes
- Considers resource requirements
- Applies constraints and affinity rules

**4. Controller Manager (kube-controller-manager)**:
- Runs controller processes
- Node controller, Replication controller, etc.
- Maintains desired state

**5. Cloud Controller Manager**:
- Integrates with cloud provider APIs
- Manages cloud-specific resources

#### Node Components

**1. Kubelet**:
- Agent running on each node
- Ensures containers are running in Pods
- Reports node status to control plane

**2. Kube-proxy**:
- Network proxy on each node
- Maintains network rules
- Enables Service abstraction

**3. Container Runtime**:
- Software that runs containers
- Docker, containerd, CRI-O

### Installing Kubernetes

#### Option 1: Minikube (Local Development)

```bash
# Install Minikube (macOS)
brew install minikube

# Start Minikube cluster
minikube start

# Verify installation
kubectl cluster-info
kubectl get nodes

# Stop Minikube
minikube stop

# Delete Minikube cluster
minikube delete
```

#### Option 2: Kind (Kubernetes in Docker)

```bash
# Install Kind
brew install kind

# Create cluster
kind create cluster --name my-cluster

# List clusters
kind get clusters

# Delete cluster
kind delete cluster --name my-cluster
```

#### Option 3: K3s (Lightweight Kubernetes)

```bash
# Install K3s
curl -sfL https://get.k3s.io | sh -

# Check status
sudo systemctl status k3s

# Get kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml
```

### Essential Kubectl Commands

```bash
# Cluster Information
kubectl version                    # Client and server version
kubectl cluster-info              # Cluster information
kubectl get nodes                 # List nodes

# Pod Management
kubectl get pods                  # List pods in default namespace
kubectl get pods -n kube-system   # List pods in specific namespace
kubectl get pods -A               # List pods in all namespaces
kubectl describe pod <pod-name>   # Detailed pod information
kubectl logs <pod-name>           # View pod logs
kubectl logs <pod-name> -f        # Follow logs
kubectl exec -it <pod-name> -- bash  # Execute command in pod
kubectl delete pod <pod-name>     # Delete pod

# Deployment Management
kubectl get deployments           # List deployments
kubectl describe deployment <name>
kubectl create -f deployment.yaml # Create from file
kubectl apply -f deployment.yaml  # Apply configuration
kubectl delete deployment <name>  # Delete deployment
kubectl scale deployment <name> --replicas=5  # Scale deployment

# Service Management
kubectl get services              # List services
kubectl describe service <name>
kubectl expose deployment <name> --port=80 --type=LoadBalancer

# Namespace Management
kubectl get namespaces
kubectl create namespace dev
kubectl delete namespace dev
kubectl config set-context --current --namespace=dev

# Configuration
kubectl config view                # View kubeconfig
kubectl config get-contexts       # List contexts
kubectl config use-context <name> # Switch context
```

---

## 13.3. Deployment of Microservices using Kubernetes

### Multi-Container Pod Example (Answering the Exam Question)

**Scenario**: Deploy a multi-container pod with MongoDB and Django

#### Step 1: Create Multi-Container Pod YAML

`mongo-django-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongo-django
  labels:
    app: web-app
spec:
  containers:
  # MongoDB container
  - name: mongo
    image: mongo:latest
    ports:
    - containerPort: 27017
    env:
    - name: MONGO_INITDB_ROOT_USERNAME
      value: "admin"
    - name: MONGO_INITDB_ROOT_PASSWORD
      value: "password"
    volumeMounts:
    - name: mongo-data
      mountPath: /data/db
  
  # Django container
  - name: django
    image: django:latest
    ports:
    - containerPort: 8000
    env:
    - name: DATABASE_URL
      value: "mongodb://admin:password@localhost:27017"
    command: ["python", "manage.py", "runserver", "0.0.0.0:8000"]
  
  volumes:
  - name: mongo-data
    emptyDir: {}
```

#### Step 2: Deploy the Pod

```bash
# Start Minikube cluster
minikube start

# Create the pod
kubectl apply -f mongo-django-pod.yaml

# Verify pod is created
kubectl get pods
```

#### Step 3: List all pods in default namespace

```bash
# List all pods in default namespace
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# List pods with labels
kubectl get pods --show-labels

# Output example:
# NAME           READY   STATUS    RESTARTS   AGE
# mongo-django   2/2     Running   0          1m
```

#### Step 4: Describe the 'mongo-django' pod

```bash
# Describe the pod
kubectl describe pod mongo-django

# Output includes:
# - Pod metadata (name, namespace, labels)
# - Container details (image, ports, state)
# - Events (creation, pulling images, starting)
# - Resource usage
# - Volume mounts
```

**Example Output**:
```
Name:         mongo-django
Namespace:    default
Node:         minikube/192.168.49.2
Start Time:   Sun, 15 Jan 2024 10:00:00 +0000
Labels:       app=web-app
Status:       Running
IP:           172.17.0.5

Containers:
  mongo:
    Container ID:   docker://abc123...
    Image:          mongo:latest
    Port:           27017/TCP
    State:          Running
    Ready:          True
    
  django:
    Container ID:   docker://def456...
    Image:          django:latest
    Port:           8000/TCP
    State:          Running
    Ready:          True

Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/mongo-django to minikube
  Normal  Pulling    2m    kubelet            Pulling image "mongo:latest"
  Normal  Pulled     1m    kubelet            Successfully pulled image
  Normal  Created    1m    kubelet            Created container mongo
  Normal  Started    1m    kubelet            Started container mongo
  Normal  Pulling    1m    kubelet            Pulling image "django:latest"
  Normal  Pulled     45s   kubelet            Successfully pulled image
  Normal  Created    45s   kubelet            Created container django
  Normal  Started    45s   kubelet            Started container django
```

#### Step 5: View logs of each container

```bash
# View logs of mongo container
kubectl logs mongo-django -c mongo

# View logs of django container
kubectl logs mongo-django -c django

# Follow logs in real-time
kubectl logs mongo-django -c mongo -f
kubectl logs mongo-django -c django -f

# View last 50 lines of logs
kubectl logs mongo-django -c mongo --tail=50
kubectl logs mongo-django -c django --tail=50

# View logs with timestamps
kubectl logs mongo-django -c mongo --timestamps
kubectl logs mongo-django -c django --timestamps
```

#### Complete Commands Summary

```bash
# 1. List all pods in default namespace
kubectl get pods
kubectl get pods -o wide
kubectl get pods --all-namespaces  # or -A for all namespaces

# 2. Describe the 'mongo-django' pod
kubectl describe pod mongo-django

# 3. View logs of each container in the pod
kubectl logs mongo-django -c mongo
kubectl logs mongo-django -c django

# Additional useful commands
kubectl logs mongo-django --all-containers=true  # All container logs
kubectl exec -it mongo-django -c mongo -- bash   # Access mongo container
kubectl exec -it mongo-django -c django -- bash  # Access django container
kubectl port-forward mongo-django 8000:8000     # Port forward to access django
kubectl delete pod mongo-django                  # Delete pod
```

### Complete Microservices Deployment Example

#### Application Architecture

```
┌─────────────────────────────────────────┐
│         Load Balancer (Service)         │
└────────────────┬────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
┌───────▼──────┐   ┌──────▼────────┐
│   Frontend   │   │   Backend     │
│  Deployment  │   │   Deployment  │
│  (React)     │   │   (Node.js)   │
└──────────────┘   └───────┬───────┘
                           │
                    ┌──────▼──────┐
                    │   Database  │
                    │  (MongoDB)  │
                    │  StatefulSet│
                    └─────────────┘
```

#### 1. MongoDB StatefulSet

`mongodb-statefulset.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb-service
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
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "admin"
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

#### 2. Backend Deployment

`backend-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
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
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          value: "mongodb://mongodb-service:27017/myapp"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
  type: ClusterIP
```

#### 3. Frontend Deployment

`frontend-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myapp/frontend:1.0
        ports:
        - containerPort: 80
        env:
        - name: API_URL
          value: "http://backend-service:3000"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### 4. ConfigMap and Secrets

`config.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "MyMicroserviceApp"
  LOG_LEVEL: "info"
  API_VERSION: "v1"
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
stringData:
  password: "mySecurePassword123"
  username: "admin"
```

#### Deployment Commands

```bash
# Create secrets and config
kubectl apply -f config.yaml

# Deploy database
kubectl apply -f mongodb-statefulset.yaml

# Wait for MongoDB to be ready
kubectl wait --for=condition=ready pod -l app=mongodb --timeout=300s

# Deploy backend
kubectl apply -f backend-deployment.yaml

# Wait for backend to be ready
kubectl wait --for=condition=ready pod -l app=backend --timeout=300s

# Deploy frontend
kubectl apply -f frontend-deployment.yaml

# Verify all deployments
kubectl get all

# Check service endpoints
kubectl get services

# Access the application
minikube service frontend-service
```

### Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 3000
```

```bash
# Enable Ingress in Minikube
minikube addons enable ingress

# Apply ingress
kubectl apply -f ingress.yaml

# Get ingress address
kubectl get ingress

# Add to /etc/hosts
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Access application
curl http://myapp.local
```

---

## 13.4. Scalability in Kubernetes

### Types of Scaling

```
┌────────────────────────────────────────┐
│         Kubernetes Scaling             │
├────────────────────────────────────────┤
│  1. Horizontal Pod Autoscaling (HPA)  │
│     - Scale pod replicas               │
│  2. Vertical Pod Autoscaling (VPA)    │
│     - Scale pod resources              │
│  3. Cluster Autoscaling                │
│     - Scale nodes                      │
└────────────────────────────────────────┘
```

### 1. Horizontal Pod Autoscaler (HPA)

**Automatically scales the number of pods based on metrics**

#### Manual Scaling

```bash
# Scale deployment manually
kubectl scale deployment backend --replicas=5

# Scale using file
kubectl scale -f backend-deployment.yaml --replicas=10

# Verify scaling
kubectl get pods -l app=backend
kubectl get deployment backend
```

#### Automatic Scaling with HPA

`hpa.yaml`:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

```bash
# Create HPA
kubectl apply -f hpa.yaml

# Or create via command
kubectl autoscale deployment backend \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# View HPA status
kubectl get hpa
kubectl describe hpa backend-hpa

# Watch HPA in action
kubectl get hpa -w

# Example output:
# NAME          REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS
# backend-hpa   Deployment/backend   45%/70%   2         10        3
```

### 2. Load Testing to Trigger Scaling

```bash
# Generate load using busybox
kubectl run -it --rm load-generator --image=busybox -- /bin/sh

# Inside the pod, run:
while true; do wget -q -O- http://backend-service:3000/api/data; done

# In another terminal, watch scaling
kubectl get hpa -w
kubectl get pods -w
```

### 3. Custom Metrics Scaling

`custom-metrics-hpa.yaml`:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  # CPU-based scaling
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  # Custom metric: requests per second
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  # External metric: queue depth
  - type: External
    external:
      metric:
        name: queue_messages_ready
        selector:
          matchLabels:
            queue_name: task_queue
      target:
        type: AverageValue
        averageValue: "30"
```

### 4. Vertical Pod Autoscaler (VPA)

**Automatically adjusts CPU and memory requests/limits**

`vpa.yaml`:
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  updatePolicy:
    updateMode: "Auto"  # Auto, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: backend
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
```

```bash
# Install VPA (Minikube)
minikube addons enable metrics-server
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

# Create VPA
kubectl apply -f vpa.yaml

# View VPA recommendations
kubectl describe vpa backend-vpa
kubectl get vpa backend-vpa -o yaml
```

### 5. Cluster Autoscaler

**Automatically adds or removes nodes based on pod demands**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler
  namespace: kube-system
data:
  cluster-autoscaler: |
    --balance-similar-node-groups
    --skip-nodes-with-system-pods=false
    --scale-down-enabled=true
    --scale-down-unneeded-time=10m
    --scale-down-delay-after-add=10m
    --skip-nodes-with-local-storage=false
```

### Scaling Best Practices

```yaml
# 1. Set Resource Requests and Limits
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"

# 2. Use Pod Disruption Budgets
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: backend-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: backend

# 3. Configure Readiness Probes
readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5

# 4. Use Anti-Affinity for HA
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: kubernetes.io/hostname
```

### Scaling Commands Reference

```bash
# Manual Scaling
kubectl scale deployment backend --replicas=5
kubectl scale statefulset mongodb --replicas=3
kubectl scale replicaset backend-abc123 --replicas=4

# HPA Management
kubectl autoscale deployment backend --min=2 --max=10 --cpu-percent=70
kubectl get hpa
kubectl describe hpa backend-hpa
kubectl delete hpa backend-hpa

# View Metrics
kubectl top nodes
kubectl top pods
kubectl top pods -n kube-system
kubectl top pod backend-xyz --containers

# Resource Limits
kubectl describe node minikube
kubectl describe quota
kubectl describe limitrange
```

---

## 13.5. Security in Kubernetes

### Kubernetes Security Layers

```
┌──────────────────────────────────────────────┐
│            Security Layers                    │
├──────────────────────────────────────────────┤
│  1. Cluster Security                         │
│     - Network policies                       │
│     - Pod security policies                  │
│  2. Authentication & Authorization           │
│     - RBAC                                   │
│     - Service accounts                       │
│  3. Secrets Management                       │
│     - Encrypted secrets                      │
│     - External secret managers               │
│  4. Container Security                       │
│     - Image scanning                         │
│     - Security contexts                      │
│  5. Network Security                         │
│     - Network policies                       │
│     - Service mesh (mTLS)                    │
└──────────────────────────────────────────────┘
```

### 1. Role-Based Access Control (RBAC)

#### Service Account

`serviceaccount.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backend-sa
  namespace: default
---
apiVersion: v1
kind: Secret
metadata:
  name: backend-sa-token
  annotations:
    kubernetes.io/service-account.name: backend-sa
type: kubernetes.io/service-account-token
```

#### Role and RoleBinding

`rbac.yaml`:
```yaml
# Role - defines permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
---
# RoleBinding - assigns role to service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: backend-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### ClusterRole and ClusterRoleBinding

```yaml
# ClusterRole - cluster-wide permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-custom
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: admin@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-custom
  apiGroup: rbac.authorization.k8s.io
```

```bash
# RBAC Commands
kubectl create serviceaccount backend-sa
kubectl get serviceaccounts
kubectl describe serviceaccount backend-sa

kubectl create role pod-reader --verb=get,list --resource=pods
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:backend-sa

kubectl get roles
kubectl get rolebindings
kubectl describe role pod-reader

# Check permissions
kubectl auth can-i list pods --as=system:serviceaccount:default:backend-sa
kubectl auth can-i create deployments --as=system:serviceaccount:default:backend-sa
```

### 2. Secrets Management

```yaml
# Opaque Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database-password: "mySecurePassword"
  api-key: "abc123def456"
---
# Using Secret in Pod
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
spec:
  containers:
  - name: backend
    image: myapp/backend:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database-password
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: api-key
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
```

```bash
# Create secrets
kubectl create secret generic app-secrets \
  --from-literal=database-password=myPassword \
  --from-literal=api-key=abc123

# From file
kubectl create secret generic ssh-key --from-file=id_rsa=~/.ssh/id_rsa

# TLS secret
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# View secrets
kubectl get secrets
kubectl describe secret app-secrets

# Decode secret
kubectl get secret app-secrets -o jsonpath='{.data.database-password}' | base64 -d

# Delete secret
kubectl delete secret app-secrets
```

### 3. Pod Security Standards

```yaml
# Pod Security Context
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### 4. Network Policies

```yaml
# Default Deny All Traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
# Allow Backend to Access MongoDB
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-to-mongodb
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 27017
---
# Allow Frontend to Access Backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 3000
---
# Allow Egress to External APIs
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 27017
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53  # DNS
    - protocol: UDP
      port: 53
  - to:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 443  # HTTPS
```

```bash
# Apply network policy
kubectl apply -f network-policy.yaml

# View network policies
kubectl get networkpolicies
kubectl describe networkpolicy backend-to-mongodb

# Test connectivity
kubectl run test-pod --image=busybox -it --rm -- /bin/sh
# Inside pod:
wget -O- http://backend-service:3000
```

### 5. Image Security

```yaml
# Image Pull Secret
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
---
# Use in Pod
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
spec:
  containers:
  - name: backend
    image: private-registry.com/myapp/backend:1.0
  imagePullSecrets:
  - name: registry-secret
```

```bash
# Create image pull secret
kubectl create secret docker-registry registry-secret \
  --docker-server=private-registry.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com

# Scan images with Trivy
trivy image myapp/backend:1.0

# Use admission controllers
# - ImagePolicyWebhook
# - AlwaysPullImages
# - PodSecurityPolicy (deprecated) → Pod Security Standards
```

### Security Best Practices

```yaml
# 1. Least Privilege Principle
- Use specific RBAC roles
- Run as non-root user
- Drop unnecessary capabilities

# 2. Network Segmentation
- Use network policies
- Implement service mesh

# 3. Secrets Management
- Use external secret managers (Vault, AWS Secrets Manager)
- Encrypt secrets at rest
- Rotate secrets regularly

# 4. Image Security
- Scan images for vulnerabilities
- Use minimal base images (alpine, distroless)
- Sign images
- Use private registries

# 5. Audit and Monitoring
- Enable audit logs
- Monitor suspicious activities
- Use security tools (Falco, OPA)
```

### Security Commands

```bash
# RBAC
kubectl create serviceaccount my-sa
kubectl create role my-role --verb=get,list --resource=pods
kubectl create rolebinding my-binding --role=my-role --serviceaccount=default:my-sa
kubectl auth can-i list pods --as=system:serviceaccount:default:my-sa

# Secrets
kubectl create secret generic my-secret --from-literal=key=value
kubectl get secrets
kubectl describe secret my-secret

# Network Policies
kubectl get networkpolicies
kubectl describe networkpolicy default-deny-all

# Security Context
kubectl exec pod-name -- id
kubectl exec pod-name -- cat /proc/1/status | grep Cap

# Audit
kubectl get events
kubectl logs -n kube-system kube-apiserver-xxx
```

---

## 13.6. CI/CD using Kubernetes

### CI/CD Pipeline Overview

```
┌──────────────────────────────────────────────────────────┐
│                  CI/CD Pipeline                           │
├──────────────────────────────────────────────────────────┤
│  1. Code Commit (Git)                                    │
│           ↓                                               │
│  2. Build & Test (Jenkins/GitLab CI/GitHub Actions)     │
│           ↓                                               │
│  3. Build Docker Image                                    │
│           ↓                                               │
│  4. Push to Registry (Docker Hub/ECR/GCR)               │
│           ↓                                               │
│  5. Deploy to Kubernetes                                  │
│           ↓                                               │
│  6. Run Tests (Integration/E2E)                          │
│           ↓                                               │
│  7. Monitor & Rollback if needed                         │
└──────────────────────────────────────────────────────────┘
```

### 1. GitHub Actions CI/CD

`.github/workflows/deploy.yml`:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  DOCKER_IMAGE: myapp/backend
  K8S_NAMESPACE: production

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Run linter
      run: npm run lint
  
  build-and-push:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ env.DOCKER_IMAGE }}:${{ github.sha }}
          ${{ env.DOCKER_IMAGE }}:latest
        cache-from: type=gha
        cache-to: type=gha,mode=max
  
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG }}
    
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/backend \
          backend=${{ env.DOCKER_IMAGE }}:${{ github.sha }} \
          -n ${{ env.K8S_NAMESPACE }}
        
        kubectl rollout status deployment/backend \
          -n ${{ env.K8S_NAMESPACE }}
    
    - name: Verify deployment
      run: |
        kubectl get pods -n ${{ env.K8S_NAMESPACE }}
        kubectl get services -n ${{ env.K8S_NAMESPACE }}
```

### 2. GitLab CI/CD

`.gitlab-ci.yml`:
```yaml
stages:
  - build
  - test
  - docker
  - deploy

variables:
  DOCKER_IMAGE: myapp/backend
  KUBERNETES_NAMESPACE: production

build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm test
    - npm run lint
  coverage: '/Coverage: \d+\.\d+%/'

docker:
  stage: docker
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker tag $DOCKER_IMAGE:$CI_COMMIT_SHA $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
    - docker push $DOCKER_IMAGE:latest
  only:
    - main

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  before_script:
    - echo "$KUBE_CONFIG" | base64 -d > kubeconfig
    - export KUBECONFIG=kubeconfig
  script:
    - kubectl config use-context production
    - kubectl set image deployment/backend backend=$DOCKER_IMAGE:$CI_COMMIT_SHA -n $KUBERNETES_NAMESPACE
    - kubectl rollout status deployment/backend -n $KUBERNETES_NAMESPACE
    - kubectl get pods -n $KUBERNETES_NAMESPACE
  only:
    - main
  when: manual
```

### 3. Jenkins Pipeline

`Jenkinsfile`:
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myapp/backend'
        DOCKER_REGISTRY = 'docker.io'
        K8S_NAMESPACE = 'production'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
                sh 'npm run lint'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        kubectl set image deployment/backend \
                            backend=${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -n ${K8S_NAMESPACE}
                        
                        kubectl rollout status deployment/backend -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl get pods -n ${K8S_NAMESPACE}'
                    sh 'kubectl get services -n ${K8S_NAMESPACE}'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
            // Rollback
            withKubeConfig([credentialsId: 'kubeconfig']) {
                sh 'kubectl rollout undo deployment/backend -n ${K8S_NAMESPACE}'
            }
        }
    }
}
```

### 4. ArgoCD (GitOps)

`argocd-application.yaml`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-k8s
    targetRevision: HEAD
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Create application
kubectl apply -f argocd-application.yaml

# CLI login
argocd login localhost:8080

# Sync application
argocd app sync myapp

# View application status
argocd app get myapp
```

### 5. Deployment Strategies

#### Rolling Update (Default)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above replicas during update
      maxUnavailable: 1  # Max pods unavailable during update
  template:
    spec:
      containers:
      - name: backend
        image: myapp/backend:v2
```

```bash
# Perform rolling update
kubectl set image deployment/backend backend=myapp/backend:v2

# Monitor rollout
kubectl rollout status deployment/backend

# Pause rollout
kubectl rollout pause deployment/backend

# Resume rollout
kubectl rollout resume deployment/backend

# Rollback
kubectl rollout undo deployment/backend

# Rollback to specific revision
kubectl rollout undo deployment/backend --to-revision=2

# View rollout history
kubectl rollout history deployment/backend
```

#### Blue-Green Deployment

```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      version: blue
  template:
    metadata:
      labels:
        app: backend
        version: blue
    spec:
      containers:
      - name: backend
        image: myapp/backend:v1
---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      version: green
  template:
    metadata:
      labels:
        app: backend
        version: green
    spec:
      containers:
      - name: backend
        image: myapp/backend:v2
---
# Service (switch between blue/green)
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
    version: blue  # Change to 'green' to switch traffic
  ports:
  - port: 3000
    targetPort: 3000
```

#### Canary Deployment

```yaml
# Stable version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-stable
spec:
  replicas: 9  # 90% of traffic
  selector:
    matchLabels:
      app: backend
      track: stable
  template:
    metadata:
      labels:
        app: backend
        track: stable
    spec:
      containers:
      - name: backend
        image: myapp/backend:v1
---
# Canary version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-canary
spec:
  replicas: 1  # 10% of traffic
  selector:
    matchLabels:
      app: backend
      track: canary
  template:
    metadata:
      labels:
        app: backend
        track: canary
    spec:
      containers:
      - name: backend
        image: myapp/backend:v2
---
# Service (load balances across both)
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend  # Matches both stable and canary
  ports:
  - port: 3000
    targetPort: 3000
```

---

## 13.7. Kubernetes Dashboard

### Installing Kubernetes Dashboard

```bash
# Deploy Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin user
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Get token
kubectl -n kubernetes-dashboard create token admin-user

# Start proxy
kubectl proxy

# Access Dashboard
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### Minikube Dashboard

```bash
# Enable and access dashboard
minikube dashboard

# Get dashboard URL
minikube dashboard --url
```

### Dashboard Features

```
┌──────────────────────────────────────────────┐
│        Kubernetes Dashboard                   │
├──────────────────────────────────────────────┤
│  1. Cluster Overview                         │
│     - Nodes status                           │
│     - Namespaces                             │
│     - Resource usage                         │
│                                              │
│  2. Workloads                                │
│     - Deployments                            │
│     - Pods                                   │
│     - ReplicaSets                            │
│     - StatefulSets                           │
│     - DaemonSets                             │
│                                              │
│  3. Discovery and Load Balancing             │
│     - Services                               │
│     - Ingresses                              │
│                                              │
│  4. Config and Storage                       │
│     - ConfigMaps                             │
│     - Secrets                                │
│     - PersistentVolumes                      │
│                                              │
│  5. RBAC                                     │
│     - ServiceAccounts                        │
│     - Roles                                  │
│     - RoleBindings                           │
└──────────────────────────────────────────────┘
```

### Alternative Dashboards

#### 1. Lens (OpenLens)

```bash
# Install Lens
brew install --cask lens

# Or download from https://k8slens.dev/
```

**Features**:
- Multi-cluster management
- Real-time metrics
- Built-in terminal
- Helm chart management
- RBAC visualization

#### 2. K9s (Terminal UI)

```bash
# Install K9s
brew install k9s

# Run K9s
k9s

# Keyboard shortcuts
# :pods - View pods
# :svc - View services
# :deploy - View deployments
# d - Describe
# l - View logs
# s - Shell into container
# Ctrl+d - Delete
```

#### 3. Octant

```bash
# Install Octant
brew install octant

# Run Octant
octant

# Access at http://localhost:7777
```

### Monitoring with Dashboard

```bash
# Install Metrics Server (required for resource metrics)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# For Minikube
minikube addons enable metrics-server

# View metrics
kubectl top nodes
kubectl top pods
kubectl top pods --all-namespaces
```

---

## Summary and Practice

### Complete Kubernetes Workflow

```bash
# 1. Start Minikube
minikube start

# 2. Create namespace
kubectl create namespace myapp

# 3. Deploy application
kubectl apply -f mongodb-statefulset.yaml -n myapp
kubectl apply -f backend-deployment.yaml -n myapp
kubectl apply -f frontend-deployment.yaml -n myapp

# 4. Verify deployment
kubectl get all -n myapp
kubectl get pods -n myapp -w

# 5. Access application
kubectl port-forward svc/frontend-service 8080:80 -n myapp
# Or
minikube service frontend-service -n myapp

# 6. Scale application
kubectl scale deployment backend --replicas=5 -n myapp

# 7. Update application
kubectl set image deployment/backend backend=myapp/backend:v2 -n myapp

# 8. Monitor
kubectl logs -f deployment/backend -n myapp
kubectl top pods -n myapp

# 9. Troubleshoot
kubectl describe pod <pod-name> -n myapp
kubectl exec -it <pod-name> -n myapp -- bash

# 10. Cleanup
kubectl delete namespace myapp
minikube stop
```

### Key Commands Cheatsheet

```bash
# Cluster Management
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>

# Pod Operations
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name> [-c container-name]
kubectl exec -it <pod-name> -- bash
kubectl delete pod <pod-name>

# Deployment Operations
kubectl get deployments
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl set image deployment/nginx nginx=nginx:1.21
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx

# Service Operations
kubectl get services
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl describe service nginx

# ConfigMap and Secrets
kubectl create configmap app-config --from-literal=key=value
kubectl create secret generic app-secret --from-literal=password=secret
kubectl get configmaps
kubectl get secrets

# Debugging
kubectl get events
kubectl top nodes
kubectl top pods
kubectl describe <resource> <name>
kubectl logs <pod-name> --previous

# Apply/Delete from file
kubectl apply -f app.yaml
kubectl delete -f app.yaml
kubectl apply -f ./k8s/
```

### Practice Exercises

1. **Deploy Multi-Container Pod**:
   - Create a pod with nginx and redis
   - Verify both containers are running
   - Access nginx and check redis connection

2. **Create Microservices Application**:
   - Deploy frontend, backend, and database
   - Configure services for inter-pod communication
   - Set up ingress for external access

3. **Implement Scaling**:
   - Create HPA for your deployment
   - Generate load to trigger scaling
   - Monitor scaling behavior

4. **Security Implementation**:
   - Create service accounts with limited permissions
   - Implement network policies
   - Use secrets for sensitive data

5. **CI/CD Pipeline**:
   - Set up automated deployments
   - Implement rolling updates
   - Configure automated rollbacks

---

## Next Steps

- Practice with real applications
- Explore Helm charts for package management
- Learn about service meshes (Istio, Linkerd)
- Study advanced topics (StatefulSets, DaemonSets, Jobs)
- Get certified (CKA, CKAD, CKS)
- Explore managed Kubernetes (EKS, GKE, AKS)

---

**Congratulations!** You now have comprehensive knowledge of Docker, Kubernetes, and Monitoring. Practice these concepts with real projects to solidify your understanding.
