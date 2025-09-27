# RBAC :

RBAC= Role Based Access Control

# Namespace-scoped RBAC:

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/7ac328fcc037361b9bd976785f10cb3897cf9e1d/kubernetes-resources/15-RBAC/rbac-role.jpg)

### 1) Role:
 A Role is a set of permissions (rules).

### 2) RoleBinding:
 RoleBinding connects (binds) those permissions to a user.

### 3) User:
 A User is the subject that actually receives the permissions

Role is a namespace based resource. It works inside namespace. We create role inside a namespace. Role provide permission in namespace. Rolebinding create mapping between user and role. Rolebinding is also a namespace based resource.

# Namespace-scoped RBAC:

## Creating user :

Reference: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

[In vscode and terminal in same location]

### steps:
#### generate certificate file :

- create private key and certificate signing request file:
 ``` openssl genrsa -out newuser.key 2048 ```

(Here username is "newuser". In vscode we see that a key is generated.)


(Now, we request for certificate signing)
 ``` openssl req -new -key newuser.key -out newuser.csr -subj "/CN=newuser" ```
(now we see that newuser.csr file generated)

- Create a CertificateSigningRequest (csr.yaml) :
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: newuser
spec:
  request: <csr _______ data>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

- Some points to note:

-- usages has to be 'client auth' -- request is the base64 encoded value of the CSR file content. You can get the content using this command:
``` cat newuser.csr | base64 | tr -d "\n" ```

- Copy the output and replace in "<csr _______ data>"

#### Apply the yaml file :
``` kubectl apply -f csr.yaml ```

#### Check csr(Certificate Signing Request) :

``` kubectl get csr ```

we can see condition is pending. Now we have to approve it.

#### Get the list of CSRs:
``` kubectl get csr ```

#### Approve the CertificateSigningRequest :
``` kubectl certificate approve csr-name ```

Now if we check ``` kubectl get csr `` . We see that csr approved.

#### Get the certificate :
``` kubectl get csr newuser -o jsonpath='{.status.certificate}'| base64 -d > newuser.crt ```

(in this command, we get csr, we take output in jsonpath,filtering certificate, decode this file in base64, put this file in newuser.crt)

Now we get a file newuser.crt in vscode.

User created...


### How we send this other one to access as a user:
- kubectx newuser
- Generate the clean config file, for sharing:
``` kubectl config view --minify --flatten --context=$(kubectl config current-context) > current-context.yaml ```

- we get an output inside vscode "current-context.yaml" file
- This is the credential for new user.
- paste yaml file in this url and share it (https://www.toptal.com/developers/hastebin)
---------------------------------------------------------------------------------

## Create Role :

#### check role :
``` kubectl get role -A ```

#### check rolebinding :
``` kubectl get rolebinding -A ```

#### This is a command to create a Role for this new user:
``` kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods ```

( In this command, role name is developer, can do: create,get,list,update,delete , in which resourse : pod )

#### Check role :
``` kubectl get role ```

#### Describe role :
``` kubectl describe role role-name ```

Our role is created
-------------------------------------------------------------------------------------------
## Create RoleBinding :

#### This is a command to create a RoleBinding for this new user:
``` kubectl create rolebinding developer-binding-myuser --role=developer --user=newuser ```

(In this command, deleloper-binding-myuser= rolebinding name, rolename=developer, username=newuser)

#### Check rolebinding:
``` kubectl get rolebinding ```
#### Describe rolebinding :
``` kubectl describe rolebinding rolebinding-name ```

Our rolebinding is created.
-------------------------------------------------------------------------


``` cat ~/.kube/config ```
Now we have to add newuser here.

### Add to kubeconfig
First, you need to add new credentials:

``` kubectl config set-credentials newuser --client-key=newuser.key --client-certificate=newuser.crt --embed-certs=true ```

(In this command, set credential for new user)

#### see output:
``` cat ~/.kube/config ```

We can see, inside manifest file, newuser created.

#### Then, you need to add the context:
``` kubectl config set-context newuser --cluster=kubernetes --user=newuser ```

#### see output:
``` cat ~/.kube/config ```
(we can see context part added newuser)

#### To test it, change the context to newuser:
``` kubectl config use-context newuser ```
(simililar like "kubectx", switching account to "newuser")

Now we get pod related all permission in newuser account...

Delete role :

``` kubectl delete role <role-name> -n <namespace> ```

Delete role binding :

``` kubectl delete rolebinding <rolebinding-name> -n <namespace> ```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


--------------------------------------------------------------------------------

# Cluster-scoped RBAC :

![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/7ac328fcc037361b9bd976785f10cb3897cf9e1d/kubernetes-resources/15-RBAC/rbac-cluster-role.jpg)


Cluster-role is a cluster based resource.It works inside cluster.We create cluster-role inside a cluster. Cluster-Role provide permission in cluster. ClusterRolebinding create mapping between user and ClusterRole. ClusterRolebinding is also a cluster based resource.



### 1) ClusterRole:
 A ClusterRole is a set of permissions (rules).

### 2) ClusterRoleBinding:
 A ClusterRoleBinding links a ClusterRole to a User

### 3) User:
 A User is the subject that actually receives the permissions
 

