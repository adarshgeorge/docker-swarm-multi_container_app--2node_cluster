## Multi Container App in Single Node Cluster using Docker Swarm.

* A DevOps Project to display Location of IP address.


![alt App](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/home.png)



### Pre-Requisite
An 2 Server/EC2 with docker installed. 

Userdata while launching an EC2 instance.
```
#!/bin/bash
sudo yum install docker -y
sudo service docker restart
sudo chkconfig docker on
sudo usermod -a -G docker ec2-user

```



First we need to initialize the Swarm Mode

```
$ docker swarm init

Swarm initialized: current node (jdza9qu4zxmgswvtqrlzdipxh) is now a manager.
To add a worker to this swarm, run the following command:
   
    docker swarm join --token SWMTKN-1-1pxslguh0yy2i8f9la85po3f53etjyejs7m27jsp7ustkkw6wa-b7s13chwrmyhn1l533zcxc9vm 172.31.6.208:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
$
```


**Creating Overlay network**
```
$ docker network create --driver overlay ipstack-net
gw6ehdudygxydxvlihbmy8shp
$

```

**Now create redis service**

```
docker service create \
--name ipstack-cache \
--replicas 1 \
--network ipstack-net \
redis:latest

```


**Creating Flask Service**


```
docker service create \
--name ipstack-app \
--network ipstack-net \
--replicas 2 \
-p 80:8080 \
-e REDIS_HOST=ipstack-cache \
-e REDIS_CACHE=600 \
-e FLASK_PORT=8080 \
-e IPSTACK_KEY='91e59ddd65394199ad3f905d35532479' \
adarshgeorge999/ipstackapp:1
```

Verify the created services

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
t7drwgzwp6xg        ipstack-app         replicated          2/2                 adarshgeorge999/ipstackapp:1   *:80->8080/tcp
wsvhvt4w2dff        ipstack-cache       replicated          1/1                 redis:latest
$

```

**Browse the Hostname or IP address.**

![alt Out1](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/container1.png) 

Refresh the page to verify app is loading from the different container. 

![alt Search](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/container2.png) 


**Output**

Search any IP

![alt Search](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/searching1.png) 


Now again search the same IP address to verify the IP address is cached in reddis container. 

![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/searching2.png) 


### We're going to increase the container replicas from 2 to 4 container.

```
$ docker service update --replicas 4  ipstack-app

ipstack-app
overall progress: 4 out of 4 tasks                                                                                                                                      1/4: running [==================================================>]                                                                        2/4: running [==================================================>]                                                                         
3/4: running [==================================================>]                                                                     
4/4: running [==================================================>]                                                                   verify: Service converged                                                                                                                   


$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
97gzs0sxqppp        ipstack-app         replicated          4/4                 adarshgeorge999/ipstackapp:1   *:80->8080/tcp
wsvhvt4w2dff        ipstack-cache       replicated          1/1                 redis:latest
$

```

**Result**

Loading from First Container

![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/replica1.png) 

Loading from Second Container
![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/replica2.png) 

Loading from Third Container

![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/replica3.png) 

Loading from Fourth Container

![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/replica4.png) 


### That's it!


Now we're Joining another Node server

```
$ docker swarm join --token SWMTKN-1-1pxslguh0yy2i8f9la85po3f53etjyejs7m27jsp7ustkkw6wa-b7s13chwrmyhn1l533zcxc9vm 172.31.6.208:2377
This node joined a swarm as a worker.
$

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Verify in Master Node

```

$ docker node ls
ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
jdza9qu4zxmgswvtqrlzdipxh *   ip-172-31-6-208.ap-south-1.compute.internal   Ready               Active              Leader              19.03.6-ce
kf6f71nmz5n95wz2crutgmq3w     ip-172-31-9-253.ap-south-1.compute.internal   Ready               Active                                  19.03.6-ce
$
```



Now we're going to update the replicas set from 4 to 6 container. 



```
$ docker ps
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS              PORTS               NAMES
565fa2873c9a        adarshgeorge999/ipstackapp:1   "python3 app.py"    12 seconds ago      Up 10 seconds                           ipstack-app.5.li74ez7a66fr2r1s6hga05s9h
83de004ce66a        adarshgeorge999/ipstackapp:1   "python3 app.py"    12 seconds ago      Up 10 seconds                           ipstack-app.6.p4f6bh6hib3xaenbgj2xltku8
$
```

Loading from Worker Node 1

![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/2.png) 

Loading from Worker Node 2


![alt Out2](https://github.com/adarshgeorge/docker-swarm-multi_container_app/blob/master/png/searching2.png) 
