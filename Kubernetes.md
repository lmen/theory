# Getting started with Kubernetes

## What is ?
> A docker container orchestration framework by Google. 

It implements a cluster of docker engines, like Docker Swarm.

## Concepts

* node : runs a docker engine. It is where the containers execute.
* pods : colocated group of conatainers that share an IP, namespace and storage volume. Pods are efemural they can go down or up at any time. At the simple level a Pos can have only one container.
* Replica Set: Manages the lifecycle of pods and ensures specified number are running.
* Service : Single stable name for a set of pods, also acts as a load balancer.
* label : used to organize and select group of objects. They allow to define metadata to objects like pods. For example, a pod can have labels and then a service can identify the pods ata have a specific label.
* Node : Machine or VM in a cluster
* Master : Central control plane, providesunified view of the cluster 
  * etcd: distributed key-value store used to persist Kubernetes system state
* Worker: Docker host running kubelet (node agent) and proxy service:
  * Runs pods and containers
  * Monitored by systemd (CentOS) or monit (Debian)
* Kubectl : is the client tool that tals with the Master node to control the kubernetes cluster. Example of commands:
  * kubectl get pods
  * kubectl create -f <filename>
  * kubectl update or delete
  * kubectl scale -replicas=3 rc/<name>


The outside requests are handled first by a Load Balance HTTP which redirects the requests to the the proxy service present in each worker node.

## Kubernetes Pod Configuration example

```
apliVerion: 1
kind: Pod
metadata:
    name: wildfly-pod
    labels:
        name: wildfly-pod
spec:
    containers:
    -   name: wildfly
        image: jboss/wildfly
        ports:
        -   containerPort: 8080
```

## replication set configuration (Replica controller)

Example of a configuration file

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: wildfly-rc
spec:
  replicas: 2
  selector: // identifies the PODs for the RSet
    app: wildfly-rc-prod
  template:
    metadata:
      labels:
        app: wildfly-rc-prod
    spec:
      containers:
      - name: wildfly
        image: jboss/wildfly
        ports:
        - containerPort: 8008
    ```

    ### Service configuration

    apiVersion: v1
    kind: Service
    metadata:
      name: couchbase-service
    spec:
      selector:
        app: couchbase-rc-pod
      ports:
        - name: admin
          port: 8091
        - name: query
          port: 8093