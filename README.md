# Kubernetes Hands-On Labs with Minikube

This repository contains my **hands-on experience with Kubernetes** using **Minikube**, covering Pods, ReplicaSets, Deployments, Services, ConfigMaps, and Secrets.  

---

## Table of Contents

1. [Setup Minikube](#setup-minikube)  
2. [Lab 1: Create a Cluster](#lab-1-create-a-cluster)  
3. [Lab 2: Create a Namespace](#lab-2-create-a-namespace)  
4. [Lab 3: Create a Pod](#lab-3-create-a-pod)  
5. [Lab 3b: ReplicaSets](#lab-3b-create-replicasets)  
6. [Lab 4: Deployments](#lab-4-create-deployments)  
7. [Lab 5: Services](#lab-5-create-services)  
8. [Lab 6: ConfigMaps](#lab-6-configmaps)  
9. [Lab 7: Secrets](#lab-7-secrets)  

---

## Setup Minikube

### Setup on Docker Desktop (Mac)

1. **Install Minikube**:

```bash
brew install minikube
minikube start --driver=docker
minikube ssh
# On Mac, kubectl might already be installed with Minikube
kubectl version --client

### Why `minikube start --driver=docker`

- Using `--driver=docker` tells Minikube to use Docker as the **hypervisor**.
- Kubernetes runs **inside a Docker container**, with a **single-node cluster**.
- This method avoids needing a VM and integrates easily with Docker Desktop.

Run kubectl without sudo:

kubectl communicates with the Kubernetes API server, fetching resource info from etcd.

Minikube with Docker driver is designed to run as a normal user, not root.

If you face permission issues, add your user (default ubuntu) to the Docker group:
sudo usermod -aG docker $USER
```

## Lab 1: Create a Namespace

```bash
kubectl get ns                             # List all namespaces
kubectl create namespace dev               # Create namespace 'dev'
kubectl config set-context --current --namespace=dev  # Switch to 'dev'
```

## Lab 2: Create a Pod

```bash
kubectl apply -f pod.yml                   # Create or update pod
kubectl get pods                           # List pods in current namespace
kubectl get pods -A                        # List pods in all namespaces
kubectl describe pod <pod-name>            # Detailed info about pod
kubectl logs <pod-name>                     # View pod logs
kubectl get pods -w                         # Watch pods live
kubectl exec -it <pod-name> -- /bin/bash   # Enter container shell
```

Example of pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web-app
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports: 
     - containerPort: 80

```

## Lab 3: ReplicaSets

```bash
kubectl get rs                     # List ReplicaSets
kubectl apply -f replicas.yml      # Create ReplicaSets

```

Example of replicas.yml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicas
  labels:
    app: replics
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: replic-web
  template:
    metadata:
      labels:
        app: replic-web
    spec:
      containers:
        - name: replica-container
          image: nginx:1.14.2
          ports:
           - containerPort: 80
 

```


## Lab 4: Deployments

```bash
kubectl get deployments
kubectl rollout status deployment/<deployment-name>   # Check rollout status
kubectl rollout undo deployment/<deployment-name>    # Undo deployment

```
#### Best practice in production

1. Rollback is usually done via GitOps: revert the Git commit → CI/CD redeploys previous stable version.

2. Manual rollback with kubectl rollout undo is avoided in production.

Example of deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  labels: 
    app: Deployment-controller
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-pod
  template:
    metadata:
      labels:
        app: deployment-pod
    spec:
      containers:
      - name: deployment-container
        image: nginx:latest
        ports:
        - containerPort: 80
        # envFrom:
        # - configMapRef:
        #     name: test-configmap
        volumeMounts:
          - name: config-mount-volume
            mountPath: /app/config
      volumes:
        - name: config-mount-volume ## volumemounts.name and volumes.name should be the same.
          configMap:
            name: volume-mount-config-map
        

```


## Lab 5: Services 

```bash
kubectl get svc                 # List services
kubectl describe svc <svc-name> # Describe service

```
#### Important Points :

 1. Clusterip : it is the ip assigned to service by kubernetes. pod can be communicate within cluster through this ip : https://service-ip-address:port

2. External Ip : Kubernetes cannot directly assign the ip to access the client outside the cluster. so in nodeport type service, client can access the application through the ***nodeip:port***.
3. In load balancer service, cloud provider assign the ip to the service and client can access the application through that ip address.
4. If in service if port is other than 80/443, client can access the service if the service is load balancer then ***https://ip-address:port-number***. If it is 80/443, it can be access directly without port.

### Service Types :

| Type         | Description                                                      |
| ------------ | ---------------------------------------------------------------- |
| ClusterIP    | Internal cluster communication only                              |
| NodePort     | Accessible externally via `NodeIP:NodePort` (range: 30000–32767) |
| LoadBalancer | Cloud provider assigns external IP to service                    |

#### Flow of Nodeport service:

              🌍 External User (Browser)
                           |
                           |
                           v
                http://<NodeIP>:30513
                           |
                           |   (1) NodePort
                           v
        ┌──────────────────────────────────┐
        │          Kubernetes Node         │
        │                                  │
        │   NodePort: 30513                │
        │        |                         │
        │        v                         │
        │   Service Port: 8080             │
        │        |                         │
        │        v                         │
        │   Pod IP: 10.x.x.x               │
        │        |                         │
        │        v                         │
        │   Container Port: 80 (nginx)     │
        └──────────────────────────────────┘

Example of service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  type: NodePort
  selector:
    app: deployment-pod
  ports:
  - port: 8080
    targetPort: 80
```


## Lab 6: ConfigMaps 

```bash
kubectl get cm
kubectl describe cm <configmap-name>
kubectl exec -it <pod-name> -- /bin/bash
env | grep <KEY_NAME>
```

Example of configmap.yml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: volume-mount-config-map
  labels:
    app: volume-mount-config-map
data:
  db-username: "ayushi"

```

And uses of configmap in deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  labels: 
    app: Deployment-controller
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-pod
  template:
    metadata:
      labels:
        app: deployment-pod
    spec:
      containers:
      - name: deployment-container
        image: nginx:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: test-configmap
        
```

Example of deployment.yml with volume mounts of configmap sets:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  labels: 
    app: Deployment-controller
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-pod
  template:
    metadata:
      labels:
        app: deployment-pod
    spec:
      containers:
      - name: deployment-container
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config-mount-volume
            mountPath: /app/config
      volumes:
        - name: config-mount-volume ## volumemounts.name and volumes.name should be the same.
          configMap:
            name: volume-mount-config-map
        
```

## Lab 7: Secrets :

```bash
kubectl get secrets
kubectl describe secret test-secrets
```

### Important point about secrets

1. Secrets are used to store sensitive information such as: Passwords, API keys, Tokens, SSH keys
2. Secrets are Base64-encoded, but NOT encrypted by default.
3. Secrets can be: Mounted as files inside Pods or Injected as environment 
4. Kubernetes stores secrets in etcd (encoded format).
5. When running:
    ```bash
    kubectl get secrets
    ```
    Note: Kubernetes displays the Base64-encoded values, not the decoded content.

#### Extra bonus point :
  ```bash
      echo -n "admin" | base64
  ```
  This converts plain text into Base64 format.

### Using stringData (Recommended)

1. Instead of manually encoding values, you can use stringData.
2. Kubernetes will automatically:
    a). Encode the values
    b). Store them in Base64 format
    c). Decode them inside the Pod when used

#### ⚠️ Security Note

1. Base64 encoding is NOT encryption. It only converts data into an encoded format.
2. For stronger security, enable:
    a). Encryption at rest
    b). External secret managers (e.g., Vault, AWS Secrets Manager)


### Example of secrets.yml with stringdata

```yaml
  apiVersion: v1
kind: Secret
metadata:
  name: encoded-secret
  labels:
    app: secrets
type: Opaque
stringData:
  db_username: "admin"
  db_password: "adminpassword"
```

### Example of secrets.yml without strindata

```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: test-secrets
    labels:
      app: test-secrets
  data:
    db-password: c3VwZXJzZWNyZXQ=
```

## Lab - 7: Ingress Controller

### 🚀 Enable Ingress in Minikube
```bash
  minikube enable ingress
```
- This installs the NGINX Ingress Controller. 


