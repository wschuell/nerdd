# Nerdd

## Infrastructure


This projects provides three parts:
* Kafka: Enables multiple services to communicate with each other. We use the Strimzi 
  Kafka Operator for setting up a Kafka cluster (see [their Github 
  project](https://github.com/strimzi/strimzi-kafka-operator/)). 
* Redpanda: A graphical user interface to track Kafka topics.
* KEDA: Automatically scales applications.


## Add a module to the cluster

* We assume that we already have a Python repository that implements a NERDD module
  (see https://github.com/molinfo-vienna/nerdd-module). Specifically, it contains a
  model class, say ```testmodule.TestModel```.
* Copy the folder ```template``` in this repository and give it the same name like your
  repository (e.g. testmodule).
* 

## Create cluster from scratch

* (maybe install docker - depends on next step)
* install a variant of kubernetes, e.g. microk8s or k3s
* enable dashboard, dns, image registry, metrics-server, hostpath-storage
```
microk8s enable dashboard
microk8s enable dns
microk8s enable registry
microk8s enable metrics-server
microk8s enable hostpath-storage
```
* create a namespace argo: ```kubectl create namespace argo```
* create a secret called github-credentials within this namespace argo and provide a
  username of the molinfo-vienna team together with a personal access token as password
```
kubectl apply -n argo -f - <<EOF
apiVersion: v1
kind: Secret
metadata:  
  name: github-credentials
type: Opaque
data:
  username: $(echo -n "<username>" | base64 -w0)
  password: $(echo -n "<access_token>" | base64 -w0)
EOF
```

* install argocd

## Add a node to the cluster
* when using microk8s:
    * on the master node (node16), run ```microk8s add-node`` to get instructions for 
      addinga new node
    * copy the command that uses the worker flag and run it on the new worker node
    * each new node has to use the image registry in the cluster
    * get the address of the current image registry 
    ```kubectl -n container-registry get service registry -o jsonpath='{.spec.clusterIP}'```
    * adapt hosts.toml on the new worker node accordingly (see https://microk8s.io/docs/registry-private#insecure-registry-1 Configuring MicroK8s)

## Contribute

* Install docker
* Install a variant of kubernetes, e.g. microk8s or k3s
* Install tilt
* ```tilt up```