# Notes from [presentation](https://www.youtube.com/watch?v=ZNdc4-yFTeA) 

## What is docker

Docker is a virtualization implementation that runs **one single application** as if it runs in a dedicated CPU, harddrive, network and operating system. For the application the running environemrnt is like is a container, where the computational resources are separed from the other applications.

Compare this with the classic virtualization implementation tht allows to run many application on top of a operating system, like a personal computer. 


The application files are place in a docker image file. A docker image has an operating system with basic features (it can be a unix - ubuntu or centos- or windows server), and other applications like for instance a JAVA EE container.

The Docker engine runs the docker image. The docker engine can run on top of linux, macox, windows. 

Package Once Deploy Anywhere

As stated a docker container can only do one task. When that task ends the container ends also.

Containers are disposible, when they start running they start always from the original image, not from the the last execution, as it happens with classical virtual machines.

Docker workflow:

* Docker registry : store the public images
* Docker client : interacts with the regestry and the docker host to run the container
* Docker host: Has a Docker Deamon to run the containers acording to the docker client rules.

Docker client most usefull comands:

* _docker images_ : display the images installed locally
* _docker run -it openjdk_ : runs image called openJdk (fetchs the image if is not available locally) in interact mode, that is run tha bash shell for more comands.
* _docker run -d -p 8080:8080 jboss/wildfly_ : runs image called wildfly in detacked mode (-d) and tells the container port 8080 should be mapped to the outside has port 8080 (-p 8080:8080)
* _docker ps_ : list the containers that are running 
* _docker stop 4ad_ : kill the container runnig as 4ad


#Docker composer

> Allows to run multiple containers in simultaneous in a **single node**.

The configuration of each container is done in a YAML file.

### define a configuration file for the containers to be run 
docker-composer.yml
```
web:
    image: jboss/wildfly
    ports:
        - 8080:8080
```

docker-compose.override.yml
```
web:
    ports:
        - 9080:8080
```

### To execute the containers 

```
docker-compose up -d
```

### To stop containers

```
docker-compose down
```

### Example ho to yse docker-compose to handle diferent configuration for diferent environments

The base configuration with de dev settings

docker-compose.yml
```
db-dev:
    image: sdddd/couchbase
    ports:
        - .....

web:
    image: dsds/wildfly
    environment:
        - COUCHBASE_URI=db-dev:8093
    ports:
        - 8080:8080
```

The changes on the configuration for the production environment.

production.yml
```
web:
    environment:
        - COUCHBASE_URI=db-prod:3093
    ports
        - 80:8080

db-prod:
    image:.....
```

To run:
`docker-composer up -f docker-composer.yml -f production.yml -d`

Note:
  Each container is register in a DNS server that knows the IP of each container. See the COUCHBASE_URI configuration.

The containers from docker-composer run in a single machine. And that machine can be the same machine where the docker client is running (local mode) or in a diferent machine (remote mode).


Common Use cases            | command line
-----------------           | ----------------------
automatic test setup        | ```docker-compose up; mv test; docker-compose down``` 
 Multiple isolated environments | docker-compose up -p <project directory> 

 ## Docker Swam

 > Creates a cluster of hosts (diferent machines with the docker engine) where each can run  a set of containers.

 Available in Docker client since version 1.12. The cluster of docker engines is created and managed by docker client.

 The cluster has the properties:
 * Has a set of docker-host master and a set of docker-host workers. Only one master is the principal, the others are in secondary and in standby for a failure from the principal.
 * Self healing. when a docker host works goes down then the containers are started on another working host.
 * It is just an option to do container management. 
 * A container runs in the worker hosts or in the master host (a master host can be configure to not run any container and has only a configuration role) 

description | command docker client 
----------- | ---------------------- 
Initialize a cluster and automatically create a host master | docker swarm init --listern-addr <ip>:2377 
add a worker host to the cluster | docker swarm join --secret <Secret> <manager ip>:2377
launch images on the cluster | docker service create --replicas 3 -name web <Image name>
remove one node from the running cluster | docker node update --availability drain <node name>

Docker swarm helps creating a distributed application but it stills needs a load balancer to distribute the request between the hosts. Because a container can be in any host Docker Swarm helps redirect the request to the host with the right container.


> A sidenote: A distributed environment has a scheduler that is a program that elects the most appropriated HOST for running an application based on constrains defined by the application and the characteristics of each host. For example, one application must run in  windows with 2 cores and 2 GB of RAM and on the same hast that has an Oracle database. the scheduler must be then find a host with this set of characteristics.

Using *Labels* on each docker host and given **filters** filters it is possible to influcient how the Docker Swam scheduler works.


## Docker and jenkins CI/CD

[see link] (https://github.com/arun-gupta/docker-jenkins-pipeline)

## Monitoring Docker containers

* docker stats
* docker remote api
* cAdvisor
  * Prometheus
  * influxDB
* Docker programing

### Docker maven integrations

There are at least two maven plug-ins for docker. One from spotify and another from fabric8io. This plugins allow to build an docker image and also to run the image in a docker host and send the image to the docker hub (registry).