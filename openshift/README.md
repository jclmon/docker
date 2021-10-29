# OPENSHIFT

Red Hat OpenShift es la plataforma de Kubernetes empresarial líder en el sector*, la cual se diseñó especialmente para formar parte de una [estrategia de nube híbrida abierta](https://www.redhat.com/es/products/open-hybrid-cloud). Gracias a que ofrece operaciones automatizadas integrales, una  experiencia uniforme en todos los entornos y la implementación de  autoservicio para los desarrolladores, los equipos pueden trabajar en  conjunto para llevar las ideas de la etapa de desarrollo a la de  producción de manera más eficiente.

## Creando el cluster openshift

La inicialización del clúster crea una nueva máquina en nuestro virtualbox e instala en ella todas las aplicaciones necesarias para que funcione openshift (la versión que se va a instalar el openshift 3.11), la instrucción que tenemos que ejecutar para que se empiece a construir el clúster es la siguiente (el proceso dura algunos minutos):

```
$ minishift start --vm-driver virtualbox
minishift start       
-- Starting profile 'minishift'
...
OpenShift server started.

The server is accessible via web console at:
    https://192.168.99.100:8443/console

You are logged in as:
    User:     developer
    Password: <any value>
```

To login as administrator:
```
oc login -u system:admin
```

## Gestionando el cluster openshift
### Para detener la máquina virtual que hemos creado:
```
$ minishift stop
```
### En cualquier otro momento podemos seguir trabajando con el cluster:
```
$ minishift start
```
### Para borrar la máquina virtual:
```
$ minishift delete
```

## Despliegue de la primera aplicación en Openshift
•	Vamos a crear una aplicación web con un servidor web apache2. Nuestra aplicación tendrá ficheros html y php.

•	Nuestra aplicación la tenemos en un repositorio de GitHub (https://github.com/josedom24/html_for_openshift)

•	Vamos a crear nuestra aplicación usando source2image: al crear la aplicación vamos a escoger una imagen con apache2 y php y vamos a indicar nuestro repositorio con el código, a partir de esta información se va a crear una imagen docker con apache2, php y nuestro código, que va a servir para desplegar la aplicación.

### Pasos a seguir
1.	Del catálogo elegimos la imagen de php y elegimos la versión de php con la que vamos a trabajar.
2.	Indicamos un nombre y el repositorio donde tenemos nuestro código.
3.	Se va a crear un build: 
	o	Se crea un pod a partir de la imagen de apache2 seleccionada.
	o	Se inyecta en ese pod el código del repositorio.
	o	Y se crea una nueva imagen con apache2 y nuestra aplicación.
4.	A partir de la imagen creada: 
	o	Se crea un deployment que crear un pod con la aplicación.
	o	Se crea un servicio de acceso a la aplicación
	o	Y se crea una ruta de acceso.
5.	Ya tenemos desplegada nuestra aplicación y podemos acceder a ella.

### Recursos que nos ofrece Openshift
•	Web Console: Aplicación web que nos permite trabajar con nuestros recursos de OpenShift

•	Catálogo de servicios: Conjunto de imágenes y plantillas bases. A partir de ellas podemos construir (builds) nuevas imágenes con s2i.

•	Proyectos: Permiten a los usuarios organizar y controlar sus aplicaciones.

•	Aplicación: Nuestra aplicación es una aplicación Kubernetes, por los están formada por distintos recursos: deployment, pods, service, routes

•	Builds: Es el proceso por el que se crea la imagen desde la que se va a crear nuestra aplicación. Tenemos muchas estrategias: 
	o	Desde una imagen, desde un fichero Dockerfile, …
	
	o	source2image
	
	o	Pipeline de Jenkins
	
	o	…

•	Registro de imágenes: Las imágenes que se crean en los builds se guardan en un registro interno.

•	Deployment: Define las características de los contenedores que se van a crear para cada aplicación.

•	Pods: Accedemos a la gestión de los pods que se han creado para nuestra aplicación.

•	Services: Tenemos información del recurso servicio que nos permite acceder a nuestra aplicación.

•	Routes: Se asocia automática una ruta (nombre) para acceder a cada aplicación.

•	Monitorig: Tenemos acceso a herramienta para monitorizar el uso de recurso de nuestra aplicación.

•	cli oc: Herramienta de línea de comandos que nos permite trabajar con nuestros recursos de OpenShift


### Tolerancia a fallos y balanceo de carga
Openshift va a asegurar que en todo momento se esté ejecutando nuestra aplicación. Para ello asegura la ejecución de los pods que hayamos indicado para ejecutar la aplicación.
Antes de ver un ejemplo de tolerancia a fallos, vamos a señalar algunas funcionalidades que podemos obtener de los pods: En la ruta Aplications->Pods podemos escoger un determinado Pod, y nos llevará a la información del pod, entre lo más interesante podemos señalar:
•	Details: Toda la información del pod.
•	Logs: Terminal donde podemos ver en tiempo real los logs que se producen en el pod
•	Terminal: Podemos acceder a la terminal del pod.
Nota: Si el pod está compuesto por varios contenedores, podremos elegir el contenedor al que queremos acceder de forma sencilla.
Vemos un ejemplo de tolerancia a fallos:
1.	Vamos a borrar el pod, eligiendo la opción Delete del botón Actions.
2.	Inmediatamente se crea un nuevo pod que sustituye al anterior.

### Escalabilidad
En cualquier momento se puede escalar de forma horizontal (crear más pods en un despliegue o eliminarlos) los pods de un despliegue. En todo momento las peticiones a nuestra aplicación se balancean entre los pods que tengamos ejecutando.
Antes de ver un ejemplo de tolerancia a fallos, vamos a señalar algunas funcionalidades que podemos obtener de los deploy: En la ruta Aplications->Deployments podemos escoger un determinado despliegue y a continuación el número de despliegue, y nos llevará a una página con la información de ese despliegue, entre lo más interesante podemos señalar:

•	Details: Toda la información del despliegue.

•	Metrics: Las métricas (uso de memoria y de CPU) del despliegue.

•	Logs: Terminal donde podemos ver en tiempo real los logs que se producen en el despliegue.


Veamos un ejemplo de escalado manual:
1.	Desde la pestaña Details podemos observar una grágica donde vemos el número de pods actuales, con dos flechas que nos permiten el escalado.

2.	Podemos aumentar y disminuir el número de pods interactuando con las dos flechas y podemos observar cómo se crean y eliminan los pods.

### Balanceo de carga
Crear tres pods en nuestro despliegue y acceder al programa info.php que simplemente nos muestra el nombre del servidor en el que se está ejecutando:
```
<?php echo "Servidor:"; echo gethostname();echo "\n"; ?>
Desde una línea de comando podemos simular un número de peticiones a nuestra aplicación y veremos cómo se está balanceando la carga:
for i in `seq 1 100`; do curl http://prueba-myproyecto1.7e14.starter-us-west-2.openshiftapps.com/info.php; done
Servidor:prueba-1-pb2sj
Servidor:prueba-1-vcm92
Servidor:prueba-1-l9zvt
Servidor:prueba-1-pb2sj
...
```
Después de escalar, con tres Pods activos:
Se ejecuta la línea de comando para realizar las 100 peticiones y se observa el balanceo.
```
[root@srvdev ~]# for i in `seq 1 100`; do curl http://ejemplo1-myproject.192.168.2.60.nip.io/info.php; done
Servidor:ejemplo1-1-qzk72
Servidor:ejemplo1-1-nqc9j
Servidor:ejemplo1-1-8flgl
Servidor:ejemplo1-1-qzk72
Servidor:ejemplo1-1-nqc9j
Servidor:ejemplo1-1-8flgl
Servidor:ejemplo1-1-qzk72
Servidor:ejemplo1-1-nqc9j
Servidor:ejemplo1-1-8flgl
…
```
## Despliegue continuo y rollback de nuestra aplicación Openshift
Actualizaciones continúas
El ciclo de vida del desarrollo de nuestra aplicación sería el siguiente:

1.	Modificamos el código de nuestra aplicación, y guardamos los cambios en el repositorio.

2.	Desde openshift realizamos un nuevo build, que construye una nueva imagen con la aplicación modificada.

3.	A continuación, de forma automática, se realizará un nuevo deployment, que borrará los pods antiguos y creará nuevos desde la nueva versión de la imagen.

Rollback de nuestra aplicación en OpenShift
En cualquier momento podemos desplegar un deployment anterior, para volver a una versión anterior de nuestra aplicación.

### Despliegue continuo en Openshift
Despliegue continuo:
Copiamos el webhook de nuestra aplicación, lo incorporamos al proyecto de github en la pestaña de configuración, usar content type json:
Tal y como se realiza el push de git con los cambios se dispara automáticamente el build:
Para publicar nuestra ip interna y que github tenga acceso al webhook se puede utilizar https://ngrok.com/

### Autoescalado, escalado automático en Openshift
El autoescalado en OpenShift es una característica muy interesante. Con el autoescalado podemos tener un número mínimo de pods de nuestra aplicación y queremos que cuando esa “máquina” llegue a una determinada carga se cree un pod nuevo. Con ello realizamos un escalado horizontal y la carga de peticiones que estamos respondiendo se reparte entre más contenedores. En el momento en que baje la carga de peticiones se eliminaran los pods que no son necesarios.
Actualmente el autoescalado se realiza dependiendo del uso del CPU, aunque también se está desarrollando la posibilidad de que la utilización de memoria sea el factor que realice el autoescalado.


### Autoescalado de un despliegue
Para crear el auescalado de un despliegue elegimos de la opción Deployments el despliegue que nos interesa y escogemos en el botón Actions la opción Add Autoescaler:

Indicamos los siguientes datos:

1.	El número mínimo de pods que vamos a tener de nuestra aplicación.

2.	El número máximo de pods que vamos a tener de nuestra aplicación.

3.	El porcentaje de utilización de la CPU que se va a considerar para crear o eliminar un pod (Nota: Hemos puesto un valor pequeño para la prueba que vamos a realizar).

Como podemos observar en un principio tenemos un sólo pod:
Si empezamos a hacer peticiones a nuestra aplicación, por ejemplo hacemos 50 peticiones concurrentes durante un minuto:

```
ab -t 60 -c 50 -k http://prueba-myproyecto1.7e14.starter-us-west-2.openshiftapps.com/
```
ab es del paquete apache2-utils para hacer pruebas de peticiones
Pasado unos segundos observaremos como de forma automática se crean nuevos pods de nuestra aplicación, hasta llegar al límite superior indicado:

## Introducción a la línea de comandos
Hasta ahora hemos manejado los recursos de nuestro cluster utilizando la página web de Openshift Online, pero también tenemos a nuestra disposición un comando CLI (command line interface) oc que podemos usar desde la línea de comandos.
Instalación de la utilidad de línea de comandos oc
Podemos bajarnos la última versión del cliente desde la página de descargas. Y siguiendo las instrucciones que encontramos en Get Started with the CLI (https://www.openshift.com/blog/installing-oc-tools-windows) podemos instalar la herramienta en los distintos sistemas operativos.
Por ejemplo para Linux Debian/Ubuntu, descomprimimos el fichero y copiamos el ejecutable oc en un directorio que este en el PATH.
Conectando a nuestra cuenta de OpenShift
Una vez instalado, debemos conectarnos a nuestro clúster. Para ello vamos a utilizar un token de acceso. Lo más sencillo es obtener el token desde la página web eligiendo la opción Copy Login Command desde las opciones de tu usuario.
Pegamos en la línea de comando y obtenemos el comando para conectarnos a nuestro clúster:

 ```
oc login https://api.starter-us-west-2.openshift.com --token=xxxxxxxxxxxxxxx.....
 ```

Cuando nos conectamos por primera vez se crea un fichero de configuración en ~/.kube/config con la información de acceso al clúster.

- Creación de un nuevo proyecto

Si accedemos al clúster y no tenemos creado un proyecto tenemos que ejecutar la siguiente instrucción para crear uno:
 ```
oc new-project miproyecto1
 ```
Y podemos comprobar el estado del clúster con:
 ```
oc status
 In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443
You have no services, deployment configs, or build configs.
Run 'oc new-app' to create an application.
 ```

### Despliegue de una aplicación con OC

Una vez que nos hemos conectado a nuestro cluster y hemos creado un proyecto, vamos a crear nuestra aplicación. En este caso, volvemos a desplegar la misma aplicación que en el ejemplo anterior, usando source2image.
Para ver todos recursos del catálogo que podemos utilizar:
```
$ oc new-app –list
```
Seguimos los mismos pasos, del catálogo elegimos la imagen de php y la versión, para buscar imágenes:
```
$ oc new-app --search php
```

Las image stream son las que tiene disponibles Openshift
A continuación creamos la nueva aplicación indicando la imagen, el repositorio donde esta el código e indicando un nombre:
```
$ oc new-app php:7.1~https://github.com/josedom24/html_for_openshift.git --name prueba
```
Podemos ver el estado de nuestra aplicación ejecutando:
```
$ oc status
In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443

svc/prueba - 172.30.27.139 ports 8080, 8443
  dc/prueba deploys istag/prueba:latest <-
    bc/prueba source builds https://github.com/josedom24/html_for_openshift.git on openshift/php:7.1 
    deployment #1 deployed 37 seconds ago - 1 pod
```
Como podemos observar se ha creado un build:
```
$ oc get bc
NAME      TYPE      FROM      LATEST
prueba    Source    Git       1
$ oc describe bc/prueba
```
Se ha creado un despliegue:
```
$ oc get dc
NAME      REVISION   DESIRED   CURRENT   TRIGGERED BY
prueba    1          1         1         config,image(prueba:latest)
$ oc describe dc/prueba
```
Y podemos ver los pods que se han creado:
```
$ oc get pods
NAME             READY     STATUS      RESTARTS   AGE
prueba-1-build   0/1       Completed   0          5m
prueba-1-n6lmz   1/1       Running     0          4m
$ oc describe pod/prueba-1-n6lmz
```
Para ver los logs de un build:
```
$ oc logs bc/prueba
```
Para ver los logs de un determinado despliegue:
```
$ oc logs dc/prueba
```
Y, por ejemplo, para ejecutar un comando en un determinado pod:
```
$ oc exec prueba-1-n6lmz  -it /bin/bash
```
También podemos ejecutar el siguiente comando para acceder al pod:
```
$ oc rsh prueba-1-n6lmz
```
Accediendo a nuestra aplicación
Cuando hemos creado una aplicación se ha creado un servicio:
```
$ oc get svc
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
prueba    ClusterIP   172.30.27.139   <none>        8080/TCP,8443/TCP   10m
```
Pero debemos crear una ruta para poder acceder a dicha aplicación, para ello:
```
$ oc expose svc/prueba
route.route.openshift.io/prueba exposed
$ oc get routes
NAME      HOST/PORT                                                     PATH      SERVICES   PORT       TERMINATION   WILDCARD
prueba    prueba-miproyecto1.7e14.starter-us-west-2.openshiftapps.com             prueba     8080-tcp                 None
```
Y ya podemos acceder a la aplicación usando la URL:
```
prueba-miproyecto1.7e14.starter-us-west-2.openshiftapps.com.
Operaciones avanzadas con OC
Tolerancia a fallos
Si borramos un pods, se va a crear inmediatamente otro para que la aplicación siga funcionando:
$ oc get pods
NAME             READY     STATUS      RESTARTS   AGE
prueba-1-build   0/1       Completed   0          18m
prueba-1-n6lmz   1/1       Running     0          17m
$ oc delete pod/prueba-1-n6lmz
pod "prueba-1-n6lmz" deleted
$ oc get pods
NAME             READY     STATUS      RESTARTS   AGE
prueba-1-b4s25   1/1       Running     0          10s
prueba-1-build   0/1       Completed   0          18m
```

### Escalabilidad
En cualquier momento se puede escalar de forma horizontal (crear más pods en un despliegue o eliminarlos) los pods de un despliegue. En todo momento las peticiones a nuestra aplicación se balancean entre los pods que tengamos ejecutando.
```
$ oc scale dc prueba --replicas=3
deploymentconfig.apps.openshift.io/prueba scaled
$ oc get pods -o wide
NAME             READY     STATUS      RESTARTS   AGE       IP               NODE                                          NOMINATED NODE
prueba-1-b4s25   1/1       Running     0          1m        10.130.22.98     ip-172-31-56-230.us-west-2.compute.internal   <none>
prueba-1-build   0/1       Completed   0          20m       10.129.115.115   ip-172-31-51-92.us-west-2.compute.internal    <none>
prueba-1-sghr9   1/1       Running     0          20s       10.128.27.215    ip-172-31-55-160.us-west-2.compute.internal   <none>
prueba-1-w4b4p   1/1       Running     0          20s       10.129.11.160    ip-172-31-52-89.us-west-2.compute.internal    <none>
```
Donde vemos en que nodo se está ejecutando cada pod, y comprobamos el balanceo de carga:
```
for i in `seq 1 100`; do curl http://prueba-miproyecto1.7e14.starter-us-west-2.openshiftapps.com/info.php; done
Servidor:prueba-1-sghr9
Servidor:prueba-1-w4b4p
Servidor:prueba-1-b4s25
...
```
### Actualizaciones continúas
Modificamos el código de nuestra aplicación, y guardamos los cambios en el repositorio. Desde openshift realizamos un nuevo build, que construye una nueva imagen con la aplicación modificada:
```
$ oc start-build bc/prueba
build.build.openshift.io/prueba-2 started
$ oc get bc
NAME      TYPE      FROM      LATEST
prueba    Source    Git       2
```
A continuación, de forma automática, se realizará un nuevo deployment, que borrará los pods antiguos y creará nuevos desde la nueva versión de la imagen:
```
$ oc get dc
NAME      REVISION   DESIRED   CURRENT   TRIGGERED BY
prueba    2          3         2         config,image(prueba:latest)
```
Rollback de nuestra aplicación
Para volver a una versión anterior de nuestra aplicación:
```
$ oc rollout undo dc/prueba
deploymentconfig.apps.openshift.io/prueba rolled back
$ oc get dc
NAME      REVISION   DESIRED   CURRENT   TRIGGERED BY
prueba    3          3         1         config
```
### Autoescalado
Para crear un autoescalado en un despliegue:
```
$ oc autoscale dc/prueba --min 1 --max 3 --cpu-percent=20
horizontalpodautoscaler.autoscaling/prueba autoscaled
$ oc get hpa
NAME      REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
prueba    DeploymentConfig/prueba   5%/20%    1         3         3          34s
$ oc describe hpa/prueba
```

## Despliegue de aplicaciones Python con OC
Vamos a desplegar una aplicación python que tenemos guardada en el repositorio: https://github.com/josedom24/python_for_openshift
Despliegue desde la consola web
Para realizar el despliegue seguimos los siguientes pasos desde la consola web:

1.	Al crear nuestra aplicación, escogemos en el catálogo la opción de imagen python.

2.	Escogemos la versión de python e indicamos el repositorio de nuestra aplicación.

3.	Durante el despliegue se van a instalar los módulos que encuentre en el fichero requirements.txt.

4.	El contenedor que vamos a crear ejecuta por defecto una aplicación qie se llama app.py.

Despliegue con el cliente de comandos oc
Borramos el proyecto anterior, y creamos uno nuevo. Y a continuación vamos a realizar el despliegue de la aplicación python de nuevo. Lo primero es buscar las imágenes python que tenemos en el catálogo:
 ```
$ oc new-app --search python
 ```
A continuación creamos la aplicación:
 ```
$ oc new-app python:3.6~https://github.com/josedom24/python_for_openshift --name app-python
Podemos ver el estado de nuestra aplicación ejecutando:
$ oc status
In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443
svc/app-python - 172.30.85.230:8080
  dc/app-python deploys istag/app-python:latest <-
    bc/app-python source builds https://github.com/josedom24/python_for_openshift on openshift/python:3.6 
    deployment #1 deployed 16 seconds ago - 1 pod
 ```

Para terminar tenemos que crear la ruta de acceso a la aplicación:
```
$ oc expose svc/app-python
route.route.openshift.io/app-python exposed
$ oc get route
NAME         HOST/PORT                                                         PATH      SERVICES     PORT       TERMINATION   WILDCARD
app-python   app-python-miproyecto1.7e14.starter-us-west-2.openshiftapps.com             app-python   8080-tcp                 None
```

Y accediendo a la URL podremos ver nuestra aplicación.
Despliegue de aplicaciones PHP con OC
Vamos a instalar un CMS PHP que utiliza una base de datos Sqlite (phpSQLiteCMS). Para ello vamos a utilizar el código de la aplicación que se encuentra en el repositorio: https://github.com/ilosuna/phpsqlitecms
Despliegue desde la consola web

Vamos a seguir los siguientes pasos desde la consola web:

1.	Al crear nuestra aplicación, escogemos en el catálogo la opción de imagen PHP.

2.	Escogemos la versión de PHP e indicamos el repositorio de nuestra aplicación.

3.	Durante el despliegue se van a instalar los módulos que encuentre en el fichero composer.json.

4.	El despliegue de la aplicación, creará un build que construirá una imagen con nuestra aplicación, posteriormente creará un deployment, encargado de crear los pods, y finalmente se creará un servicio y una ruta de acceso a la aplicación.
Despliegue con el cliente de comandos oc

Borramos el proyecto anterior, y creamos uno nuevo. Y a continuación vamos a realizar el despliegue de la aplicación php de nuevo. Para ello ejecutamos:
```
$ oc new-app php:7.1~https://github.com/ilosuna/phpsqlitecms --name appphp
Podemos ver el estado de nuestra aplicación ejecutando:
$ oc status
In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443
svc/appphp - 172.30.244.6 ports 8080, 8443
  dc/appphp deploys istag/appphp:latest <-
    bc/appphp source builds https://github.com/ilosuna/phpsqlitecms on openshift/php:7.1 
    deployment #1 deployed about a minute ago - 1 pod
Para terminar tenemos que crear la ruta de acceso a la aplicación:
$ oc expose svc/appphp
route.route.openshift.io/appphp exposed
$ oc get routes
NAME      HOST/PORT                                                     PATH        SERVICES   PORT       TERMINATION   WILDCARD
appphp    appphp-miproyecto1.7e14.starter-us-west-2.openshiftapps.com             appphp     8080-tcp                 None
```

Y accediendo a la URL podremos ver nuestra aplicación.
Los contenedores son efímeros
Como hemos comentado los contenedores son efímeros, la información que se guarda en ellos se pierde al eliminar el contenedor, además si tenemos varias replicas de una misma aplicación (varios pods) la información que se guarda en cada una de ellas es independiente. Vamos a comprobarlo:

1.	En el directorio /cms/data se encuentran las bases de datos de la aplicación.

2.	Cuando escalemos nuestra aplicación se va a crear otro pod con la base de datos inicial, en este nuevo pod no tenemos el mismo contenido que el original.

3.	Si realizamos un nuevo despliegue los nuevos pods perderán los datos de la base de datos.

4.	Necesitamos volúmenes persistentes

## Despliegue de aplicaciones PHP con almacenamiento persistente
En este ejemplo vamos a utilizar un volumen (almacenamiento persistente) para guardar las bases de datos de la aplicación (que están guardadas en el directorio /cms/data). Vamos a continuar trabajando donde lo dejamos en la unidad anterior.
Estrategia de despliegue. En este momento tenemos instalado phpSQLiteCMS pero sin almacenamiento persistente. Para trabajar con almacenamiento persistente en el próximo paso vamos a crear un volumen, pero antes vamos a cambiar la estrategia de despliegue de la aplicación:

•	Por defecto la estrategia es Rolling: En este caso se crean los nuevos pods de la nueva versión del despliegue se comprueban que funcionan y posteriormente se eliminan los pods de la versión anterior.

•	Nosotros deseamos que la estrategia sea Recreate: En esta situación se eliminan los pods de la versión actual y posteriormente se crean los nuevos pods de la nueva versión. Si vamos a trabajar con volúmenes, necesitamos configurar este tipo de estrategia, ya que la primera nos da errores al intentar conectar el volumen a un nuevo pod cuando sigue conectado al anterior pod.

Para cambiar la estrategia: Elegimos el despliegue de appphp y en el botón Actions elegimos la opción Edit:

### Creación del volumen
A continuación vamos a crear un nuevo volumen:

Al crear el volumen tenemos que elegir entre varios medio de almacenamiento (Storage Class). Las posibilidades de medios de almacenamiento dependerán de la infraestructura donde está instalado OpenShift. Como medios de almacenamiento podemos tener: NFS, HostPath, GlusterFS, Ceph RBD, OpenStack Cinder, AWS Elastic Block Store (EBS), GCE Persistent Disk, iSCSI, Fibre Channel,…
OpenShiftOnline está instalado en AWS por lo que tenemos dos medios de almacenamiento: ebs: Elastic Block Store que proporciona volúmenes de almacenamiento, y gp2-encrypted, similar al anterior pero la información está cifrada.
Dependiendo del medio de almacenamiento que tenga nuestra infraestructura tendremos distintas formas de acceso a la información guardada en el volumen:

•	ReadWriteOnce: lectura y escritura solo para un nodo (RWO)

•	ReadOnlyMany: solo lectura para muchos nodos (ROX)

•	ReadWriteMany: lectura y escritura para muchos nodos (RWX)

Por ejemplo los volúmenes de tipo ebs no soportan el modo ReadWriteMany, por consecuencia, si tenemos un despliegue al que hemos conectado un volumen con tipo de acceso ReadWriteOnce, este despliegue no podrá replicarse, es decir no podemos tener varios pods replicados ya que no podríamos montar el volumen al mismo tiempo en los distintos pods.
En la siguiente tabla podemos ver la relación entre los medios de almacenamiento y los tipos de acceso para cada uno de los proveedores cloud que soportan:
En todos ellos al menos un pod podrá acceder al volumen, si esto no existe no podremos dar la posibilidad de elasticidad.

### Añadir el volumen a un despliegue
En el Pod que se está ejecutando tenemos el terminal, de ahí vemos que la ruta donde se almacenan los ficheros de SQLite son las siguientes:

El volumen que hemos creado lo conectamos al despliegue appphp en el directorio /opt/app-root/src/cms/data: Elegimos el despliegue de appphp y en el botón Actions elegimos la opción Add Storage:

En el momento que hemos añadido el almacenamiento a nuestra aplicación se produce un nuevo despliegue de forma automática que implantará la aplicación con el volumen persistente.
En el directorio cms/data de phpSQLiteCMS se guardan las bases de datos sqltile de la aplicación. Por lo tanto es el directorio que necesitamos que este guardado en un volumen persistente.
Pero al montar el volumen en el directorio indicado, hemos perdido su contenido, y al acceder a la aplicación nos da un error:

Para solucionarlo vamos a copiar las bases de datos desde nuestro ordenador:

He clonado el repositorio de phpsqlitecms en mi ordenador:
 ```
$ git clone https://github.com/ilosuna/phpsqlitecms.git
$ cd phpsqlitecms/cms
 ```
Vamos a copiar el contenido de este directorio al volumen, para ello:
 ```
$ oc get pods
NAME             READY     STATUS      RESTARTS   AGE
appphp-1-build   0/1       Completed   0          12m
appphp-2-tj5jm   1/1       Running     0          1m

$ oc cp data appphp-2-tj5jm:cms/
 ```
Comprobamos que hemos copiado los ficheros:
 ```
$ oc exec appphp-2-tj5jm ls cms/data
 content.sqlite
 entries.sqlite
 userdata.sqlite 
 ```

Podemos concluir que cada vez que hagamos un nuevo despliegue se creara de nuevo el sistema de fichero de los contenedores, a excepción del directorio cms/data cuya información esta guardada en el volumen.
Ya podemos acceder a nuestra aplicación. Para terminar el ejercicio podemos comprobar que las modificaciones que hacemos en la configuración de la página se mantienen cuando se borran los pods, por ejemplo al crear un nuevo despliegue.

Despliegue de aplicaciones PHP en OpenShift con almacenamiento persistente
Desde Openshift se pueden desplegar aplicaciones con distintas bases de datos:

Desde línea de comandos
Buscamos los templates relacionados con mysql

### Creamos la BBDD

Conecto con la BBDD y compruebo que se ha creado. Despliegue de una base de datos mysql sin almacenamiento persistente
Lo primero que vamos a hacer es buscar en el catálogo los recursos que podemos desplegar relacionados con mysql:
```
$ oc new-app --search mysql
Templates (oc new-app --template=<template>)
-----
mysql-persistent
...
Image streams (oc new-app --image-stream=<image-stream> [--code=<source>])
-----
mysql
...
```
Como podemos observar podemos utilizar una imagen mysql sin almacenamiento persistente o un template mysql-persistent que nos ofrece almacenamiento persistente.
Despliegue de una aplicación mysql sin almacenamiento persistente
Para crear una aplicación mysql utilizamos la siguiente instrucción indicando algunas variables de entorno para la configuración del servidor:
```
$ oc new-app mysql -e MYSQL_USER=user MYSQL_PASSWORD=pass MYSQL_DATABASE=prueba --name mysql
$ oc status
In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443
svc/mysql - 172.30.160.193:3306
  dc/mysql deploys openshift/mysql:5.7 
    deployment #1 running for 9 seconds - 1 pod
Podemos ver el pod que se ha creado y acceder a él para ejecutar el cliente mysql:
$ oc get pods
NAME            READY     STATUS    RESTARTS   AGE
mysql-1-rr5kk   1/1       Running   0          13s
$ oc rsh mysql-1-rr5kk
sh-4.2$ mysql -u user -p
Enter password: 
...
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| prueba             |
+--------------------+
2 rows in set (0.02 sec)
A continuación vamos a crear una tabla en la base de datos:
mysql> use prueba;
Database changed
mysql> create table tabla_prueba(id INT);
Query OK, 0 rows affected (0.06 sec)
mysql> show tables;
+------------------+
| Tables_in_prueba |
+------------------+
| tabla_prueba     |
+------------------+
1 row in set (0.01 sec)
¿Qué ocurre si se elimina el pod?
$ oc delete pod/mysql-1-rr5kk 
pod "mysql-1-rr5kk" deleted
$ oc get pods
NAME            READY     STATUS    RESTARTS   AGE
mysql-1-pr57f   1/1       Running   0          9s
$ oc rsh mysql-1-pr57f       
sh-4.2$ mysql -u user -p
Enter password: 
...
mysql> use prueba;
Database changed
mysql> show tables;
Empty set (0.00 sec)
```
Como podemos ver se ha perdido la información de la base de datos, ya que estamos usando un pod sin un volumen de almacenamiento.
Despliegue de una base de datos mysql con almacenamiento persistente
En esta ocasión vamos a usar el template mysql-persistent, en un nuevo proyecto ejecutamos la siguiente instrucción:
```
$ oc new-app mysql-persistent --param=MYSQL_USER=user --param=MYSQL_PASSWORD=pass --param=MYSQL_DATABASE=prueba --name mysql
$ oc status
In project miproyecto1 on server https://api.starter-us-west-2.openshift.com:443
svc/mysql - 172.30.201.218:3306
  dc/mysql deploys openshift/mysql:5.7 
    deployment #1 deployed about a minute ago - 1 pod
Podemos ver el pod que se ha creado y acceder a él para ejecutar el cliente mysql:
$ oc get pods
NAME            READY     STATUS    RESTARTS   AGE
mysql-1-wzqjl   1/1       Running   0          1m

$ oc rsh mysql-1-wzqjl 
sh-4.2$ mysql -u user -p 
Enter password: 
...
mysql> use prueba;
Database changed

mysql> create table tabla_prueba(id INT);
Query OK, 0 rows affected (0.18 sec)
mysql> show tables;
+------------------+
| Tables_in_prueba |
+------------------+
| tabla_prueba     |
+------------------+
1 row in set (0.00 sec)
¿Qué ocurre si se elimina el pod?
$ oc delete pod/mysql-1-wzqjl
pod "mysql-1-wzqjl" deleted
$ oc get pods
NAME            READY     STATUS              RESTARTS   AGE
mysql-1-jbqdb   0/1       ContainerCreating   0          15s

$ oc rsh mysql-1-jbqdb
sh-4.2$ mysql -u admin -p
Enter password: 
...
mysql> use prueba;
Database changed
mysql> show tables;
+------------------+
| Tables_in_prueba |
+------------------+
| tabla_prueba     |
+------------------+
1 row in set (0.00 sec)
Como podemos ver en esta ocasión no se ha perdido la información de la base de datos, ya que estamos utilizando un volumen para guardar la información que no queremos perder.
```

## Despliegue de aplicación WordPress
En esta unidad vamos a instalar WordPress, un CMS escrito en PHP, en OpenShift aplicando todos los conocimientos que hemos estudiado en unidades anteriores:

### Creación de la base de datos
Vamos a crear un nuevo proyecto y en ella vamos a crear una aplicación con la base de datos mysql. Para ellos escogemos en el catálogo la imagen de mysql e indicamos las variables para su creación:

### Despliegue de WordPress
A continuación vamos a crear una aplicación PHP con el despliegue de WordPress para ello vamos a utilizar el repositorio oficial de la aplicación:
https://github.com/WordPress/WordPress:

A continuación, antes de trabajar con el volumen, y como explicábamos en unidades anteriores vamos a cambiar la estrategia de despliegue, para ello elegimos el despliegue de wordpress y en el botón Actions elegimos la opción Edit:

### Creación del volumen
A continuación vamos a crear un nuevo volumen:
Y lo conectamos al despliegue wordpress en el directorio /opt/app-root/src/wp-content: Elegimos el despliegue de wordpress y en el botón Actions elegimos la opción Add Storage:
En el momento que hemos añadido el almacenamiento a nuestra aplicación se produce un nuevo despliegue de forma automática que implantará la aplicación con el volumen persistente.
En el directorio wp-content de WordPress se guardada todos los datos de la aplicación (documentos que subimos, plugins, temas,…). Por lo tanto es el directorio que necesitamos que este guardado en un volumen persistente.
Pero al montar el volumen en el directorio indicado, hemos perdido su contenido, por lo que vamos a copiarlo desde nuestro ordenador:
•	He clonado el repositorio de WordPress en mi ordenador:
``` 
$ git clone https://github.com/WordPress/WordPress
$ cd WordPress
```
•	Vamos a copiar el contenido de este directorio al volumen, para ello:
```
$ oc get pods
        NAME                READY     STATUS      RESTARTS   AGE
        mysql-1-r9vjk       1/1       Running     0          22m
        wordpress-1-build   0/1       Completed   0          20m
        wordpress-2-dsgnr   1/1       Running     0          2m

        $ oc cp wp-content wordpress-2-dsgnr:/opt/app-root/src/
```
•	Comprobamos que hemos copiado los ficheros:
```
$ oc exec wordpress-2-dsgnr ls wp-content
        index.php
        plugins
        themes
```
Podemos concluir que cada vez que hagamos un nuevo despliegue se creara de nuevo el sistema de fichero de los contenedores, a excepción del directorio wp-content cuya información esta guardada en el volumen. Seguimos teniendo un problema: el fichero wp-config.php donde se guarda la configuración de WordPress se perderá cada vez que hacemos un despliegue. Para solucionarlo, vamos a copiar al volumen persistente dos ficheros:
1. Un fichero wp-config.php con la configuración de wordpress con los datos para el acceso a la base de datos:
```
$ cd ficheros
$ oc cp wp-config.php wordpress-2-dsgnr:/opt/app-root/src/wp-content
```
2. Un fichero run.sh que vamos a ejecutar cada vez que hagamos un nuevo despliegue y va a crear un enlace simbólico al fichero de configuración que tenemos guardado:
```
$ oc cp run.sh wordpress-2-dsgnr:/opt/app-root/src/wp-content
El contenido del fichero run.sh es:
#!/bin/bash
ln -s /opt/app-root/src/wp-content/wp-config.php /opt/app-root/src/wp-config.php
exec /usr/libexec/s2i/run
```
Y tenemos que darle permiso de ejecución:
```
$ oc exec wordpress-2-dsgnr chmod +x wp-content/run.sh
```
Ejecutando un comando en el despliegue
Como hemos indicado anteriormente cada vez que hagamos un despliegue tenemos que ejecutar el script anterior para que se cree el enlace directo al fichero wp-config.php, para ello elegimos el despliegue de wordpress y en el botón Actions elegimos la opción Edit YAML y vamos a añadir la sección command:
```
spec:
  containers:
    - image: >-
        172.30.254.23:5000/proyecto-wordpress/wordpress@sha256:7458214fb3da2a6deec8fa13dbdf8af42b09a75e98735937bf67f6da72adb468
      imagePullPolicy: Always
      name: wordpress
      command:
        - /opt/app-root/src/wp-content/run.sh
...
```
En el momento que hemos modificado la configuración a nuestra aplicación se produce un nuevo despliegue de forma automática que implantará la modificación indicada.

## Despliegue de WordPress
Para concluir accedemos a la URL de nuestra aplicación y terminamos de configurar la instalación de WordPress (como podemos observar se ha saltado el paso donde nos preguntan por las credenciales de la base de datos, ya que hemos indicado un fichero wp-config.php con la configuración).
Despliegue de aplicación wordpress con template
En la unidad anterior, hemos detallamos los pasos necesarios para crear una instancia de WordPress creando todos los recursos paso a paso en openShift. En esta última unidad, le mostraremos una forma mucho más fácil de instalar y ejecutar una nueva instancia de WordPress en OpenShift utilizando una plantilla y una imagen personalizada de WordPress.
Una plantilla describe un conjunto de objetos que se pueden parametrizar y procesar para desplegar de forma automática los recursos necesarios en OpenShift. en nuestro caso vamos a utilizar un template que de forma automática crea la aplicación WordPress, la aplicación mysql, los volúmenes necesarios para que la información sea persistente, …

## Creación del template
El template que vamos a utilizar lo puedes encontrar en el repositorio GitHub: https://github.com/openshift-evangelists/wordpress-quickstart. Lo primero que tenemos que hacer es crear el recurso template en nuestro proyecto:
```
$ oc create -f https://raw.githubusercontent.com/openshift-evangelists/wordpress-quickstart/master/templates/classic-standalone.json
```
A continuación si examinamos el catálogo de aplicaciones, veremos una nueva opción: WordPress (Classic / Standalone). Si lo elegimos podremos indicar los parámetros de configuración de nuestras aplicaciones: mysql y wordpress.
Como podemos comprobar se han creado las dos aplicaciones, Se han creado los dos volúmenes Y finalmente podemos acceder a nuestro wordpress accediendo a la ruta de la aplicación