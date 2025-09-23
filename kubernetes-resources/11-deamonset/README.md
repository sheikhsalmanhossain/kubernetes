# Deamonset :


![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/8aac25b14d938c7f0fe7ea34657be5c674c2defe/kubernetes-resources/11-deamonset/demonset.jpg)


In Kubernetes, a DaemonSet is a type of workload object that ensures a copy of a Pod runs on all nodes in a cluster.

### Charactertistics of DaemonSet:

- When you create a DaemonSet, Kubernetes automatically schedules one Pod per node.

- If you add a new node to the cluster, Kubernetes will automatically start the DaemonSet Pod on that new node.

- If a node is removed, the Pod is also removed.

### Common use cases:

- Running cluster storage daemons (e.g., glusterd, ceph)

- Running log collectors (e.g., fluentd, logstash)

- Running monitoring agents (e.g., Prometheus Node Exporter, Datadog agent)

- Running networking services (e.g., kube-proxy, CNI plugins)




demonset.yaml:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: busybox-logger
  namespace: default
spec:
  selector:
    matchLabels:
      app: busybox-logger
  template:
    metadata:
      labels:
        app: busybox-logger
    spec:
      containers:
      - name: busybox
        image: nginx
```

### Apply yaml file :
``` kubectl apply -f daemonset.yaml ```

### check deamonset :
``` kubectl get daemonset ```

### check pod :
``` kubectl get pod ```

(we can see our demonset pod is running)

### check node :
``` kubectl get node ```

Deamonset will run on every node(accept master). It will create a pod and collect log  data from node and export it to dashboard. It create 1 pod per node. If we want to run on master node too, we have to remove taint from master node ("edit node master" command). But its not good practice. If we want to add demonset on master node in production. We have to use "taint and toleration" method. This means if taints stay, just we have to add toleration.  
