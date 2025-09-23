# Ingress :
Ingress is a reverse proxy in Kubernetes. Ingress is a custom controller in Kubernetes. Ingress means inbound of traffic.If traffic goes to external, its called egress. In deployment,service,secret etc has a controller inside Kubernetes by default. But ingress have no controller built in Kubernetes. so we have to import this. Implement these commands (https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx) to deploy ingress controller in our cluster.


## step 1 :
### Install helm : 

( reference: https://helm.sh/docs/intro/install/ )

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

## Step 2 :
### setting up ingress controller :

 ( reference: https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx )

#### Get Repo Info :
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

#### Install Chart :

``` helm install [RELEASE_NAME] ingress-nginx/ingress-nginx ```
(replace your RELEASE_NAME)
( In the background there will create some jobs running in pod. Wait untill its done. Also you can check ``` kubectl describe pod pod-name ``` to see whats it doing. )
After successful, a controller pod will run automatically.


## Step 3 :
Now we run our application.

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/dc89e5e6931010346c86cc849584e65e63c903d4/kubernetes-resources/10-ingress/ingress.jpg)

We setup a custom controller in our cluster. We create a deployment and create a service(clusterIp) for this deployment. Now we point this service by ingress. And ingress will provide us an external endpoint (ip) as like load balancer. Now we can browse this ip. If we want we can add domain in ingress. Then we can browse our application by domain. We can also add ssl certificate here.




deployment.yaml :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service      #### if there's no service type. so its clusterip service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### Apply yaml file :
``` kubectl apply -f deployment.yaml ```
( this yaml will create a deployment and a service )


ingress.yaml :

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
spec:
  ingressClassName: nginx          #### hence its nginx ingress(nginx will install behind the seen)
  rules:
    - http:
        paths:
          - pathType: Prefix
            backend:               #### whose behind ingress(see in image)
              service:
                name: nginx       #### from deployment.yaml service name
                port:
                  number: 80      #### port from deployment.yaml
            path: /               #### "/" means "ip/" and log will execute this path
```

### Apply yaml file :
``` kubectl apply -f ingress.yaml ```

### check ingress :
``` kubectl get ingress ```

( we did not get address )
### Check service :
``` kubectl get svc ```

[we can see that ingress controller creating load balancer behind the seen, but external ip pending(watch loadBalancerService to solve this). {But If we use cloud cluster we do not need metallb, by default we get a public ip. For load balancing we will use aws load balancer then}]

Now, We have to install another custom controller for metallb. (follow metallb setup page & setup metallb)

### check service :
``` kubectl get svc ```

Now we get our external ip in ingress controller created by loadBalancer .
### check ingress :
``` kubectl get ingress ```

After few times we can see our address.
Now we can browse our application by this ip.

Project summary :

We create a deployment and expose by clusterip internally. Then we catch this service by ingress.


We can see here, Ingress works like load balancer here.

### delete all yaml file :
``` kubectl delete -f deployment.yaml ```
``` kubectl delete -f ingress.yaml ```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Another example :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/dc89e5e6931010346c86cc849584e65e63c903d4/kubernetes-resources/10-ingress/ingess2.jpg)



Deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache
spec:
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: apache
spec:
  selector:
    app: apache
  ports:
  - port: 80
    targetPort: 80
```

### Apply yaml file :
``` kubectl apply -f deployment.yaml ```

We create deployment. Now we have to write ingress for these. 

ingress.yaml :

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: example
spec:
  ingressClassName: nginx   #### thus we use nginx
  rules:
    - http:
        paths:
          - pathType: Prefix
            backend:
              service:
                name: apache
                port:
                  number: 80
            path: /apache

          - pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
            path: /nginx
```

We have to manage 2 service. Thats why in path section we created 2 path. 

In first path, we said, our backend have a service, which is apache, running on port 80, and if you request in "/apache" then we send in "service:name:apache" .
Similarly to nginx.

When a request come in ingress, ingress will match the request path, is "/apache" or "/nginx".

### Apply yaml file :
``` kubectl apply -f ingress.yaml ```
#### check ingress :
``` kubectl descibe ingress ingress-name ```


Anottion is a logic in ingress. It will add a special feaure inside menifest. "rewrite-target" means when a request came and match with path (/apache or /nginx) and then request will go to service, then request will be rewite and come in "/". 
