# Secrets :

In Kubernetes, Secrets are special objects that store sensitive data (like passwords, API keys, tokens, or certificates) so you don’t have to put them directly inside Pod definitions or ConfigMaps.
#### secrets is a namespace specific resource.

 - Purpose: Protect confidential information.

 - Encoding: Stored as Base64 (not strong encryption, but avoids plain text)

## different types of Secrets:

1) Use "Opaque" for normal sensitive data.

2) Use "dockerconfigjson" for private registries.

3) Use "tls" for certificates.

4) "Service account tokens" & "bootstrap tokens" are system-related.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Secrets :
```
apiVersion: v1
kind: Secret
metadata:
  name: name-secret
type: Opaque
data:
  name: c2FsbWFu        #### Base64 encodsc value of "salman"
```


### encode the secrets (on Linux) :
``` echo -n "salman" | base64 ```

### decode the secret :
``` echo "c2FsbWFu" | base64 -d ```


## Deployment :
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sc-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sc-app
  template:
    metadata:
      labels:
        app: sc-app
    spec:
      containers:
      - image: alpine
        name: secret-container
        command: ['sh', '-c', 'while true; do echo $NAME; sleep 1; done']
        env:
        - name: NAME
          valueFrom:
            secretKeyRef:
              name: name-secret      #### name from secrets name
              key: name              #### key value from secrets data value
```



### Apply yaml files:
``` kubectl apply -f deployment.yaml ```
``` kubectl apply -f secrets.yaml ```

### Get secrets :
``` kubectl get secrets ```

### Describe secret:
``` kubectl describe secret secret-name ```

### Watch pod logs :
``` kubectl logs pod-name -f ```
