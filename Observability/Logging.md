# Kubernetes Logging

Logging is a critical component of Kubernetes observability that allows you to collect, store, and analyze container output. Understanding how logging works in Kubernetes is essential for effective application monitoring, troubleshooting, and compliance.

## Kubernetes Logging Basics

In Kubernetes, logs are streams of information written by containers to `stdout` and `stderr`. By default, these streams are captured by the container runtime and made accessible via the Kubernetes API.

### Native Kubernetes Logging

The basic logging architecture in Kubernetes includes:
1. **Container logs**: Applications writing to stdout/stderr
2. **Node-level logging**: Container runtime captures logs
3. **Kubernetes API access**: Use `kubectl logs` to retrieve logs
4. **Log rotation**: Handled by the container runtime or kubelet

## Accessing Container Logs

### Using kubectl logs

The primary way to access container logs is through the `kubectl logs` command:

```bash
# Basic log retrieval
kubectl logs <pod-name>

# Get logs for a specific container in a multi-container pod
kubectl logs <pod-name> -c <container-name>

# Follow logs in real-time (similar to tail -f)
kubectl logs <pod-name> -f

# Show logs since a specific time
kubectl logs <pod-name> --since=1h

# Show only the most recent 100 lines
kubectl logs <pod-name> --tail=100

# Show logs from all containers in a pod
kubectl logs <pod-name> --all-containers=true
```

### Logs for Previous Container Instances

If a container has restarted, you can access logs from previous container instances:

```bash
kubectl logs <pod-name> --previous
```

## Multi-Container Pod Logging

When pods contain multiple containers, you need to specify which container's logs you want to view:

```bash
kubectl logs <pod-name> -c <container-name>
```

### Sidecar Logging Pattern

A common pattern for centralized logging is the sidecar pattern:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging-sidecar
spec:
  containers:
  - name: app
    image: my-app
    volumeMounts:
    - name: log-storage
      mountPath: /var/log/app
  - name: log-collector
    image: logging-agent
    volumeMounts:
    - name: log-storage
      mountPath: /var/log/app
  volumes:
  - name: log-storage
    emptyDir: {}
```

In this pattern:
1. The main application writes logs to a file in a shared volume
2. The sidecar container reads from that volume and forwards logs to a central logging system

## Log Aggregation

For production environments, you'll typically want to aggregate logs from across your cluster:

### Common Log Aggregation Stacks:

1. **EFK Stack**:
   - Elasticsearch: Storage and search
   - Fluentd/Fluent Bit: Collection and forwarding
   - Kibana: Visualization and analysis

2. **PLG Stack**:
   - Promtail: Collection
   - Loki: Storage
   - Grafana: Visualization

3. **ELK Stack**:
   - Elasticsearch: Storage and search
   - Logstash: Collection and processing
   - Kibana: Visualization and analysis

### Basic Fluentd Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>
    
    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch
      port 9200
      logstash_format true
      flush_interval 5s
    </match>
```

## Structured Logging

Structured logging improves the searchability and analytics capabilities of your logs by formatting them as structured data (usually JSON):

```json
{
  "timestamp": "2023-05-15T08:12:34.567Z",
  "level": "INFO",
  "message": "Request processed successfully",
  "method": "GET",
  "path": "/api/users",
  "statusCode": 200,
  "responseTime": 45,
  "userId": "user-123"
}
```

### Benefits of Structured Logging:

1. Easier filtering and searching
2. Better analytics capabilities
3. Consistent format across services
4. Machine-readable for automated processing
5. Contextual data alongside log messages

## Log Levels

Well-designed logging should use appropriate log levels:

- **ERROR**: Failed operations or unexpected conditions that require attention
- **WARN**: Situations that might cause problems but don't stop normal operation
- **INFO**: Normal operational messages, useful for tracking application flow
- **DEBUG**: Detailed diagnostic information for troubleshooting
- **TRACE**: Most granular information, typically only used during development

## Application Logging Best Practices

1. **Write to stdout/stderr**: Don't write logs to files inside containers
2. **Use structured logging**: Format logs as JSON where possible
3. **Include request IDs**: Add correlation IDs to track requests across services
4. **Add context**: Include relevant information in each log entry
5. **Use appropriate log levels**: Don't log everything at ERROR level
6. **Handle sensitive data**: Don't log passwords, tokens, or personal information
7. **Consider volume**: Balance detail with the amount of log data generated
8. **Include timestamps**: Even though Kubernetes adds timestamps, including them in your application logs can be useful

## Kubernetes Event Logging

Besides container logs, Kubernetes also generates events that provide insights into cluster operations:

```bash
kubectl get events
kubectl get events --field-selector involvedObject.name=<pod-name>
```

Events are particularly useful for troubleshooting scheduling, scaling, and other cluster-level issues.

## Storage Considerations

Log retention and storage should be planned:

1. **Rotation**: Configure log rotation to prevent disk space issues
2. **Retention**: Determine how long logs should be kept
3. **Storage class**: Consider using appropriate storage for logs
4. **Compression**: Compress older logs to save space

## Troubleshooting with Logs

### Common Troubleshooting Patterns:

1. **Check recent logs first**:
   ```bash
   kubectl logs <pod-name> --tail=50
   ```

2. **Look for errors**:
   ```bash
   kubectl logs <pod-name> | grep -i error
   ```

3. **Examine logs around a specific time**:
   ```bash
   kubectl logs <pod-name> --since=5m
   ```

4. **Compare with previous container instances**:
   ```bash
   kubectl logs <pod-name> --previous
   ```

5. **Check events and logs together**:
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

## CKAD Exam Tips

1. **Master `kubectl logs` command**: Know the key options like `-f`, `-c`, `--tail`, and `--previous`
2. **Understand multi-container logging**: Know how to access logs from specific containers in a pod
3. **Know the sidecar logging pattern**: Understand how to configure a logging sidecar container
4. **Practice troubleshooting**: Be able to quickly find relevant information in logs
5. **Remember the basics of structured logging**: Know why it's better than unstructured logs
6. **Understand Kubernetes events**: Know how to check events related to specific resources

## Example CKAD Tasks

### Task 1: Access Container Logs

**Task**: You have a pod named `web-app` in the `default` namespace. View the last 20 lines of its logs.

**Solution**:

```bash
kubectl logs web-app --tail=20
```

### Task 2: Follow Logs from a Specific Container

**Task**: You have a pod named `multi-container-pod` with two containers: `app` and `sidecar`. Stream the logs from the `app` container.

**Solution**:

```bash
kubectl logs multi-container-pod -c app -f
```

### Task 3: Create a Pod with Logging to a File and a Sidecar

**Task**: Create a pod with two containers: an application container that writes logs to a file at `/var/log/app/app.log`, and a sidecar container that reads those logs and outputs them to its stdout.

**Solution**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging-sidecar
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - while true; do
        echo "$(date): Application log entry" >> /var/log/app/app.log;
        sleep 5;
      done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app
  - name: log-sidecar
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - tail -f /var/log/app/app.log
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app
  volumes:
  - name: log-volume
    emptyDir: {}
```

## Relevant Documentation

- [Kubernetes Documentation: Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [kubectl logs Command Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
- [Kubernetes Events](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#events)
- [EFK Stack on Kubernetes](https://kubernetes.io/docs/tasks/debug/debug-cluster/logging-elasticsearch-kibana/)