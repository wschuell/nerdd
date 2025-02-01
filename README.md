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

## Contribute

* Install a kubernetes distribution, e.g. [microk8s](https://microk8s.io/), 
  [k3s](https://k3s.io/), [k0s](https://k0sproject.io/) or [Talos](https://www.talos.dev/)
* Install [tilt](https://docs.tilt.dev/)
* `tilt up`