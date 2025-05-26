# Resource Requirements in Kubernetes

Resource requirements allow you to specify how much CPU and memory (RAM) each container in a pod needs. This is crucial for efficient scheduling, resource allocation, and overall cluster stability.

## Understanding Kubernetes Resources

### Resource Types

1. **CPU** - Measured in CPU units. 1 CPU unit equals:
   - 1 AWS vCPU
   - 1 GCP Core
   - 1 Azure vCore
   - 1 Hyperthread on a bare-metal processor

2. **Memory** - Measured in bytes, but typically specified with suffixes:
   - `Ki`: Kibibyte (1024 bytes)
   - `Mi`: Mebibyte (1024 Ki)
   - `Gi`: Gibibyte (1024 Mi)
   - `Ti`: Tebibyte (1024 Gi)

## Resource Requests and Limits

### Requests

Requests specify the minimum amount of resources needed by a container:

- Used by the scheduler to find a suitable node
- Guaranteed to be available to the container
- Node must have enough allocatable resources to satisfy the request

### Limits

Limits specify the maximum amount of resources a container can use:

- Container can't use more than the specified limit
- No guaranteed minimum (requests handle that)
- If a container exceeds its memory limit, it might be terminated (OOMKilled)
- If a container tries to use more CPU than its limit, it will be throttled

## Configuring Resource Requirements

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-example
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "500m"     # 500 milliCPU (0.5 CPU)
      limits:
        memory: "256Mi"
        cpu: "1"        # 1 CPU
```

## Resource Units

### CPU Units

CPU resources are specified in "CPU units":

- `1` or `1000m` = 1 full CPU core
- `500m` = 0.5 CPU (half a core)
- `100m` = 0.1 CPU (10% of a core)
- `10m` = 0.01 CPU (1% of a core)

### Memory Units

Memory can be specified in several ways:

- Plain integer (bytes): `128974848`
- Powers of 2 with suffixes: `128Mi` (Mebibytes)
- Powers of 10 with suffixes: `100M` (Megabytes)

## Quality of Service (QoS) Classes

Kubernetes assigns one of three QoS classes to pods based on their resource configurations:

### Guaranteed

- **Requirements**: Every container in the pod must have CPU and memory requests equal to limits
- **Priority**: Highest priority, least likely to be evicted
- **Example**:
  ```yaml
  resources:
    requests:
      memory: "256Mi"
      cpu: "500m"
    limits:
      memory: "256Mi"
      cpu: "500m"
  ```

### Burstable

- **Requirements**: At least one container has a request or limit set, but not meeting "Guaranteed" criteria
- **Priority**: Medium priority, might be evicted if resources are needed
- **Example**:
  ```yaml
  resources:
    requests:
      memory: "128Mi"
      cpu: "250m"
    limits:
      memory: "256Mi"
      cpu: "500m"
  ```

### BestEffort

- **Requirements**: No resource requests or limits specified
- **Priority**: Lowest priority, first to be evicted if needed
- **Example**: No resource specifications at all

## LimitRange Resource

LimitRange objects set default, minimum, and maximum values for resource requests and limits for containers within a namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-constraints
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 100m
      memory: 64Mi
    default:
      cpu: 200m
      memory: 128Mi
    min:
      cpu: 50m
      memory: 32Mi
    max:
      cpu: 1
      memory: 512Mi
```

## ResourceQuota

ResourceQuota objects limit the total amount of resources that can be used within a namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
    pods: "10"
```

## Best Practices

1. **Always specify requests and limits** for production workloads
2. **Set realistic resource requests** based on actual application needs
3. **Monitor actual resource usage** to optimize requests and limits
4. **Be conservative with memory limits** as exceeding them causes termination
5. **Consider using LimitRange** for default values in development namespaces
6. **Use ResourceQuotas** to prevent resource exhaustion at namespace level
7. **Strive for "Guaranteed" QoS** class for critical workloads

## Common Pitfalls

1. **Setting CPU limits too low** - Can throttle applications unnecessarily
2. **Setting memory limits too low** - Can cause OOMKilled pod terminations
3. **Very high resource requests** - Can lead to underutilized nodes and inefficient scheduling
4. **Ignoring resource requirements** - Can lead to node resource contention and instability
5. **Using wrong units** - E.g., "MB" vs "Mi" can cause significant differences

## CKAD Exam Tips

1. **Know the syntax** - Be able to quickly write resource requests and limits for pods
2. **Understand units** - Know the difference between `m` for CPU and `Mi`, `Gi` for memory
3. **Remember QoS classes** - Understand how they're determined and their implications
4. **Practice troubleshooting** - Recognize issues related to resource constraints
5. **Know how to check resource usage** with `kubectl top pods` and `kubectl describe pod`

## Example CKAD Task

**Task**: Create a pod with the following specifications:
- Name: `resource-pod`
- Image: `nginx`
- CPU request: `200 milliCPU`
- CPU limit: `500 milliCPU`
- Memory request: `256 Mebibytes`
- Memory limit: `512 Mebibytes`

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
```

## Relevant Documentation

- [Kubernetes Documentation: Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Kubernetes Documentation: Quality of Service](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)
- [Kubernetes Documentation: Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Kubernetes Documentation: Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)