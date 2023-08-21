---
layout: post
excerpt: "Traffic flows in Kubernetes Flannel"
title: "Traffic flows in Kubernetes Flannel"
categories: [Kubernetes, Flannel, CNI]
author: Matyas
published: true
comments: true
---

#### Overview
I'm currently working on Kubernetes project and Flannel so I put together very short post about Kubernetes Flannel and how Flannel works from design and traffic flow perspective. It will not explain how to set up Flannel I may will do it in dedicated post.

&nbsp;
#### Flannel
Flannel is currently the most tested Kubernetes CNI Provider. Flannel has very simple configuration and supports overlay and non-overlay backends. Flannel does not have any network policy support therefore if any security or segmentation required it has to be provided by other solutions.

##### Overlay mode

- VXLAN
    - For any cloud environments
    - Private and Secure network
- IPSec
    - Public cloud environments
    - Non-Secure network

##### Non-overlay mode

- Host-GW
    - Better performance
    - Public cloud environments
- AWS VPC
    - Experimental only
- GCE
    - Experimental only

I will focus on Flannel in the VXLAN overlay mode in this post. For the current project I'm working on this is the most suitable model and provide the features required for the inter-pod communication and performance. It will also enable to have single /16 across Kubernetes cluster.

&nbsp;
#### High-level Kubernetes Cluster networking diagram

![Diagram](../../img/2020-03-22-kubernetes-flannel/flannel.png)

###### vEthx interface
For the simplicity reasons vEthx is not in the diagram however it sits between container ethx interface and cni0. It is used for the routing traffic between cni0 and containers.

###### cni0 interface
cni0 interface is the default gateway for the pods which are connected to the Pod's CIDR subnet. Pods are using this as their default gateway therefore any traffic which is not within the ethx subnet is redirected to cni0 interface.

###### flannel.1 interface
Flannel.1 interface enable to communicate between pods. It terminates the subnet which is configured and defined by flanneld.

###### flanneld
Flanneld  speaks to etcd which is making sure that routing information about pods are exchanged between nodes.

&nbsp;
#### Multiple network support
Flannel does *not* support multiple network deployment however this can be done with running multiple flannel daemons connected to the different networks. Based on the requirements this is not an issue but for any future deployment this is good to keep in mind. Multiple network support for Flannel has been previously in experimental stage however it has been removed.

&nbsp;
#### Kubernetes cluster flows
In this chapter I will try to explain what will be the different scenarios of various flows between pods and nodes within the cluster. It will be important to mention that I will cover all the scenarios from the network perspective. Please be aware on top of the Kubernetes Flannel it will be also security layer so even though I will be presenting some traffic flows in next few chapters it doesn't mean this will or should be allowed in the production environment.

To allow any flows between nodes Kubernetes nodes should open UDP port 8472 (VXLAN) and TCP port 9099 (healthcheck).


##### Inter-pod communication
For the purpose of explaining the inter-pod flows it is assumed there will be no service mesh solution (like HasiCorp Consul pr Istio) or firewalls. Flows between Pods and Nodes will be in the same namespace and same Kubernetes cluster therefore in the same physical place.

1. Package leaves **Pod 1** via **eth1** and reaches **cni0** via **veth1**, looking for **Pod 2's** address.
2. Package leaves **cni0** and is redirected to **veth2**.
3. Package reaches the **Pod 2** via **eth2** interface.

![Diagram](../../img/2020-03-22-kubernetes-flannel/flannel-pod-to-pod.png)


##### Inter-node communication within the same cluster
For the purpose of explaining the inter-node flows it is assumed there will be no service mesh solution (like HasiCorp Consul pr Istio) or firewalls. Flows between Pods and Nodes will be in the same namespace and same Kubernetes cluster therefore in the same physical place.

1. Package leaves **Pod 1** via **eth1** interface and reaches the root network through the virtual interface **veth1**.
2. Package leaves **veth1** and reaches **cni0**, looking for **Pod 4's** address.
3. Package leaves **cni0** and is redirected to **eth4** via **Node 2 eth0**.
4. Package leaves **eth0** from **Node 1** and reaches the gateway.
5. Package leaves the gateway and reaches **eth0** interface on **Node 2**.
6. Package leaves **eth0** and reaches **cni0**, looking for **Pod 4’s** address.
7. Package leaves **cni0** and is redirected to **veth4** virtual interface.
8. Package leaves through **veth4** and reaches **Pod 4** via **eth4** interface.

```
matyas@node1:~ $ ip route
default via 192.168.1.1 dev eth0 src 192.168.1.2 metric 202
10.1.11.0/24 via 10.1.11.0 dev flannel.1 onlink
10.1.10.0/24 dev cni0 proto kernel scope link src 10.1.10.1
10.1.10.0 dev flannel.1 proto kernel scope link src 10.1.10.0 metric 205
```

![Diagram](../../img/2020-03-22-kubernetes-flannel/flannel-node-to-node.png)

&nbsp;
##### Inter-namespace communication
Since Flannel networking does *not* support network policy Pods from different namespaces can communicate between each other without any restrictions. In case the restriction for inter-namespace communication will be required service mesh like for instance Consul will have to handle this.
