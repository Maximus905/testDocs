* yum-utils must be installed

1. ```
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```
2. ```
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
3. Set SELinux in permissive mode (effectively disabling it)
    * `setenforce 0`
    * ```
      sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
      ```
4. `yum check-update -y`
5. `yum install -y docker-ce`
6. `systemctl enable --now docker`
7. `sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`
8. `systemctl enable --now kubelet`
9. `swapoff <my_swap_device_or_file> (also swap should be disabled in /etc/fstab )`
10. ```
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