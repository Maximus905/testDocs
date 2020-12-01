Сomponents calculations:  
__th2 env = Base + Core + Building blocks + Custom + Cassandra *__<br>

| Base & Core Components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|
| th2 infra | 1000 MB | 800 m | helm, infra-mgr, infra-editor, infra-operator |
| th2 core | 2500 MB | 2000 m | mstore, estore, rpt-provider, rpt-viewer |
| Rabbitmq replica 1 | 2000 MB | 1000 m | need to test |
| Monitoring | 1500 MB | 2000 m | |
| Other supporting components | 500 MB | 250 m | e.g. in-cluster CD system, ingress and etc |

| Custom & Building blocks components | Memory (MB) | CPU (millicores) | Comment |
|-----|------|---------|-------------|
| th2 in-cluster connectivity services | 200 MB * n | 200 m * n | Depends on number of connectivity instances. 1 Connectivity service * n e.g. if we have 10 connectivity instances: 200 MB * 10 = 2000 MB|
| th2 codec, act | 200 MB * n | 200 m * n |  |
| th2 check1 |  |  |  |
| th2 C++ read | 2000 MB * n | 200 m * n |  |
| th2 Java read  | 200 MB * n | 200 m * n |  |
| th2 recon | 200 MB * n | 200 m * n | cacheSize = (podMemoryLimit - 70MB) / (AvrRawMsgSize * 10 * (SUM(number of group in rule))) |
| th2 check2 | 800 MB * n  | 200 m * n |  |
| th2 hand | 300 MB * n | 400 m * n |  |

## Apache Cassandra cluster hardware requirements
   Though it is possible to use Cassandra single-node installation, generally it’s recommended to setup at least 3-nodes cluster. Requirements to each node are the same.
   
 | Apache Cassandra node | Memory (MB) | CPU (Сores) | Disk space (GB) |
 |-----|------|---------|-------------|  
 | Cassandra node_n | 4000 MB | 2 | 15 GB for “/ “ mount + 200 GB for “/var“ mount |
   
## Software requirements:
* [Kubernetes - before you begin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin)
* Docker registry access from cluster nodes - nexus.exp.exactpro.com:9000, hub.docker.com
* Test box access to - github.com, index.docker.io, quay.io, k8s.gcr.io, grafana.github.io, charts.bitnami.com, kubernetes-charts.storage.googleapis.com, mirror.centos.org, puppet.exp.exactpro.com
* Chrome 75 or newer
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
* __Git repositories__ for apps and infrastructure code
* __Helm 3+__ utility for th2 components deployment into kubernetes
* __Cassandra 3.11.6__<br>
    Cassandra in-cluster installed in kubernetes (developer mode)
    Cassandra cluster installed separately (production mode)
    	
    _Cassandra standalone one-node installation for Centos 7_:
    ```
    yum install java-1.8.0-openjdk
    ```
    python 2.7.5 or higher is also required
    ```
    cat <<EOF | sudo tee /etc/yum.repos.d/cassandra.repo
    [cassandra]
    name=Apache Cassandra
    baseurl=https://downloads.apache.org/cassandra/redhat/311x/
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://downloads.apache.org/cassandra/KEYS
    EOF
    ```
    ```
    yum install cassandra
    ```
    Your `/etc/cassandra/cassandra.yaml` should also contain the following settings:
    ```
    authenticator: PasswordAuthenticator
    authorizer: org.apache.cassandra.auth.CassandraAuthorizer
    auto_bootstrap: true
    cluster_name: test-Universum
    commitlog_directory: "/var/lib/cassandra/commitlog"
    commitlog_sync: periodic
    commitlog_sync_period_in_ms: 10000
    data_file_directories:
    - "/var/lib/cassandra/data"
    endpoint_snitch: GossipingPropertyFileSnitch
    hints_directory: "/var/lib/cassandra/hints"
    listen_interface: <YOUR NETWORK INTERFACE for example eth0>
    num_tokens: 256
    partitioner: org.apache.cassandra.dht.Murmur3Partitioner
    saved_caches_directory: "/var/lib/cassandra/saved_caches"
    read_request_timeout_in_ms: 30000
    range_request_timeout_in_ms: 20000
    seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
      - seeds: <IP ADDRESS OF THIS NODE>
    start_native_transport: true
    start_rpc: false
    ```
    ```
    service cassandra start
    ```
    ```
    chkconfig cassandra on
    ```

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

