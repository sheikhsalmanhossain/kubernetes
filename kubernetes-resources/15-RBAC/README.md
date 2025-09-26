# RBAC :

RBAC= Role Based Access Control

# Role:

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/7ac328fcc037361b9bd976785f10cb3897cf9e1d/kubernetes-resources/15-RBAC/rbac-role.jpg)

### 1) Role:
 A Role is a set of permissions (rules).

### 2) RoleBinding:
 RoleBinding connects (binds) those permissions to a user.

### 3) User:
 A User is the subject that actually receives the permissions

Role is a namespace based resource. It works inside namespace. We create role inside a namespace. Role provide permission in namespace. Rolebinding create mapping between user and role. Rolebinding is also a namespace based resource.


# Cluster Role :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/7ac328fcc037361b9bd976785f10cb3897cf9e1d/kubernetes-resources/15-RBAC/rbac-cluster-role.jpg)


Cluster-role is a cluster based resource.It works inside cluster.We create cluster-role inside a cluster. Cluster-Role provide permission in cluster. ClusterRolebinding create mapping between user and ClusterRole. ClusterRolebinding is also a cluster based resource.



### 1) ClusterRole:
 A ClusterRole is a set of permissions (rules).

### 2) ClusterRoleBinding:
 A ClusterRoleBinding links a ClusterRole to a User

### 3) User:
 A User is the subject that actually receives the permissions
