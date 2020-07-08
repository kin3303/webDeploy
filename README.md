Test Web Deploy

```
$ git clone https://github.com/kin3303/webDeploy.git
$ cd webDeploy
$ docker build -t kin3303/webDeploy:latest .
$ docker push kin3303/webDeploy:latest
```


Create Docker Swarm - master

```
$ sudo su 
$ amazon-linux-extras install -y docker 
$ systemctl start docker

# Master Node 의 Private IP 를 넣어야 함
# Master-Slave 간 통신은 2377(랜덤포트) 
$ docker swarm init --advertise-addr {swarm-master-01-private-ip}
Swarm initialized: current node (6tamvt1xodakxlz6w05b9e76w) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2dy5wg1mj2aopmprnqlqy0h6yrykgayq34evklv1tn6lvuwume-0ndbtmaqbjoa69bh66sw6r2l2 172.31.43.223:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Create Docker Swarm - worker

```
$ sudo su 
$ amazon-linux-extras install -y docker 
$ systemctl start docker
$ docker swarm join --token SWMTKN-1-2dy5wg1mj2aopmprnqlqy0h6yrykgayq34evklv1tn6lvuwume-0ndbtmaqbjoa69bh66sw6r2l2 172.31.43.223:2377
```

Create Services - By commands

```
$ docker service create \
--name traefik \
--constraint 'node.role == manager' \ 
-p 80:80 \
-p 8080:8080 \
--mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
--network external \
traefik:v1.7 \
--docker \
--docker.swarmmode \
--docker.watch \
--web


$ docker service create \
--name counter \
--network=external \
--network=internal \
--replicas 3 \
-e REDIS_HOST=redis \
--label traefik.port=4567 \
--label traefik.frontend.rule=Host:counter.local.dev \
subicura/counter
 

$ docker service create \
--name web \
--replicas 3 \
--network external \
--with-registry-auth \
--label traefik.port=80 \
--label traefik.frontend.rule=Host:web.com \
subicura/whoami:2

 
$ docker service create \
--name=visualizer \
--publish=8081:8080/tcp \
--constraint=node.role==manager \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
dockersamples/visualizer 


$ docker service create \
--name portainer \
--constraint 'node.role==manager' \
-p 9000:9000 \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
--network external \
portainer/portainer
```

Create Services - By docker-compose

```
$ docker stack deploy \
-c docker-compose.yaml \
teststack
```
