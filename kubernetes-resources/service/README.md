# Service :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/14557dd04fdd19c790d62cb760732d9170559dba/kubernetes-resources/service/Service.jpg)


## Private IP Based Pod Communication :
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
## But if i restart or replace with new pod then how this pod can be find out ?
That's why we need " Cluster IP Service "

## Cluster IP Service :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/14557dd04fdd19c790d62cb760732d9170559dba/kubernetes-resources/service/Clusterip%20Service.jpg)
