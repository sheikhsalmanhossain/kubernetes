# Service :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/14557dd04fdd19c790d62cb760732d9170559dba/kubernetes-resources/service/Service.jpg)


# Private IP Based Pod Communication :
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/79afea7d9765418688f892fd537aaaf8675e7f6a/kubernetes-resources/service/ip%20based%20pod%20communication.jpg)


```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    name: pod1
spec:
  containers:
  - name: myapp
    image: nginx
    ports:
      - containerPort: 80

---

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    name: pod2
spec:
  containers:
  - name: myapp
    image: nginx
    ports:
      - containerPort: 80
```

### Apply this yaml file:
``` kubectl apply -f filename.yaml ```

### Checks pods :
``` kubectl get pods ```

### Access inside pod2 :
``` kubectl exec -it pod2 -- bash ```
or,
### To go inside pod:
``` kubectl exec -it pod-name -- bash ```

### Watch logs of pod1(use another terminal):
``` kubectl logs pod1 -f ```
or
### Watch logs of a pod:
``` kubectl logs pod-name -f ```


### Hit request from pod 2 to pod 1:
``` curl 10.233.171.74 ``` (private ip of pod1)

NOW WE CAN CONNECT FROM POD2 TO POD1 USING POD1 PRIVATE IP
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### But if i restart this pod or replace with new pod & IP will change. Then how this pod can be find out ?
### That's why we need " Cluster IP Service "

# 1) Cluster IP Service :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/14557dd04fdd19c790d62cb760732d9170559dba/kubernetes-resources/service/Clusterip%20Service.jpg)


## For Pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    name: pod1
spec:
  containers:
  - name: pod1
    image: nginx
    ports:
      - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: pod1-service
spec:
  selector:
    app: pod1    #### ("selector" and "levels" name should be same to introduce service about pod)
  ports:
  - port: 80    #### (Service expose port)
    targetPort: 80 #### (target port is container port)

```



( We create a service with pod1, so if pod1 restart or deleted and create new one automatically. Then pod IP will change. So, If we use service with pod. We can call service instead of pod. And service call pod1. )

### (We should add service per deployment/pod)

### Apply yaml file:
``` kubectl apply -f pod1.yaml ```
### Check pods :
``` kubectl get pods ```
### Check service :
``` kubectl get service ```

### Show details about sevice :

``` kubectl describe service service-name ```
( Now we can see that Endpoint give an ip address. That means "selector" and "levels" name are same to introduce service about pod)

### Delete a service:
``` kubectl delete service service-name ```

## Now we create another pod (pod2) yaml file:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    name: pod2
spec:
  containers:
  - name: pod2
    image: nginx
```
### Apply this yaml file:
``` kubectl apply -f pod2.yaml ```

### Enter inside pod2:
``` kubectl exec -it pod2 -- bash ```

### Watch logs pod1 (In another terminal):
``` kubectl logs pod1 -f ```

### Now from pod2:
``` curl service-name ``` 

(instead of private IP, we will use service name.  Because IP can be change after restart/new pod)


Now, If i change inside "Service", service export port from 80 to 8080, i cant curl service. Then i should curl service like this:
 
``` curl service-name:8080 ```



## [(For Deployment :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
          - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp #### ("labels" and "selector" should be same)
  ports:
  - port: 80
    targetPort: 80

```

)]

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 2) Node Port Service :
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/cfcea8848d766f006ca99be979f69aba0dd004aa/kubernetes-resources/service/NodePort%20Service.jpg)


If we want to browse our application outside cluster, then we need to use NodePort.
## Purpose Of NodePort:
The purose of nodePort is expose port inside node and creating cluster IP for application.

## How NodePort Works :


NodePort Service reach app through lebels. When we expose a application on nodeport, then nodeport will create port(same port number) in every node (master and worker).

If someone hit on master node ip with master node port (example 10.173.179.70:30000). Then port(30000) will forward this to "NodePort Service". Similarly, If someone hit on worker node ip with worker node port (example 10.174.179.71:30000). Then port(30000) will forward this to "NodePort Service".


(NodePort Range: 30000-32767).

## Creating NodePort Service :

```
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    name: test
spec:
  containers:
  - name: test
    image: nginx
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    name: test          #### ("selector" and "levels" name should be same to introduce service about pod)
  ports:
    - port: 80           ### (does not matter on nodeport. Because It works for cluster IP, if we need specific cluster ip then its important.
                          ### Example: if port 8080 then we have to curl ip:8080 to get hit)
      targetPort: 80      ##(container port)
      nodePort: 30000     #### (if we do not define node port, it will choose randomly & change after restart)

```

### Apply this yaml file:
``` kubectl apply -f filename.yaml ```

### Watch pod, service, node :
``` kubectl get pod,service,node -o wide ```


### TO check in browser:
1) Take any node IP (example: 192.168.0.101)
2) From Service check "NodePort service" and copy outside port (example: 30000)
3) Now browse it from any browser. (example : 192.168.0.101:30000)

### If get any error & need to debug :
``` kubectl describe service service-name ```
( Here we use nodeport service name )


When we apply yaml file, nodePort service will create a cluster ip. Cluster ip will catch application. Cluster Ip contain a name. Other application can reach this application by this name (ClustrtIP Service). (If we create nodePort cluster ip automatically crate inside to make connection between application).

 

Now we see that, we have to browse "ip:port" to get in application from outside world. Its not a good solution. So wee need load balancer to solve this problem.


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 3) Load Balancer :

