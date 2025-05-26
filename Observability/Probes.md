# Kubernetes Probes

Kubernetes probes are diagnostic tools used by the kubelet to determine the health and readiness of a container. They are critical for ensuring application reliability, proper service availability, and efficient resource utilization in a Kubernetes cluster.

## Types of Probes

Kubernetes offers three types of probes:

### 1. Liveness Probe

The liveness probe determines if a container is running properly. If the liveness probe fails, Kubernetes will restart the container based on the pod's `restartPolicy`.

**Use cases:**
- Detect application deadlocks or frozen states
- Restart crashed or hung applications
- Recover from temporary errors that require a restart

### 2. Readiness Probe

The readiness probe determines if a container is ready to serve requests. If the readiness probe fails, Kubernetes will remove the pod's IP address from the service endpoints, preventing traffic from being sent to it.

**Use cases:**
- Prevent traffic to pods that are initializing
- Handle temporary unavailability without restarting
- Graceful application startup and shutdown
- Database connection initialization

### 3. Startup Probe

The startup probe determines if an application within the container has started. If provided, it disables liveness and readiness checks until it succeeds, allowing for slow-starting containers to avoid being killed by other probes during startup.

**Use cases:**
- Applications with long initialization times
- Legacy applications with unpredictable startup patterns
- Preventing premature liveness probe execution

## Probe Mechanisms

Kubernetes supports three mechanisms for implementing probes:

### 1. HTTP GET Request

Performs an HTTP GET request to the container's IP address on a specified port and path. Status codes between 200 and 399 indicate success.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 10
  periodSeconds: 10
```

### 2. TCP Socket Check

Attempts to establish a TCP connection to the specified port. If the connection can be established, the probe succeeds.

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

### 3. Exec Command

Executes a command inside the container. If the command exits with status code 0, the probe succeeds.

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Probe Configuration Parameters

All probes support the following common parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `initialDelaySeconds` | Time to wait after container starts before initiating probe | 0 |
| `periodSeconds` | How often to perform the probe | 10 |
| `timeoutSeconds` | Time after which the probe times out | 1 |
| `successThreshold` | Minimum consecutive successes for the probe to be considered successful | 1 |
| `failureThreshold` | Number of consecutive failures before declaring probe failure | 3 |

## Best Practices

### Liveness Probe Best Practices

1. **Keep probe logic light**: Avoid resource-intensive checks that could impact application performance
2. **Set appropriate timeouts**: Ensure timeouts are longer than the expected execution time
3. **Focus on critical functionality**: Only check essential components, not external dependencies
4. **Include appropriate delays**: Use `initialDelaySeconds` to prevent premature checks
5. **Be cautious with restarts**: Remember that failed liveness probes cause container restarts

### Readiness Probe Best Practices

1. **Check all critical dependencies**: Include checks for databases, caches, and other services
2. **Implement graceful degradation**: Design applications to properly report readiness even in partial failure
3. **Set appropriate thresholds**: Use `failureThreshold` to prevent flapping (rapid addition/removal from service)
4. **Include startup checks**: Ensure the application is fully initialized before accepting traffic
5. **Balance specificity with reliability**: Tests should be comprehensive but not so strict that they cause unnecessary removal from service

### General Probe Best Practices

1. **Always define probes** for production workloads
2. **Use different endpoints** for liveness and readiness
3. **Make health check endpoints lightweight** and non-authenticated
4. **Consider the impact of restarts** on application state
5. **Log probe failures** to aid in troubleshooting
6. **Test probes thoroughly** in staging environments
7. **Adjust timing parameters** based on application behavior

## Common Anti-Patterns

1. **Using readiness probes for liveness**: Checking external dependencies in liveness probes
2. **Identical liveness and readiness probes**: Both probes should serve different purposes
3. **Resource-intensive probes**: Causing CPU or memory pressure through heavy probe operations
4. **Insufficient delays**: Not giving applications enough time to initialize before probing
5. **Health checks that mask failures**: Probes that don't accurately reflect application health
6. **Over-reliance on liveness restarts**: Using restarts as a primary recovery mechanism

## Examples for Common Applications

### Web Application

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-app
    image: nginx
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
```

### Database Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: db
    image: postgres
    livenessProbe:
      exec:
        command:
        - pg_isready
        - "-U"
        - postgres
      initialDelaySeconds: 30
      periodSeconds: 20
    readinessProbe:
      exec:
        command:
        - pg_isready
        - "-U"
        - postgres
      initialDelaySeconds: 10
      periodSeconds: 10
```

### Java Application with Startup Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: java-app
spec:
  containers:
  - name: java-app
    image: myapp:1.0
    startupProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /actuator/health/liveness
        port: 8080
      periodSeconds: 15
    readinessProbe:
      httpGet:
        path: /actuator/health/readiness
        port: 8080
      periodSeconds: 5
```

## Troubleshooting Probes

### Common Issues and Solutions

1. **Probe failing but application seems healthy**
   - Check endpoint path, port, and headers
   - Verify timeout settings
   - Ensure probe execution environment matches expectations

2. **Container restarts loop due to liveness failures**
   - Increase `initialDelaySeconds`
   - Examine container logs for startup issues
   - Check if probe targets are available internally

3. **Pod never becomes ready**
   - Confirm all dependencies are available
   - Review readiness check implementation
   - Check for permission issues or network policies blocking probes

### Debugging Commands

```bash
# Check pod status and events
kubectl describe pod <pod-name>

# Check logs for signs of probe failures
kubectl logs <pod-name>

# Test HTTP probe manually via port-forward
kubectl port-forward <pod-name> 8080:8080
curl http://localhost:8080/healthz

# Get detailed probe information from kubelet logs
kubectl logs -n kube-system <kubelet-pod-name>
```

## CKAD Exam Tips

1. **Know the differences** between liveness and readiness probes
2. **Memorize the basic syntax** for all probe types (HTTP, TCP, Exec)
3. **Understand common timing parameters** and their impact
4. **Practice creating pods** with different types of probes
5. **Be prepared to troubleshoot** failing probes in the exam
6. **Remember the purpose of startup probes** for slow-starting applications
7. **Know how to implement** quick HTTP health endpoints

## Example CKAD Tasks

### Task 1: Implement a Liveness Probe

**Task**: Create a pod with an nginx container and add a liveness probe that checks the path `/healthz` on port 80 every 10 seconds.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-liveness
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      periodSeconds: 10
      initialDelaySeconds: 15
```

### Task 2: Add Both Liveness and Readiness Probes

**Task**: Create a pod with a container using the image `busybox` that runs a simple HTTP server. Add both liveness and readiness probes.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-server
spec:
  containers:
  - name: server
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'HTTP/1.1 200 OK\n\nHello World!' | nc -l -p 8080; done"]
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 1
      periodSeconds: 5
```

### Task 3: Implement Exec Probe

**Task**: Create a pod that uses an exec liveness probe to verify a file exists in the container.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-liveness
spec:
  containers:
  - name: liveness
    image: busybox
    command: ["/bin/sh", "-c", "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"]
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Relevant Documentation

- [Kubernetes Documentation: Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes Blog: Liveness and Readiness Probes](https://kubernetes.io/blog/2020/01/22/kubeinvaders-gamified-chaos-engineering-tool-for-kubernetes/)
- [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)