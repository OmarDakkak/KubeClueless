# SecurityContext in Kubernetes

A SecurityContext defines privilege and access control settings for Pods or Containers. These settings include:

- User and group IDs to run the container processes
- Linux capabilities to grant or remove
- SELinux context
- Running containers as privileged or unprivileged
- And other security settings

## Why Use SecurityContext?

Security context constraints help:
- Prevent privilege escalation
- Apply least privilege principle
- Enforce stronger isolation between containers and host
- Implement security best practices across your cluster

## Setting Security Contexts

Security contexts can be defined at two levels:

1. **Pod level** - applies to all containers in the pod
2. **Container level** - applies only to specific containers (overrides pod-level settings)

## SecurityContext Properties

### User and Group Settings

- `runAsUser`: User ID (UID) to run container processes
- `runAsGroup`: Group ID (GID) for container processes
- `runAsNonRoot`: Boolean that prevents containers from running as root
- `fsGroup`: Supplemental group applied to mounted volumes

### Linux Capabilities

- `capabilities.add`: List of capabilities to add
- `capabilities.drop`: List of capabilities to remove

### Privilege Settings

- `privileged`: Boolean that enables privileged mode (similar to root on host)
- `allowPrivilegeEscalation`: Boolean that controls if processes can gain more privileges than parent process

### File System Settings

- `readOnlyRootFilesystem`: Boolean that makes the root filesystem read-only

## Pod-Level Security Context Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data/demo
  volumes:
  - name: data-volume
    emptyDir: {}
```

In this example:
- All container processes will run with user ID 1000 and group ID 3000
- Any volume mounts will have GID 2000

## Container-Level Security Context Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
        drop: ["ALL"]
```

In this example:
- Pod-level user ID is set to 1000 (applies to all containers unless overridden)
- Container-specific user ID is 2000 (overrides pod-level)
- Container prohibits privilege escalation
- Container drops all default capabilities and adds just NET_ADMIN and SYS_TIME

## Non-Root User Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - name: non-root-container
    image: nginx:1.19
    securityContext:
      runAsUser: 1000
```

In this example:
- The pod requires all containers to run as non-root
- The container explicitly sets user ID 1000

## Read-Only Root Filesystem

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: readonly-container
    image: nginx:1.19
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: temp-volume
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run
    - name: var-cache
      mountPath: /var/cache/nginx
  volumes:
  - name: temp-volume
    emptyDir: {}
  - name: var-run
    emptyDir: {}
  - name: var-cache
    emptyDir: {}
```

In this example:
- Container has a read-only root filesystem
- Specific writable directories are provided via volume mounts for temporary files and runtime needs

## Linux Capabilities

Linux capabilities break down the privileges available to the root user into smaller, more specific privileges. Some common capabilities:

- `CAP_NET_ADMIN`: Configure network interfaces, firewalls
- `CAP_SYS_TIME`: Modify system clock
- `CAP_CHOWN`: Make arbitrary changes to file UIDs and GIDs
- `CAP_DAC_OVERRIDE`: Bypass file read, write, and execute permission checks

Example of setting capabilities:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capabilities-demo
spec:
  containers:
  - name: capability-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
        drop: ["ALL"]
```

## Combining Security Settings

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: comprehensive-security-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true
  containers:
  - name: secure-container
    image: nginx:1.19
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    volumeMounts:
    - name: var-run
      mountPath: /var/run
    - name: var-cache
      mountPath: /var/cache/nginx
  volumes:
  - name: var-run
    emptyDir: {}
  - name: var-cache
    emptyDir: {}
```

## Security Context vs PodSecurityPolicy

- **SecurityContext**: Applied at the Pod or Container level
- **PodSecurityPolicy (Deprecated)**: Cluster-wide policy for Pod creation and updates
- **Pod Security Admission (Replacement for PSP)**: Enforces Pod Security Standards at the namespace level

## Common Pitfalls

1. **Container images with hardcoded user**: Some images specify users in their Dockerfile, potentially conflicting with SecurityContext
2. **Permission issues**: Setting incorrect user IDs can prevent applications from accessing required files
3. **Volume permission issues**: Applications may need specific permissions on volume mounts
4. **Capability requirements**: Some applications need specific Linux capabilities to function
5. **Overly restrictive contexts**: Can prevent applications from working correctly

## CKAD Exam Tips

1. **Know the common securityContext fields**: Especially runAsUser, runAsNonRoot, and capabilities
2. **Understand which settings apply at pod vs container level**
3. **Practice troubleshooting permission issues** related to SecurityContext
4. **Be familiar with capability management** - adding and dropping Linux capabilities
5. **Know how to make containers more secure** by applying least privilege principles

## Example CKAD Task

**Task**: Create a pod with the following security requirements:
- Pod name: `secure-pod`
- Container name: `secure-container`
- Image: `nginx:1.19`
- All processes should run as user ID 1000
- The container should not be able to gain additional privileges
- The root filesystem should be read-only
- Mount an emptyDir volume at `/var/cache/nginx` to allow nginx to write cache data

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: secure-container
    image: nginx:1.19
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: cache-volume
      mountPath: /var/cache/nginx
  volumes:
  - name: cache-volume
    emptyDir: {}
```

## Relevant Documentation

- [Kubernetes Documentation: Pod Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Kubernetes Documentation: Configure a Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Linux Capabilities Reference](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)