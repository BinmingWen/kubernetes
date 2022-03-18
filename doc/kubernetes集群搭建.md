## 搭建kubernetes集群

### 一.安装搭建docker

 **1.安装需要的软件包， yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖** 

>yum install -y yum-utils device-mapper-persistent-data lvm2
>
>



**2.设置 yum 源**

设置一个yum源，下面两个都可用

> yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo（中央仓库）
>
> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（阿里仓库）
>



 3.选择docker版本并安装 

（1）查看可用版本有哪些 

> yum list docker-ce --showduplicates | sort -r

 （2）选择一个版本并安装：`yum install docker-ce-版本号` 

> yum install docker-ce docker-ce-cli containerd.io（安装最新版的docker)



 4.启动 Docker 并设置开机自启 

```javascript
systemctl start docker
systemctl enable docker
```



连接：https://cloud.tencent.com/developer/article/1701451

**3.设置阿里云加速器**

~~~docker
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://wq4o23ow.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
        "max-size": "100m"
  }
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

{
  "registry-mirrors": ["https://wq4o23ow.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
        "max-size": "100m"
  }
}

~~~

### 二.安装kubernetes所需要的插件

##### 1.配置阿里云

~~~shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
~~~

#### 2.安装插件

**先安装阿里云kubernetes源配置**

~~~shelll
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
~~~



> yum install -y kubelet kubeadm kubectl(最新版本插件)
>
> yum install kubelet-1.20.0 kubeadm-1.20.0 kubectl-1.20.0 -y



**查看是否安装成功**

~~~
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl
~~~

#### 3.重启docker以及kubelet服务

>systemctl enable docker.service
>
>systemctl enable kubelet.service

#### 4.关闭相对应的设置

~~~
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
ufw disable # ubuntu

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在master添加hosts (换成自己的IP)
cat >> /etc/hosts << EOF
8.134.215.221 alik8s-master 
8.134.216.109 alik8s-node
8.134.222.247 alik8s-node2
EOF

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
~~~

#### 5.修改hostname

> hostnamectl set-hostname Alik8s-master
>
> hostnamectl set-hostname Alik8s-node
>
> hostnamectl set-hostname Alik8s-node2

### 三.编写kubeadm.yml文件，启动kubeadm

~~~yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
imageRepository: registry.aliyuncs.com/google_containers
controllerManager:
  extraArgs:
    horizontal-pod-autoscaler-use-rest-clients: "true"
    horizontal-pod-autoscaler-sync-period: "10s"
    node-monitor-grace-period: "10s"
apiServer:
  extraArgs:
    runtime-config: "api/all=true"
etcd:
  local:
    dataDir: /data/k8s/etcd
~~~

#### 1.修改deamom.json文件，解决cgroup驱动不一致

~~~json
{
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
                "max-size": "100m"
        }
}

~~~

#### 2.对于存在问题，使用kubeadm reset，清除缓存

>[ERROR FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]:  /etc/kubernetes/manifests/kube-apiserver.yaml already exists

#### 3.获取token

~~~
kubeadm join 172.27.150.232:6443 --token 5x8vri.25ribaulo9qv2akx \
	--discovery-token-ca-cert-hash sha256:c3f195230339c315711275450f462dc9376544111c84d534bc594abfc243d4d1 
kubeadm join 8.134.215.221:6443 --token yroouh.wyr57ivdzdfw869e --discovery-token-ca-cert-hash sha256:0cc3002c28b946580c0116609837a549e8582d0214ec00031d13724765e5987a --v=2

kubeadm join 172.27.150.232:6443 --token 5x8vri.25ribaulo9qv2akx --discovery-token-ca-cert-hash sha256:c3f195230339c315711275450f462dc9376544111c84d534bc594abfc243d4d1 --v=2

~~~

#### 4.使用集群配置命令

~~~
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

### 四.卸载Kubeadm

```
# 卸载服务
kubeadm reset

# 删除rpm包
rpm -qa|grep kube*|xargs rpm --nodeps -e

# 删除容器及镜像
docker images -qa|xargs docker rmi -f
```

### 五.安装网络协议

>NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized

#### 1.部署网络插件

>kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
>#kubeadm config print init-defaults可以告诉我们kubeadm.yaml版本信息。

#### 2.安装网络插件

>kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=1.23.0"

#### 3.搭建集群

~~~
cat >> /etc/hosts << EOF
192.168.172.134 k8smaster
119.29.88.22 k8snode
EOF
~~~

#### 4.解决**/proc/sys/net/ipv4/ip_forward contents are not set to 1**

> echo "1" > /proc/sys/net/ipv4/ip_forward
>
> service network restart

![image-20220318143411482](https://jepusi-image.oss-cn-beijing.aliyuncs.com/img/image-20220318143411482.png)

### 六.安装储存插件(暂时没有成功)

~~~shell

kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/common.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml

kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
~~~

### 七.集群搭建成功

![image-20220318143605488](https://jepusi-image.oss-cn-beijing.aliyuncs.com/img/image-20220318143605488.png)

**·集群详细信息·**

| 主节点（alik8s-master) | 节点1(alik8s-node)     | 节点2(alik8s-node2)    |
| ---------------------- | ---------------------- | ---------------------- |
| 公网ip：8.134.215.221  | 公网ip：8.134.216.109  | 公网ip：8.134.222.247  |
| 私网ip: 172.27.150.232 | 私网ip: 172.27.150.233 | 私网ip: 172.27.150.234 |

