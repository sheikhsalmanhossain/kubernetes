![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/17a97d7653587ef0c2cb61fe973ad6b30d9bf2b3/kubernetes%20architecture/Kubernetes%20Architecture.jpg)

1) Kubernetes command execute in "kube api server" via "api".
2) Scheduler: It schedule containers where it should be run from(in which worker node).
3) Every node have an agent. which is called "Kublet".
4) etcd: Its a nosql database, data stay in key value pair. It contains all information of the cluster. (Example: If etcd contain information that 2 container should running. But somehow a container deleted, then scheduler will see that etcd contain 2 container information, so scheduler call kube api server to order kubelet create a new container.)
5) Controller manager: To manage all kind of workload in kubernetes. Its the core backend of kubernetes.
6) Kube proxy: It will make connection application component which are running in different worker node.
7) Coredns: Behave like a dns server. coredns make connection between multiple worker node through IP address.
 
How kubernetes Architecture works:

Any comannd after execute, request will go to kubernetes "api server". Then it will communicate with "scheduler" to know which node will create the container. These info stored in "etcd". "controller manager" will manage the kubernetes resources. Then request will go to worker node, inside this worker node "kubelet" will run container. When application will run inside different node, application make communication through "kube proxy". Container can communicate with each other using IP address which is called "coredns".
