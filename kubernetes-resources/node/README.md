![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/fb574f24670ca1b8e772fc50885eb4cb92eeb571/kubernetes-resources/node/node.jpg)

## General Node Management
 ### List all nodes in the cluster:

``` kubectl get nodes ```

### Get detailed information about a specific node:

``` kubectl describe node <node-name> ```

### Display node resources(output) in YAML format:

``` kubectl get node <node-name> -o yaml ```

### Display node extra info:

``` kubectl get node <node-name> -o wide ```

### View resources and statuses of all nodes:

``` kubectl get nodes -o wide ```

### Label a node:

``` kubectl label node <node-name> <label-key>=<label-value> ```

### Remove a label from a node:

``` kubectl label node <node-name> <label-key>- ```
