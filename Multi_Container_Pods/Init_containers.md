# Init Containers in Kubernetes

Init containers are specialized containers that run before app containers in a pod. They are exactly like regular containers, except they always run to completion and each init container must complete successfully before the next one starts.

## Purpose of Init Containers

Init containers enable separation of concerns:

1. **Run setup operations** before application containers start
2. **Delay application startup** until preconditions are met
3. **Enhance security** by including utilities that would be a security risk in app containers
4. **Initialize shared volumes** with data that app containers will use

## How Init Containers Work

1. Init containers are executed in order, one at a time
2. Each init container must exit successfully (exit code 0) before the next begins
3. If any init container fails, Kubernetes restarts the pod according to its restart policy
4. Only when all init containers have succeeded will app containers be started

## Basic Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

In this example:
1. The pod has two init containers that check for service dependencies
2. Each init container uses the `nslookup` command to check if a service is available
3. The application container only starts after both services are detected

## Common Use Cases

### 1. Service Dependency Checks

Wait for dependent services to be ready before starting the main application:

```yaml
initContainers:
- name: wait-for-database
  image: busybox
  command: ['sh', '-c', 'until nc -z db-service 5432; do echo waiting for database; sleep 2; done;']
```

### 2. Volume Setup and Data Population

Prepare shared volumes before the application starts:

```yaml
initContainers:
- name: copy-config-data
  image: busybox
  command: ['sh', '-c', 'cp /source-config/* /config/']
  volumeMounts:
  - name: config-volume
    mountPath: /config
  - name: source-config
    mountPath: /source-config
```

### 3. Application Configuration

Set up application configurations:

```yaml
initContainers:
- name: generate-config
  image: config-generator
  command: ['generate', '--output=/config/app.conf']
  volumeMounts:
  - name: config-volume
    mountPath: /config
```

### 4. Delay Application Start

Wait for specific conditions to be met:

```yaml
initContainers:
- name: wait-for-migration-job
  image: kubectl
  command: ['sh', '-c', 'until kubectl get job migration-job -o jsonpath="{.status.succeeded}" | grep 1; do sleep 5; done']
```

## Important Init Container Properties

### Resource Handling

- **Resource limits**: Apply to init containers just like normal containers
- **QoS tier**: Determined from init and app containers together
- **Resource requests/limits**: The highest request/limit for each resource is used

```yaml
initContainers:
- name: init-container
  image: busybox
  resources:
    limits:
      cpu: 200m
      memory: 100Mi
    requests:
      cpu: 100m
      memory: 50Mi
```

### Volume Usage

Init containers can prepare volumes that will later be used by app containers:

```yaml
volumes:
- name: data-volume
  emptyDir: {}

initContainers:
- name: data-preparer
  image: data-preparer
  volumeMounts:
  - name: data-volume
    mountPath: /data

containers:
- name: app
  image: myapp
  volumeMounts:
  - name: data-volume
    mountPath: /data
```

### Failure and Restart Behavior

- If an init container fails, the pod restart policy applies
- For `restartPolicy: Always`, the failed init container restarts indefinitely
- For `restartPolicy: Never` or `OnFailure`, the pod becomes failed if an init container fails

## CKAD Exam Tips

1. **Know the Syntax**: Be able to define init containers in a pod specification
2. **Understand Order**: Remember that init containers run sequentially, not in parallel
3. **Troubleshoot**: Know how to check init container status:
   ```bash
   kubectl describe pod <pod-name>
   ```
4. **View Logs**: Know how to view init container logs:
   ```bash
   kubectl logs <pod-name> -c <init-container-name>
   ```
5. **Resource Management**: Understand how resources are calculated for init containers
6. **Volume Sharing**: Be comfortable configuring volumes shared between init and app containers

## Example CKAD Task

**Task**: Create a Pod with two init containers. The first init container should create a file called `ready.txt` in a shared volume. The second init container should wait for that file to exist before completing. The main application container should then read the content of this file.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'cat /workdir/ready.txt && echo "App is running" && sleep 3600']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  initContainers:
  - name: init-create-file
    image: busybox
    command: ['sh', '-c', 'echo "Init complete" > /workdir/ready.txt']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  - name: init-check-file
    image: busybox
    command: ['sh', '-c', 'until [ -f /workdir/ready.txt ]; do echo waiting; sleep 2; done;']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  volumes:
  - name: workdir
    emptyDir: {}
```

## Relevant Documentation

- [Kubernetes Documentation: Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Init Container Patterns](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/)