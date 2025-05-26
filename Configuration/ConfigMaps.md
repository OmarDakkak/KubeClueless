# ConfigMaps in Kubernetes

ConfigMaps allow you to decouple configuration artifacts from container image content to keep containerized applications portable. They store non-confidential data in key-value pairs and can be consumed by pods in various ways.

## Creating ConfigMaps

### Imperative Methods

#### From Literal Values
```bash
kubectl create configmap <configmap-name> \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

Example:
```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false
```

#### From Files
```bash
kubectl create configmap <configmap-name> --from-file=<path-to-file>
```

Example:
```bash
kubectl create configmap game-config --from-file=./game.properties
```

#### From Multiple Files in a Directory
```bash
kubectl create configmap <configmap-name> --from-file=<directory-path>
```

Example:
```bash
kubectl create configmap game-configs --from-file=./configs
```

#### From Environment File
```bash
kubectl create configmap <configmap-name> --from-env-file=<path-to-env-file>
```

Example:
```bash
kubectl create configmap env-config --from-env-file=./config.env
```

### Declarative Method (YAML)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
  config.properties: |
    property1=value1
    property2=value2
```

## Using ConfigMaps in Pods

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
```

### Using envFrom to Load All Values

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

### As Files in a Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Mounting Specific Items from ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: game.properties
        path: game.properties
      - key: user-interface.properties
        path: ui.properties
```

## ConfigMap Restrictions and Limitations

1. ConfigMaps must be created before they are consumed in pods.
2. ConfigMaps reside in a specific namespace.
3. Pods can only reference ConfigMaps in the same namespace.
4. ConfigMap data is limited to 1MB in size.
5. Container using a ConfigMap as a subPath volume mount won't receive ConfigMap updates.

## Updating ConfigMaps

When you update a ConfigMap:

1. **Environment variables** from ConfigMaps won't update automatically - the pod needs to restart.
2. **Files projected using volumes** will eventually update automatically, but it may take some time.

## Best Practices

1. Use meaningful names for ConfigMaps that reflect their purpose.
2. Group related configurations in a single ConfigMap.
3. Consider namespace segregation for configuration by environment.
4. Keep sensitive data in Secrets, not ConfigMaps.
5. Use file projections for larger configuration files.

## CKAD Exam Tips

1. **Know the Commands**: Memorize the imperative commands for creating ConfigMaps from different sources.
2. **Understand Mounting Options**: Know how to use ConfigMaps as environment variables and as files.
3. **Troubleshooting**: Be prepared to troubleshoot issues with ConfigMaps, such as incorrect keys or missing references.
4. **YAML Structure**: Be comfortable writing YAML to create and use ConfigMaps.
5. **Practice**: Create pods that consume ConfigMaps in different ways.

## Example CKAD Task

**Task**: Create a ConfigMap named `game-config` with the following data:
- key `game.properties` containing value:
  ```
  enemies=aliens
  lives=3
  ```
- key `ui.properties` containing value:
  ```
  color.good=purple
  color.bad=yellow
  ```
Create a pod that uses this ConfigMap to mount these properties as files in `/etc/config`.

**Solution**:

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-config
data:
  game.properties: |
    enemies=aliens
    lives=3
  ui.properties: |
    color.good=purple
    color.bad=yellow
---
# Pod using the ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: game-pod
spec:
  containers:
  - name: game
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: game-config
```

## Relevant Documentation

- [Kubernetes Documentation: ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Tasks: Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)