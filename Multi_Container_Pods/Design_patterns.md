# Multi-Container Pod Design Patterns

Multi-container pods are pods that run more than one container that work together to provide a single service. These containers share the same network namespace, which means they can communicate with each other using `localhost`, and they share the same storage volumes.

## Common Design Patterns

There are three common patterns for multi-container pods in Kubernetes:

1. **Sidecar Pattern**
2. **Ambassador Pattern**
3. **Adapter Pattern**

## Sidecar Pattern

A sidecar container extends and enhances the functionality of the main container without changing it.

### Use Cases:
- Logging agents
- Monitoring agents
- Sync services
- Proxies

### Example: Logging Sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver-with-logging-sidecar
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: log-collector
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  volumes:
  - name: logs-volume
    emptyDir: {}
```

In this example:
- The `nginx` container writes logs to `/var/log/nginx`
- The `log-collector` sidecar reads these logs
- Both containers share the `logs-volume` volume

## Ambassador Pattern

An ambassador container proxies network traffic to and from the main container, serving as a representative of the main container to the outside world.

### Use Cases:
- Connection pooling
- TLS termination
- Service discovery
- Rate limiting

### Example: Redis Ambassador

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-ambassador
spec:
  containers:
  - name: redis
    image: redis
  - name: redis-ambassador
    image: redis-ambassador
    ports:
    - containerPort: 6379
```

In this example:
- The `redis` container handles data storage
- The `redis-ambassador` handles network connections, potentially providing retry logic or connection pooling

## Adapter Pattern

An adapter container transforms the main container's output to conform to a standard interface or expected format.

### Use Cases:
- Data transformation
- Normalizing monitoring output
- API adaptation

### Example: Monitoring Adapter

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-monitoring-adapter
spec:
  containers:
  - name: app
    image: my-app
    ports:
    - containerPort: 8080
  - name: monitoring-adapter
    image: monitoring-adapter
    ports:
    - containerPort: 9091
```

In this example:
- The `app` container exposes metrics in a non-standard format
- The `monitoring-adapter` transforms those metrics to Prometheus format

## Key Considerations for Multi-Container Pods

1. **Container Lifecycle**: Containers in a pod start and terminate together. Plan for coordinated lifecycle management.

2. **Resource Allocation**: Each container specifies its own resource requests and limits.

3. **Inter-Container Communication**: Containers can communicate via:
   - Shared volumes
   - `localhost` network
   - Process signals (if supported by container)

4. **Readiness and Liveness Probes**: Each container can have its own probes that accurately reflect its state.

5. **Shared Fate**: If one container fails critically, the entire pod may be restarted.

## CKAD Exam Tips

1. **Know the Patterns**: Understand each pattern and their typical use cases.
2. **Volume Sharing**: Be able to configure shared volumes between containers.
3. **Container Ordering**: Know that Kubernetes doesn't guarantee startup order (use init containers if order matters).
4. **Debugging**: Know how to get logs from and exec into specific containers:
   ```bash
   kubectl logs <pod-name> -c <container-name>
   kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
   ```
5. **Resource Management**: Understand how to allocate resources properly to each container.

## Relevant Documentation

- [Kubernetes Documentation: Pod Design](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Kubernetes Blog: Multi-Container Pod Patterns](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)
- [Resource Sharing and Communication](https://kubernetes.io/docs/concepts/workloads/pods/#resource-sharing-and-communication)