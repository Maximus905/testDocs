# Operator Box
## Software requirements:
* __kubectl__ (Kubernetes command-line tool)
* __Helm 3+__ utility for th2 components deployment into kubernetes
* __Chrome 75__ or newer 

# QA Box
* __Chrome 75__ or newer 

#Cassandra Box
* __Cassandra 3.11.6__<br>
    Cassandra in-cluster installed in kubernetes (developer mode)<br>
    Cassandra cluster installed separately (production mode)<br>
## Apache Cassandra cluster hardware requirements
   Though it is possible to use Cassandra single-node installation, generally it’s recommended to setup at least 3-nodes cluster. Requirements to each node are the same.
   
 | Apache Cassandra node | Memory (MB) | CPU (Сores) | Disk space (GB) |
 |-----|------|---------|-------------|  
 | Cassandra node_n | 4000 MB | 2 | 15 GB for “/ “ mount + 200 GB for “/var“ mount |
 
#External Resources
* __Git repositories__ for apps and infrastructure code

# Test Platform Box
Components calculations:  
__th2 env = Base + Core + Building blocks + Custom + Cassandra *__<br>

| Base & Core Components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|
| th2 infra | 1000 MB | 800 m | helm, infra-mgr, infra-editor, infra-operator |
| th2 core | 2500 MB | 2000 m | mstore, estore, rpt-provider, rpt-viewer |
| Rabbitmq replica 1 | 2000 MB | 1000 m | need to test |
| Monitoring | 1500 MB | 2000 m | |
| Other supporting components | 500 MB | 250 m | e.g. in-cluster CD system, ingress and etc |
| __Total:__ | __7500 MB__ | __6050 m__ |  |

| Custom & Building blocks components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|
| th2 in-cluster connectivity services | 200 MB * n | 200 m * n | Depends on number of connectivity instances. 1 Connectivity service * n e.g. if we have 10 connectivity instances: 200 MB * 10 = 2000 MB|
| th2 codec, act | 200 MB * n | 200 m * n |  |
| th2 check1 |  |  |  |
| th2 Java read  | 200 MB * n | 200 m * n |  |
| th2 recon | 200 MB * n | 200 m * n | cacheSize = (podMemoryLimit - 70MB) / (AvrRawMsgSize * 10 * (SUM(number of group in rule))) |
| th2 check2 | 800 MB * n  | 200 m * n |  |
| th2 hand | 300 MB * n | 400 m * n |  |
| __Total:__ | __1900 * n__ | __1400 * n__ |  |

## Software requirements:
* Kubernetes Cluster accessible from other boxes 


# Unallocated stuff!!!

[Kubernetes - before you begin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin)

* __Docker CE__ 
installed with the following parameters in /etc/docker/daemon.json
```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m" 
  },
  "storage-driver": "overlay2"
}
```
* [__Overlay2 storage driver prerequisites__](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#prerequisites)
* __Docker registry__ with push permissions for storing containerized application images
* __Kubernetes__
  Kubernetes cluster installed (single master node as development mode, master and 2+ workers as production mode) with the [flannel CNI plugin](https://coreos.com/flannel/docs/latest/kubernetes.html#the-flannel-cni-plugin) . [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
    
  Flannel CNI installation:
  ```
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  ```
  
  If you want to be able to schedule Pods on the control-plane node, for example for a single-machine Kubernetes cluster for development, run:
  ```
  kubectl taint nodes --all node-role.kubernetes.io/master-
  ```
* __Python__
* __JAVA 11__	

## Hardware requirements:
### High availability configuration cluster

* Three machines that meet kubeadm’s minimum requirements for the workers
* One or more machines running one of:
    * Ubuntu 16.04+
    * Debian 9+
    * CentOS 7
    * Red Hat Enterprise Linux (RHEL) 7
    * Fedora 25+

* Full network connectivity between all machines in the cluster (public or private network is fine)
* Unique hostname, MAC address, and product_uuid for every node. Click here for more details.
* Certain ports are open on your machines. Click here for more details.
* Swap disabled. You MUST disable swap in order for the kubelet to work properly.
* Full network connectivity between all machines in the cluster (public or private network)
* sudo privileges on all machines
* SSH access from one device to all nodes in the system
* `kubeadm` and `kubelet` installed on all machines. `kubectl` is optional.
