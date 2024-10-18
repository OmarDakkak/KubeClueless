
# Kubernetes Namespaces Cheat Sheet

This cheat sheet provides useful commands for working with Kubernetes namespaces and pods within those namespaces.

## 1. To List All Namespaces

Run the following command to list all available namespaces in your Kubernetes cluster:
```bash
kubectl get namespace
```

This will display all namespaces currently defined in the cluster. You can also count the number of namespaces by piping the output to `wc -l`:
```bash
kubectl get namespace | wc -l
```

## 2. To List All Pods in a Specific Namespace

To list all the pods within a particular namespace, use the following command, replacing `<namespace-name>` with the actual namespace name:
```bash
kubectl get pods --namespace=<namespace-name>
```

This command will show all pods running in the specified namespace.

## 3. To Create a Pod in a Specific Namespace

To create a pod inside a specific namespace, run the following command. Replace `<pod-name>`, `<image>`, and `<namespace-name>` with the desired pod name, image, and namespace:
```bash
kubectl run <pod-name> --image=<image> -n <namespace-name>
```

For example, to create a pod named `my-pod` using the `nginx` image in the `development` namespace:
```bash
kubectl run my-pod --image=nginx -n development
```

## 4. To List All Pods Across All Namespaces

To view pods across all namespaces, use the following command:
```bash
kubectl get pods --all-namespaces
```

This will list all running pods in all namespaces.

### To Search for a Specific Pod Across All Namespaces

If you're looking for a specific pod in any namespace, you can use the `grep` command to search for its name. Replace `<pod-name>` with the name of the pod you're looking for:
```bash
kubectl get pods --all-namespaces | grep <pod-name>
```

This will help you locate the pod across multiple namespaces.
