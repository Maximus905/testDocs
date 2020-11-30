* `yum-utils` must be installed

* ```
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```
* ```
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
   exclude=kubelet kubeadm kubectl
   EOF
   ```
* Set SELinux in permissive mode (effectively disabling it)
    * `setenforce 0`
    * ```
      sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
      ```
* `yum check-update -y`
* `yum install -y docker-ce`
* `systemctl enable --now docker`
* `sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`
* `systemctl enable --now kubelet`
* `swapoff <my_swap_device_or_file> (also swap should be disabled in /etc/fstab )`
* ```
  cat <<EOF | sudo tee /etc/docker/daemon.json
  {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "100m"
    },
    "storage-driver": "overlay2"
  }
  EOF
  ```
* `systemctl restart docker`
* `systemctl disable firewalld && systemctl stop firewalld`
* [Letting iptables see bridged traffic](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#letting-iptables-see-bridged-traffic)
    ```
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    ```
* `sudo sysctl --system`
* `echo 1 > /proc/sys/net/ipv4/ip_forward`
* `kubeadm init --pod-network-cidr=10.244.0.0/16`
* `mkdir -p <exactpro_user_home_dir>/.kube`
* `cp -i /etc/kubernetes/admin.conf <exactpro_user_home_dir>/.kube/config`
* `chown $(id <exactpro_user> -u):$(id <exactpro_user> -g) <exactpro_user_home_dir>/.kube/config`
* `openssl genrsa -out th2-adm.key 2048`
* `openssl req -new -key th2-adm.key -out th2-adm.csr -subj "/CN=th2-adm"`
* `openssl x509 -req -in th2-adm.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out th2-adm.crt -days 500`
* `mkdir <exactpro_user_home_dir>/.certs`
* `mv th2-adm.crt th2-adm.key <exactpro_user_home_dir>/.certs/`
* `chown $(id <exactpro_user> -u):$(id <exactpro_user> -g) <exactpro_user_home_dir>/.certs/*`
* `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml`
* `kubectl taint node mymasternode node-role.kubernetes.io/master:NoSchedule-`
* ```
  cat <<EOF | sudo tee th2-adm_clusterRoleBinding.yaml
  kind: ClusterRoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: th2-adm
    namespace: default
  subjects:
  - kind: User
    name: th2-adm
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  EOF
  ```
* `kubectl apply -f th2-adm_clusterRoleBinding.yaml`
