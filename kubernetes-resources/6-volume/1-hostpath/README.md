# HostPath Volume
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/650d83408ed4310c0c50dced0350f9a3725f6807/kubernetes-resources/6-volume/1-hostpath/hostpath%20volume.jpg)
In time running of an application inside pod, it will allocate a directory inside server(pod). Where application is running and store files. This directory will mount another directory which is outside our application server(pod) and copying files there.So If pod dies volume will not lost.

### Edit a node:
``` kubectl edit node node-name ```

## Creating HostPath:
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/650d83408ed4310c0c50dced0350f9a3725f6807/kubernetes-resources/6-volume/1-hostpath/hostpath%20volume1.jpg)

From pod data will go to pv through pvc. Then pv store data in "/mnt/data".

```
#### persistent volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"  #### path inside node
```

### Apply yaml file:
``` kubectl apply -f pv.yaml ```

### check persistent volume:
``` kubectl get pv ```

```
#### persistent volume claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  accessModes:
    - ReadWriteOnce #### must same as pv
  resources:
    requests:
      storage: 1Gi #### storage taken from pv storage
```
### Apply yaml file:
``` kubectl apply -f pvc.yaml ```

### Check pvc :
``` kubectl get persistentvolumeclaim ```

### check pv,pvc:
``` kubectl get pv,pvc ```


```
#### Creating pod with volume attached
apiVersion: v1
kind: Pod
metadata:
  name: my-web-server
  labels:
    app: my-web-server
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim   #### name shuold be same as pvc name, by this name pvc will available inside pod
  containers:
    - name: web-server
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"   #### mount Path inside pod
          name: task-pv-storage      #### name should be same as volumes name
```

## Or,

### Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web-server
  labels:
    app: my-web-server
spec:
  selector:
    matchLabels:
      app: my-web-server
  template:
    metadata:
      labels:
        app: my-web-server
    spec:
      volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: task-pv-claim
      containers:
        - name: web-server
          image: nginx
          ports:
            - containerPort: 80
              name: "http-server"
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: task-pv-storage
```

### (Pvc will automatically find pv, hence they inside same node)

### Apply yaml file:
``` kubectl apply -f pod.yaml ```

### Check pods:
``` kubectl get pods ```

[ If pod runs , everything works fine]

### Go inside master node:
``` ssh master@192.168.0.100 ```

### Go inside node mount path:

``` cd /mnt/data ```

-------------------------------------------------------------------------------------------------------
### Use another terminal to go inside pod:
``` kubectl exec -it pod-name -- bash ```

### Go inside pod mount path:
``` cd /usr/share/nginx/html ```

Now write a file here.... (example: touch abc)
You will see file in "/mnt/data" using ls command.

That means our data copying from pod to node. So if our pod will die and create new one, our data will not lost.
Data will be persistent.

This is use in development purpose, not production.
