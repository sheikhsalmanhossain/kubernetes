# HPA :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/c6095f79e6a6312a1b700bd4bb2f4d1155cc5511/kubernetes-resources/14-HPA/hpa.jpg)

HPA means Horizontal Pod Autoscaler .

We have to add matric server first.
Reference : https://github.com/kubernetes-sigs/metrics-server

Matric server monitor matrics usage of our Kubernetes resource.
Using matric server our pod will scale up/down.

To install the latest Metrics Server release from the components.yaml manifest, run the following command:
``` kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml ```

This command will create a pod/deployment name "matric-server" in kube-system namespace. go to log of this pod/deployment. In log we see that it cant work.That means this pod/deployment can not be ready. 
Now describe this pod :

 ``` kubectl describe pod-name ```

In describe we see that "Ready=false"

That means this pod/deployment failed to get ready. So we have to edit kube-system namespace deployment.

### see deployments :

``` kubectl get deployment -n namespace-name ```

### Edit deployment :

``` kubectl edit deployment matrics-server -n kube-system ```
``` kubectl edit deployment deployment-name -n namespace-name ```

Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing --kubelet-insecure-tls to Metrics Server)[from: https://github.com/kubernetes-sigs/metrics-server]


Now we have to add  " --kubelet-insecure-tls "  this line inside " spec:containers:args: " and save it.

Now we have to restart this deployment.
After few minitues our deployment get ready.
Our matrics server is ready..............

### Check all pods cpu and memory usage :

" kubectl top pods -A "

Now we can see pods cpu usage and memory usage.
That means matrics server added.

### check node cpu and memory usage :
``` kubectl top node ```

### sort by cpu usage :
``` kubectl top pods --sort-by=cpu -A ```

### sort by memory usage :
``` kubectl top pods --sort-by=memory -A ```

### nodes sort by cpu :
``` kubectl top nodes --sort-by=cpu ```

### nodes sort by memory :
``` kubectl top nodes --sort-by=memory ```



deployment.yaml :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default   #### define namespace
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 400m   #### max the pod can use, but HPA doesn’t use this for scaling
          requests:     #### Target 50% → means pod scaling triggers at 100m CPU usage
            cpu: 200m   #### baseline for scaling.
---
apiVersion: v1
kind: Service    #### we expose this deployment by clusterIp service
metadata:
  namespace: default   #### define namespace
  name: php-apache
  labels:               #### using levels, service catch deployment
    run: php-apache 
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    run: php-apache
```



hpa.yaml :

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default     #### define namespace
spec:
  scaleTargetRef:         #### whom i target to scale
    apiVersion: apps/v1
    kind: Deployment      #### target a deployment
    name: php-apache      #### deployment name
  minReplicas: 1          #### minimum pod
  maxReplicas: 10         #### maximum pod
  metrics:                #### follow matric for scaling
    - type: Resource
      resource:
        name: cpu         #### cpu usage type resource
        target:
          type: Utilization
          averageUtilization: 50   #### if cpu utilization get in 50%
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30      #### Wait 30 sec before scaling down
      policies:
        - type: Percent
          value: 10                       #### Scale down by 10% of current replicas
          periodSeconds: 30               #### Evaluate every 30 seconds
    scaleUp:
      policies:
        - type: Percent
          value: 50                       # Scale up by 50% of current replicas
          periodSeconds: 30               # Evaluate every 30 seconds
      stabilizationWindowSeconds: 0       # No stabilization for scaling up
```


### Apply yaml file :
``` kubectl apply -f . ```

### check HPA :

``` kubectl get hpa -A ```

### watch HPA :
``` watch kubectl get hpa -A ```
(keep open this terminal)

In another terminal,
Now we add traffic to check autocaling works or not.

load.yaml :

```
apiVersion: v1
kind: Pod
metadata:
  name: load-generator
  namespace: default
spec:
  containers:
  - name: load-generator
    image: busybox:1.28
    command:
    - /bin/sh
    - -c
    - "while sleep 0.01; do wget -q -O- http://php-apache; done"
  restartPolicy: Never
```
In this yaml file we run a loop which hit(curl) "http://php-apache" this url every 0.01 sec. This url is inside deployment's service url, which expose outside our deployment.



### Go to "default" namespace:
``` kubens default ```

### watch pods:
``` watch kubectl get pods ```
(keep open this terminal)

Use another terminal

### Apply yaml file :

``` kubectl apply -f load.yaml ```

Now we can see "hpa" terminal showing load increasing % . And in "pods" terminal pods are creating(up scaling).

If we go "kube-system" namespace "kube-controller" pods log we can see that pods are scaling.
if we decribe hpa we can see info also.

-----------------------------------------------------------------------------
Now we remove our load.yaml to scale down pods.
### Delete load generator pod
``` kubectl delete -f load.yaml ```

After few minitues wee see that pods are scaling down.
Now if resource utilization get below 10% for 30 secend, one pod get deleted, after 30 sec delete another automatically................
Thats how autoscaling works...........

### check events happen in kubernetes cluster:
``` kubectl get events ```

### check only autoscaler events :
``` kubectl get events | grep autoscaler ```
