# TO KNOW HOW MANY PODS EXISTS IN THE CURRENT NAMESPACE
kubectl get pods

# TO CREATE A NEW POD WITH THE NGINX IMAGE
kubectl run nginx-pod --image=nginx

# TO CREATE A MANIFEST FILE USING kubectl run WITH --dry-run=client -o yaml OPTION
kubectl run pod-name --image=image --dry-run=client -o yaml > pod.yaml

# TO CREATE A RESOURCE FROM THE MANIFEST FILE
kubectl create -f pod.yaml

# TO KNOW THE IMAGE USED TO CREATE A POD & # TO KNOW HOW MANY CONTAINERS ARE PART OF A POD
kubectl describe pod pod-name

The command: kubectl describe pod pod-name provides detailed information about a specific pod in Kubernetes. 
This information includes:

- Recent events: Any recent events that have occurred related to the pod.
- Containers and their images: Details about the containers within the pod and the images they are using.
- Mounted volumes: Information about any volumes that are mounted within the pod.
- Pod conditions: The current conditions of the pod, such as whether it is running, pending, or failed.
- IP addresses: The IP addresses assigned to the pod.
- Annotations and labels: Any annotations and labels that have been applied to the pod.

=> To know the image Under Containers you have just to look in Image to know which Image is used to create the POD
=> To know how many containers are part of a pod look in the output, look for the "Containers" section and count the number of containers listed. Each container will have its own subsection under "Containers".
=> To know the state of a container in a pod, look for the "State" section under each container. The state can be "Waiting", "Running", or "Terminated".
=> To know what causes a container in a pod to be in error or waiting, look for the "Reason" and "Message" fields under the "State" section. These fields will provide details about the cause.
=> If the state is `Waiting` and the reason is `ImagePullBackOff`, it indicates that Kubernetes is unable to pull the container image. Check the "Reason" and "Message" fields for more details. Common causes include incorrect image name, lack of permissions, or network issues.

# TO KNOW WHICH NODES A POD WAS PLACED ON 
kubectl get pods -o wide

This command will display a list of pods along with additional information, including the nodes they are running on.

# What does the READY column in the output of the kubectl get pods command indicate?

The `READY` column shows the ratio of containers that are ready to serve requests to the total number of containers in the pod. It is displayed in the format `X/Y`, where `X` is the number of containers that are ready, and `Y` is the total number of containers in the pod.

For example, if a pod has 2 containers and both are ready, the `READY` column will show `2/2`. If only one container is ready, it will show `1/2`.

# TO DELETE A POD
kubectl delete pod pod-name

This command will delete the specified pod from the Kubernetes cluster.

