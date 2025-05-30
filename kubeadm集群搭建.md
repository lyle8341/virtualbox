# kubeadm

### 安装前准备
+ 1.配置hosts文件
+ 2.关闭swap
  ```shell
  sudo swapoff -a
  # \s 表示空白字符
  # (/\sswap\s/)匹配独立单词 swap（前后必须有空白字符）
  # ^ 表示行首。#? 表示 0 或 1 个 # 字符[(* 0个或多个) (+ 1个或多个)]
  sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
  ```
+ 3.Set SELinux to permissive mode:
  ```shell
  # Set SELinux in permissive mode (effectively disabling it)
  sudo setenforce 0
  # 替换指定内容: sed -i 's/新的内容/要替换内容/'
  sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
  ```
+ 4.关闭防火墙
  + 或者放开端口`[6443 10250]`
+ 5.~~修改内核参数~~
+ 6.时间同步(否则添加worker报错failed to verify certificate)
  ```shell
  dnf -y install chrony
  # 内网部署的话，应该配置内网的时间同步服务
  echo "server ntp.aliyun.com iburst" >> /etc/chrony.conf
  systemctl enable --now chronyd
  systemctl restart chronyd
  # 同步时间
  chronyc makestep
  # 检查同步状态
  chronyc sources -v
  # 查看时间：timedatectl
  ```


---
### 安装容器运行时
+ 1.Enable IPv4 packet forwarding
  ```shell
  # sysctl params required by setup, params persist across reboots
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.ipv4.ip_forward = 1
  EOF
  
  # Apply sysctl params without reboot
  sudo sysctl --system
  ```

+ 2.设置docker仓库源（里面包含 containerd）
  ```shell
  sudo dnf -y install dnf-plugins-core
  sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  ```
+ 3.安装containerd
  ```shell
  sudo dnf list containerd.io
  sudo dnf install containerd.io
  ```
+ 4.查看 cgroup version
  ```shell
  stat -fc %T /sys/fs/cgroup/
  
  #For cgroup v2, the output is cgroup2fs.
  #For cgroup v1, the output is tmpfs.
  #The systemd cgroup driver is recommended if you use cgroup v2.
  ```
+ 5.配置
  ```shell
  containerd config default > /etc/containerd/config.toml
  # SystemdCgroup = true
  #[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  #...
  #[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  #  SystemdCgroup = true
  ```
+ 6.启动
  ```shell
  systemctl daemon-reload
  systemctl enable containerd
  sudo systemctl restart containerd
  ```
+ 7.套接字路径
  ```shell
  /var/run/containerd/containerd.sock
  /run/containerd/containerd.sock #(link)
  ```


---


### 开始安装kubeadm等
+ 1.Kubernetes yum repository
  ```shell
  # This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
  enabled=1
  gpgcheck=1
  gpgkey=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/repodata/repomd.xml.key
  exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
  EOF
  ```
+ 2.Install kubelet, kubeadm and kubectl:
  ```shell
  sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
  ```
+ 3.(Optional) Enable the kubelet service before running kubeadm:
  ```shell
  sudo systemctl enable --now kubelet
  ```
+ 4.镜像准备
  ```shell
  # registry.k8s.io/kube-apiserver:v1.33.1
  kubeadm config images list # 查看需要哪些镜像
  # registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.33.1
  kubeadm config images list --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
  # 拉取镜像
  kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers
  ```
  

+ 5.查看下载的镜像
  > crictl images
+ 6.镜像重命名
  + ctr -n k8s.io i tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.33.1 registry.k8s.io/kube-apiserver:v1.33.1
+ 7.~~删除registry.aliyuncs.com开头的镜像~~(不能删除，因为删除后tag页一起被删除了)
  + crictl rmi registry.aliyuncs.com/google_containers/coredns:v1.12.0

+ 初始化
  + 如果是从国内镜像拉取需要指定参数（--image-repository）
    + kubeadm init --image-repository registry.aliyuncs.com/google_containers
  + 如果是标准的镜像（registry.k8s.io开头）
    + kubeadm init --apiserver-advertise-address=10.0.2.180 --pod-network-cidr=172.16.0.0/16
    + 初始化失败后，需要执行reset，否则再次init会报错
      + kubeadm reset
      + 查看日志报错原因（下载不了pause:3.8）
        + journalctl -u containerd.service(systemctl status containerd)
        + crictl pull registry.aliyuncs.com/google_containers/pause:3.8(下载并修改tag)
        + kubeadm token list(查看init生成的token)
        + openssl x509 -in /etc/kubernetes/pki/ca.crt -pubkey -noout |openssl pkey -pubin -outform DER |openssl dgst -sha256(获取discovery-token-ca-cert-hash)
  + [Kubectl autocomplete自动补全](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion)
    ```shell
    source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
    echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
    ```
+ 14.命令
  + kubectl get pod --all-namespaces(kubectl get pods -A)
  + kubectl get nodes(kubectl get node -o wide)
  + kubectl get ns(命名空间)
  + kubectl get pods -n [namespace]

+ 15.加入worker
  + kubeadm join 10.0.2.180:6443 --token....
+ 16.给节点打上标签
  + kubectl label nodes k8s-2 node-role.kubernetes.io/work=work
  + kubectl label nodes k8s-3 node-role.kubernetes.io/work=work
  + 效果如下：
    ```shell
    [root@k8s-1 ~]# kubectl get nodes
    NAME    STATUS     ROLES           AGE     VERSION
    k8s-1   NotReady   control-plane   40m     v1.33.1
    k8s-2   NotReady   work            4m29s   v1.33.1
    k8s-3   NotReady   work            4m19s   v1.33.1
    ```

### Installing a Pod network add-on
+ 1.your Pods can communicate with each other.
+ 2.Cluster DNS (CoreDNS) will not start up before a network is installed.

+ [Install Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)
  + 1.Install the Tigera operator and custom resource definitions.
  ```shell
  kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/operator-crds.yaml
  kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml
  #如果已经存在可以先 kubectl delete -f ....
  ```
  - 验证新创建的namespace
    - kubectl get ns
    - kubectl get pods -n tigera-operator
  + 2.下载yaml文件，修改网段
    ```shell
    wget https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
    或
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml -O
    ```
  + 3.Create the manifest to install Calico.
    + kubectl create -f custom-resources.yaml
  + 4.Verify Calico installation in your cluster.
    + watch kubectl get pods -n calico-system
  + 5.安装失败回退
    + kubectl delete -f https://raw....
    + **遇到问题，看容器日志，大概率是镜像下载失败**
    + **遇到问题，看容器日志，大概率是镜像下载失败**
  + 6.安装长时间失败总结
    ```shell
    我将proxy代理给了宿主机，这意味着我的Pod会把流量转发给宿主机，通过宿主机进行通信，
    而Pod要通信的对端IP地址正是我定义的Pod网段（10.96.0.0/12，172.16.0.0/16），
    这通过宿主机进行通信肯定是找不到对端的。
    ```


























