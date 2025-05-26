# Secrets in Kubernetes

Secrets are similar to ConfigMaps but are specifically intended for confidential data such as passwords, OAuth tokens, and SSH keys. Using Secrets helps prevent exposing sensitive configuration in your application stack.

## Why Use Secrets?

- Keep sensitive information separate from application code
- Limit access to sensitive data
- Reduce risk of accidental exposure in logs, screen dumps, etc.
- Apply different security settings to secret data

## Types of Secrets

Kubernetes provides several built-in types of secrets:

- `Opaque`: Default, arbitrary user-defined data
- `kubernetes.io/service-account-token`: Service account token
- `kubernetes.io/dockercfg`: Serialized `~/.dockercfg` file
- `kubernetes.io/dockerconfigjson`: Serialized `~/.docker/config.json` file
- `kubernetes.io/basic-auth`: Credentials for basic authentication
- `kubernetes.io/ssh-auth`: Data for SSH authentication
- `kubernetes.io/tls`: Data for a TLS client or server
- `bootstrap.kubernetes.io/token`: Bootstrap token data

## Creating Secrets

### Imperative Methods

#### From Literal Values

```bash
kubectl create secret generic <secret-name> \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

Example:
```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t
```

#### From Files

```bash
kubectl create secret generic <secret-name> \
  --from-file=key1=/path/to/file1 \
  --from-file=key2=/path/to/file2
```

Example:
```bash
kubectl create secret generic ssl-certs \
  --from-file=cert=/path/to/cert.crt \
  --from-file=key=/path/to/key.key
```

#### Creating TLS Secret

```bash
kubectl create secret tls <secret-name> \
  --cert=path/to/tls.cert \
  --key=path/to/tls.key
```

### Declarative Method (YAML)

For declarative creation, the secret data must be base64 encoded:

```bash
echo -n "admin" | base64
YWRtaW4=
echo -n "s3cr3t" | base64
czNjcjN0
```

Then create a YAML file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=
  password: czNjcjN0
```

Alternatively, you can use the `stringData` field which accepts unencoded strings:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: admin
  password: s3cr3t
```

## Using Secrets in Pods

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: database
    image: mysql
    env:
    - name: MYSQL_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: MYSQL_PASSWORD
      valueFrom:
        secretKeyRef:
          key: password
          name: db-credentials
```

### Using envFrom to Load All Values

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: database
    image: mysql
    envFrom:
    - secretRef:
        name: db-credentials
```

### As Files in a Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: database
    image: mysql
    volumeMounts:
    - name: secrets-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secrets-volume
    secret:
      secretName: db-credentials
```

## Secret Restrictions and Best Practices

1. Secrets are namespaced and can only be referenced by pods in the same namespace
2. Secret data is stored in etcd in unencrypted format by default (enable encryption at rest for better security)
3. Limit access with RBAC to protect who can read/write secret objects
4. Consider external secret management systems for production environments (HashiCorp Vault, AWS Secrets Manager, etc.)
5. Secrets have a 1MB size limit

## Security Considerations

1. **Pods can access all secrets in their namespace** by default. Use RBAC to restrict access.
2. **Secret data appears in kubelet's logs and memory**. Be careful when debugging.
3. **Secret data is passed to containers as environment variables or files**. Protect access to containers.
4. **Secrets are stored in etcd**. Secure your etcd properly.
5. **Secret data can be seen by anyone with API access**. Control API access carefully.

## CKAD Exam Tips

1. **Command Mastery**: Remember the imperative commands for creating different types of secrets.
2. **Know the Difference**: Understand when to use ConfigMaps vs Secrets.
3. **Pod Configuration**: Practice creating pods that consume secrets in different ways.
4. **Encoding/Decoding**: Know how to encode and decode base64 values.
5. **Troubleshooting**: Be prepared to debug applications that use secrets incorrectly.

## Example CKAD Task

**Task**: Create a secret named `database-creds` with username `admin` and password `password123`. Then create a pod that uses this secret as environment variables for a database container.

**Solution**:

```yaml
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: database-creds
type: Opaque
stringData:
  username: admin
  password: password123
---
# Pod using the Secret
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: db
    image: postgres
    env:
    - name: POSTGRES_USER
      valueFrom:
        secretKeyRef:
          name: database-creds
          key: username
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: database-creds
          key: password
```

## Relevant Documentation

- [Kubernetes Documentation: Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes Tasks: Distributing Credentials Securely](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)
- [Best Practices for Secret Management](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)