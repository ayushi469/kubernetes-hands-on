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
