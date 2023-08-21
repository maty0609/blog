---
layout: post
excerpt: "Running Home Assistant on Kubernetes cluster"
title: Running Home Assistant on Kubernetes cluster
categories: [Kubernetes, Home Assistant]
author: Matyas
published: true
---


### Intro
I have been using Home Assistant for a while to control my home devices. I have been running Home Assistant on one of my Raspberry Pis but with the cadence of new releases it became a bit annoying to upgrade Home Assistant every few weeks with occasionally doing roll back. To address those challenges I have decided to run Home Assistant in docker and since I was already running Kubernetes cluster at home I have decided to run Home Assistant on the cluster.

### Overview
We will be deploying Home Assistant with four YAML configuration files. It will be for deployment, storage and services. Each of these files will ensure consistent deployment. My Kubernetes cluster runs on 4 Raspberry Pi therefore this post is mainly for ARM version however can be easily applied for any other architecture.

### Zigbee and Zwave USB
Main advantage of Kubernetes is that it provides high level of resiliency. Essentially you have master node which proxying any requests between who consumes the services running on Kubernetes cluster (for instance laptop) and one of Kubernetes cluster nodes based on its availability or load. This is a challenge since we need docker container (pod in case of Kubernetes) running on particular node where Zigbee and Z-Wave USB receivers are physically connected. We have two options: we will hair pin the pod to the particular node or we will make USB reachable over TCP across the whole cluster.

This time we will choose easiest way and we will hair pin the pod to the particular node. I will explain a bit more in deployment chapter.

### Namespace
I personally prefer to logically separate Home-assistant from other pods and therefore we will be deploying Home Assistant in dedicated namespace ```homeassistant```. We will not be defining any dedicated resources for any pods in the namespace.

### Storage
Since we will be pinning the pod to the particular node we will be storing all configuration files and database to the same node. Storage for Home Assistant will be PersistentVolume type of the object which will create path for any Home Assistant store to ```/mnt/kubernetes/homeassistant``` providing 100M of storage which should be enough for the database. It will be named ```pv-ha``` which will be called by PersistentVolumeClaim.

```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-ha
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/kubernetes/homeassistant"
```

Purpose of the PersistentVolumeClaim is to ensure that storage will be available and tagged for particular pod. Important part is also ```namespace``` which will ensure that storage will be available for the pod which will be sitting in ```homeassistant``` namespace. Name will ensure that any pod which will be running anywhere on the cluster will link its storage to this PVC based on the tag called ```name``` from ```metadata``` attribute.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: homeassistant-storage
 namespace: homeassistant
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

### Deployment
When the storage has been defined than the actual pod can be deployed. We will define the pod with deployment YAML. Let's start with some generic information which we will be defining at the top of YAML file. It will be defined that this deployment will be running in ```homeassistant``` namespace with tag ```app: homeassistant``` which will be important when we will be propagating this deployment outside of the cluster via ```service```.

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: homeassistant
  name: homeassistant
  namespace: homeassistant
```

Now when the basic information about the deployment has been defined we can start defining actual information about the pod for instance what volumes will be attached to the pod. As you can see we are not only attaching storage via ```persistentVolumeClaim``` but also USB devices which will be essential for controlling our home devices. It clearly shows that we will use USB links from ```/dev/``` directory and link them to the pod.

```
spec:
  volumes:
   - name: ha-storage
     persistentVolumeClaim:
      claimName: homeassistant-storage
   - name: dev-usb0
     hostPath:
       path: /dev/ttyACM0
   - name: dev-ama0
     hostPath:
       path: /dev/ttyAMA0
```

Last but not least will be the container attributes. ```image``` defines what docker image will be used. As you can see we are using particular version in this case ```0.94.1```. The reason why we define the version is to avoid deploying newest version after every re-deployment. This becomes handy with homeassistant since some new versions breaking 'stuff' so it is better to stick to previous releases:)

```volumeMounts``` define what volumes will be mounted to the pod and what will be the path


```
containers:
- image: homeassistant/raspberrypi3-homeassistant:0.94.1
  name: home-assistant
  volumeMounts:
   - mountPath: "/config"
     name: ha-storage
   - mountPath: "/dev/ttyACM0"
     name: dev-usb0
   - mountPath: "/dev/ttyAMA0"
     name: dev-ama0
```


### Home Assistant Configuration

As it has been already mentioned in the previous chapter we will be mounting new path ```/config``` to volumeMount with name ```ha-storage``` which is defined in ```homeassistant-deployment.yaml``` and it is linked to already defined claimName ```homeassistant-storage```

```
spec:
  volumes:
   - name: ha-storage
     persistentVolumeClaim:
      claimName: homeassistant-storage
```

In this path we will be storing and reading all the configuration data like for instance ```configuration.yaml``` or ```automation.yaml```.


### Home Assistant Services

This is all nice and cool but so far we were deploying application which will be reachable only internally from the cluster. To expose the application to the outside world we will need to deploy object called ```service```. We will be basically exposing Home-assistant on the external IP address of the cluster on particular cluster port. In this case it will be port 31123.

```
spec:
  ports:
  - nodePort: 31123
    port: 8123
    protocol: TCP
  selector:
    app: homeassistant
  type: NodePort
```

### Links

All we have discussed in this post can be downloaded from my git:

<https://git.matyasprokop.com/mprokop/k8s_home/tree/master/homeassistant>

### Summary

Now you should be able to access home assistant via port 31123 on your Kubernetes cluster.
