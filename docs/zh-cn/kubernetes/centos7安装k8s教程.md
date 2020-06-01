kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

```
# 创建一个 Master 节点
kubeadm init

# 将一个 Node 节点加入到当前集群中
kubeadm join <Master节点的IP和端口 >
```

## 1. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

## 2. 学习目标

1. 在所有节点上安装Docker和kubeadm
2. 部署Kubernetes Master
3. 部署容器网络插件
4. 部署 Kubernetes Node，将节点加入Kubernetes集群中
5. 部署Dashboard Web页面，可视化查看Kubernetes资源


## 3. 准备环境
```python
三台主机
IP:  192.168.153.34     主机名：k8s-master    系统： centos 7.6      配置： 2C 2G
IP:  192.168.153.35     主机名：k8s-node1     系统： centos 7.6      配置： 2C 2G
IP:  192.168.153.36     主机名：k8s-node2     系统： centos 7.6      配置： 2C 2G
```

```python
三台主机全部关闭防火墙:
# systemctl stop firewalld
# systemctl disable firewalld

三台主机全部关闭selinux：
# sed -i 's/enforcing/disabled/' /etc/selinux/config
# setenforce 0

三台主机全部关闭swap：
# swapoff -a  # 临时关闭
# vim /etc/fstab 注释到swap那一行 # 永久关闭

添加主机名与IP对应关系(三台主机都执行)：
# cat >> /etc/hosts << EOF
192.168.153.34 k8s-master
192.168.153.35 k8s-node1
192.168.153.36 k8s-node2
EOF

设置 192.168.153.34主机主机名
# hostnamectl set-hostname  k8s-master

设置 192.168.153.35主机主机名
# hostnamectl set-hostname  k8s-node1

设置 192.168.153.36主机主机名
# hostnamectl set-hostname  k8s-node2

将桥接的IPv4流量传递到iptables的链(三台主机都执行)：
# cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# sysctl --system
```

## 4. 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

### 4.1 安装Docker

每台机器上安装Docker，建议使用18.09版本。

```python
下载阿里云的docker yum源,并安装
# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# yum -y install docker-ce-18.09.9-3.el7

启动docker,并设置docker开机自启
# systemctl start docker
# systemctl enable docker
```

镜像下载加速
进入到阿里云后台控制界面
![image_1e2kr61hi1oeq2ck1ktq7nssk79.png-141.4kB][1]

点击产品与服务,选择容器镜像服务
![image_1e2kraqj12i07a1mm011kcc8v13.png-73.9kB][2]

左下角有个镜像加速器
![image_1e2krfndu1g191sag2tb1uu64q1g.png-86.4kB][3]

选择自己的加速器地址，修改就行了
```sh
 cat >  /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://6ze43vnb.mirror.aliyuncs.com"]
}
```

设置cgroup驱动，推荐systemd：
```sh
# cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://6ze43vnb.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

重启docker
```sh
systemctl daemon-reload
systemctl restart docker
```


### 4.2 添加阿里云YUM软件源

```sh
# cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 4.3 安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号部署：

```sh
# yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0
# systemctl enable kubelet
```

## 5. 部署Kubernetes Master
在192.168.153.34（Master）执行。
```
# kubeadm init \
  --apiserver-advertise-address=192.168.153.34 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.16.0 \
  --service-cidr=10.1.0.0/16 \
  --pod-network-cidr=10.244.0.0/16
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址

执行完,出现如图所示
![image_1e2ku14lmdvd4n16i49uf1nr29.png-136.5kB][4]

使用kubectl工具(192.168.153.34执行)：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


## 6. 加入Kubernetes Node

在192.168.153.35/36（Node）分别执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：
```
kubeadm join一行是master端 kubeadm init 生成的,实际你的会变化
# kubeadm join 192.168.153.34:6443 --token 6qf11n.pdyzp2zki1ydb2fc \
    --discovery-token-ca-cert-hash sha256:e9055d8b3cfcf40330124f5da18e820ebcb6eb9ff28eb64c0f593e0fb154b755
```
## 7. 安装Pod网络插件（CNI）
```
# kubectl apply -f kube-flannel.yml
# kubectl get pods -n kube-system
```
确保能够访问到quay.io这个registery。

如果下载失败，可以改成这个镜像地址：lizhenliang/flannel:v0.11.0-amd64


## 8. 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```
# kubectl create deployment nginx --image=nginx
# kubectl expose deployment nginx --port=80 --type=NodePort
# kubectl get pod,svc
```

访问地址：http://NodeIP:Port  

## 9. 部署 Dashboard

```
# kubectl apply -f dashboard.yaml
# kubectl get pods -n kubernetes-dashboard
```

访问地址：http://NodeIP:30001

创建service account并绑定默认cluster-admin管理员集群角色：

```
# kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
# kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
# kubectl describe secrets -n kubernetes-dashboard $(kubectl -n kubernetes-dashboard get secret | awk '/dashboard-admin/{print $1}')
```
使用输出的token登录Dashboard。

最后的UI界面
![image_1e2kv5hh41joqdp715dr1jfko09m.png-169.3kB][5]


  [1]: http://static.zybuluo.com/sjl--3306/infx1q3dgmspgstngku4cwdb/image_1e2kr61hi1oeq2ck1ktq7nssk79.png
  [2]: http://static.zybuluo.com/sjl--3306/jiv7slaem8w0y68ii9srzjce/image_1e2kraqj12i07a1mm011kcc8v13.png
  [3]: http://static.zybuluo.com/sjl--3306/1wwve1m0al9iwinuqrtxlbez/image_1e2krfndu1g191sag2tb1uu64q1g.png
  [4]: http://static.zybuluo.com/sjl--3306/df4qzms8bgm1nsuoe520nqj9/image_1e2ku14lmdvd4n16i49uf1nr29.png
  [5]: http://static.zybuluo.com/sjl--3306/4izcm8woidrix6cc1alxq14l/image_1e2kv5hh41joqdp715dr1jfko09m.png
