# CronJob :
Cronjob is a namespace specific resource.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service    #### if we do not mention service type, it will choose clusterip automatically
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
```

### Apply yaml file :
``` kubectl apply -f deployment.yaml ```

## cronjob yaml file :

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: myjob
spec:
  schedule: "* * * * *"     #### job will run in every minitue
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello     #### container name
            image: nginx    #### cronjob pod image
            command:
            - /bin/sh       #### this command will run inside cronjob pod
            - -c
            - curl myapp
          restartPolicy: OnFailure     #### if it fails, it will restart again
```
Schedule : to setting up schedule you can use this site (https://crontab.guru/ )

In every minitue cron job will run a pod, execute command then completed.
our task is, we are doing curl on myapp. "myapp" is a clusterIP service in deployment service. 
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/3dd78517a1f725693e36872f8e6405d6dfd27e6d/kubernetes-resources/9-cronjob%26jobs/cronjob/cronjob.jpg)

### Apply yaml file :
``` kubectl apply -f cronjob.yaml ```

### Check cronjob :
``` kubectl get cronjob ```

### keep watch on cronjob :
``` watch kubectl get cronjob ```

### Watch deployment log :
``` kubectl logs deployments/deployment-name -f ```
---------------------------------------------------------------------------------------------------------------

## Another example :


### MySQL.yaml :

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
        - name: MYSQL_ROOT_PASSWORD   #### use these env as secret.yaml
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

### watch pods live :
``` watch kubectl get pod ```

### check pod details :
``` kubectl describe pod pod-name ```

### go inside pod :
``` kubectl exec -it pod-name -- bash ```

### Connect with mysql :
``` mysql -u(user-name) -p(password) ```

### show database :
`` show databases ```

### create database :
``` create database database-name ```

Suppose databe is hudge , now we have to take backup our database.

So here we use cronjob to keep our data backup automatically after few times.

## databackup cronjob :

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup-cronjob
spec:
  schedule: "* * * * *"      #### Runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: mysql-backup
            image: mysql:8.0
            command:
            - "/bin/sh"
            - "-c"
            - >
              mysqldump -h mysql-service -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE > /backup/db-backup-$(date +\%Y\%m\%d\%H\%M\%S).sql;
            env:
            - name: MYSQL_USER
              value: "root"
            - name: MYSQL_PASSWORD
              value: "rootpassword"
            - name: MYSQL_DATABASE
              value: "exampledb"
            volumeMounts:
            - name: backup-volume    #### name from volumes name
              mountPath: /backup     #### use this path in command
          restartPolicy: OnFailure
          volumes:
          - name: backup-volume
            hostPath:
              path: /data/mysql-backups  #### this path will inside /backup
              type: DirectoryOrCreate    #### if this directory does not exist , it will create
```

In command we use mysql service(clusterIP service) to transfer our backup data to another directory/pod .And file name will be date. for this we use volume(hostpath volume). This volume create a directory inside node but out of this pod.
Node will sync backup data from pod. So further if pod get deleted data will persist. In worker node data will persist in "/data/mysql-backups" this path.
[
### take backup manually :
``` mysqldumb -u(username) -p(password) database-name ```

### keep inside file :
``` mysqldumb -u(username) -p(password) database-name > test.sql ```

### watch backupfile :
``` cat test.sql ```
]

### Apply yaml file :

``` kubectl apply -f cronjob.yaml ```

### get pod and cronjob :
``` kubectl get pod,cronjob ```

Now we find our backup data inside node "/data/mysql-backups" this path.

## How cronjob take and store backups :
![Image Alt](https://github.com/sheikhsalmanhossain/kubernetes/blob/3dd78517a1f725693e36872f8e6405d6dfd27e6d/kubernetes-resources/9-cronjob%26jobs/cronjob/cronjobBackup.jpg)


### How cronjob take and store backups :
Cronjob running in cronjob pod. Then it takes backup from mysql pod and keep it inside cronjob pod. Then we mount our volume (/backup). And send this(/backup) to hostpath. After sending this to hostpath, cronjob pod will get stop.


Now we ssh our worker node and go "/data/mysql-backups" path & we can see our backups.
So if someone delete mysql, still we have backups.
