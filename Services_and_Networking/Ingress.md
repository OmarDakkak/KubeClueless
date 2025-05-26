# Ingress in Kubernetes

Ingress is an API resource that manages external access to services in a Kubernetes cluster, typically HTTP and HTTPS traffic. It provides features not available in a simple Service, such as URL path-based routing, name-based virtual hosting, TLS termination, and more.

## Key Concepts

- **Ingress**: The Kubernetes resource that manages external access to services
- **Ingress Controller**: The software component that fulfills the Ingress rules (e.g., nginx, traefik, HAProxy)
- **Ingress Rules**: Definition of how HTTP/HTTPS traffic should be routed to services
- **Paths**: URL paths that are mapped to Kubernetes services
- **Hosts**: Domain names used for virtual hosting

## Why Use Ingress?

1. **Centralized routing**: Manage all incoming HTTP traffic in one place
2. **Path-based routing**: Route traffic based on URL paths
3. **Name-based virtual hosting**: Host multiple sites on the same IP address
4. **TLS/SSL termination**: Manage certificates and encryption
5. **Load balancing**: Distribute traffic across pods
6. **Simplified networking**: Avoid managing multiple external LoadBalancer services

## Basic Ingress Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx       # Specifies which controller to use
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix        # Prefix, Exact, or ImplementationSpecific
        backend:
          service:
            name: test
            port:
              number: 80
```

## Ingress Controllers

Ingress resources don't do anything by themselves - you need an Ingress controller to implement the rules. Some popular controllers:

- **NGINX Ingress Controller**: Most commonly used
- **Traefik**: Modern cloud-native edge router
- **HAProxy**: High-performance load balancer
- **Ambassador/Emissary**: Kubernetes-native API Gateway
- **Istio**: Service mesh that includes Ingress capabilities

### Installing NGINX Ingress Controller Example:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

## Common Ingress Examples

### 1. Single Service Ingress

Routes all traffic to a single service:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: single-service-ingress
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: my-service
      port:
        number: 80
```

### 2. Path-Based Routing

Routes traffic to different services based on URL path:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-routing
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### 3. Name-Based Virtual Hosting

Routes traffic based on the hostname:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: foo.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: foo-service
            port:
              number: 80
  - host: bar.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bar-service
            port:
              number: 80
```

### 4. TLS/SSL Configuration

Configures TLS termination using a specified secret containing the certificate:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - secure.example.com
    secretName: example-tls-secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-service
            port:
              number: 80
```

The TLS secret should contain the certificate and private key:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-tls-secret
type: kubernetes.io/tls
data:
  tls.crt: base64_encoded_cert
  tls.key: base64_encoded_key
```

You can create this secret from local files:

```bash
kubectl create secret tls example-tls-secret --cert=path/to/tls.crt --key=path/to/tls.key
```

## Ingress PathType Options

- **Prefix**: Matches the URL path prefix. For example, `/foo` would match `/foo`, `/foo/`, and `/foobar`
- **Exact**: Exactly matches the URL path. For example, `/foo` only matches `/foo`, not `/foo/` or `/foobar`
- **ImplementationSpecific**: The matching is dependent on the IngressClass controller implementation

## Important Annotations

Annotations allow you to customize the behavior of specific Ingress controllers:

### NGINX Ingress Controller Annotations:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "30"
```

## Complete Ingress Example with Annotations

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: comprehensive-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/enable-cors: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    secretName: example-tls-secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: main-service
            port:
              number: 80
```

## Debugging Ingress

### Check Ingress Status

```bash
kubectl get ingress
kubectl describe ingress <ingress-name>
```

### Check Ingress Controller Logs

```bash
kubectl logs -n <ingress-controller-namespace> <ingress-controller-pod-name>
```

### Check Ingress Controller Service

```bash
kubectl get service -n <ingress-controller-namespace>
```

### Test with curl

```bash
# Add a temporary host entry for testing
curl -H "Host: example.com" http://<ingress-IP>
```

## IngressClass Resource

Starting with Kubernetes 1.18, the IngressClass resource is used to specify which controller should implement the Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```

## Common Issues and Solutions

1. **Ingress not working**: Ensure the Ingress controller is running
2. **Rules not applied**: Check if the Ingress class matches the available controller
3. **Service not found**: Verify the service referenced in the Ingress exists and has endpoints
4. **TLS not working**: Check if the TLS secret exists and is correctly formatted
5. **Path matching issues**: Double check path types and any regex configurations
6. **Timeout issues**: Adjust timeout values in annotations if necessary

## Best Practices

1. **Use a consistent naming scheme** for your Ingress resources
2. **Organize services logically** behind Ingress controllers
3. **Use TLS** for production workloads
4. **Be cautious with regex paths** as they can impact performance
5. **Configure appropriate timeouts** based on application needs
6. **Monitor Ingress controller metrics** for performance issues
7. **Regularly update your Ingress controller** to get security fixes
8. **Consider using wildcard certificates** for multiple subdomains
9. **Document annotations used** in your Ingress configurations

## CKAD Exam Tips

1. **Know the basic structure**: Be able to create simple Ingress resources quickly
2. **Understand pathType options**: Know when to use Prefix vs Exact
3. **Be familiar with backend service configuration**: How to specify service name and port
4. **Know how to configure TLS**: Create secrets for TLS termination
5. **Understand multiple hosts and paths**: Configure virtual hosts and path-based routing
6. **Be aware of Ingress class**: Specify the correct IngressClass for your controller

## Example CKAD Tasks

### Task 1: Create a Basic Ingress

**Task**: Create an Ingress resource named `app-ingress` that routes traffic to the service `web-app` on port 80 for all paths.

**Solution**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
```

### Task 2: Create a Multi-Path Ingress

**Task**: Create an Ingress resource that routes traffic to different services based on path:
- `/api` should go to `api-service:8080`
- `/` should go to `web-service:80`

**Solution**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

## Relevant Documentation

- [Kubernetes Documentation: Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Controller Documentation](https://kubernetes.github.io/ingress-nginx/)
- [Kubernetes Documentation: TLS Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets)
- [Ingress Controllers Comparison](https://docs.google.com/spreadsheets/d/191WWNpjJ2za6-nbG4ZoUMXMpUK8KlCIosvQB0f-oq3k/)