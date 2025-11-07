# Kubernetes Commands Cheat Sheet

## 1. Basic Cluster Management

### Cluster Information Commands
```bash
# Check cluster info
kubectl cluster-info

# View nodes in the cluster
kubectl get nodes

# Check k8s version
kubectl version --short
```

### Cluster Architecture Overview
```mermaid
flowchart TD
    A[Control Plane] --> B[API Server]
    A --> C[etcd]
    A --> D[Scheduler]
    A --> E[Controller Manager]
    
    B --> F[Worker Nodes]
    F --> G[Kubelet]
    F --> H[Kube Proxy]
    F --> I[Pods]
    I --> J[Containers]
```

---

## 2. Pod Operations

### Pod Management Commands
```bash
# List all pods
kubectl get pods

# List pods with details
kubectl get pods -o wide

# Get pod details including events
kubectl describe pod <pod-name>

# Delete a pod
kubectl delete pod <pod-name>

# Stream pod logs
kubectl logs <pod-name>

# Stream logs from multiple pods
kubectl logs -f deployment/<deployment-name>
```

### Pod Lifecycle
```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Running
    Running --> Succeeded
    Running --> Failed
    Running --> Unknown
    Failed --> [*]
    Succeeded --> [*]
    Unknown --> [*]
```

---

## 3. Deployments & ReplicaSets

### Deployment Commands
```bash
# List deployments
kubectl get deployments

# Create a deployment
kubectl create deployment <name> --image=<image>

# Scale a deployment
kubectl scale deployment/<name> --replicas=3

# Update deployment image
kubectl set image deployment/<name> <container>=<new-image>

# Check rollout status
kubectl rollout status deployment/<name>

# Rollback a deployment
kubectl rollout undo deployment/<name>
```

### Deployment Architecture
```mermaid
graph TB
    A[Deployment] --> B[ReplicaSet v1]
    A --> C[ReplicaSet v2]
    B --> D[Pod 1]
    B --> E[Pod 2]
    C --> F[Pod 3]
    C --> G[Pod 4]
    
    style B fill:#e1f5fe
    style C fill:#f3e5f5
```

---

## 4. Services

### Service Management Commands
```bash
# List services
kubectl get svc

# Expose a deployment as a service
kubectl expose deployment/<name> --port=80 --target-port=8080 --type=LoadBalancer

# Get service details
kubectl describe svc <service-name>
```

### Service Types and Traffic Flow
```mermaid
flowchart LR
    A[External User] --> B[LoadBalancer]
    B --> C[ClusterIP]
    C --> D[Pod]
    C --> E[Pod]
    C --> F[Pod]
    
    G[Internal App] --> C
    
    subgraph Cluster
        C
        D
        E
        F
    end
```

---

## 5. ConfigMaps & Secrets

### Configuration Management Commands
```bash
# Create a ConfigMap from a file
kubectl create configmap <name> --from-file=<path-to-file>

# Create a Secret (generic)
kubectl create secret generic <name> --from-literal=key=value

# List ConfigMaps/Secrets
kubectl get configmaps
kubectl get secrets
```

### ConfigMap/Secret Usage in Pods
```mermaid
flowchart TD
    A[ConfigMap] --> C[Pod]
    B[Secret] --> C
    C --> D[Container 1]
    C --> E[Container 2]
    
    A --> F[Environment<br/>Variables]
    B --> G[Volume Mount]
```

---

## 6. Namespaces

### Namespace Commands
```bash
# Switch namespaces
kubectl config set-context --current --namespace=<namespace>

# List all namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace <name>
```

### Namespace Isolation
```mermaid
graph TD
    A[Kubernetes Cluster] --> B[default]
    A --> C[kube-system]
    A --> D[production]
    A --> E[development]
    A --> F[monitoring]
    
    B --> B1[App Pods]
    C --> C1[System Pods]
    D --> D1[Prod App]
    E --> E1[Dev App]
    F --> F1[Monitoring Tools]
```

---

## 7. Debugging & Inspection

### Debugging Commands
```bash
# Execute a command in a pod
kubectl exec -it <pod-name> -- /bin/bash

# Port-forward to a pod
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Check resource usage
kubectl top pods
kubectl top nodes
```

### Debugging Workflow
```mermaid
flowchart TD
    A[Issue Detected] --> B[Get Pod Status]
    B --> C[kubectl get pods]
    C --> D{All Pods Running?}
    D -->|No| E[Check Events]
    D -->|Yes| F[Check Logs]
    
    E --> E1[kubectl describe pod]
    F --> F1[kubectl logs]
    
    E1 --> G[Identify Issue]
    F1 --> G
    
    G --> H[Execute Debug Commands]
    H --> I[kubectl exec]
```

---

## 8. YAML Management

### YAML Operations
```bash
# Create resources from YAML
kubectl apply -f <file.yaml>

# Delete resources from YAML
kubectl delete -f <file.yaml>

# Generate YAML for a resource
kubectl create deployment <name> --image=<image> --dry-run=client -o yaml
```

### Resource Creation Flow
```mermaid
sequenceDiagram
    participant U as User
    participant K as kubectl
    participant A as API Server
    participant E as etcd
    
    U->>K: kubectl apply -f config.yaml
    K->>A: POST /apis/apps/v1/namespaces/default/deployments
    A->>E: Store deployment spec
    A->>K: Return success
    K->>U: deployment.apps/my-app created
```

---

## 9. Context & Configuration

### Context Management
```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>
```

### Multi-Context Architecture
```mermaid
graph TB
    A[Local Machine] --> B[Context: dev-cluster]
    A --> C[Context: prod-cluster]
    A --> D[Context: staging-cluster]
    
    B --> E[Development<br/>K8s Cluster]
    C --> F[Production<br/>K8s Cluster]
    D --> G[Staging<br/>K8s Cluster]
```

---

## 10. Cleanup Operations

### Cleanup Commands
```bash
# Delete all pods in a namespace
kubectl delete pods --all

# Delete all resources in a namespace
kubectl delete all --all
```

### Resource Hierarchy for Cleanup
```mermaid
graph TD
    A[Namespace] --> B[Deployments]
    A --> C[Services]
    A --> D[ConfigMaps]
    A --> E[Secrets]
    
    B --> F[ReplicaSets]
    F --> G[Pods]
    
    C --> H[Endpoints]
    
    style A fill:#ffebee
```

---

## Quick Reference Tips

### Common Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-n` | Namespace | `kubectl get pods -n kube-system` |
| `-o wide` | Detailed output | `kubectl get nodes -o wide` |
| `--watch` | Real-time monitoring | `kubectl get pods --watch` |
| `--dry-run` | Simulation mode | `kubectl create --dry-run=client` |

### Resource Types Shortcuts
| Shortcut | Full Name |
|----------|-----------|
| `po` | pods |
| `deploy` | deployments |
| `svc` | services |
| `cm` | configmaps |
| `ns` | namespaces |

### Helpful Tips
- Use `kubectl explain <resource>` for documentation
- Use `kubectl api-resources` to see all available resources
- Add `--v=6` for debug-level logging
- Use `kubectl get events --sort-by=.metadata.creationTimestamp` to see recent events