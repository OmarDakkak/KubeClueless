# Kubernetes Debugging

Debugging applications in Kubernetes requires a systematic approach and knowledge of specialized tools. This guide covers the essential debugging techniques and commands you'll need for effective troubleshooting in Kubernetes environments.

## Debugging Process Overview

A structured debugging process for Kubernetes applications typically includes:

1. **Identify the issue**: Determine what's not working as expected
2. **Check pod status**: Examine the current state of affected pods
3. **View logs**: Check container logs for error messages
4. **Examine events**: Look for cluster events related to the issue
5. **Check configurations**: Verify that resource definitions are correct
6. **Inspect the environment**: Check networking, storage, and other dependencies
7. **Test directly**: Connect to containers to run diagnostic commands
8. **Apply a fix**: Make necessary changes and verify the solution

## Essential Debugging Commands

### Pod Status Investigation

```bash
# Get overall pod status
kubectl get pods

# Get detailed pod information
kubectl describe pod <pod-name>

# Check pod status with custom output format
kubectl get pod <pod-name> -o wide

# Watch pod status in real-time
kubectl get pods -w
```

### Checking Pod and Container Logs

```bash
# Get container logs
kubectl logs <pod-name>

# Get logs from a specific container in a multi-container pod
kubectl logs <pod-name> -c <container-name>

# Stream logs in real-time
kubectl logs <pod-name> -f

# Get logs from a previous container instance (if it crashed)
kubectl logs <pod-name> --previous
```

### Examining Events

```bash
# Get all events in the namespace
kubectl get events

# Get events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

# Get events for a specific resource
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Accessing Running Containers

```bash
# Get an interactive shell in a container
kubectl exec -it <pod-name> -- /bin/sh

# Run a specific command in a container
kubectl exec <pod-name> -- <command>

# Access a specific container in a multi-container pod
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

### Creating Debugging Pods

```bash
# Create a debugging pod that runs in the same namespace
kubectl run debug-pod --image=busybox --restart=Never -- sleep 3600

# Create a temporary interactive debugging pod
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
```

### Checking Resource Definitions

```bash
# Get YAML representation of a resource
kubectl get pod <pod-name> -o yaml

# Compare current state with original manifest
kubectl get pod <pod-name> -o yaml > current-pod.yaml
diff original-pod.yaml current-pod.yaml
```

## Common Issues and Debugging Approaches

### Pod Stuck in Pending State

**Possible causes:**
- Insufficient cluster resources
- PersistentVolumeClaim not bound
- Node affinity/anti-affinity rules preventing scheduling

**Debugging steps:**
```bash
kubectl describe pod <pod-name>  # Look for events with scheduling errors
kubectl get nodes                # Check node status and capacity
kubectl describe pvc <pvc-name>  # If using PVCs, check binding status
```

### Pod Stuck in CrashLoopBackOff

**Possible causes:**
- Application errors
- Incorrect container command or arguments
- Missing dependencies or configuration
- Resource constraints

**Debugging steps:**
```bash
kubectl logs <pod-name> --previous  # Check logs from failed container
kubectl describe pod <pod-name>     # Check for termination messages
```

### Pod Stuck in ImagePullBackOff

**Possible causes:**
- Invalid image name or tag
- Private repository without credentials
- Network connectivity issues

**Debugging steps:**
```bash
kubectl describe pod <pod-name>  # Check image name and pull status
kubectl get secret               # Check if image pull secrets exist
```

### Pod Running but Application Not Working

**Possible causes:**
- Application configuration issues
- Network connectivity problems
- Service or Ingress misconfiguration
- Resource limits too restrictive

**Debugging steps:**
```bash
kubectl logs <pod-name>                                   # Check for application errors
kubectl exec -it <pod-name> -- curl localhost:<port>      # Test local connectivity
kubectl get svc                                           # Check service configuration
kubectl describe ingress <ingress-name>                   # Check ingress configuration
kubectl exec -it <pod-name> -- cat /etc/resolv.conf       # Check DNS configuration
```

## Specialized Debugging Techniques

### Network Connectivity Debugging

**Check if a service is accessible within the cluster:**
```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -qO- http://service-name:port
```

**Check DNS resolution:**
```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name.namespace.svc.cluster.local
```

**Check network policies:**
```bash
kubectl get networkpolicy
kubectl describe networkpolicy <policy-name>
```

### Resource Usage Debugging

**Check resource usage at node level:**
```bash
kubectl top nodes
```

**Check resource usage at pod level:**
```bash
kubectl top pods
```

**Check container resource limits and requests:**
```bash
kubectl describe pod <pod-name> | grep -A 3 Limits
```

### Volume and ConfigMap/Secret Debugging

**Check if volumes are properly mounted:**
```bash
kubectl describe pod <pod-name> | grep -A 10 Volumes
```

**Verify ConfigMap or Secret content:**
```bash
kubectl get configmap <configmap-name> -o yaml
kubectl get secret <secret-name> -o yaml
```

**Access mounted ConfigMap or Secret inside container:**
```bash
kubectl exec -it <pod-name> -- cat /path/to/mounted/config
```

## Advanced Debugging Tools

### Port Forwarding

Access services directly without exposing them:

```bash
kubectl port-forward pod/<pod-name> 8080:80
# Now access through localhost:8080
```

### Proxy

Access Kubernetes API and services through local proxy:

```bash
kubectl proxy
# API now available at http://localhost:8001/api/
```

### Debugging with Ephemeral Containers (Kubernetes 1.23+)

Debug a running pod without modifying it:

```bash
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

### Using kubectl debug (Kubernetes 1.18+)

Create a copy of a pod with debugging tools:

```bash
kubectl debug <pod-name> -it --image=busybox --share-processes --copy-to=<pod-name>-debug
```

## Debugging Strategies for Multi-Container Pods

### Check all containers

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name> --all-containers
```

### Check specific container

```bash
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

### Verify communication between containers

```bash
kubectl exec -it <pod-name> -c <container1> -- curl localhost:<container2-port>
```

## JSON Path for Advanced Debugging

Extract specific information from resources:

```bash
# Get specific field from a pod
kubectl get pod <pod-name> -o jsonpath='{.status.phase}'

# Get container statuses
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state}'

# List all pod images in a namespace
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# Check environment variables in a container
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].env}'
```

## Debugging Init Containers

```bash
# Check init container status
kubectl describe pod <pod-name>

# Get logs from init container
kubectl logs <pod-name> -c <init-container-name>
```

## CKAD Exam Tips

1. **Know the essential commands**: `kubectl get`, `kubectl describe`, `kubectl logs`, `kubectl exec`
2. **Understand pod lifecycle**: Know the different pod phases and what they mean
3. **Quick debugging sequence**: Know the logical order of debugging steps
4. **Check events first**: Events often provide clear indicators of issues
5. **Be comfortable with container access**: Be able to quickly exec into containers
6. **Use jsonpath when needed**: Extract specific information efficiently
7. **Create debugging pods**: Know how to create temporary pods for testing
8. **Use port-forwarding**: Test services directly without modifying configs

## Example CKAD Tasks

### Task 1: Debug a Crashing Pod

**Task**: You have a pod named `app-pod` that's continuously crashing. Find out why and fix the issue.

**Approach**:

```bash
# Check pod status to confirm it's crashing
kubectl get pod app-pod

# Check pod events and status for clues
kubectl describe pod app-pod

# Check logs from the failing container
kubectl logs app-pod --previous

# If needed, check the container configuration
kubectl get pod app-pod -o yaml
```

### Task 2: Diagnose Network Connectivity

**Task**: One service can't communicate with another service named `backend-service`. Debug the issue.

**Approach**:

```bash
# Verify service existence and endpoints
kubectl get service backend-service
kubectl get endpoints backend-service

# Check if DNS resolution works properly
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup backend-service.default.svc.cluster.local

# Try to connect to the service from a test pod
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -qO- http://backend-service:80

# Check for network policies that might be blocking traffic
kubectl get networkpolicy
```

### Task 3: Debug Resource Constraints

**Task**: A pod named `resource-pod` is getting evicted or failing to schedule. Determine if it's related to resource constraints.

**Approach**:

```bash
# Check pod status and events
kubectl describe pod resource-pod

# Check node resource usage
kubectl top nodes

# Check pod resource requests and limits
kubectl get pod resource-pod -o yaml | grep -A 10 resources:

# Check if other pods with similar resource requirements are running
kubectl get pods -o wide
```

## Relevant Documentation

- [Kubernetes Documentation: Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Kubernetes Documentation: Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
- [kubectl Commands Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- [Kubernetes JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)