# Network Policy :

Network policy decide which pod can communicate with which pod and which pod can not.

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/6f0792990f4b345b7d9f64c762f2e33967fc9626/kubernetes-resources/13-networkpolicy/networkPolicy.jpg)

Inside a cluster we create a namespace. Inside namespace we create 2 pod. We provide label two pod. Because using label we handle network policy. Now we create a network policy hence pod1 can communicate with pod2. And we create another pod, hence new pod will unable to communicate. Network policy same as like security group. It only follow allow rule. If there is no allow rule, it will denied by default.



namespace.yaml :

```
apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
```

### Apply yaml file :
``` kubectl apply -f namespace.yaml ```

### Check namespace :
``` kubectl get namespace namespace-name ```

Now we create two pod inside this namespace.

pods.yaml :

```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: test-namespace  #### specify our namespace name
  labels:                    #### use label
    app: pod1
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: test-namespace
  labels:
    app: pod2
spec:
  containers:
  - name: nginx
    image: nginx
```

### Apply yaml file :
``` kubectl apply -f pod.yaml ```

Now we create a network that pod1 can reach out pod2.


networkpolicy.yaml :

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-pod1-to-pod2
  namespace: test-namespace    #### same namespace
spec:
  podSelector:
    matchLabels:
      app: pod2           #### creating ingress rule for pod2
  policyTypes:
  - Ingress               #### hence traffic inbound, otherwise if outbound it will egress
  ingress:
  - from:                 #### from where traffic comes
    - podSelector:
        matchLabels:
          app: pod1       #### we allow traffic from pod1 by networkpolicy
```
(pod to pod communication. So we dont use port or ip.)

### apply yaml file :
``` kubectl apply -f networkpolicy.yaml ```

Now, we have to switch our namespace:
``` kubens namespace-name ```

### Check pod :
``` kubectl get pod ```

### shell access to pod1 :
``` kubectl exec -it pod-name -- bash ```

In another terminal
### watch log pod2 :
``` kubectl logs -f pod-name ```

### Inside pod1 curl to pod2 ip :
``` curl pod2-ip ```

Now in pod1 we can see output, "welcome to nginx". And in pod2 logs, we can see that new logs added.


Finally we successfully communicate from pod1 to pod2.
---------------------------------------------------------------------
Now we create another pod :
Create another pod named pod3, now we try to communicate with pod 2 by this pod.


pod3.yaml :

```
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  namespace: test-namespace
  labels:
    app: pod3
spec:
  containers:
  - name: nginx
    image: nginx
```

### Apply yaml file:
``` Kubectl apply -f pod3.yaml ```
### shell access to pod3 :
``` kubectl exec -it pod-name -- bash ```

From inside pod3, we curl to pod2 ip :
``` curl pod2-ip ```
Now this will hang, because can not curl to pod2. This can not reach because its out of network policy.

### delete all yaml file :
``` kubectl delete -f . ```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Another example :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/6f0792990f4b345b7d9f64c762f2e33967fc9626/kubernetes-resources/13-networkpolicy/networkPolicy2.jpg)

Here we create three namespace. Now we create a pod in every namespace. We create level in every namespace. Then we write a rule in network policy that, only from namespace b request will accept by namespace a. And request will not accept from namespace c. 

namespace.yaml :

```
apiVersion: v1
kind: Namespace
metadata:
  name: a
  labels:
    name: a
---
apiVersion: v1
kind: Namespace
metadata:
  name: b
  labels:
    name: b
```

### Apply yaml file :
``` kubectl apply -f namespace.yaml ```

Create pod inside these namespace :
pod.yaml :

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: a   #### define namespace
  labels:
    app: pod-a
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  namespace: b
  labels:
    app: pod-b
spec:
  containers:
  - name: nginx
    image: nginx
```

### Apply yaml file :
``` kubectl apply -f pod.yaml ```

Now check we create 2 namespace. And every namespace has a pod.

If i carl from one pod to another pod ip, it can curl without any restriction. Because we do not set network policy.

Now we have to set a network policy.

networkpolicy.yaml :

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-b
  namespace: a
spec:
  podSelector: {}     #### Selects all pods in namespace a
  policyTypes:        #### network policy type
  - Ingress           #### inbound request . so ingress
  ingress:
  - from:             #### where request come from
    - namespaceSelector:     #### from a specific selector
        matchLabels:
          name: b            #### where lebel name is b
      podSelector:           #### namespaceSelector + podSelector
        matchLabels:
          app: pod-b         #### pod level b
```
crosscheck network policy : https://orca.tufin.io/netpol/
(in this site you can provide your network policy and can see output)

### Apply yaml file :
``` kubectl apply -f networkpolicy.yaml ```
Now we curl from pod b to pod a.
### ssh pod b :
``` kubectl exec -it pod-name -- bash ```
### curl to pod a :
``` curl pod-a-ip ```
Curl successful. Now we can send request from pod-b to pod-a.

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/6f0792990f4b345b7d9f64c762f2e33967fc9626/kubernetes-resources/13-networkpolicy/networkPolicy3.jpg)

Description of this networkpolicy :
In namespace b which pods have lavel in b, only they can communicate with all pods in namespace a. That means namespace b and level b can access in namespace a all pods.




------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Now we create another namespace and pod :
pod.yaml :

```
apiVersion: v1
kind: Namespace
metadata:
  name: c
  labels:
    name: c
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
  namespace: c
  labels:
    app: pod-c
spec:
  containers:
  - name: nginx
    image: nginx
```

### Apply yaml file :
``` kubectl apply -f pod.yaml ```

### go to c namespace and ssh c pod
### from there curl on namespace a:
``` curl pod-a-ip ```
Now it can not curl successfully and hang . That means its out of our network policy, so it can not communicate with pod a.

Our network policy work successfully.
