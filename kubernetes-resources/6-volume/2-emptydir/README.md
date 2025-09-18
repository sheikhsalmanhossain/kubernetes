# Emptydir :

(This used in production)
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/ef0008813a330bdc2c67550e4bbd6c06528ee20d/kubernetes-resources/6-volume/2-emptydir/emptydir.jpg)

Inside pod, container will mount volume with external volume. But the External volume is ephemeral. If pod get deleted, external volume will be deleted too.

```
#### pod with mount external volume
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  volumes:
  - name: cache-volume
    emptyDir:             #### volume type
      sizeLimit: 500Mi
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /cache     #### path inside pod
      name: cache-volume    #### same as volumes name
```

### Apply yaml file:

``` kubectl apply -f pod.yaml ```

### check pod :
``` kubectl get pod ```

### Go inside pod :

``` kubectl exec -it pod-name -- bash ```

### go mount path :
``` cd /cache/ ```

create some file using "touch" and check by "ls"

### Now delete the pod & create new one:
``` kubectl delete -f pod.yaml ```
``` kubectl apply -f pod.yaml ```
### Go inside pod :
``` kubectl exec -it pod-name -- bash ```
go to " cd /cache/ " and run "ls",
you will see nothing there.
That means data deleted. After deleted pod, data deleted automatically. So  data is not persistent.


```
apiVersion: v1
kind: Pod
metadata:
  name: ed-pod
spec:
  containers:
  - image: alpine
    name: myvolumes-container-1
    command: ['sh', '-c', 'sleep 3600']
    
    volumeMounts:
    - mountPath: /demo1
      name: demo-volume

  - image: alpine
    name: myvolumes-container-2
    command: ['sh', '-c', 'sleep 3600']      #### sh=shell, -c=command, sleep 3600 sec means makes the process pause(do nothing) for 3600 seconds. After that time, the shell continues with the next command.
    
    volumeMounts:
    - mountPath: /demo2                    #### mount path inside volume
      name: demo-volume                    #### same as volumes name

  volumes:
  - name: demo-volume
    emptyDir: {}
```

we can see that, inside a pod have 2 container. now we have to mount these 2 container inside a single volume outside. so two containers can share same data.


### Apply the yaml file :
``` kubectl apply -f pod.yaml ```

Hence i have 2 container inside a pod. Then how we can access a specific pod?
### Go inside random container :

``` kubectl exec -it pod-name --sh ```

### Go inside specific container :

``` kubectl exec -it pod-name -c container-name --sh ```

Now we login inside 2 container in different tab and go mount path " cd /demo1 " " cd /demo2 " for first and second container.
Now if we write file inside a container we get same data inside another one. So they shares same volume outside container.

## Use case of this directory :
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/ef0008813a330bdc2c67550e4bbd6c06528ee20d/kubernetes-resources/6-volume/2-emptydir/emptydir1.jpg)

Hence "second container" get same log data, so we can add an exporter as a side car here and send this log data to "external service" where log data stored permanently.
Thats how emptydir volume works.
