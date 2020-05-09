# ISTIO

•	Los Microservicios añaden complejidad en la red

•	Aparecen crosscutting concerns

•	Se implementan en código alejándonos de la lógica de negocio

•	Los Service Meshes solucionan el problema

•	Istio el service mesh del futuro

# INSTALACION

## Arrancar minikube
```
minikube start --cpus 2 --memory 8192
```
## Instalar Istio
 ```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.5.2/
istioctl manifest apply --set profile=demo
kubectl label namespace default istio-injection=enabled
```
## Comprobar instalación
```
kubectl get namespaces | grep istio
```

# TUTORIAL

Desplegar nuestra aplicación basada en microservicios a Kubernetes dentro de la service mesh. Además se explica cómo exponer un servicio.

## Crear namespace tutorial
```
kubectl create namespace tutorial
```
## Hacer el clone del proyecto
```
git clone https://github.com/jclmon/istio-tutorial
cd istio-tutorial/customer/java/springboot/
```
## Empaquetado del proyecto
```
export JAVA_TOOL_OPTIONS="-Dhttps.protocols=TLSv1.2"
mvn clean package
```
## Asociar a istio el entorno de docker
```
eval $(minikube -p istio docker-env)
```
## Construcción del proyecto
```
docker build -f Dockerfile -t example/customer .
istioctl kube-inject -f ../../kubernetes/Deployment.yml | kubectl apply -n tutorial -f -
istioctl kube-inject -f ../../kubernetes/Service.yml | kubectl apply -n tutorial -f -

cd ../../../preference/java/springboot/
mvn clean package
docker build -f Dockerfile -t example/preference:v1 .
istioctl kube-inject -f ../../kubernetes/Deployment.yml | kubectl apply -n tutorial -f -
istioctl kube-inject -f ../../kubernetes/Service.yml | kubectl apply -n tutorial -f -

cd ../../../recommendation/java/vertx/
mvn clean package
docker build -f Dockerfile -t example/recommendation:v1 .
istioctl kube-inject -f ../../kubernetes/Deployment.yml | kubectl apply -n tutorial -f -
istioctl kube-inject -f ../../kubernetes/Service.yml | kubectl apply -n tutorial -f -

nano .\src\main\java\com\redhat\developer\demos\recommendation\RecommendationVerticle.java
mvn clean package
docker build -f Dockerfile -t example/recommendation:v2 .
istioctl kube-inject -f ../../kubernetes/Deployment-v2.yml | kubectl apply -n tutorial -f -
```

## Exponer los servicios, redirección de puertos
```
kubectl port-forward $(kubectl get pod -n tutorial | grep "customer" | awk '{ print $1 }') -n tutorial 8080:8080
```
o también
```
$ kubectl get pods -n tutorial
NAME                                 READY   STATUS    RESTARTS   AGE
customer-55c8cc5979-fnkg5            2/2     Running   1          22m
preference-v1-55b4d789d7-nldc6       2/2     Running   0          14m
recommendation-v1-8467dd859c-8696x   2/2     Running   0          13m
recommendation-v2-5b956bd67f-r9dvg   2/2     Running   0          5m31s
$ kubectl port-forward customer-55c8cc5979-fnkg5 8080:8080
```

# BLUEGREEN DEPLOYMENT

Fichero Istio para despliegue de dos versiones como DestinationRule destination-rule-recommendation-v1-v2.yml
Ahora lo que se requiere es que todo el tráfico se redirija a la versión 2.

Para ello se crea el fichero:
``` 
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: recommendation
spec:
  host: recommendation
  subsets:
  - labels:
      version: v1
    name: version-v1
  - labels:
      version: v2
    name: version-v2
---
```
Creación:
```
istioctl create –f istiofiles/destination-rule-recom-v1-v2.yml –n tutorial
istioctl create –f istiofiles/virtual-service-recomendation-v2.yml
istioctl get virtualservices –n tutorial
isitioctl get destinationrules –n tutorial
```
Vemos que las invocaciones las redirige a la versión 2
```
curl customer-tutorial.$(minishift ip).nip.io
```
Creamos la imagen
```
cd istio-tutorial/istiofiles/
kubectl create -f .\destination-rule-recommendation-v1-v2.yml
destinationrule.networking.istio.io/recommendation created
```
Compruebo que se han creado
```
kubectl get destinationrules -n tutorial
NAME             HOST             AGE
recommendation   recommendation   6s
```
Para volver a la versión 1 creo virtual-service-recommendation-v1.yml:
Creo el fichero:
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
spec:
  hosts:
  - recommendation
  http:
  - route:
    - destination:
        host: recommendation
        subset: version-v1
      weight: 100
---
```
Reemplazo:
```
isitioctl replace –f istiofiles/virtual-service-recommendation-v1.yml -n tutorial
```

# CANARY RELEASE
No manda el 100% del tráfico a una versión o a otra, el cambio se va haciendo incrementalmente desde la versión 1 a la versión 2
Creo el fichero para un 50 - 50
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
spec:
  hosts:
  - recommendation
  http:
  - route:
    - destination:
        host: recommendation
        subset: version-v1
      weight: 50
    - destination:
        host: recommendation
        subset: version-v2
      weight: 50
---
```
Creamos la imagen:
``` 
istioctl create –f istiofiles/virtual-service-recommendation-v1_and_v2_50_50.yml
istioctl get virtualservices –n tutorial
```

# CONTROL DE TRAFICO POR CONTENIDO
Redirecciona el tráfico según el contenido de los paquetes y no por porcentaje. El contenido estará contenido en el header de HTTP
En el ejemplo user agent que identifica al navegador para safari pasa por la versión 2.
Creo el fichero:
```  
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
spec:
  hosts:
  - recommendation
  http:
  - match:
    - headers:
        baggage-user-agent:
          regex: .*Safari.*
    route:
    - destination:
        host: recommendation
        subset: version-v2
  - route:
    - destination:
        host: recommendation
        subset: version-v1
---
```
Creamos la imagen:
```  
istioctl replace –f istiofiles/virtual-service-safari-recommendation-v2.yml
curl -A Safari customer-tutorial.$(minishift ip).nip.io
curl -A Firefox customer-tutorial.$(minishift ip).nip.io
```

# DARK LAUNCHERS SHADOWING TRAFIC MIRRORING TRAFIC
Se podría monitorizar para que la respuesta la de la versión 1 pero también las peticiones se envíen a la versión 2 con el fin de probar cual es su respuesta. Al cliente se le envía la respuesta de la versión 1.
Se ejecutarán las dos versiones las dos recibirán las request.
``` 
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: recommendation
spec:
  host: recommendation
  subsets:
  - labels:
      version: v1
    name: version-v1
  - labels:
      version: v2
    name: version-v2
---
``` 
Creamos la imagen:
``` 
kubectl create -f .\destination-rule-recommendation-v1-v2.yml
kubectl get destinationrules -n tutorial
NAME             HOST             AGE
recommendation   recommendation   3m21s
kubectl describe destinationrules recommendation -n tutorial
…
  Subsets:
    Labels:
      Version:  v1
    Name:       version-v1
    Labels:
      Version:  v2
    Name:       version-v2
Events:         <none>
```
Ver los logs
---
```
kubectl logs -f `kubectl get pods | grep recommmendation-v2 | awk '{print $1}'` -c recommendation
```

# EGRESS
Acceso a servicios externos al cluster, se crean reglas egress que configura que tráfico permitimos para salida de istio.
Por ejemplo usamos este servicio que nos proporciona la hora:
```
curl http://worldclockapi.com/api/json/cet/now
```
Creación de un service entry con egress:
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: worldclockapi-egress-rule
spec:
  hosts:
  - worldclockapi.com
  ports:
  - name: http-80
    number: 80
    protocol: http
```
Creamos la imagen:	
```
kubectl create -f .\service-entry-egress-worldclockapi.yml
```

# SERVICIOS RESILENTES
- Circuit breaker, estados del servicio circuit breaker (Closed, Open, Half Open)
- El reset timeout es el tiempo que tenemos para revisar de nuevo la conexión.
- Cuando el circuito está abierto podemos responder con un valor determinado. 
- Istio implementa circuit breaker y retry para los reintentos.

## LOAD BALANCING
Empezamos desde cero:
```
kubectl get virtualservice -n tutorial
No resources found in tutorial namespace.
kubectl get destinationrule -n tutorial
No resources found in tutorial namespace.
```
Creamos el destination rule y el balanceo 50 - 50
```
kubectl apply -f .\destination-rule-recommendation-v1-v2.yml
destinationrule.networking.istio.io/recommendation created
kubectl apply -f .\virtual-service-recommendation-v1_and_v2_50_50.yml
virtualservice.networking.istio.io/recommendation created
```

## TIMEOUT
Cuando las llamadas sufren retraso en una de las versiones se puede balancear a otra.
Incluyo un timer en:
.\src\main\java\com\redhat\developer\demos\recommendation\RecommendationVerticle.java para emular retraso
Para emular un retraso de la v2
```
mvn clean install
docker build -f Dockerfile -t example/recommendation:v2 .
// borro el pod para que kubernetes lo recree
kubectl delete pod –l app=recommendation, versión=v2
```
Si accedemos a la versión 2 se visualiza que el tiempo de respuesta es mucho más lento. 
Lo que no quiere el servicio 2 es tener más request.

Emulo las llamadas
```
siege –r 2 –c 20 –v customer-tutorial.$(minishift ip).nip.io
```

## CIRCUIT BREAKER

Este patrón que implementa por ejemplo Hystrix por código, Istio ya lo tiene implementado.
Cuantos errores consecutivos deben pasar para que se abra el circuito, cual es la duración mínima que un host podrá estar expulsado para poder ser elegible para el envío de tráfico.
Según al ejemplo del apartado anterior configuramos para que no se de la misma carga a la versión 2.
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  creationTimestamp: null
  name: recommendation
  namespace: tutorial
spec:
  host: recommendation
  subsets:
    - name: version-v1
      labels:
        version: v1
    - name: version-v2
      labels:
        version: v2
      trafficPolicy:
        connectionPool:
          http:
            http1MaxPendingRequests: 1
            maxRequestsPerConnection: 1
          tcp:
            maxConnections: 1
        outlierDetection:
          baseEjectionTime: 120.000s
          consecutiveErrors: 1
          interval: 1.000s
          maxEjectionPercent: 100
```
Reemplazo el destination rule:
```
istioctl replace –f istiofiles/destination-rules-cb_policy_version_v2.yml
```
 
## POOL EJECTION
Si uno de los pods del pool comienza a devolver errores valores 500 de HTTP
Reescalamos el pool de recursos de la versión 2
```
kubectl scale deployment recommendation-v2 –replicas=2 –n tutorial 
kubectl get pods –l app=recommendation,versión=v2
```
Fichero destination-rule-crecommendation_cb_policy_pool_ejection.yml
```
destination-rule-recommendation_cb_policy_pool_ejection.yml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  creationTimestamp: null
  name: recommendation
  namespace: tutorial
spec:
  host: recommendation
  subsets:
  - labels:
      version: v1
    name: version-v1
    trafficPolicy:
      connectionPool:
        http: {}
        tcp: {}
      loadBalancer:
        simple: RANDOM
      outlierDetection:
        // con un error hace ejection solo realizara una petición y esperara 
		// 15 segundos para hacer la siguiente
        baseEjectionTime: 15.000s
        consecutiveErrors: 1
        interval: 5.000s
        maxEjectionPercent: 100
  - labels:
      version: v2
    name: version-v2
    trafficPolicy:
      connectionPool:
        http: {}
        tcp: {}
      loadBalancer:
        simple: RANDOM
      outlierDetection:
        baseEjectionTime: 15.000s
        consecutiveErrors: 1
        interval: 5.000s
        maxEjectionPercent: 100
```
Reemplazo:
```		
istioctl replace –f istiofiles/destination-rule-crecommendation_cb_policy_pool_ejection.yml
```

## SOLUCION FINAL
Con la solución anterior al menos uno de los usuarios recibirá un error, el siguiente error se dará a los 15s. Con esta solución no se devuelve error al usuario.
Ahora se hace retry automático por cada error para que se resuelva la petición desde otro pod.
Se configura para 3 reintentos:
Fichero: virtual-service-recommendation-v2_retry.yml
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
  namespace: tutorial
spec:
  hosts:
  - recommendation
  http:
  - route:
    - destination:
        host: recommendation
    retries:
      attempts: 3
      perTryTimeout: 2s
```

# TESTING
Pruebas sobre el mesh.

## TESTING CAOTICO
Inyectamos errores en las invocaciones para ver la respuesta del service mesh.
El 50% de los request a recommedation devolverán error 503
Se utiliza label de kubernetess, fichero virtual-service-recommendation-503.yml
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
spec:
  hosts:
  - recommendation
  http:
  - fault:
      abort:
        httpStatus: 503
        percent: 50
    route:
    - destination:
        host: recommendation
        subset: app-recommendation
---
```
Creamos la imagen:
```
istioctl create –f istiofiles/virtual-service-recommendation-503.yml
```

## RETRASOS
Inyecta retrasos, añade delays a las llamadas de servicios para ver cómo se comportan los consumidores.
Añade un sleep de 7 segundos al 50 por ciento de las llamadas.
Fichero virtual-service-recommendation-delay.yml
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendation
spec:
  hosts:
  - recommendation
  http:
  - fault:
      delay:
        fixedDelay: 7.000s
        percent: 50
    route:
    - destination:
        host: recommendation
        subset: app-recommendation
---
```

# TRAZABILIDAD
Observabilidad outofthebox de Istio, Jaeger ui permite revisar la trazabilidad, Comunicaciones entre servicios y trazabilidad 

## MÉTRICAS
Métricas outofthebox de Istio, Prometheus es un sistema para la visualización de métricas OpenSource que además de guardar datos tiene un sistema de alertas para notificar datos.
Utiliza Graphana como Data Visualizer, además tiene la consola de prometheus.

## KIALI
Que están haciendo los servicios en el service mesh, sirve para visualizar gráficamente las comunicaciones entre servicios y como están configurados.
Instalación:
```
kubectl apply kiali.yml
```

# SEGURIDAD
## WHITE LISTS
Reglas para la comunicación entre los servicios, evita comunicaciones no esperadas. Con esto evitamos comprometer los servicios.
Preferenceservice solo aceptará comunicaciones de recommendation service
Fichero acl-whitelist.yml
```
apiVersion: "config.istio.io/v1alpha2"
kind: listchecker
metadata:
  name: preferencewhitelist
spec:
  overrides: ["recommendation"]
  blacklist: false
---
apiVersion: "config.istio.io/v1alpha2"
kind: listentry
metadata:
  name: preferencesource
spec:
  value: source.labels["app"]
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: checkfromcustomer
spec:
  match: destination.labels["app"] == "preference"
  actions:
  - handler: preferencewhitelist.listchecker
    instances:
    - preferencesource.listentry
```
	
## BLACK LISTS
Se indicarán cuáles son las comunicaciones que no están permitidas.
Para hacer la prueba vamos a denegar la comunicación entre customers y preferences.
Denegaremos la comunicación e indicamos como informar al usuario.
Fichero acl-blacklist.yml
```
apiVersion: "config.istio.io/v1alpha2"
kind: denier
metadata:
  name: denycustomerhandler
spec:
  status:
    code: 7
    message: Not allowed
---
apiVersion: "config.istio.io/v1alpha2"
kind: checknothing
metadata:
  name: denycustomerrequests
spec:
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: denycustomer
spec:
  match: destination.labels["app"] == "preference" && source.labels["app"]=="customer"
  actions:
  - handler: denycustomerhandler.denier
    instances: [ denycustomerrequests.checknothing ]
```

## MUTUAL TLS
Con Istio la comunicación entre los proxy de cada uno de los servicios será con HTTPS y nuestros servicios puedan seguir tratando HTTP. Con esto se evita crear e instalar certificados.

Fichero authentication-enable-tls.yml
```
apiVersion: "authentication.istio.io/v1alpha1"
kind: "Policy"
metadata:
  name: "default"
spec:
  peers:
  - mtls: {}
```
Fichero destination-rule-tls.yml
```
apiVersion: "networking.istio.io/v1alpha3"
kind: "DestinationRule"
metadata:
  name: "default"
spec:
  host: "*.tutorial.svc.cluster.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```
Aplicamos:
```
curl http://${GATEWAY_URL}
istioctl apply –f istiofiles/authentication-enable-tls.yml
istioctl apply –f istiofiles/destination-rule-tls.yml
istioctl authn tls-check | grep tutorial
``` 
Podríamos hacer sniffer del tráfico entre servicios para revisarlo.
``` 
Kubectl exec –it $CUSTOMER_POD –c istio-proxy /bin/bash
Ifonfig para ver la ip 172.17.0.11 y desde la máquina
sudo tcpdump –vvvv –A –i eth0 ‘((dst port 8080) and (net 172.17.0.11))’
``` 
Deshabilitamos:
```
istioctl delete –f istiofiles/authentication-enable-tls.yml
istioctl delete –f istiofiles/destination-rule-tls.yml
```
Y si hacemos lo mismo, la información aparece…
