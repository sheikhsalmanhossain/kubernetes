# Configmap :

In Kubernetes, a ConfigMap is an API object used to store non-confidential configuration data in key-value pairs.

It allows you to decouple configuration from your application container images, so you can modify settings without rebuilding and redeploying the container.

- Use Secrets instead of ConfigMaps if you need to store sensitive data (like passwords, API keys).

- ConfigMaps are stored in etcd, so avoid putting large files inside them.
configmap is a namespace specific resource.


## Configmap:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: name-config
  namespace: default    #### if i do not declare it will choose default one
data:
  name: "test-name-111"
```

## Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cm-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cm-app
  template:
    metadata:
      labels:
        app: cm-app
    spec:
      containers:
      - image: alpine
        name: cm-container
        command: ['sh', '-c', 'while true; do echo $NAME; sleep 1; done']       #### 'sh'=shell, '-c'=command, NAME=declare env, which call configmap[Run a shell (sh -c) that loops forever, printing the value of $NAME once per second.]
        env:
        - name: NAME               #### this is environment name, which we should declare in command
          valueFrom:
            configMapKeyRef:
              name: name-config    #### same as configmap name
              key: name            #### from configmap data
```

### Apply yaml file:
``` kubectl apply -f configmap.yaml ```
``` kubectl apply -f deployment.yaml ```

### Check pods :
``` kubectl get pod ```

### check deployment logs :

``` kubectl logs deployment-name -f ```   (-f= follow, similar like "kubectl watch")


![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/5f8da33689153415fcb08d0723e120ab3827fe56/kubernetes-resources/7-configmap/configmap.jpg)


In time of pod creation, pod collect configmap-data from configmap and run. Now if i change something inside configmap and apply it, pod will not be aware of this change. So pod still running with previous configmap.
So we have to restart our pods that pods again collect data from configmap.

### Restart deployment :

``` kubectl rollout restart deployment depkoyment-name ```

(so pods get deleted and create new pods)

### check pods :
``` kubectl get pods ```
### check pod logs :
``` kubectl logs pod-name -f ```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Lets try another example :

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  player_initial_lives: "3"
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    


---
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ['sh', '-c', 'while true; do echo player_initial_lives: $PLAYER_INITIAL_LIVES; echo game_properties: $GAME_PROPERTIES; sleep 1; done']
      env:
        - name: PLAYER_INITIAL_LIVES 
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: player_initial_lives

        - name: GAME_PROPERTIES
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: game.properties
```

In pod.yaml file inside env we call configmap data. Then we pass env names inside command as veriable.

### Apply whole yaml file together:
``` kubectl apply -f cm.yaml ```


### Now watchpod logs :
``` kubectl logs pod-name -f ```
