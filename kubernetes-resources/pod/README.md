# What is Pod
In Kubernetes, a Pod is the smallest and simplest unit of deployment. It represents a single instance of a running process in your cluster. A Pod can contain one or more containers, which are tightly coupled and share resources such as networking and storage.

### Create an NGINX Pod via imperative command:
``` kubectl run pod-name --image=nginx ```

### Check pod list:
``` kubectl get pods ```

### get pod output in "yaml" format and keep inside "abc.yaml" :
``` kubectl get pod pod-name -o yaml > abc.yaml ```

### details about pods :
``` kubectl get pod -o wide ```

### describe about pod details :
``` kubectl describe pod pod-name ```

### Create an NGINX Pod with manifest file
``` kubectl apply -f abc.yaml ```

### Dry run: Print the corresponding API objects without creating them:
(dry run means it will not create pod actually, it just shows it works(test purpose only))

``` kubectl run nginx --image=nginx --dry-run=client ```

### Generate POD Manifest YAML file (-o yaml) with (–dry-run):
``` kubectl run nginx --image=nginx --dry-run=client -o yaml ```

### Get the manifest file of a pod:
``` kubectl get pod podname -o yaml ```

### Run with Labels, Example tier:
``` kubectl run redis -l tier=db --image=redis:alpine ```

### Deploy a redis pod using the redis:alpine image with the labels set to ``` tier=db ``` :
``` kubectl run redis --image=redis:alpine --labels tier=db ```

### List all pods , with more details:
``` kubectl get pods -o wide ```

### Port forward:
(after creating a pod, if i want to see my dockerImage which is running inside pod & its output)
``` kubectl port-forward pods/pod-name 8080:80 ```

### delete pod:
``` kubectl delete pod podname ```





# Creating a pod:

## "kubectl run pod-name --image=nginx" Convert this command in "yaml" format :

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
       - containerPort: 80
```
