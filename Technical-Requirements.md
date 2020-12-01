Сomponents calculations:  
__th2 env = Base + Core + Building blocks + Custom + Cassandra *__<br>
| Base & Core Components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|
| th2 infra | 1000 MB | 800 m | helm, infra-mgr, infra-editor, infra-operator |

| Custom & Building blocks components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|

## Apache Cassandra cluster hardware requirements
   Though it is possible to use Cassandra single-node installation, generally it’s recommended to setup at least 3-nodes cluster. Requirements to each node are the same.
   
 | Apache Cassandra node | Memory (MB) | CPU (Сores) | Disk space (GB) |
 |-----|------|---------|-------------|  
 | Cassandra node_n | 4000 MB | 2 | 15 GB for “/ “ mount + 200 GB for “/var“ mount |
   
## Software requirements:
Kubernetes cluster with Docker installed or as cloud managed service

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin

* Apache Cassandra cluster installed
* Docker registry access from cluster nodes - nexus.exp.exactpro.com:9000, hub.docker.com
* Distributed storage (not yet)
* Test box access to - gitlab.exactpro.com, index.docker.io, quay.io, k8s.gcr.io, grafana.github.io, charts.bitnami.com, kubernetes-charts.storage.googleapis.com, mirror.centos.org, puppet.exp.exactpro.com
* Chrome 75 or newer

## Hardware requirements:
### Single-control plane cluster
You need:

* One or more machines running a deb/rpm-compatible Linux OS; for example: Ubuntu or CentOS.
* 2 GB or more of RAM per machine–any less leaves little room for your apps.
* At least 2 CPUs on the machine that you use as a control-plane node.
* Full network connectivity among all machines in the cluster. You can use either a public or a private network.

## High availability configuration

* Three machines that meet kubeadm’s minimum requirements for the masters
* Three machines that meet kubeadm’s minimum requirements for the workers
* One or more machines running one of:
    * Ubuntu 16.04+
    * Debian 9+
    * CentOS 7
    * Red Hat Enterprise Linux (RHEL) 7
    * Fedora 25+
    * HypriotOS v1.0.1+
    * Container Linux (tested with 1800.6.0)
* 2 GB or more of RAM per machine (any less will leave little room for your apps)
* 2 CPUs or more
* Full network connectivity between all machines in the cluster (public or private network is fine)
* Unique hostname, MAC address, and product_uuid for every node. Click here for more details.
* Certain ports are open on your machines. Click here for more details.
* Swap disabled. You MUST disable swap in order for the kubelet to work properly.
* Full network connectivity between all machines in the cluster (public or private network)
* sudo privileges on all machines
* SSH access from one device to all nodes in the system
* `kubeadm` and `kubelet` installed on all machines. `kubectl` is optional.
   
For the external etcd cluster only, you also need:
* Three additional machines for etcd members   
   
   
   
   
   
   
   
   











