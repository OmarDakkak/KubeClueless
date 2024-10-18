
# Kubernetes Deployments Cheat Sheet

## 1. To List All Deployments
To list all the deployments in your Kubernetes cluster, you can use one of the following commands:

```bash
kubectl get deployment
```
or
```bash
kubectl get deploy
```

The command will display a table with details such as the deployment name, desired number of replicas, available replicas, and the current state. To count the number of deployments, simply observe the output or pipe the result to `wc -l` for a quick count:

```bash
kubectl get deployment | wc -l
```

## 2. To Check the Image Used in a Deployment
To find out which container image is used to create the Pods in a specific deployment, use the `kubectl describe` command, which provides detailed information:

```bash
kubectl describe deployment <deployment-name>
```

Look for the **Containers** section and find the `Image:` field to know the image being used. This is particularly useful when you're debugging or checking for image versions.

## 3. To Create an Editable YAML File for a Deployment
If you want to create a deployment and save its configuration in a YAML file for further editing, you can use the `kubectl create` command with the `--dry-run=client` option to avoid actually creating the deployment. Instead, this will generate the YAML file:

```bash
kubectl create deployment <deployment-name> --image=<image-name>:<image-version> --dry-run=client -o yaml > <deployment-name>-deployment.yaml
```

Replace `<deployment-name>`, `<image-name>`, and `<image-version>` with the appropriate values for your deployment. This command will save the YAML configuration to a file, which you can edit and apply later using:

```bash
kubectl apply -f <deployment-name>-deployment.yaml
```

---

### Additional Tips:

- **Scaling Deployments**: You can scale a deployment to increase or decrease the number of replicas (pods) using the `kubectl scale` command:
  ```bash
  kubectl scale deployment <deployment-name> --replicas=<number>
  ```

- **Viewing Deployment History**: To see the history of changes made to a deployment (useful for rolling updates), you can use:
  ```bash
  kubectl rollout history deployment <deployment-name>
  ```

- **Rolling Back to a Previous Revision**: If you need to undo a deployment, you can roll back to a previous version:
  ```bash
  kubectl rollout undo deployment <deployment-name>
  ```

---

This cheat sheet provides essential commands for working with deployments in Kubernetes, allowing for easier management and troubleshooting of your applications.
