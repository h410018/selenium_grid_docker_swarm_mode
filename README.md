# selenium_grid_docker_swarm_mode

## 1. Install docker on Ubuntu 16.04

   https://docs.docker.com/engine/install/ubuntu/

## 2. Install Virtualbox on Ubuntu 16.04

https://tecadmin.net/install-oracle-virtualbox-on-ubuntu/

## 3. Install docker-machine

https://docs.docker.com/machine/install-machine/

## 4. Create docker machines

```
$ docker-machine create -d virtualbox v1 v2 v3
```
```
$ docker-machine ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
v1     -        virtualbox   Running   tcp://192.168.99.101:2376           v19.03.5
v2     -        virtualbox   Running   tcp://192.168.99.102:2376           v19.03.5
v3     -        virtualbox   Running   tcp://192.168.99.100:2376           v19.03.5
```

## 5. Create docker swarm mode

```
#ssh v1 machine
$ ssh docker@192.168.99.101
docker@192.168.99.100's password:
   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net
 
docker@v1:~$
```
* set swarm manager on v1 machine
```
#Set swarm manager
docker@v1:~$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (fmetlacx245l5j03cqgxe8p3v) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-16rfxmms57qbf0ogpbmdr6t73gfu8ewekgb42k1eydg92pr4gr-b50uwb0ho7jc85fu3on8uvmu5 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
* v2 v3 add in swarm 
```
docker@v2:~$ docker swarm join --token SWMTKN-1-16rfxmms57qbf0ogpbmdr6t73gfu8ewekgb42k1eydg92pr4gr-b50uwb0ho7jc85fu3on8uvmu5 192.168.99.100:2377
This node joined a swarm as a worker.
```
```
docker@v3:~$ docker swarm join --token SWMTKN-1-16rfxmms57qbf0ogpbmdr6t73gfu8ewekgb42k1eydg92pr4gr-b50uwb0ho7jc85fu3on8uvmu5 192.168.99.100:2377
This node joined a swarm as a worker.
```
* Check node status
```
docker@v1:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
fmetlacx245l5j03cqgxe8p3v *   v1                  Ready               Active              Leader              19.03.5
tx640qpziuf3kt0fxlafuldoo     v2                  Ready               Active                                  19.03.5
pw5izhxowz19cxp03vs4rrxcs     v3                  Ready               Active                                  19.03.5
```
## 6. Create overlay network for selenium grid
```
docker@v1:~$ docker network create -d overlay seleniumnet
```
## 7. Create selenium hub service
```
docker@v1:~$ docker service create --name selenium-hub --constraint node.role==manager --network seleniumnet -p 4040:4444 selenium/hub
ccwlrpee5u1sht4u5snjkwp4l
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```
* Port forward using vboxmanage
```
$ cd <virtualbox folder>
$ docker-machine stop v1
$ vboxmanage modifyvm "v1" --natpf1 "selenium-grid,tcp,,4040,,4040"
```
Then you can access grid console page via localhost:4040

## 8. Create chrome-node and firefox-node service
```
docker@v1:~$ docker service create --name node-chrome --replicas 1 --constraint node.hostname==v2 -p 7901:5900 --network seleniumnet -e HUB_PORT_4444_TCP_ADDR=selenium-hub -e HUB_PORT_4444_TCP_PORT=4444 selenium/node-chrome-debug bash -c 'SE_OPTS="-host $HOSTNAME" /opt/bin/entry_point.sh'

docker@v1:~$ docker service create --name node-firfox --replicas 1 --constraint node.hostname==v3 -p 7900:5900 --network seleniumnet -e HUB_PORT_4444_TCP_ADDR=selenium-hub -e HUB_PORT_4444_TCP_PORT=4444 selenium/node-firefox-debug bash -c 'SE_OPTS="-host $HOSTNAME" /opt/bin/entry_point.sh'
```

Here's using docker-swarm-visualizer plugin to see how swarm cluster looks like. 
<br>
ref:https://github.com/dockersamples/docker-swarm-visualizer

![image](https://github.com/h410018/selenium_grid_docker_swarm_mode/blob/master/%E6%88%AA%E5%9C%96%202020-04-17%20%E4%B8%8B%E5%8D%883.07.18.png)


Then you can check out the console page to see if node register the hub.
![image](https://github.com/h410018/selenium_grid_docker_swarm_mode/blob/master/%E6%88%AA%E5%9C%96%202020-04-17%20%E4%B8%8B%E5%8D%883.07.02.png)

