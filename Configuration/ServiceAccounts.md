# ServiceAccounts in Kubernetes

ServiceAccounts provide an identity for processes running in a Pod, allowing them to interact with the Kubernetes API server. They're essentially a way to authenticate Pods to the Kubernetes API.

## Key Concepts

- Every namespace has a default ServiceAccount created automatically
- A Pod is automatically assigned the default ServiceAccount in its namespace unless specified otherwise
- ServiceAccounts can be bound to roles with specific permissions using RBAC (Role-Based Access Control)
- ServiceAccounts are namespace-scoped resources

## When to Use ServiceAccounts

ServiceAccounts are useful when:

1. Your application needs to interact with the Kubernetes API
2. You want to limit what actions your application can perform in the cluster
3. You need to authenticate to external services that support Kubernetes authentication
4. You're implementing automation tools that interact with Kubernetes

## Basic Commands

### List ServiceAccounts in a namespace:

```bash
kubectl get serviceaccounts
# or
kubectl get sa
```

### Create a ServiceAccount:

```bash
kubectl create serviceaccount <name>
```

### Describe a ServiceAccount:

```bash
kubectl describe serviceaccount <name>
```

## Creating ServiceAccounts

### Imperative Approach:

```bash
kubectl create serviceaccount app-service-account
```

### Declarative Approach:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default
```

## Using ServiceAccounts in Pods

You can specify which ServiceAccount a Pod should use:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: app-service-account
  containers:
  - name: app
    image: nginx:1.19
```

## ServiceAccounts and Secrets

When you create a ServiceAccount, Kubernetes automatically:

1. Creates an API token stored as a Secret
2. Mounts this token into Pods using the ServiceAccount at `/var/run/secrets/kubernetes.io/serviceaccount`

For newer Kubernetes versions (1.24+), this behavior changes:
- Tokens are no longer automatically created
- Tokens are obtained through the TokenRequest API
- Mounted tokens are short-lived and automatically renewed

To manually create a long-lived token:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-sa-token
  annotations:
    kubernetes.io/service-account.name: app-service-account
type: kubernetes.io/service-account-token
```

## ServiceAccounts and RBAC

To make ServiceAccounts useful, you need to grant them permissions using RBAC:

### Creating a Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### Binding the Role to a ServiceAccount:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## ClusterRoles and ClusterRoleBindings

For cluster-wide permissions (not namespaced):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## Using ServiceAccount Tokens in Containers

Inside a Pod, the ServiceAccount token is mounted at:

```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

You can use this token to authenticate to the Kubernetes API:

```bash
# From inside a Pod:
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt

curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
```

## ServiceAccount Token Projection

In Kubernetes 1.20+, you can project tokens with specific audiences and expiration times:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: token-projection-pod
spec:
  containers:
  - name: app
    image: nginx:1.19
    volumeMounts:
    - name: token-vol
      mountPath: /var/run/secrets/tokens
  volumes:
  - name: token-vol
    projected:
      sources:
      - serviceAccountToken:
          path: token
          expirationSeconds: 3600
          audience: "my-service"
```

## Automount Token

You can disable automatic token mounting:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-auto-token
automountServiceAccountToken: false
```

Or disable it for specific Pods:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: app-service-account
  automountServiceAccountToken: false
  containers:
  - name: app
    image: nginx:1.19
```

## ServiceAccount with imagePullSecrets

You can add image pull secrets to a ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
imagePullSecrets:
- name: regcred
```

All Pods using this ServiceAccount will automatically have access to these pull secrets.

## Common Pitfalls

1. **Insufficient permissions**: ServiceAccount doesn't have necessary RBAC permissions
2. **Using default ServiceAccount**: Default ServiceAccount typically has limited permissions
3. **Missing namespace**: Creating RBAC resources in wrong namespace
4. **Not waiting for propagation**: Changes to ServiceAccounts/RBAC may take time to propagate
5. **Token renewal**: Not accounting for token expiration in older applications

## Security Best Practices

1. **Least Privilege**: Grant only the permissions a Pod needs
2. **Unique ServiceAccounts**: Create separate ServiceAccounts for different applications
3. **Disable Default Mounting**: Set `automountServiceAccountToken: false` when not needed
4. **Restrict ServiceAccount Creation**: Control who can create ServiceAccounts
5. **Audit ServiceAccount Usage**: Regularly review which Pods use which ServiceAccounts
6. **Use Short-lived Tokens**: Prefer TokenRequest API over long-lived tokens

## CKAD Exam Tips

1. **Know how to create ServiceAccounts**: Both imperatively and declaratively
2. **Understand how to assign a ServiceAccount to a Pod**
3. **Be familiar with RBAC**: Create Roles and RoleBindings for ServiceAccounts
4. **Practice troubleshooting**: Debug common ServiceAccount issues
5. **Know token mount paths**: Understand where and how tokens are mounted
6. **Remember automountServiceAccountToken**: Know how to disable token mounting

## Example CKAD Task

**Task**: Create a ServiceAccount named `monitoring-sa` in the default namespace. Create a Role that allows reading Services and Endpoints. Bind this Role to the ServiceAccount. Then create a Pod that uses this ServiceAccount and verify it can access the allowed resources.

**Solution**:

```yaml
# Create ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-sa
  namespace: default
---
# Create Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: service-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["services", "endpoints"]
  verbs: ["get", "list", "watch"]
---
# Create RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-service-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: default
roleRef:
  kind: Role
  name: service-reader
  apiGroup: rbac.authorization.k8s.io
---
# Create Pod using the ServiceAccount
apiVersion: v1
kind: Pod
metadata:
  name: monitoring-pod
spec:
  serviceAccountName: monitoring-sa
  containers:
  - name: monitoring
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

## Relevant Documentation

- [Kubernetes Documentation: Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes Documentation: Managing Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)
- [Kubernetes Documentation: RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes Blog: Bound Service Account Tokens](https://kubernetes.io/blog/2021/04/20/kubelet-credential-provider/)