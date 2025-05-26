# Network Policies in Kubernetes

Network Policies are specifications of how groups of pods are allowed to communicate with each other and with other network endpoints. They act as a firewall for Kubernetes pods, enabling you to define rules for ingress (incoming) and egress (outgoing) traffic.

## Key Concepts

- **Network Policy**: Kubernetes resource that specifies how pods are allowed to communicate
- **Ingress Rules**: Control incoming traffic to pods
- **Egress Rules**: Control outgoing traffic from pods
- **Selectors**: Determine which pods a policy applies to and which traffic is allowed
- **Default Deny**: By default, pods accept traffic from any source in the absence of network policies

## Prerequisites

Network policies are enforced by the network plugin, not Kubernetes itself. Your cluster must be using a network plugin that supports NetworkPolicy (e.g., Calico, Weave Net, Cilium, Antrea).

## Basic Network Policy Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-network-policy
  namespace: default
spec:
  podSelector:             # Selects the pods to which this policy applies
    matchLabels:
      role: db
  policyTypes:             # Types of policy rules to include
  - Ingress                # Include ingress rules
  - Egress                 # Include egress rules
  ingress:                 # Ingress rules (incoming traffic)
  - from:                  # Sources allowed to access
    - ipBlock:             # IP CIDR range
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:   # Select namespaces by label
        matchLabels:
          project: myproject
    - podSelector:         # Select pods by label
        matchLabels:
          role: frontend
    ports:                 # Ports that may be accessed
    - protocol: TCP
      port: 6379
  egress:                  # Egress rules (outgoing traffic)
  - to:                    # Destinations pods can access
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 5978
```

## Common Network Policy Examples

### 1. Default Deny All Ingress Traffic

Blocks all incoming traffic to pods in a namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}          # Applies to all pods
  policyTypes:
  - Ingress                # Only include ingress rules
```

### 2. Default Deny All Egress Traffic

Blocks all outgoing traffic from pods in a namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}          # Applies to all pods
  policyTypes:
  - Egress                 # Only include egress rules
```

### 3. Allow Traffic from a Specific Application

Allow incoming traffic only from pods with label `app=frontend`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow-frontend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

### 4. Allow Traffic Only to Specific External Services

Allow outgoing traffic only to specific IPs (e.g., database server):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-allow-external-db
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24
    ports:
    - protocol: TCP
      port: 5432
```

### 5. Allow Traffic Between Namespaces

Allow pods in namespace `frontend` to communicate with pods in namespace `backend`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
  namespace: backend
spec:
  podSelector: {}          # Applies to all pods in backend namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
```

### 6. Allow DNS Traffic

Allows DNS queries (necessary in many networks):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

## Complex Selectors in Network Policies

### Multiple Conditions with AND Logic

This example shows how to select pods that have both labels `role=api` AND `environment=production`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-prod-policy
spec:
  podSelector:
    matchLabels:
      role: api
      environment: production
  # ...policy rules...
```

### Multiple Sources with OR Logic

This creates OR logic between different sources (any match allows traffic):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-multiple-sources
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  - from:
    - podSelector:
        matchLabels:
          app: monitoring
```

### Complex Example with Multiple Rules

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complex-network-policy
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: authorized-client
      namespaceSelector:
        matchLabels:
          environment: production
    ports:
    - protocol: TCP
      port: 8443
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: allowed-destination
    ports:
    - protocol: TCP
      port: 443
  - to:
    - ipBlock:
        cidr: 10.0.0.0/16
    ports:
    - protocol: TCP
      port: 8080
```

## Testing Network Policies

### 1. Check if policy is being enforced

```bash
kubectl get networkpolicies
```

### 2. Test connectivity from a pod

Create a test pod and try to connect to the restricted resource:

```bash
kubectl run test-pod --rm -it --image=busybox -- /bin/sh
# Inside the pod:
wget -q --timeout=5 http://service-name --spider
```

### 3. Check logs for denied connections

If your CNI plugin supports it, check logs for denied connections.

## Troubleshooting Network Policies

1. **Policy has no effect**: Verify your network plugin supports Network Policies
2. **Too restrictive**: Ensure you've allowed necessary traffic (DNS, API server)
3. **Unexpected behavior**: Check for conflicting policies or selector mismatches
4. **Debugging steps**:
   - Look at pod labels
   - Check namespace labels
   - Verify policy selectors
   - Test with simpler policies first

## Best Practices

1. **Start with default deny**: Begin with a restrictive stance, then add allowances
2. **Use namespaces for isolation**: Group related applications and create policies per namespace
3. **Label pods consistently**: Use consistent labeling for easier policy management
4. **Test thoroughly**: Ensure policies work as expected before deploying to production
5. **Document policies**: Maintain documentation about network flows and corresponding policies
6. **Don't forget DNS**: Always allow DNS traffic in restrictive environments
7. **Consider egress restrictions**: Limit outbound traffic to reduce attack surface

## CKAD Exam Tips

1. **Know the syntax**: Be familiar with writing NetworkPolicy YAML
2. **Understand selectors**: Be comfortable with podSelector and namespaceSelector
3. **Practice policy creation**: Be able to quickly create policies for common scenarios
4. **Understand combination logic**: Know how multiple rules and selectors interact
5. **Test effectively**: Know how to verify that policies are working as expected

## Example CKAD Tasks

### Task 1: Restrict Database Access

**Task**: Create a NetworkPolicy that allows only pods with the label `role=api` to access pods with the label `role=db` on port 5432.

**Solution**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 5432
```

### Task 2: Create a Default Deny Policy with DNS Allowed

**Task**: Create a NetworkPolicy that denies all traffic to pods in the `restricted` namespace except for DNS queries.

**Solution**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-with-dns
  namespace: restricted
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

## Relevant Documentation

- [Kubernetes Documentation: Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes API Reference: NetworkPolicy v1](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#networkpolicy-v1-networking-k8s-io)
- [NetworkPolicy Examples](https://github.com/ahmetb/kubernetes-network-policy-recipes)