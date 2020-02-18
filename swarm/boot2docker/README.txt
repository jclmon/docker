docker-machine create --driver hyperv --hyperv-virtual-switch "VM-External-Switch" --hyperv-boot2docker-url file://C:/tmp/boot2docker.iso swarm1
docker-machine env swarm1
docker-machine ssh swarm1

// crear el manager
root@swarm1:/home/docker# docker swarm init

// crear worker
root@swarm2:/home/docker# docker swarm join --token SWMTKN-1-3hdxke0ifihew0ndkicb36vpsv3vtir93amvxvtygmmn3hd1e9-6yl1ypyj0lrgei0vdk0vi0ww9 10.228.51.190:2377

// ver nodos
root@swarm1:/home/docker# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
z69nst3etrhfsk9k5wvhgfhtk *   swarm1              Ready               Active              Leader              19.03.5
p53ivhbplywab5wxlpgff6f1i     swarm2              Ready               Active                                  19.03.5

root@swarm2:/home/docker# docker node ls
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager 
node or promote the current node to a manager.

-- ver https://github.com/docker/compose/releases para instalar docker-compose
root@swarm1:/home/docker# cd /tmp
root@swarm1:/tmp# curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
root@swarm1:/tmp#chmod +x /usr/local/bin/docker-compose



root@swarm1:/tmp# mkdir node-redis-example
root@swarm1:/tmp# cd node-redis-example
-- crear fichero docker-compose.yml

-- probar compose
root@swarm1:/tmp/node-redis-example#docker-compose up -d
-- 
root@swarm1:/tmp/node-redis-example#docker ps
root@swarm1:/tmp/node-redis-example#docker-compose down


-- levantar compose en cloud
root@swarm1:/tmp/node-redis-example# docker stack deploy -c docker-compose.yml redis-node
-- 
root@swarm1:/tmp/node-redis-example#docker ps

Una instancia esta en el manager y otra en el nodo segun las reglas del deploy


root@swarm1:/tmp/node-redis-example# docker stack services redis-node
ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
s0fet9zmwlna        redis-node_redis    replicated          1/1                 redis:alpine
vhcgmorg1znz        redis-node_web      replicated          2/2                 katacoda/redis-node-docker-example:latest   *:8432->3000/tcp

root@swarm1:/tmp/node-redis-example# docker stack rm redis-node
Removing service redis-node_redis
Removing service redis-node_web
Removing network redis-node_appnet1

HERRAMIENTA PORTAINER
cd /tmp
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
docker stack deploy --compose-file=portainer-agent-stack.yml portainer

HERRAMIENTA SWARMPIT
git clone https://github.com/swarmpit/swarmpit
docker stack deploy -c swarmpit/docker-compose.yml swarmpit
