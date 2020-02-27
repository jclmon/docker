# APLICACION FUNCIONAL GO

```
\kubernetes\app\k8s-hands-on\backend\src> docker run -p 9090:9090 --rm -dti -v $PWD/:/go --name golang golang bash
1c8f541489467668856f296cf30a4f5d857d209ac4481e0753706d479ff574e4

\kubernetes\app\k8s-hands-on\backend\src> docker exec -ti 1c8f541489467668856f296cf30a4f5d857d209ac4481e0753706d479ff574e4 bash
root@1c8f54148946:/go# go run main.go
```

## Construir el dockerfile

### Crear imagen
```
\kubernetes\app\k8s-hands-on\backend\src> docker build -t k8s-hands-on -f Dockerfile .
...
Successfully tagged k8-hands-on:latest
```

### Correr imagen
```
\kubernetes\app\k8s-hands-on\backend\src> docker run -d -p 9091:9090 --name k8s-hands-on k8s-hands-on:latest
6c925b14a227db2dc5011132c1fb57fbdab2ed25a5de938c2b722ec396372ea6

\kubernetes\app\k8s-hands-on\backend\src> docker ps -l
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
6c925b14a227        k8-hands-on:latest   "./app"             14 seconds ago      Up 11 seconds       0.0.0.0:9091->9090/tcp   k8s-hands-on
```

### Busco el atributo del maniefesto PullPolicy para cambiarlo a ifnotpresent, esto hará que no se descargue
```
\kubernetes\app\k8s-hands-on\backend> kubectl get deployments backend-k8s-hands-on -o yaml | grep -i PullPolicy
-C 12
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: backend
    spec:
      containers:
      - image: k8s-hands-on
        imagePullPolicy: Always
        name: backend
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2020-02-13T13:49:55Z"
```
