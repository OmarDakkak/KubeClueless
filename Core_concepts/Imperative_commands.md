
# Kubernetes Imperative Commands Cheat Sheet

This cheat sheet provides useful Kubernetes commands for creating and managing Kubernetes objects imperatively.

## 1. Create a Pod
To create a pod with a specific image, run:

```bash
kubectl run <pod-name> --image=<image-name>
```

**Example:**
```bash
kubectl run nginx-pod --image=nginx:alpine
```

## 2. Deploy a Pod with Labels
To create a pod and assign labels, use the `-l` flag to specify labels:

```bash
kubectl run <pod-name> -l <label-key>=<label-value> --image=<image-name>
```

**Example:**
```bash
kubectl run redis --image=redis:alpine -l tier=db
```

## 3. Expose a Pod with a Service
To expose a pod as a service within the cluster, use the `expose` command:

```bash
kubectl expose pod <pod-name> --port=<service-port> --name=<service-name>
```

**Example:**
```bash
kubectl expose pod redis --port=6379 --name=redis-service
```

## 4. Create a Deployment
To create a deployment with a specified number of replicas, use the following command:

```bash
kubectl create deployment <deployment-name> --image=<image-name> --replicas=<number-of-replicas>
```

**Example:**
```bash
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

## 5. Create a Pod with a Specific Container Port
To create a pod with a specific container port:

```bash
kubectl run <pod-name> --image=<image-name> --port=<container-port>
```

**Example:**
```bash
kubectl run custom-nginx --image=nginx --port=8080
```

## 6. Create a Namespace
To create a new namespace:

```bash
kubectl create namespace <namespace-name>
```

**Shortcut:**
```bash
kubectl create ns <namespace-name>
```

**Example:**
```bash
kubectl create namespace dev-ns
```

## 7. Create a Deployment in a Specific Namespace
To create a deployment in a specific namespace:

```bash
kubectl create deployment <deployment-name> --image=<image-name> --replicas=<number-of-replicas> -n <namespace-name>
```

**Example:**
```bash
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

## 8. Create a Pod and Expose It as a Service
To create a pod and expose it as a service of type ClusterIP:

```bash
kubectl run <pod-name> --image=<image-name> --port=<container-port> --expose
```

**Example:**
```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose
```
s