# Kubernetes Deployment:
Kubernetes Deployment is a resource object in Kubernetes that provides a declarative way to manage Pods and ReplicaSets.
It’s one of the most commonly used controllers in Kubernetes because it allows you to define what you want your application to look like,
and Kubernetes makes sure the cluster matches that desired state.

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/5714367e30d21f81e3eea1e6169472ed6b8c2caa/kubernetes-resources/deployment/Deployment.jpg)

------------------------------------------------------------------------------------------------------------------------------------------------------------

# Deployment scaling:
## Deployment scaling in Kubernetes means increasing or decreasing the number of Pods that a Deployment manages.

##  Why Scaling?

- To handle more traffic (scale up).

- To save resources/costs when load is low (scale down).

- To maintain high availability.

##  How Scaling Works

- When you scale a Deployment, Kubernetes adjusts the ReplicaSet associated with it.

- Scale up → New Pods are created and scheduled on available nodes.

- Scale down → Extra Pods are terminated.

 - The Deployment controller always ensures the number of running Pods matches the desired replica count.





![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/5714367e30d21f81e3eea1e6169472ed6b8c2caa/kubernetes-resources/deployment/Deployment2.jpg)
