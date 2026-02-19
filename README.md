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

```
kubectl get ns                             # List all namespaces
kubectl create namespace dev               # Create namespace 'dev'
kubectl config set-context --current --namespace=dev  # Switch to 'dev'
```

## Lab 2: Create a Pod

```
kubectl apply -f pod.yml                   # Create or update pod
kubectl get pods                           # List pods in current namespace
kubectl get pods -A                        # List pods in all namespaces
kubectl describe pod <pod-name>            # Detailed info about pod
kubectl logs <pod-name>                     # View pod logs
kubectl get pods -w                         # Watch pods live
kubectl exec -it <pod-name> -- /bin/bash   # Enter container shell
```

Example of pod.yml

<img width="399" height="359" alt="image" src="https://github.com/user-attachments/assets/511add9c-f447-4894-87b2-2dd8ca0da64f" />


## Lab 3: ReplicaSets

```
kubectl get rs                     # List ReplicaSets
kubectl apply -f replicas.yml      # Create ReplicaSets

```

Example of replicas.yml

<img width="506" height="597" alt="image" src="https://github.com/user-attachments/assets/4cb41d98-7833-42c8-aab7-e5a873ebe7c6" />

## Lab 4: Deployments

```
kubectl get deployments
kubectl rollout status deployment/<deployment-name>   # Check rollout status
kubectl rollout undo deployment/<deployment-name>    # Undo deployment

```
#### Best practice in production

1. Rollback is usually done via GitOps: revert the Git commit → CI/CD redeploys previous stable version.

2. Manual rollback with kubectl rollout undo is avoided in production.

Example of deployment.yml

<img width="900" height="782" alt="image" src="https://github.com/user-attachments/assets/0ce583dd-5625-4372-9f3a-6955396f4a86" />

## Lab 5: Services 

```

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

<img width="435" height="412" alt="image" src="https://github.com/user-attachments/assets/f2172ed9-c537-4b4a-9c56-1347b3d15710" />


## Lab 6: ConfigMaps 

```

kubectl get cm
kubectl describe cm <configmap-name>
kubectl exec -it <pod-name> -- /bin/bash
env | grep <KEY_NAME>

```

Example of configmap.yml

<img width="309" height="249" alt="image" src="https://github.com/user-attachments/assets/cd300834-3738-464b-87be-50f18c90ee9e" />

Example of deployment.yml with volume mounts:

<img width="988" height="838" alt="image" src="https://github.com/user-attachments/assets/47b75599-1c85-412f-89ae-5ab60ca2c5ac" />



