# Commands and Arguments in Kubernetes

Commands and arguments in Kubernetes allow you to override the default command and arguments defined in a container image's Dockerfile. Understanding how to set and use them is crucial for container configuration.

## Key Concepts

- **Command (Entrypoint)**: Defines the executable that runs when a container starts
- **Arguments (Cmd)**: Provides arguments to the command
- These override the default `ENTRYPOINT` and `CMD` instructions in a container's Dockerfile
- Once set at Pod creation, commands and arguments cannot be changed

## Docker vs. Kubernetes

Understanding the relationship between Docker and Kubernetes configurations is essential:

| Docker         | Kubernetes                       | Description                               |
|---------------|----------------------------------|-------------------------------------------|
| `ENTRYPOINT`  | `command` in container spec      | The executable to run                      |
| `CMD`         | `args` in container spec         | Arguments passed to the executable        |

## Basic Examples

### Command in Kubernetes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
```

### Equivalent Docker Command

```dockerfile
FROM debian
ENTRYPOINT ["printenv"]
CMD ["HOSTNAME", "KUBERNETES_PORT"]
```

## Setting Commands and Arguments

### Using Array Syntax (recommended)

```yaml
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["/bin/sh", "-c"]
    args: ["echo hello; sleep 3600"]
```

### Using String Syntax (not recommended)

```yaml
spec:
  containers:
  - name: app
    image: ubuntu
    command: /bin/sh -c
    args: echo hello; sleep 3600
```

## Environment Variables in Commands and Arguments

You can use environment variables in commands and arguments:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-cmd-demo
spec:
  containers:
  - name: env-cmd-demo-container
    image: debian
    command: ["printenv"]
    args: ["$(HOSTNAME)", "$(KUBERNETES_PORT)"]
    env:
    - name: HOSTNAME
      value: "example-host"
    - name: KUBERNETES_PORT
      value: "tcp://10.0.0.1:443"
```

## Using ConfigMap Values in Commands

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  param1: "value1"
  command-script: |
    #!/bin/sh
    echo "The first parameter is: $1"
    echo "The second parameter is: $2"
    echo "The command name is: $0"
---
apiVersion: v1
kind: Pod
metadata:
  name: configmap-cmd-demo
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["/bin/sh", "-c", "source /etc/config/command-script value1 value2"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      defaultMode: 0755
```

## Shell vs. Exec Form

### Shell Form (not recommended in Kubernetes)
In a Dockerfile:
```dockerfile
ENTRYPOINT echo "Hello, $NAME"
```

### Exec Form (recommended in Kubernetes)
In a Dockerfile:
```dockerfile
ENTRYPOINT ["/bin/echo", "Hello, $NAME"]
```

In Kubernetes:
```yaml
command: ["/bin/echo", "Hello, $(NAME)"]
```

## Running Interactive Shells

Sometimes you need an interactive shell:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shell-demo
spec:
  containers:
  - name: shell-demo-container
    image: debian
    command: ["/bin/bash"]
    args: ["-c", "while true; do sleep 10; done"]
```

## Common Use Cases

### 1. Running a Specific Command

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo 'Hello World' > /data/hello.txt; sleep 3600"]
```

### 2. Overriding Default Entry Points

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-custom
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["/usr/sbin/nginx"]
    args: ["-g", "daemon off;", "-q"]
```

### 3. Running Health Checks

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-cmd
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"]
    livenessProbe:
      exec:
        command: ["cat", "/tmp/healthy"]
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Command vs. Container Lifecycle Hooks

Don't confuse command/args with container lifecycle hooks:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from preStop handler > /usr/share/message"]
```

## Best Practices

1. **Use array syntax**: Always use the array syntax `[]` for both command and args
2. **Keep commands idempotent**: Commands should be safe to run multiple times
3. **Handle signals properly**: Ensure your application handles SIGTERM for graceful shutdown
4. **Set resource limits**: Always set resource limits for containers with commands
5. **Use environment variables**: Parameterize commands with environment variables
6. **Log to stdout/stderr**: Don't log to files, log to standard output/error
7. **Avoid interactive commands**: Avoid commands that require user input

## Common Pitfalls

1. **Shell interpretation**: Be aware of shell interpretation differences
2. **Not handling signals**: Failing to handle SIGTERM can lead to abrupt terminations
3. **Hard-coding values**: Embedding configuration in commands instead of using ConfigMaps
4. **Missing executable permissions**: Ensure scripts are executable
5. **Not considering init containers**: Some setup work belongs in init containers

## CKAD Exam Tips

1. **Know the syntax**: Be comfortable with both command and args syntax
2. **Understand parameter substitution**: Know how to use environment variables in commands
3. **Master imperative commands**: Know how to use kubectl run with custom commands
4. **Practice common scenarios**: Create Pods with custom commands and arguments
5. **Understand Dockerfile relation**: Know the relationship between Dockerfile instructions and Kubernetes settings

## Imperative Command Examples

Create a Pod with a specific command imperatively:

```bash
kubectl run nginx --image=nginx -- /bin/sh -c "echo hello; sleep 3600"
```

## Example CKAD Tasks

### Task 1: Override Default Command

**Task**: Create a Pod named `custom-command-pod` using the `busybox` image that runs a command to print "Hello, CKAD!" every 5 seconds.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-command-pod
spec:
  containers:
  - name: custom-command-container
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["while true; do echo 'Hello, CKAD!'; sleep 5; done"]
```

### Task 2: Use Environment Variables in Commands

**Task**: Create a Pod that uses an environment variable in its command.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-command-pod
spec:
  containers:
  - name: env-command-container
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["echo Hello $(WHO)!; sleep 3600"]
    env:
    - name: WHO
      value: "CKAD Candidate"
```

### Task 3: Run a Custom Script from ConfigMap

**Task**: Create a ConfigMap with a shell script and a Pod that executes this script.

**Solution**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: script-configmap
data:
  script.sh: |
    #!/bin/sh
    echo "ConfigMap script is running!"
    echo "Arguments provided: $@"
    sleep 3600
---
apiVersion: v1
kind: Pod
metadata:
  name: script-pod
spec:
  containers:
  - name: script-container
    image: busybox
    command: ["/scripts/script.sh"]
    args: ["arg1", "arg2"]
    volumeMounts:
    - name: script-volume
      mountPath: /scripts
  volumes:
  - name: script-volume
    configMap:
      name: script-configmap
      defaultMode: 0755
```

## Relevant Documentation

- [Kubernetes Documentation: Define a Command and Arguments for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
- [Kubernetes Documentation: Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)
- [Docker Documentation: Dockerfile reference - ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)
- [Docker Documentation: Dockerfile reference - CMD](https://docs.docker.com/engine/reference/builder/#cmd)