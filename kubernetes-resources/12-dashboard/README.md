### Kubectx :
kubectx is a command-line utility that makes it easier to switch between multiple Kubernetes clusters.

### Install kubectx :
``` sudo snap install kubectx --classic ```
### If you want to move one cluster to another:
``` kubectx <cluster-name> ```
### You can check all the available Kubernetes clusters in your kubeconfig :
``` kubectx ```

--------------------------------------------------------------------
### kubens :
kubens helps you switch between namespaces inside a cluster.

### if you want to move one namespace to another inside a cluster :
``` kubens <namespace-name> ```
### check namespace list :
``` kubens ```

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Dashboard :

Reference : https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Installing kubernetes dahsboard :


#### Add kubernetes-dashboard repository: 
``` helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/ ```
#### Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart:
``` helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard ```


Now we can see that "namespace-dashboard" created. Inside namespace there running few pods. These pods help to monitoring the cluster.

#### Command line proxy :
You can enable access to the Dashboard using the kubectl command-line tool, by running the following command:

``` kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443 ```

#### Kubectl will make Dashboard available at https://localhost:8443.

Now our dashboard want permission to pull cluster data. So we need a password or token to get these data.

Now we have to create a manifest file permission.yaml .


permission.yaml :

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

### Apply the yaml file :
``` kubectl aply -f permission.yaml ```

### Create a tocken for dashboard :
``` kubectl -n kubernetes-dashboard create token admin-user ```

NOW, we get a output token after this command applied. We copy the output token. And paste in Kubernetes dashboard page.
we finally enter our dashboard Successfully. 
