# Desarrollo rápido con docker-compose y docker Swarm

## Importar máquina en windows 10
```
docker-machine create --driver hyperv --hyperv-virtual-switch "VM-External-Switch" --hyperv-boot2docker-url file://C:/tmp/boot2docker.iso swarm1
docker-machine env swarm1
docker-machine ssh swarm1
docker-machine create --driver hyperv --hyperv-virtual-switch "VM-External-Switch" --hyperv-boot2docker-url file://C:/tmp/boot2docker.iso swarm2
docker-machine env swarm2
docker-machine ssh swarm2
```

## Crear el manager
```
root@swarm1:/home/docker# docker swarm init
```

## Crear worker
```
root@swarm2:/home/docker# docker swarm join --token SWMTKN-1-3hdxke0ifihew0ndkicb36vpsv3vtir93amvxvtygmmn3hd1e9-6yl1ypyj0lrgei0vdk0vi0ww9 10.228.51.190:2377
```

### Ver nodos
```
root@swarm1:/home/docker# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
z69nst3etrhfsk9k5wvhgfhtk *   swarm1              Ready               Active              Leader              19.03.5
p53ivhbplywab5wxlpgff6f1i     swarm2              Ready               Active                                  19.03.5

root@swarm2:/home/docker# docker node ls
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager 
node or promote the current node to a manager.
```

## Instalar docker-compose en la máquina
ver: https://github.com/docker/compose/releases
```
root@swarm1:/home/docker# cd /tmp
root@swarm1:/tmp# curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
root@swarm1:/tmp#chmod +x /usr/local/bin/docker-compose
root@swarm1:/tmp# mkdir node-redis-example
root@swarm1:/tmp# cd node-redis-example
```

## Crear fichero docker-compose.yml
```
version: "3"
services:
 redis:
  image: redis:alpine
  volumes:
   - db-data:/data
  networks:
   appnet1:
    aliases:
     - db
  deploy:
   placement:
    constraints: [node.role == manager]

 web:
  image: katacoda/redis-node-docker-example
  networks:
   - appnet1
  depends_on:
   - redis
  ports:
   - 8432:3000
  deploy:
   mode: replicated
   replicas: 2
   labels: [APP=WEB]
   resources:
    limits:
     cpus: '0.25'
     memory: 512M
    reservations:
     cpus: '0.25'
     memory: 256M
   restart_policy:
     condition: on-failure
     delay: 5s
     max_attempts: 3
     window: 120s
   update_config:
     parallelism: 1
     delay: 10s
     failure_action: continue
     monitor: 60s
     max_failure_ratio: 0.3
   placement:
     constraints: [node.role == worker]

volumes:
 db-data:
networks:
 appnet1:
```

## Probar compose
```
root@swarm1:/tmp/node-redis-example#docker-compose up -d 
root@swarm1:/tmp/node-redis-example#docker ps
root@swarm1:/tmp/node-redis-example#docker-compose down
```

## Levantar compose en cloud
```
root@swarm1:/tmp/node-redis-example# docker stack deploy -c docker-compose.yml redis-node
root@swarm1:/tmp/node-redis-example#docker ps
```
Una instancia esta en el manager y otra en el nodo segun las reglas del deploy
```
root@swarm1:/tmp/node-redis-example# docker stack services redis-node
ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
s0fet9zmwlna        redis-node_redis    replicated          1/1                 redis:alpine
vhcgmorg1znz        redis-node_web      replicated          2/2                 katacoda/redis-node-docker-example:latest   *:8432->3000/tcp
root@swarm1:/tmp/node-redis-example# docker stack rm redis-node
Removing service redis-node_redis
Removing service redis-node_web
Removing network redis-node_appnet1
```
## Instalar herramienta portainer
```
cd /tmp
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```

## Instalar herramenta swarmpit
```
git clone https://github.com/swarmpit/swarmpit
docker stack deploy -c swarmpit/docker-compose.yml swarmpit
```
## Incluir stacks desde la aplicación p.e. portainer
```
version: '3.1'
services:

 wordpress:
  image: wordpress
  restart: always
  ports:
   - 8080:80
  environment:
   WORDPRESS_DB_HOST: db
   WORDPRESS_DB_USER: exampleuser
   WORDPRESS_DB_PASSWORD: examplepass
   WORDPRESS_DB_NAME: expampledb

 db:
  image: mysql:5.7
  restart: always
  environment:
    MYSQL_DATABASE: exampledb
    MYSQL_USER: exampleuser
    MYSQL_PASSWORD: examplepass
    MYSQL_RANDOM_ROOT_PASSWORD: '1'
```
También se puede ejecutar el stack desde línea de comandos:
```
docker stack deploy -c docker-compose-wp.yml wordpress
```
