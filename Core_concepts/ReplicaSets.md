# To know the number of ReplicaSets
Run the command : 
``` bash
kubectl get replicaset
```
or 
``` bash
kubectl get rs`
```

# Definition of the ReplicaSet
A ReplicaSet is a Kubernetes resource that ensures a specified number of pod replicas are running at any given time. It helps maintain the desired state of the application by automatically creating or deleting pods as needed to match the specified number of replicas. ReplicaSets are often used to ensure high availability and fault tolerance for applications.

# Example Usage
To create a ReplicaSet, you can use a YAML manifest file and the `kubectl apply -f` command.

Create a YAML manifest file for a ReplicaSet (e.g., replicaset.yaml):

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image
```



Apply the manifest file to create the ReplicaSet:

```bash
kubectl apply -f replicaset.yaml
```

# Edit a ReplicaSet

Run the command: 
```bash
kubectl edit replicaset <replica-set-name>
```

Modify the image name and then save the file, then delete the previous pods in order to get new ones with the desired modifications.

# Scale a ReplicaSet

To scale a ReplicaSet, you can use the `kubectl scale` command. This command allows you to increase or decrease the number of replicas managed by the ReplicaSet.

## Example Usage

1. To scale a ReplicaSet named `my-replicaset` to 5 replicas:

```sh
kubectl scale --replicas=5 replicaset my-replicaset
```

# Delete ReplicaSet

Run the command: 
```bash
kubectl delete replicaset <replicaset-name>
```
or 
```bash
kubectl delete -f <file-name>.yaml
```

# N.B:
- In the YAML file definition of the ReplicaSet, the `apiVersion` always needs to be `apps/v1`.
- The [`matchLabels`]field in the [`selector`] specifies the labels that the ReplicaSet uses to identify the pods it should manage.
- The [`labels`] field in the pod template metadata specifies the labels assigned to the pods created by the ReplicaSet.
- The values of [`matchLabels`]and [`labels`] must match to ensure that the ReplicaSet can correctly identify and manage the pods.