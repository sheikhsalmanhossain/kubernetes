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


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Create a deployment:
``` kubectl create deployment deployment-name --image nginx ```

## Check deployment & pods:
``` kubectl get deployments,pods ```

## Check deployment details:
``` kubectl get deployments -o wide ```

## Deployment describe :
``` kubectl describe deployments deployment-name ```

## Delete deployment:
``` kubectl delete deployment deployment-name ```

## Create deployment with replicas:
``` kubectl create deployment deployment-name --image nginx --replicas 3 ```

## Scale(up/down) deployment (pod replicas) :
``` kubectl scale deployment deployment-name --replicas 10 ```

## Show pod logs :
``` kubectl logs pod-name ```
## Show deployment logs (all pods):
``` kubectl logs deployments/deployment-name -f ```


# Deployment file with .yaml format:

``` nginx-deployment.yaml ```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-name
  labels:
    app: nginx
spec:
  replicas: 3 //(make 3 replicas of this template)
  selector:
    matchLabels:
      app: nginx
  template:
//(info below this actually describe a pod configuration)
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: container-name
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## Deploy this .yaml file:
``` kubectl apply -f nginx-deployment.yaml ```

## Delete deployment :
``` kubectl delete -f nginx-deployment.yaml ```

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Create deployment yaml file using command :

## Create a deployment.yaml file by command with dry run(where yaml wil not create directly):
``` kubectl create deployment deployment-name --image image-name --replicas 3 --dry-run=client ```

## Create a deployment.yaml file by command with dry run(where yaml wil not create directly) & get output yaml format(generate the Deployment manifest):

``` kubectl create deployment deployment-name --image image-name --replicas 3 --dry-run=client -o yaml ```

## Put this yaml file inside ``` nginx-deployment.yaml ``` :
``` kubectl create deployment deployment-name --image image-name --replicas 3 --dry-run=client -o yaml > nginx-deployment.yaml ```

## Deploy this .yaml file:
``` kubectl apply -f nginx-deployment.yaml ```

## Updating nginx-deployment deployment:
### Let's update the nginx Pods to use the nginx:1.16.1 image instead of the nginx:1.14.2 image:

``` kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 ```

### Alternatively, you can edit the Deployment and change:

``` kubectl edit deployment/nginx-deployment ```

## Edit a deployment :
``` kubectl edit deployments deployment-name ```

