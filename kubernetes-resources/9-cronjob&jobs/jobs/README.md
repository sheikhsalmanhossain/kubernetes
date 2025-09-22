# Job :
Job is a namespace specific resource.
Job will apply only one time. It does not contain any schedule. 

```
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
  namespace: default
spec:
  template:
    spec:
      containers:
      - name: testjob
        image: mysql:8.0
        command:
        - "/bin/sh"
        - "-c"
        - >
          mysqldump -h mysql-service -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE;
        env:
        - name: MYSQL_USER
          value: "root"
        - name: MYSQL_PASSWORD
          value: "rootpassword"
        - name: MYSQL_DATABASE
          value: "exampledb"

      restartPolicy: Never
```

### mysql.yaml :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        - name: MYSQL_DATABASE
          value: "exampledb"
        - name: MYSQL_USER
          value: "exampleuser"
        - name: MYSQL_PASSWORD
          value: "examplepassword"
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
  type: ClusterIP
```

### Apply yaml file :
``` kubectl apply -f mysql.yaml ```
``` kubectl apply -f job.yaml ```

### see pod, job :
``` kubectl get pod,job ```

After applying yaml file pod and job will create. Pod will execute command. After executing command pod will go completed status. This is our job lifecycle. It is used for update patches in our applications.
