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
Sending build context to Docker daemon  3.584kB
Step 1/8 : FROM golang:1.13 as builder
1.13: Pulling from library/golang
Digest: sha256:de83180c8374e56166542909101c91f7f653edc525f017b2d58f55b33cd28883
Status: Downloaded newer image for golang:1.13
 ---> 6586e3d10e96
Step 2/8 : WORKDIR /app
 ---> Running in 0631240bc2a1
Removing intermediate container 0631240bc2a1
 ---> 5de58ef973a8
Step 3/8 : COPY main.go .
 ---> 77b929434a74
Step 4/8 : RUN CGO_ENABLED=0 GOOS=linux GOPROXY=https://proxy.golang.org go build -o app ./main.go
 ---> Running in 30538b23da01
Removing intermediate container 30538b23da01
 ---> 049e4de58066
Step 5/8 : FROM alpine:latest
latest: Pulling from library/alpine
c9b1b535fdd9: Pull complete
Digest: sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d
Status: Downloaded newer image for alpine:latest
 ---> e7d92cdc71fe
Step 6/8 : WORKDIR /app
 ---> Running in 92b2bfe20d03
Removing intermediate container 92b2bfe20d03
 ---> fdcbc64c7b36
Step 7/8 : COPY --from=builder /app/app .
 ---> 9f74d89189bb
Step 8/8 : CMD ["./app"]
 ---> Running in c7b07079e424
Removing intermediate container c7b07079e424
 ---> 09a63b2f046d
Successfully built 09a63b2f046d
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