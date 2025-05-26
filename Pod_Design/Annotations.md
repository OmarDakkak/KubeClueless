# Annotations in Kubernetes

Annotations in Kubernetes are non-identifying metadata attached to objects that can be used to store additional information. Unlike labels, annotations are not used to identify and select objects, but they can hold much larger and more complex data.

## Annotations vs. Labels

| Feature | Annotations | Labels |
|---------|------------|--------|
| Purpose | Store non-identifying metadata | Identify and select objects |
| Used for | Metadata, tool integration, build info | Grouping, filtering, selecting resources |
| Size limit | Can store larger data | Limited to short strings |
| Selector support | Not selectable | Can be used with selectors |
| Example use case | Build info, timestamp, URLs, config details | Environment, tier, version, app name |

## Annotation Structure

Like labels, annotations are key-value pairs:

```yaml
metadata:
  annotations:
    key1: "value1"
    key2: "value2"
```

Annotation keys can optionally include a prefix and a slash, e.g., `example.com/key: value`.

## Common Annotation Use Cases

### 1. Build and Release Information

```yaml
metadata:
  annotations:
    build.kubernetes.io/commit: "abc123"
    build.kubernetes.io/repository: "github.com/org/repo"
    build.kubernetes.io/branch: "main"
    release.kubernetes.io/version: "v1.2.3"
    jenkins.io/build-number: "42"
```

### 2. Tool Configuration

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

### 3. Deployment Management

```yaml
metadata:
  annotations:
    kubernetes.io/change-cause: "Updated application to v2.0"
    deployment.kubernetes.io/revision: "3"
```

### 4. Documentation and References

```yaml
metadata:
  annotations:
    doc.kubernetes.io/description: "Frontend web application"
    doc.kubernetes.io/team-owner: "frontend-team"
    doc.kubernetes.io/contact: "frontend-team@example.com"
    doc.kubernetes.io/docs: "https://wiki.example.com/frontend"
```

## Adding and Managing Annotations

### In YAML Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-example
  annotations:
    description: "This is a test pod"
    owner: "team-alpha"
    version: "1.0.0"
    lastModified: "2023-05-15"
spec:
  containers:
  - name: nginx
    image: nginx:1.19
```

### Using kubectl

```bash
# Add annotations to a new resource
kubectl run nginx --image=nginx --annotations="description=web server,owner=operations"

# Add annotations to existing resources
kubectl annotate pods nginx description="web server" owner="operations"

# Update an existing annotation
kubectl annotate pods nginx description="main web server" --overwrite

# Remove an annotation
kubectl annotate pods nginx description-
```

## Annotations for System Components

Kubernetes itself uses annotations for various purposes:

### Ingress Controller Annotations

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  # ingress spec
```

### Service Annotations

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:region:account-id:certificate/cert-id
spec:
  # service spec
```

### Deployment History and Rollback

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
  annotations:
    kubernetes.io/change-cause: "Update to version 2.0"
spec:
  # deployment spec
```

To see deployment history with annotations:
```bash
kubectl rollout history deployment example-deployment
```

## Working with JSON Data in Annotations

Annotations can contain complex JSON data:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: complex-config
  annotations:
    config-schema: |
      {
        "type": "object",
        "properties": {
          "port": {
            "type": "integer",
            "minimum": 1024
          },
          "environment": {
            "type": "string",
            "enum": ["dev", "staging", "prod"]
          }
        },
        "required": ["port", "environment"]
      }
```

## Best Practices for Annotations

1. **Use namespaced keys**: Prefix annotation keys with a domain you control
2. **Document annotations**: Make sure the purpose of each annotation is clear
3. **Avoid sensitive data**: Don't store passwords or secrets in annotations
4. **Use structured data formats**: JSON or YAML is better for complex data
5. **Be consistent**: Follow a naming convention for your annotations
6. **Consider size limitations**: While larger than labels, annotations still have size limits
7. **Don't overuse**: Use only for necessary metadata, not for all information

## Annotations for Specific Kubernetes Features

### HorizontalPodAutoscaler Annotations

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
  annotations:
    metric-config.pods.custom-metric.scaling: '{"targetValue": 10}'
spec:
  # HPA spec
```

### StatefulSet Annotations

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-sts
  annotations:
    statefulset.kubernetes.io/pod-name-format: "%s"
spec:
  # StatefulSet spec
```

### Job and CronJob Annotations

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
  annotations:
    cronjob.kubernetes.io/instantiate: "manual"
spec:
  # Job spec
```

## Field References vs. Annotations

Consider using the Kubernetes API fields if they exist for your use case:

```yaml
# Using annotations (not optimal for this case)
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  annotations:
    restartPolicy: "Always"  # This should be in spec
spec:
  containers:
  - name: nginx
    image: nginx

# Using proper API fields (better)
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  restartPolicy: Always  # Proper field in spec
  containers:
  - name: nginx
    image: nginx
```

## CKAD Exam Tips

1. **Know the difference between labels and annotations**: Understand that labels are for selection and annotations are for metadata
2. **Be comfortable with kubectl annotate commands**: Know how to add, update, and remove annotations
3. **Understand common annotation patterns**: Recognize how annotations are used in Ingress, Services, and Deployments
4. **Know when to use annotations vs. labels**: Be able to choose the right metadata type for a given scenario
5. **Understand when to use annotations vs. built-in fields**: Don't use annotations for data that belongs in the spec
6. **Recognize important system annotations**: Be familiar with annotations used by Kubernetes components

## Example CKAD Tasks

### Task 1: Add Descriptive Annotations to a Deployment

**Task**: Add annotations to an existing deployment named `web-app` to track the following information:
- A description of "Frontend web application"
- The responsible team email "frontend@example.com"
- The current version "2.1.5"

**Solution**:
```bash
kubectl annotate deployment web-app \
  description="Frontend web application" \
  team="frontend@example.com" \
  version="2.1.5"
```

### Task 2: Configure a Pod for Prometheus Monitoring

**Task**: Create a pod with appropriate annotations to enable Prometheus scraping on port 8080 at the path `/metrics`.

**Solution**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prometheus-example
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  containers:
  - name: app
    image: app:latest
    ports:
    - containerPort: 8080
```

### Task 3: Update a Change-Cause Annotation

**Task**: Update the `kubernetes.io/change-cause` annotation on a deployment named `api-service` to reflect a configuration change.

**Solution**:
```bash
kubectl annotate deployment api-service kubernetes.io/change-cause="Updated database connection settings" --overwrite
```

## Relevant Documentation

- [Kubernetes Documentation: Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
- [Kubernetes Documentation: Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
- [kubectl annotate command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#annotate)