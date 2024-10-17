## To know how many pods exist in the current namespace
```bash
kubectl get pods
```

## To create a new pod with the nginx image
```bash
kubectl run nginx-pod --image=nginx
```

## To create a manifest file using kubectl run with --dry-run=client -o yaml option
```bash
kubectl run pod-name --image=image --dry-run=client -o yaml > pod.yaml
```

## To create a resource from the manifest file
```bash
kubectl create -f pod.yaml
```

## To know the image used to create a pod & to know how many containers are part of a pod
```bash
kubectl describe pod pod-name
```

The command `kubectl describe pod pod-name` provides detailed information about a specific pod in Kubernetes. This information includes:

- **Recent events**: Any recent events that have occurred related to the pod.
- **Containers and their images**: Details about the containers within the pod and the images they are using.
- **Mounted volumes**: Information about any volumes that are mounted within the pod.
- **Pod conditions**: The current conditions of the pod, such as whether it is running, pending, or failed.
- **IP addresses**: The IP addresses assigned to the pod.
- **Annotations and labels**: Any annotations and labels that have been applied to the pod.

- To know the image used to create the pod, look under **Containers** for the `Image` field.
- To know how many containers are part of the pod, count the number of containers listed under the **Containers** section.
- To know the state of a container, check the **State** section under each container, which can be "Waiting", "Running", or "Terminated".
- To understand why a container is in error or waiting, check the **Reason** and **Message** fields under the **State** section for details.
- If the state is `Waiting` and the reason is `ImagePullBackOff`, it indicates that Kubernetes is unable to pull the image. Check the **Reason** and **Message** fields for more details, which might include an incorrect image name, permission issues, or network problems.

## To know which nodes a pod was placed on
```bash
kubectl get pods -o wide
```

This command displays a list of pods along with additional information, including the nodes they are running on.

## What does the READY column in the output of kubectl get pods indicate?

The `READY` column shows the ratio of containers that are ready to serve requests compared to the total number of containers in the pod. The format is `X/Y`, where `X` is the number of containers that are ready, and `Y` is the total number of containers in the pod.

For example:
- If a pod has 2 containers and both are ready, it will show `2/2`.
- If only one container is ready, it will show `1/2`.

## To delete a pod
```bash
kubectl delete pod pod-name
```

This command will delete the specified pod from the Kubernetes cluster.

## Edit a pod
```bash
kubectl edit pod pod-name
```

## Update the pod
```bash
kubectl apply -f pod-name.yaml
```
