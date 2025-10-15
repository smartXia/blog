---
title: k8s-kubeadmin安装
categories:
  - k8s
date: 2024-03-18 11:30:15
tags:
  - k8s
cover: /img/k8s.jpg
---

通过kubeadm工具，部署k8s集群。操作步骤如下：

1. 准备工作：

2. 1. 配置yum源，repo(防被墙)
   2. 安装常用工具，同步时间
   3. 关闭防火墙，将SELinux配置为Permissive模式，关闭swap
   4. 加载ipvs模块，优化内核

3. 在所有机器上安装docker

4. 在所有机器上安装kubeadm, kubelet, kubectl

5. 部署集群Master节点

6. 部署集群工作节点

7. 安装CNI网络插件



- 一个Kubernetes集群Master节点。k8s官网现在将master节点称为control plane node(控制平面节点)
- 一个Kubernetes集群Slave节点。k8s官网叫worker node(工作节点)。下文中Slave节点和工作节点含义含义相同。

| hostname | ip         | 备注            |                               |
| -------- | ---------- | --------------- | ----------------------------- |
| master   | master.k8s | 192.168.246.133 | k8s主节点(control plane node) |
| slave    | slave.k8s  | 192.168.246.132 | k8s 工作节点(worker node)     |

## 0. **系统要求**

安装之前，请确保操作系统满足如下要求：

1. Linux内核操作系统，如CentOS，Ubuntu等
2. 至少2 CPU， 2GB
3. 集群中所有机器之间的网络必须是通的(公共或私有网络都可以)。
4. 每个节点都有唯一的主机名、MAC地址和product_uuid
5. 部署时要保证能连外网

```shell
## 查看操作系统
cat /proc/version
hostnamectl

## 查看IP和MAC命令
ip link
ifconfig -a

## 查看product_uuid
sudo cat /sys/class/dmi/id/product_uuid
```

## **1. 准备工作**

**在所有节点上运行**

### **1.1 配置yum源，repo**

由于众所周知的原因，为防止在安装时出现资源下载失败的问题，在所有节点上配置yum源，repo。shell如下



```shell
# yum源
curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# docker repo
curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# kubernetes repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 缓存
yum clean all && yum makecache
```

### **1.2 安装常用的工具，同步时间**



```shell
yum -y install tree vim wget bash-completion bash-completion-extras lrzsz net-tools sysstat iotop iftop htop unzip nc nmap telnet bc  psmisc httpd-tools ntpdate

# 时区修改,如果/etc/localtime有软连接,不是Shanghai,可以直接删除,在软链接
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ntpdate ntp2.aliyun.com            # 同步阿里云服务器上的时间.
/sbin/hwclock --systohc            # 写入到bios系统    
```

### **1.3 检查防火墙是否关闭，将SELinux配置为Permissive模式，关闭swap**



```shell
## 查看防火墙状态
sudo firewall-cmd --state

## 如果防火墙是running，关闭防火墙
sudo systemctl disable firewalld && systemctl stop firewalld

## Set SELinux to permissive mode.将SELinux配置为Permissive模式
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

## 也可以直接关闭SELinux 
#sudo setenforce 0
#sudo sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config

# 临时关闭swap。如果不关闭，kubelet会启动失败
sudo swapoff -a
# 永久防止开机自动挂载swap
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### **1.4 加载ipvs模块，优化内核**

如下参数不修改，会导致`kubeadm init`运行失败。

```shell
# 加载ipvs模块
modprobe br_netfilter
modprobe -- ip_vs
modprobe -- ip_vs_sh
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- nf_conntrack_ipv4

# 验证ip_vs模块
lsmod |grep ip_vs
ip_vs_wrr              12697  0 
ip_vs_rr               12600  0 
ip_vs_sh               12688  0 
ip_vs                 145458  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          139264  2 ip_vs,nf_conntrack_ipv4
libcrc32c              12644  3 xfs,ip_vs,nf_conntrack

# 内核文件 
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
vm.max_map_count=262144
EOF

# 生效并验证内核优化
sysctl -p /etc/sysctl.d/k8s.conf
```

## **2. 安装docker**

*在所有节点上运行*

### **2.1 安装启动docker**

注意：安装docker需要root权限。

```shell
## 1. yum 安装
## 运行sudo yum install docker-ce 也是可以的，docker-ce依赖了docker-ce-cli, containerd.io, docker-buildx-plugin, docker-compose-plugin。这些依赖会同步install
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

## 2. 配置docker
cat <<EOF > /etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://registry.hub.docker.com",
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
} 
EOF
   
## 3. 启动docker
systemctl start docker
```

验证是否安装正确

```sh
## 4. docker运行正常
[shirley@master k8s_install]$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2023-10-10 09:23:41 CST; 19s ago
     Docs: https://docs.docker.com
 Main PID: 16381 (dockerd)
    Tasks: 8
   Memory: 27.4M
   CGroup: /system.slice/docker.service
           └─16381 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/con...
[shirley@master k8s_install]$

## 运行docker命令，验证镜像下载正常，容器运行正常
[shirley@slave k8s_install]$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete
Digest: sha256:4f53e2564790c8e7856ec08e384732aa38dc43c52f02952483e3f003afbf23db
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
... ...
```

### **2.2 配置containerd中pause镜像地址**



为了防止安装过程中出现pause镜像下载失败的问题，建议运行`containerd config dump > /etc/containerd/config.toml `命令，将当前配置导出到文件，并修改`sandbox_image`配置。

```shell
## 如果没有/etc/containerd/config.toml文件，将默认配置导出到/etc/containerd/config.toml。
containerd config default > /etc/containerd/config.toml

## 修改配置文件/etc/containerd/config.toml， 更改sandbox_image配置
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"

## PS: 如果生成的/etc/containerd/config.toml中没有如上配置项，可以运行如下命令导出当前所有配置项后再修改文件/etc/containerd/config.toml
# containerd config dump > /etc/containerd/config.toml

## 重启containerd
systemctl restart containerd
```

## **3. 部署kubeadm, kubelet, kubectl**

kubeadm：启动k8s集群的工具

kubelet: 该组件在集群中的所有机器上运行，并执行启动pod和容器之类的任务。

kubectl: 与集群通信的工具。可以只在master节点上安装。

```text
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

## 开机启动kubelet
sudo systemctl enable --now kubelet
```

验证

```sh
# 查看kubeadm版本
[root@master ~]# sudo kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.2", GitCommit:"89a4ea3e1e4ddd7f7572286090359983e0387b2f", GitTreeState:"clean", BuildDate:"2023-09-13T09:34:32Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
```

## **4.** 初始化Master节点（kubeadm init）

*在Master节点运行*

### **4.1. 更改配置文件**

`kubeadm init`命令用于初始化master节点。kubeadm init 的参数可以通过**命令行**或**yaml文件**进行配置。本文介绍如何通过yaml文件进行配置。可以通过`kubeadm config print init-defaults`命令得到一份默认配置，然后对其进行修改。

```sh
 kubeadm config print init-defaults > kubeadm.yaml
```

对kubeadm.yaml进行编辑，修改内容如下：

1. 修改`advertiseAddress`为master IP地址
2. `imageRepository`修改为`registry.aliyuncs.com/google_containers`，防止镜像拉不下来
3. 建议将`networking.podSubnet`修改为`10.244.0.0/16`， 和后续安装的flannel CNI 插件的默认配置保持一致。

```text
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.246.133    ## change the IP of apiserver.
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers   ## change imageRepository to aliyun.
kind: ClusterConfiguration
kubernetesVersion: 1.28.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16  ## add this line to config POD network. Same with CNI config.
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

如上配置文件等价于命令行：

```text
kubeadm init \
  --apiserver-advertise-address=192.168.246.133 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.28.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 
```

- –apiserver-advertise-address 集群通告地址
- –image-repository 由于默认拉取镜像地址[http://k8s.gcr.io](https://link.zhihu.com/?target=http%3A//k8s.gcr.io)国内无法访问，这里指定阿里云镜像仓库地址
- –kubernetes-version K8s版本，与上面安装的一致
- –service-cidr 集群内部虚拟网络，Pod统一访问入口
- –pod-network-cidr Pod网络，**与下面部署的CNI网络组件yaml中保持一致**



### **4.2 提前pull镜像（可选）**

为了使后续安装更快，建议先把安装需要的镜像pull下来。

```sh
## 验证配置文件格式是否正确
[shirley@master k8s_install]$ kubeadm config validate --config kubeadm.yaml
ok

## 查看需要下载哪些镜像。需要关注下载的repository地址是否正确。
## 如上在kubeadm.yaml文件配置了imageRepository：registry.aliyuncs.com/google_containers，因此images会从aliyun的repo下载。
[shirley@master k8s_install]$ kubeadm config images list --config kubeadm.yaml
registry.aliyuncs.com/google_containers/kube-apiserver:v1.28.0
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.28.0
registry.aliyuncs.com/google_containers/kube-scheduler:v1.28.0
registry.aliyuncs.com/google_containers/kube-proxy:v1.28.0
registry.aliyuncs.com/google_containers/pause:3.9
registry.aliyuncs.com/google_containers/etcd:3.5.9-0
registry.aliyuncs.com/google_containers/coredns:v1.10.1

 
## pull镜像。pull镜像有点慢，第一个镜像pull成功后才有日志输出。命令运行后发现没有日志不要着急，多等一会。
[shirley@master k8s_install]$ sudo kubeadm config images pull --config kubeadm.yaml
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.28.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.28.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.28.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.28.0
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.9
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.9-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.10.1
```

### **4.3 kubeadm init初始化**

运行`kubeadm init --config kubeadm.yaml`， 当看到`Your Kubernetes control-plane has initialized successfully!`时表示安装成功。

```sh
[root@master k8s_install]# kubeadm init --config kubeadm.yaml
... ...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.246.133:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:9f7d37c768e658119242dfb49675eaeb3cbdbb7d191526bfa197dd92373b40ab
```

**PS：最好将最后一行log需要记下来，worker节点安装会用到。**

根据提示，退出root账户后运行命令

```sh
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



### **4.4 验证**

验证master节点是否部署成功。

```sh
## kubectl 运行正常. STATUS=NotReady是因为CNI插件还没装。
[shirley@master k8s_install]$ kubectl get node
NAME   STATUS     ROLES           AGE   VERSION
node   NotReady   control-plane   20m   v1.28.2

## 通过crictl命令可以查看到运行的container
[shirley@master k8s_install]$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
955b0c87ad621       ea1030da44aa1       26 minutes ago      Running             kube-proxy                0                   78efbee65dfac       kube-proxy-kp9kw
f69d8c3246904       73deb9a3f7025       26 minutes ago      Running             etcd                      0                   77180bc7ff0a8       etcd-node
3efba65f263d3       f6f496300a2ae       26 minutes ago      Running             kube-scheduler            0                   f89fb4bb60e2e       kube-scheduler-node
5dfb28390f30b       4be79c38a4bab       26 minutes ago      Running             kube-controller-manager   0                   b716cb4652e1c       kube-controller-manager-node
b8cfce31fa842       bb5e0dde9054c       26 minutes ago      Running             kube-apiserver            0                   006db1ce43cfe       kube-apiserver-node
```

这里有大家可能有个疑惑，为什么docker ps看不到运行的容器

```sh
## 运行docker命令，发现没有container
[shirley@master k8s_install]$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

这是因为kubernetes使用的containerd作为容器运行时，而不是Docker engine. kubernetes支持4中容器运行时：



| Runtime                                             | Path to Unix domain socket（CRI socket）   |
| --------------------------------------------------- | ------------------------------------------ |
| containerd                                          | unix:///var/run/containerd/containerd.sock |
| CRI-O                                               | unix:///var/run/crio/crio.sock             |
| Docker Engine (using cri-dockerd)                   | unix:///var/run/cri-dockerd.sock           |
| Mirantis Container Runtime (MCR)(using cri-dockerd) | unix:///var/run/cri-dockerd.sock           |

By default, Kubernetes uses the Container Runtime Interface (CRI) to interface with your chosen container runtime.

默认情况下，kubernetes会用CRI找到选择的容器运行时。在我们安装docker时，安装了containerd，因此k8s找到了containerd作为容器运行时。在kubeadm.yam文件中也能看到相应的配置

```shell
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
```

为什么k8s不选用docker engine作为容器运行时？因为如果使用docker engine, 还需要安装cri-dockerd，才能作为容器时被k8s识别。而如上操作并未安装。

此外，k8s推荐直接使用containerd作为容器运行时。

## **5. 初始化工作节点**

### **5.1 运行kubeadm join将工作节点加入集群**

在master节点kubeadm init安装完成后，会有如下类似log

```sh
.. ... 
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.246.133:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:9f7d37c768e658119242dfb49675eaeb3cbdbb7d191526bfa197dd92373b40ab
```

将上述命令粘贴到工作节点，将工作节点添加到集群

```shell
[root@slave ~]# kubeadm join 192.168.246.133:6443 --token abcdef.0123456789abcdef \
>         --discovery-token-ca-cert-hash sha256:9f7d37c768e658119242dfb49675eaeb3cbdbb7d191526bfa197dd92373b40ab
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "slave.k8s" could not be reached
        [WARNING Hostname]: hostname "slave.k8s": lookup slave.k8s on 192.168.246.2:53: no such host
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

### **5.2 验证**

到master节点运行kubelet get node,可以看到加进来的node

```shell
[shirley@master k8s_install]$ kubectl get nodes
NAME        STATUS     ROLES           AGE    VERSION
node        NotReady   control-plane   121m   v1.28.2
slave.k8s   NotReady   <none>          28s    v1.28.2
```

当前STATUS都是NotReady，这是因为还没有安装网络插件CNI

### **5.3 忘记token怎么办**

如果master节点安装时，没有记录下token，或token超时(默认24小时)，可以运行如下命令重新生成

```shell
[shirley@master k8s_install]$ kubeadm token create --print-join-command
kubeadm join 192.168.246.133:6443 --token by8q23.65btteq9iud7ypso --discovery-token-ca-cert-hash sha256:9f7d37c768e658119242dfb49675eaeb3cbdbb7d191526bfa197dd92373b40ab
```

## **6. 安装CNI网络插件**

Kubernetes 它需要网络插件来提供集群内部和集群外部的网络通信。以下是一些常用的 k8s 网络插件：

- Flannel：Flannel 是最常用的 k8s 网络插件之一，它使用了虚拟网络技术来实现容器之间的通信，支持多种网络后端，如 VXLAN、UDP 和 Host-GW。
- Calico：Calico 是一种基于 BGP 的网络插件，它使用路由表来路由容器之间的流量，支持多种网络拓扑结构，并提供了安全性和网络策略功能。
- Canal：Canal 是一个组合了 Flannel 和 Calico 的网络插件，它使用 Flannel 来提供容器之间的通信，同时使用 Calico 来提供网络策略和安全性功能。
- Weave Net：Weave Net 是一种轻量级的网络插件，它使用虚拟网络技术来为容器提供 IP 地址，并支持多种网络后端，如 VXLAN、UDP 和 TCP/IP，同时还提供了网络策略和安全性功能。
- Cilium：Cilium 是一种基于 eBPF (Extended Berkeley Packet Filter) 技术的网络插件，它使用 Linux 内核的动态插件来提供网络功能，如路由、负载均衡、安全性和网络策略等。
- Contiv：Contiv 是一种基于 SDN 技术的网络插件，它提供了多种网络功能，如虚拟网络、网络隔离、负载均衡和安全策略等。
- Antrea：Antrea 是一种基于 OVS (Open vSwitch) 技术的网络插件，它提供了容器之间的通信、网络策略和安全性等功能，还支持多种网络拓扑结构。

| 提供商  | 网络模型                    | 路线分发 | 网络策略 | 网格 | 外部数据存储    | 加密 | Ingress/Egress 策略 |
| ------- | --------------------------- | -------- | -------- | ---- | --------------- | ---- | ------------------- |
| Canal   | 封装 (VXLAN)                | 否       | 是       | 否   | K8s API         | 是   | 是                  |
| Flannel | 封装 (VXLAN)                | 否       | 否       | 否   | K8s API         | 是   | 否                  |
| Calico  | 封装（VXLAN，IPIP）或未封装 | 是       | 是       | 是   | Etcd 和 K8s API | 是   | 是                  |
| Weave   | 封装                        | 是       | 是       | 是   | 否              | 是   | 是                  |
| Cilium  | 封装 (VXLAN)                | 是       | 是       | 是   | Etcd 和 K8s API | 是   | 是                  |

Calico 和 Flannel都是常用的CNI，如下介绍如何安装flannel网络插件

### **6.1 安装flannel网络插件**

1. 下载kube-flannel.yml

```shell
## 1. 下载kube-flannel.yml
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```



`kube-flannel.yaml`文件中，需要注意，net-conf.json里的network要和kubeadm.yaml里配置的networking.podSubnet相同

```shell
... ...
  net-conf.json: |
    {
      ### 这里的network和kubeadm.yaml里配置的networking.podSubnet相同 
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
... ...
```

2. 容器部署flannel

```shell
[shirley@master k8s_install]$ kubectl apply -f kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

### **6.2 验证CNI**

安装后，运行`kubectl -n kube-system get pod -o wide`， 可以看到在master节点和slave节点分别运行了一个kube-proxy

```shell
## 每个节点上都运行了kube-proxy
[shirley@master k8s_install]$ kubectl -n kube-system get pod -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP                NODE        NOMINATED NODE   READINESS GATES
coredns-66f779496c-d8rws       1/1     Running   0          4h40m   10.244.0.3        node        <none>           <none>
coredns-66f779496c-fzmjm       1/1     Running   0          4h40m   10.244.0.2        node        <none>           <none>
etcd-node                      1/1     Running   0          4h41m   192.168.246.133   node        <none>           <none>
kube-apiserver-node            1/1     Running   0          4h41m   192.168.246.133   node        <none>           <none>
kube-controller-manager-node   1/1     Running   0          4h41m   192.168.246.133   node        <none>           <none>
kube-proxy-bpv8d               1/1     Running   0          160m    192.168.246.132   slave.k8s   <none>           <none>
kube-proxy-kp9kw               1/1     Running   0          4h40m   192.168.246.133   node        <none>           <none>
kube-scheduler-node            1/1     Running   0          4h41m   192.168.246.133   node        <none>           <none>


## node状态显示为Ready
[shirley@master k8s_install]$ kubectl get node
NAME        STATUS   ROLES           AGE     VERSION
node        Ready    control-plane   4h38m   v1.28.2
slave.k8s   Ready    <none>          158m    v1.28.2
```

自此，集群搭建完成。



## **其他说明**

另外，在初始安装的Master节点上也启动了`kubelet`和`kube-proxy`，在默认情况下并不参与工作负载的调度。如果希望Master节点也作为Node角色，则可以运行下面的命令（删除Master节点的：`node-role.kubernetes.io/control-plane:NoSchedule`），让Master节点也成为一个Node：

```text
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```