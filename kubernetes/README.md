# MINIKUBE

## Habilitar Hyper-V en windows 10
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
## Arrancar minikube

Debe estar creado con el Administrador de Hyper-V el conmutador virtual VM-External-Switch como external a la red del equipo (eth0 o wlan)
```
minikube start --vm-driver "hyperv" --hyperv-virtual-switch "VM-External-Switch"
```
### Si hubiese algún problema:
* Minikube delete
```
minikube delete
```
* Borrar carpetas de configuración
```
C:\Users\usuario\.kube
C:\Users\usuario\.minikube
```
Volver a arrancar minikube

## Asignar el environment de docker a minikube
```
> minikube docker-env | Invoke-Expression
```
## Parar minikube
```
> minikube ssh
$sudo poweroff
```

# KUBERNETES

# PODS

### Crear un pod
```
kubectl run --generator=run-pod/v1 podtest --image=nginx:alpine
```
### Listar pods
```
kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
podtest   1/1     Running   0          23s
```
READY 1 contenedor de 1 contenedor esperado, esta la estrategia más utilizada un contenedor por pod
### Ver logs del pod
```
kubectl describe pod podtest
Name:         podtest
Namespace:    default
Priority:     0
Node:         minikube/172.17.107.109
Start Time:   Tue, 11 Feb 2020 16:08:07 +0100
Labels:       run=podtest
Annotations:  <none>
Status:       Running
IP:           172.18.0.4
IPs:
  IP:  172.18.0.4
Containers:
  podtest: ...
```
### Eliminar pod
```
kubectl delete pod nombre
```
### Información del pod con salida yaml
```
kubectl get pod podtest -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-11T15:08:06Z"
  labels:
    run: podtest
  name: podtest
  namespace: default
  resourceVersion: "6180"
  selfLink: /api/v1/namespaces/default/pods/podtest
  uid: 2d730bf6-f22c-4be9-8a66-3c75207652c4
```
### Ver los logs
```
kubectl logs podtest
172.18.0.1 - - [11/Feb/2020:15:43:10 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.66.0" "-"
172.18.0.1 - - [11/Feb/2020:15:44:15 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.66.0" "-"
172.18.0.1 - - [11/Feb/2020:15:44:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.66.0" "-"
172.18.0.1 - - [11/Feb/2020:15:47:40 +0000] "GET / HTTP/1.1" 200 3 "-" "curl/7.66.0" "-"
```
### Ver los contenedores
```
$ docker ps -f name=podtest
```
### Ejecutar pod y entrar al ssh
```
$ kubectl exec -ti podtest -- sh
/ #
```
### Para entrar a la imagen de minikube
```
$ minikube ssh
```
### Versiones del api para manifiestos yaml
```
kubectl api-versions
```
### Recursos para manifiestos yaml, en este caso relacionados con pods
```
kubectl api-resources | grep Pod
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
```
### Aplicar manifiesto yaml para crear objeto
```
kubectl apply -f pod.yaml
pod/podtest2 created
```
### Eliminar en base a manifiesto
```
kubectl delete -f pod.yaml
```
### Lista de pods, informa IP y STATUS
```
kubectl get pods -o=custom-columns=NAME:.metadata.name,IP:.status.podIP
```
### Logs de un contenedor
```
kubectl logs doscont -c cont1
```
### Comprobar contenedor
```
kubectl exec -ti doscont -- sh
```

## Labels
### Labels metadata en manifiestos
```
kubectl get pods -l app=backend
kubectl get pods -l app=frontend
kubectl get pods -l env=dev
```

# REPLICASETS

Está a un nivel más alto del Pod, se le puede indicar cuantas replicas del Pod se necesitan. 
Los Pods tienen que estar etiquetados, para que el replicaset sepa los pods que tiene que mantener replicados.
```
kubectl get rs rs-test -o yaml
kubectl run --generator=run-pod/v1 podtest1 --image=nginx:alpine
kubectl run --generator=run-pod/v1 podtest2 --image=nginx:alpine
```
Incluir label al pod que esta corriendo (kubectl label pods)
```
kubectl label pods podtest1 app=pop-label
```

# DEPLOYMENTS

La gestión de los replicaset para actualizaciones de versión directa, los deployments son un objeto de alto nivel 
que se hace cargo de los replicaset. 

* Maxv porcentaje de replicaset que pueden caer (disponibilidad)
* Maxs porcentaje de escalado

Los replicasets se quedan en el historico despues de la actualización, por defecto máximo 10 revisionHistoryLimit

```
> kubectl get rs -l app=front
NAME                         DESIRED   CURRENT   READY   AGE
deployment-test-84467d95b6   0         0         0       5m28s
deployment-test-86677fd487   3         3         3       22m
```

## Utilizar CHANGE-CAUSE

```
> kubectl apply -f .\dev.yaml --record
deployment.apps/deployment-test configured

> kubectl rollout history deployment deployment-test
deployment.apps/deployment-test
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl.exe apply --filename=.\dev.yaml --record=true
```
Si incluimos metadata:   
annotations:
    kubernetes.io/change-cause: "Cambio de puerto 90"
```
> kubectl rollout history deployment deployment-test
deployment.apps/deployment-test
REVISION  CHANGE-CAUSE
2         kubectl.exe apply --filename=.\dev.yaml --record=true
3         Cambio de puerto 90
```
Para ver los cambios
```
> kubectl rollout history deployment deployment-test --revision=3
deployment.apps/deployment-test with revision #3
Pod Template:
  Labels:       app=front
        pod-template-hash=7995b9c4f7
  Annotations:  kubernetes.io/change-cause: Cambio de puerto 90
  Containers:
   nginx:
    Image:      nginx:alpine
    Port:       90/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
  
```
## Utilizar ROLLBACK para marcha atrás de despliegue
```
> kubectl rollout history deployment deployment-test
deployment.apps/deployment-test
REVISION  CHANGE-CAUSE
2         kubectl.exe apply --filename=.\dev.yaml --record=true
3         Cambio de puerto 90
4         Cambio de puerto 90
5         Despliegue mal configurado por imagen

> kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
deployment-test-7995b9c4f7-4lzsf   1/1     Running             0          6m54s
deployment-test-7995b9c4f7-j2grb   1/1     Running             0          7m3s
deployment-test-7995b9c4f7-jdjl5   1/1     Running             0          7m10s
deployment-test-ccf97ff6c-nbmqr    0/1     ContainerCreating   0          8s

> kubectl rollout undo deployment deployment-test --to-revision=3
deployment.apps/deployment-test rolled back

> kubectl rollout status deployment deployment-test
deployment "deployment-test" successfully rolled out

> kubectl rollout history deployment deployment-test
deployment.apps/deployment-test
REVISION  CHANGE-CAUSE
2         kubectl.exe apply --filename=.\dev.yaml --record=true
4         Cambio de puerto 90
5         Despliegue mal configurado por imagen
6         Cambio de puerto 90
```

# SERVICES
Es un objeto aislado que observa los pods con un label configurado, el servicio mantendrá una IP única para realizar entre otras cosas consulta de estado a los pods. Además el servicio hace balanceo de 
carga realizando peticiones en principio con aplicando round-robin. Esto lo hará independientemente de que el pod esté incluido en un replicaset.

Lo realiza independientemente de que los pods caigan y se levanten con la probabilidad de que cambie la IP de este. Los servicios realizan esto utilizando ENDPOINTS ya que mapea las IPS con los labels.

Los tipos de servicio son: ClusterIP, NodePort y LoadBalancer

```
> kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP    21h
my-service   ClusterIP   10.98.183.0   <none>        8080/TCP   5m49s
```

```
> kubectl describe svc my-service
Name:              my-service
Namespace:         default
Labels:            app=front
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"front"},"name":"my-service","namespace":"default"},"spec...
Selector:          app=front
Type:              ClusterIP
IP:                10.98.183.0
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         172.18.0.4:80,172.18.0.7:80,172.18.0.8:80
Session Affinity:  None
Events:            <none>
```

## Ver endpoints
```
> kubectl get endpoints
NAME         ENDPOINTS                                   AGE
kubernetes   172.17.107.109:8443                         21h
my-service   172.18.0.4:80,172.18.0.7:80,172.18.0.8:80   11m
```

```
> kubectl get pods -l app=front -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
deployment-test-86677fd487-8ms9j   1/1     Running   0          11m   172.18.0.8   minikube   <none>           <none> 
deployment-test-86677fd487-9zbxt   1/1     Running   0          11m   172.18.0.4   minikube   <none>           <none> 
deployment-test-86677fd487-bwxgx   1/1     Running   0          11m   172.18.0.7   minikube   <none>           <none>
```

### Buscar propiedades del deployment
```
> kubectl get deployments backend-k8s-hands-on -o yaml | grep -i PullPolicy
```

### Imagen temporal cuando se sale del ssh se elimina, validación de otros pods
```
> kubectl run --rm -ti --generator=run-pod/v1 podtest3 --image=nginx:alpine -- sh
```

# NAMESPACES

```
> kubectl get namespaces
NAME              STATUS   AGE
default           Active   39h
kube-node-lease   Active   39h
kube-public       Active   39h
kube-system       Active   39h
```

Si no especifico namespace los pods van al namespace default, kube-system es el namespace de kubernetes no tocar.

```
> kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
pod/coredns-6955765f44-pfbbz           1/1     Running   0          39h
...
```

* Crear namespace

```
> kubectl create namespace test-ns
namespace/test-ns created
> kubectl get namespaces
NAME              STATUS   AGE
default           Active   39h
kube-node-lease   Active   39h
kube-public       Active   39h
kube-system       Active   39h
test-ns           Active   4s
> kubectl get namespaces --show-labels
NAME              STATUS   AGE   LABELS
default           Active   39h   <none>
kube-node-lease   Active   39h   <none>
kube-public       Active   39h   <none>
kube-system       Active   39h   <none>
test-ns           Active   22s   <none>
> kubectl describe namespaces test-ns
Name:         test-ns
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

## DNS
Full qualified domain name 
```
nombreServicio + nombreNameSpace + svc.cluster.local (dns del cluster)
Se podra acceder desde cualquier namespace
curl backend-k8s-hands-on.ci.svc.cluster.local
```
### Contextos y namespaces
Da un contexto para las operaciones, de esta forma no es necesario especificar el namespace
```
\kubernetes\namespaces> kubectl config current-context
minikube
```
El contexto esta guardado en la configuración de kubernetes fichero config .kube\config
```
\kubernetes\namespaces> kubectl config view
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: ""
...
```
* Para crear un nuevo contexto y poder cambiar de contexto
```
\kubernetes\namespaces> kubectl config set-context ci-context --namespace=ci --cluster=minikube --user=minikube
Context "ci-context" created.

\kubernetes\namespaces> kubectl config view
apiVersion: v1
clusters:
...
\kubernetes\namespaces> kubectl config use-context ci-context
Switched to context "ci-context".
\kubernetes\namespaces> kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
backend-k8s-hands-on-74b989d848-889fp   1/1     Running   0          13m
backend-k8s-hands-on-74b989d848-lr9sj   1/1     Running   0          13m
backend-k8s-hands-on-74b989d848-vjxsd   1/1     Running   0          13m
\kubernetes\namespaces> kubectl get svc
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
backend-k8s-hands-on   ClusterIP   10.100.215.85   <none>        80/TCP    13m
```
* Creamos un pod que supera el limite de memoria
```
\kubernetes\limits\request> kubectl apply -f .\limits-ram2.yaml
pod/memory-demo created
\kubernetes\limits\request> kubectl get pods --watch
NAME          READY   STATUS      RESTARTS   AGE
memory-demo   0/1     OOMKilled   0          11s
memory-demo   0/1     OOMKilled   1          15s
memory-demo   0/1     CrashLoopBackOff   1          16s
memory-demo   0/1     OOMKilled          2          33s
memory-demo   0/1     CrashLoopBackOff   2          34s
memory-demo   0/1     OOMKilled          3          68s
memory-demo   0/1     CrashLoopBackOff   3          69s

\kubernetes\limits\request> kubectl describe pod memory-demo
```
* Para ver las estadisticas de uso ram y cpu
```
\kubernetes\limits\request> kubectl describe node minikube
```

## QoS
Si coincide en limits y requests está garantizado, Guaranteed
Si puede subir porque limits es mayor a request, Burstable
Cuando no tiene límites, los más peligrosos ya que puede colapsar un nodo, BestEffort

### LimitRange
Para namespaces se pueden dar límites por defecto 
```
\kubernetes\limits\limits-range-namespaces> kubectl get limitrange -n dev
NAME                  CREATED AT
mem-cpu-limit-range   2020-02-14T09:28:21Z
\kubernetes\limits\limits-range-namespaces> kubectl describe limitrange -n dev
Name:       mem-cpu-limit-range
Namespace:  dev
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Container   cpu       -    -    500m             1              -
Container   memory    -    -    256Mi            512Mi          -

\kubernetes\limits\limits-range-namespaces> kubectl describe limitranges -n prod
Name:       mem-min-max-demo-lr
Namespace:  prod
Type        Resource  Min   Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---  ---------------  -------------  -----------------------
Container   cpu       100m  1    1                1              -
Container   memory    100M  1Gi  1Gi              1Gi            -
```
### ResourceQuota
LimitRange limita los recursos para el namespace, es útil para limitar recursos individualmente. Los Pods deberán de cumplir con el LimitRange.
ResourceQuota no actúa a nivel de objeto lo hace a nivel de namespace. Limita mediante el sumatorio de los recursos individuales. Cuando se cumple la cuota no permite crear el Pod.
Se suelen utilizar conjuntamente, a nivel de objeto y a nivel de namespace independientemente del número de objetos.
```
\kubernetes\resourcequota> kubectl apply -f .\res-quota.yml
namespace/uat created
resourcequota/mem-cpu-demo created
\kubernetes\resourcequota> kubectl describe resourcequotas -n uat
...
Resource Quotas
 Name:     pod-demo
 Resource  Used  Hard
 --------  ---   ---
 pods      3     3
No LimitRange resource.
```

# PROBES
Pruebas de diagnóstico sobre los pods
Kubelet es quien sondea con probes a los pods, esto lo hace:
1.	Comando: se ejecuta un comando si devuelve 0 está OK si no no
2.	TCP: testeo de puerto, si responde OK si no no
3.	HTTP: hace un get sobre /path y si la respuesta es entre 200 y 399 es OK si no no
Tipos, cualquiera de ellos se ejecuta a través de Comando, TCP o HTTP: 

* Liveness 
Diagnóstico para ver si la aplicación funciona como debería, se ejecuta cada n intervalo de tiempo.

* Readiness 
La aplicación se inició como debería, si levanto un pod hasta que no esté listo no se utiliza como 
endpoint no se incorporará al servicio. Si el puerto no está abierto desregistra la IP de los endpoints de servicio para que el pod no reciba más carga hasta que no esté en un estado correcto.

* Startup
Aplicaciones que se retrasan en el inicio, se ejecuta el primero Liveness y Readiness tienen que 
esperarlo se pausan. Esperarán hasta que estemos seguros de que la aplicación está lista.

## Liveness
```
\kubernetes\probes> kubectl apply -f .\liveness.yml
\kubernetes\probes> kubectl describe pod liveness
...
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  <unknown>            default-scheduler  Successfully assigned default/liveness-exec to minikube
  Warning  Unhealthy  50s (x6 over 2m20s)  kubelet, minikube  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    50s (x2 over 2m10s)  kubelet, minikube  Container liveness failed liveness probe, will be restarted
  Normal   Pulling    19s (x3 over 3m)     kubelet, minikube  Pulling image "k8s.gcr.io/busybox"
  Normal   Pulled     17s (x3 over 2m55s)  kubelet, minikube  Successfully pulled image "k8s.gcr.io/busybox"
  Normal   Created    16s (x3 over 2m54s)  kubelet, minikube  Created container liveness
  Normal   Started    15s (x3 over 2m53s)  kubelet, minikube  Started container liveness

\kubernetes\probes> kubectl apply -f .\liveness-tcp.yml  

\kubernetes\probes> kubectl describe pod goproxy
Name:         goproxy
...
\kubernetes\probes> kubectl get pods liveness-http -o yaml | grep -i liv -A12
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"liveness"},"name":"liveness-http","namespace":"default"},"spec":{"containers":[{"args":["/server"],"image":"k8s.gcr.io/liveness","livenessProbe":{"httpGet":{"httpHeaders":[{"name":"Custom-Header","value":"Awesome"}],"path":"/healthz","port":8080},"initialDelaySeconds":3,"periodSeconds":3},"name":"liveness"}]}}
  creationTimestamp: "2020-02-24T15:44:23Z"
  labels:
    test: liveness
```

# CONFIGMAP
Maneja el versionado de dockerfiles, separa las configuraciones para hacer más portable los Pods, la configuración no se hace hardcoded en el Pod si no que se hace en el ConfigMap.
Los configmaps están compuestos por clave, valor. P.e. Nginx:port, 80
Ng.conf , para que el manifest de un Pod vea el configmap se pueden utilizar variables de entorno.

* Configuración de nginx
```
\kubernetes\envs> kubectl run --rm -ti --generator=run-pod/v1 podtest3 --image=nginx:alpine -- sh
If you don't see a command prompt, try pressing enter.
/ #
/ #
/ # cat etc/nginx/conf.d/default.conf
server {
    listen       80;
...
\kubernetes\configmap> kubectl create configmap nginx-config --from-file .\configmap-samples\nginx.conf
configmap/nginx-config created
```
* Tambien se puede crear desde una carpeta
```
\kubernetes\configmap> kubectl create configmap nginx-config1 --from-file .\configmap-samples
configmap/nginx-config1 created
\kubernetes\configmap> kubectl get cm
NAME           DATA   AGE
nginx-config   1      27s
\kubernetes\configmap> kubectl describe cm nginx-config
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>
Data
...
\kubernetes\configmap> kubectl apply -f .\cm-nginx-vol.yaml
\kubernetes\configmap> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-7f57955494-sktn6   1/1     Running   0          114s
\kubernetes\configmap> kubectl exec -ti deployment-test-7f57955494-sktn6 -- sh
/ # cat /etc/nginx/conf.d/default.conf
server {
    listen       9090;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

\kubernetes\configmap> kubectl apply -f .\cm-nginx-env.yaml
configmap/nginx-config configured
configmap/vars unchanged
deployment.apps/deployment-test created
\kubernetes\configmap> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-69f4684694-ht9k4   1/1     Running   0          42m
```

# SECRETS
Es un objeto parecido al configmap, el secret guarda datos sensibles. Los configmaps guardan datos no sensibles.
```
\kubernetes\secrets\secrets-files> kubectl create secret generic mysecret --from-file=.\test.txt
secret/mysecret created
\kubernetes\secrets\secrets-files> kubectl apply -f .\secret-data.yaml
secret/mysecret created
\kubernetes\secrets\secrets-files> kubectl get secrets mysecret -o yaml
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
...
\kubernetes\secrets\secrets-files> echo MWYyZDFlMmU2N2Rm | base64 --decode
1f2d1e2e67df/usr/bin/base64: invalid input
```
a través del comando envsubst se sustutiye los valores $USER y $PASS por los valores de variables de entorno
```
\kubernetes\secrets\secrets-files> set PASSWORD=pass
\kubernetes\secrets\secrets-files> echo $PASSWORD
\kubernetes\secrets\secrets-files> envsubst < secure.yaml > tmp.yaml
```

# VOLUMES
Se utiliza para guardar el estado en las imágenes de docker, igualmente lo hace kubernetes. Con esto se hacen las aplicaciones STATEFULL

## EmptyDir
Cuando kubernetes crea un contenedor nuevo en un Pod porque el anterior ha caído, el directorio se replica como directorio vacio. El nuevo contenedor monta el mismo directorio.
La carpeta /foo existe dentro del Pod, si el Pod muere si se pierde por lo que está asociado al lifecycle del Pod.

## HostPath
Para no peder la carpeta montada cuando se recrea el Pod se mantiene a nivel del Nodo. No suele ser una opción para producción porque los distintos nodos no van a compartir el volumen.


# CLOUDVOLS
El tipo de disco en AWS es EBS (Elastic Block Store) en Google Cloud PS (Persisted disk)
Estos volúmenes se asocian a las instancias. Se gestionan de manera independiente.
Los VolMounts se asocian a estos volúmenes, hay que especificar el Id del volumen, tipo del sistema de archivos etc. Por esto existe PVC y PV

PV (PersistentVolume) Guarda: Nombre, Tamaño, Aprovisionador (AWS, Google Cloud) y pide estas características al aprovisionador.
PVC (PersistentVolumeClaim), reclamación de volumen persistente, lo que reclama es un PV al recurso Cloud. Reclamación directa al espacio del PV que solicita a su vez el espacio en el Cloud. Utiliza selector a través de labels para identificar los PV.

## PV
```
\kubernetes\volumes> kubectl apply -f .\pv-pvc.yaml
persistentvolume/task-pv-volume created
\kubernetes\volumes> kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Available           manual                  6s

\kubernetes\volumes> kubectl get pv --show-labels
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE   LABELS
task-pv-volume   1Gi        RWO            Retain           Available           manual                  46s   type=local
\kubernetes\volumes> kubectl describe pv task-pv-volume
Name:            task-pv-volume
Labels:          type=local
Annotations:     kubectl.kubernetes.io/last-applied-configuration:
                   {"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"labels":{"type":"local"},"name":"task-pv-volume"},"spec":{"acce...
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /test
    HostPathType:
Events:            <none>
```
Unimos PVC a PV
```
\kubernetes\volumes> kubectl apply -f .\pvc-pv-claim.yaml
persistentvolume/task-pv-volume unchanged
persistentvolumeclaim/task-pv-claim created
\kubernetes\volumes> kubectl get pvc
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound    task-pv-volume   1Gi        RWO            manual         7s
\kubernetes\volumes> kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Bound    default/task-pv-claim   manual                  5m35s
```
Unimos PVC a PV con selector
```
\kubernetes\volumes> kubectl apply -f .\pvc-pv-claim-selector.yaml
persistentvolume/task-pv-volume created
persistentvolume/task-pv-volume2 created
persistentvolumeclaim/task-pv-claim2 created
\kubernetes\volumes> kubectl get pv
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                    STORAGECLASS   REASON   AGE
task-pv-volume    10Gi       RWO            Retain           Available                            manual                  12s
task-pv-volume2   10Gi       RWO            Retain           Bound       default/task-pv-claim2   manual                  12s
\kubernetes\volumes> kubectl get pv --show-labels
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                    STORAGECLASS   REASON   AGE   LABELS
task-pv-volume    10Gi       RWO            Retain           Available                            manual                  27s   type=local
task-pv-volume2   10Gi       RWO            Retain           Bound       default/task-pv-claim2   manual                  27s   mysql=ready1
```
