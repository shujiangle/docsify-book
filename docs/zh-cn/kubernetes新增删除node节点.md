# 1. kubernetes 新增node节点

标签（空格分隔）： k8s学习

---

## 1.1 master端执行
```
[root@k8s-master ~]# kubeadm token list
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION   EXTRA GROUPS
nih4fv.mys97k5ozjkm1nkk   22h       2020-03-10T14:54:32+08:00   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

604e2a4161b05050b5bf75583b0a7b8d9944e0d338eac20b869bcae1e82ecac5
```

### 1.1.1 新增node节点ip和主机名到hosts
```echo "192.168.153.35 k8s-node4" >> /etc/hosts```


## 1.2 node端执行

### 1.2.1主机关闭防火墙selinux 和swap:

```
# systemctl stop firewalld
# systemctl disable firewalld
主机关闭selinux：
# sed -i 's/enforcing/disabled/' /etc/selinux/config
# setenforce 0



主机关闭swap：
# swapoff -a  # 临时关闭
# vim /etc/fstab 注释到swap那一行 # 永久关闭

将桥接的IPv4流量传递到iptables的链(三台主机都执行)：
# cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# sysctl --system
```
### 1.2.2主机安装Docker/kubeadm/kubelet
```
下载阿里云的docker yum源,并安装
# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# yum -y install docker-ce-18.09.9-3.el7
启动docker,并设置docker开机自启
# systemctl start docker
# systemctl enable docker
```

### 1.2.3设置cgroup驱动和镜像加速，推荐systemd：
```
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
### 1.2.4 重启docker
```
systemctl daemon-reload
systemctl restart docker
```

### 1.2.5 添加阿里云YUM软件源
```
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

### 1.2.6 安装kubeadm，kubelet和kubectl
```
# yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0
# systemctl enable kubelet
```

### 1.2.7 将node节点加入集群
```
kubeadm join 192.168.153.31:6443 --token ak0mcr.lvmyq9svm18xv59h --discovery-token-ca-cert-hash sha256:604e2a4161b05050b5bf75583b0a7b8d9944e0d338eac20b869bcae1e82ecac5
```




# 2. kubernetes 删除node节点(例如删除k8s-node4节点)
## 2.1 master端执行
### 2.1.1 查看查看所有的节点状态
```
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   118d   v1.16.0
k8s-node1    Ready    <none>   118d   v1.16.0
k8s-node2    Ready    <none>   118d   v1.16.0
k8s-node3    Ready    <none>   46m    v1.16.0
k8s-node4    Ready    <none>   19m    v1.16.0
```
### 2.1.2 删除一个节点
```
[root@k8s-master ~]# kubectl delete nodes k8s-node4
node "k8s-node4" deleted
```

## 2.2 node端执行
### 2.2.1 重置node节点
```
[root@k8s-node4 ~]# kubeadm reset
按y重置该节点
```
