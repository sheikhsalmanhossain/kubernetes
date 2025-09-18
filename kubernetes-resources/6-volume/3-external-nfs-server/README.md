# nfs-server:
- Used in production
- Data persist
- External env/server
[image]
Inside application there is a path which always sync with external storage. If application dies external storage volume will stay. Or even Kubernetes cluster dies our data will untouched. Then we create a new cluster and sync data like before. This is a good way to save data from disaster.

### [SETUP "nfs-server" FROM "kubernetes_setup/nfs-Server-setup"]

image 2

Now added a controller/manager using helm.
Now if any pod want to access external storage. pod call controller/manager. Controller/manager make a connection between pod and external storage. Then pods data stored in external storage 
. we make this connection by command "helm-nfs".
(Check inside nfs setup)

### Check pods :
`` kubectl get pods ```

If any issue occurs.
### Check pods details :

``` kubectl describe pod pod-name ```

## Now Create A Pod Which need to make a connection with external nfs server :
### Check storage class :

``` kubectl get storageclass ```

We have to use this storage class. Storage class is a middleman between pod and manager/controller.

```

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test
spec:
  storageClassName: nfs-client    #### from "kubectl get storageclass" name
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi

```

### Apply the yaml file :

``` kubectl apply -f pod.yaml ```

Now created persistent volume claim.

### Check pvc :

``` kubectl get pvc ```

If status " bound " means it works.

We created a volume. Now we have to add this volume with a pod/deployment:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
spec:
  replicas: 2        #### Creating 2 pod
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: test-container
        image: nginx
        volumeMounts:
        - mountPath: /data         #### mounting path inside container
          name: test-volume        #### Must have to same as volumes name
      volumes:
      - name: test-volume
        persistentVolumeClaim:
          claimName: test         #### Must have to same as persistent volume claim name


```

### Apply this yaml file :

``` kubectl apply -f pod.yaml ```

### Check pods :

``` kubectl get pods ```

### Get inside pod :
``` kubectl exec -it pod-name -- bash ```

# Go to mountpath :
``` cd /data/ ```

This location inside our pod.
---------------------------------------------
Now another terminal,

### Our nfs ssh:
### Go to mount path :

``` sudo cd /var/k8-nfs/data/ ```

------------------------------------------

Now if we write a file our pod mount path, we see this same nfs mount path.

Now, in nfs:
``` kubectl get pvc ```
Now Cd this directory and command ls.


BOOM, Now we can see our data........
Similar way we can see another pod (we created 2 pod)
