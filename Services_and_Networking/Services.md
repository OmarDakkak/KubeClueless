# Kubernetes Services

Services in Kubernetes are an abstraction that defines a logical set of pods and a policy to access them. Services enable network connectivity to a set of pods, allowing them to communicate within the cluster or be exposed to the outside world.

## Service Types

Kubernetes offers several types of services to meet different networking requirements:

### 1. ClusterIP (Default)
- Internal service only, accessible within the cluster
- Provides a stable internal IP address
- Enables pod-to-pod communication

### 2. NodePort
- Exposes the service on each node's IP at a static port
- Accessible from outside the cluster using `<NodeIP>:<NodePort>`
- NodePort range: 30000-32767 by default

### 3. LoadBalancer
- Integrates with cloud provider's load balancer
- Exposes the service externally using a cloud provider's load balancer
- Automatically creates NodePort and ClusterIP services as well

### 4. ExternalName
- Maps a service to a DNS name
- No proxying is set up
- Used for service discovery without selectors

## Basic Service Definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - port: 80          # Port exposed by the service
      targetPort: 9376  # Port on the pods to which traffic is directed
  type: ClusterIP       # Service type (ClusterIP is default)
```

## Creating Services Imperatively

### ClusterIP Service

```bash
kubectl create service clusterip my-service --tcp=80:8080
```

### NodePort Service

```bash
kubectl create service nodeport my-service --tcp=80:8080
```

### LoadBalancer Service

```bash
kubectl create service loadbalancer my-service --tcp=80:8080
```

## Accessing Services

### Inside the Cluster

- **DNS**: `<service-name>.<namespace>.svc.cluster.local`
- **Environment Variables**: For each active service, kubelet adds environment variables like:
  - `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT`

### Outside the Cluster

- **NodePort**: Access via `<node-ip>:<nodeport>`
- **LoadBalancer**: Access via external IP assigned by cloud provider
- **Ingress**: Use an Ingress controller (more advanced)

## Service Discovery

Services enable pods to discover and communicate with each other:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
spec:
  containers:
  - name: client
    image: nginx
    command: ["/bin/sh", "-c"]
    args:
    - while true; do
        curl -s http://my-service.default.svc.cluster.local;
        sleep 1;
      done
```

## Exposing Deployments as Services

```bash
kubectl expose deployment my-deployment --name=my-service --port=80 --target-port=8080 --type=ClusterIP
```

## Port Terminology

- **port**: The port the service is exposed on within the cluster
- **targetPort**: The port on the pod to which traffic is forwarded
- **nodePort**: The port on each node where the service is exposed (NodePort and LoadBalancer types)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  type: ClusterIP
```

## Service without Selector

Services can be defined without pod selectors to point to:
- External service
- Service in a different namespace
- IP addresses

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 192.168.1.100
  ports:
  - port: 80
```

## Headless Services

Used when you don't need load balancing or a single Service IP, but still want DNS records:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: headless-app
  ports:
  - port: 80
    targetPort: 8080
```

## SessionAffinity

Enable "sticky sessions" to route requests from a client to the same pod:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: session-affinity-service
spec:
  selector:
    app: my-app
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  ports:
  - port: 80
    targetPort: 80
```

## Common Use Cases

### Backend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### Public Web Application

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

### Database Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: db
  ports:
  - port: 3306
    targetPort: 3306
  type: ClusterIP
```

## Service Networking Troubleshooting

### Check Service Details
```bash
kubectl describe service my-service
```

### Test Service Connectivity
```bash
kubectl run -i --tty --rm debug --image=busybox --restart=Never -- wget -qO- http://my-service:80
```

### Check Endpoints
```bash
kubectl get endpoints my-service
```

## CKAD Exam Tips

1. **Know how to create services**: Be comfortable with both declarative and imperative methods
2. **Understand service types**: Know when to use ClusterIP, NodePort, LoadBalancer, and ExternalName
3. **Master port configurations**: Understand the difference between port, targetPort, and nodePort
4. **Practice service troubleshooting**: Know how to check endpoints and test connectivity
5. **Expose deployments**: Be able to quickly expose deployments as services

## Example CKAD Tasks

### Task 1: Create a Service for a Deployment

**Task**: Create a service named `nginx-service` that exposes the `nginx-deployment` with port 80 targeting container port 80.

**Solution**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Or imperatively:

```bash
kubectl expose deployment nginx-deployment --name=nginx-service --port=80 --target-port=80
```

### Task 2: Create a NodePort Service

**Task**: Create a NodePort service named `web-service` that exposes port 80 on the pods with label `app=web` on port 30080 on the nodes.

**Solution**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

## Relevant Documentation

- [Kubernetes Documentation: Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Tasks: Connect Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
- [Kubernetes Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)