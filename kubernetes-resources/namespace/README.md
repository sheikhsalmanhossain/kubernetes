# Namespace:

 ![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/57fe628a22d7fda76e05ead6f7c6e0b4c6a666a2/kubernetes-resources/namespace/namespace.jpg)

In Kubernetes, a namespace is a way to logically divide and organize cluster resources.

Think of it like separate "virtual clusters" inside the same physical Kubernetes cluster.

## Key Points about Namespaces:

### Network Isolation:

 They provide a way to separate resources (pods, services, deployments, etc.) into groups. For example, you can have a dev, staging, and production namespace within the same cluster.

### Resource Management:

 Namespaces can be used to apply resource quotas (like CPU, memory) so one team or application doesn’t consume everything.

### Name Scoping:

 Resource names only need to be unique within a namespace, not across the entire cluster. For example, you can have a nginx deployment in both dev and prod namespaces.

### Access Control:

 You can use RBAC (Role-Based Access Control) to give different permissions for different namespaces.


## Create a namespace:

``` kubectl create namespace namespace-name ```

## Namespace list:
``` kubectl get namespace ```

## Create a deployment inside a namespace:

``` kubectl create deployment deployment-name --image nginx  -n namespace-name ```

## Create a pod inside a namespace:

``` kubectl run pod-name --image nginx -n namespace-name ```

## Check pods inside a namespace :

``` kubectl get pod -n namespace-name ```

## Crete namespace yaml using dry run:

``` kubectl create namespace namespace-name --dry-run=client -o yaml ```

## Namespace yaml:
```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    environment: dev
    team: backend
```

## Apply this yaml file:
``` kubectl apply -f abc.yaml ```

## Delete namespace :
``` kubectl delete namespaces namespace-name ```
