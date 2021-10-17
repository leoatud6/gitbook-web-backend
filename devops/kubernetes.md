---
description: Docker运维，developed from Google, 也是cluster concept
---

# Kubernetes (container orchestrator)

#### [https://www.yuque.com/leifengyang/oncloud/mbvigg](https://www.yuque.com/leifengyang/oncloud/mbvigg)

### History:

Apache Mesos (Twitter) --> k8s

Docker Swarm (Docker) --> component to make cluster, lightweight

Kubernetes (Compass logo, control Docker whale, Google) --> write in Go, low cost, open-source, elastically, load balance(IPVS), 

有状态服务（DBMS） VS 无状态服务（LVS, APACHE）

Kubeadm 源码修改-->更改证书时限（1年-->10年）

Kubernetes 高可用集群



**pod**: 调度最小单位 （docker  + pause）

**deployment**: 维持pod数量恒定（应对container挂掉）

**service**：多个pod抽象为一个服务

**dns**： default has, each service has auto map its ip address 

**ingress**: 针对外界如何暴露ip, virtual ip map to 外网ip

```bash
kubectl cluster-info
kubectl get po
kubectl run dl --image httpd:alpine --port 80
kubectl get deployments.
kubectl expose deployment d1 --target-port-80 --type NodePor
kubectl get svc
apk add curl # install curl in docker images, no default
cat /etc/hosts  # no ipconfig, use this to get ip mapping
kubectl apply -f fileName.yml  # apply for config file
```

## 基础概念

![deployment](<../.gitbook/assets/image (354).png>)

### 编排：：：编 + 排  -->提供分布式系统框架

* load balance, service discovery, cluster
* disk persistence orchestration
* auto deployment and rollback
* sandbox automatically calculation
* 自我修复
* 灰度部署

### 工作方式：：：

 Kubernetes **Cluster** **=** N **Master** Node **+** N **Worker** Node：N主节点+N工作节点； N>=1

 Master Node control Worker Node, 董事会选择领导，少数服从多数原则。

> master一般都是选择奇数个-->高可用 

![](<../.gitbook/assets/image (356).png>)

节点（**Node**）: 服务器（server）

  Control Panel: 总部，负责发号施令。master nodes。

 **CM**: controller manager： 决策者，领导人。

 **kubelet**： 每个厂的厂长，worker node leader

 **ETCD**： 资料库， persistence store

** k-proxy: **每个厂的看门大爷 ，每个厂的k-proxy都是信息同步的。

 **kubectl**: 通过commandline control whole thing

` API-Server 作为秘书部，所有部门需要通过API 打交道-->门面模式`

![](<../.gitbook/assets/image (357).png>)

####  每个cluster里面必要元素：

* container environement
* kube-proxy (sync all the mapping data)
* api-server
* kubelet (worker node learder)

### 部署集群：：：

 kubeadm + kubectl (master)+ **kubelet** + **container/docker** 

![](<../.gitbook/assets/image (358).png>)

```bash
kubeadm init 
# Make this master node --> headquarter
```

创建VPC-->创建private network -->创建instance/VM 并选择在VPC里面。

![](<../.gitbook/assets/image (359).png>)

![](<../.gitbook/assets/image (360).png>)

 组内互信，instance之间必须允许互联。

```bash
google instances:::
    10.128.0.3 --> instance 1
    10.128.0.2 --> instance 2
    10.128.0.4 --> instance 3

#after setup the environment, must be in the same VPC

yum install -y yum utils
#install the yum and yum util

yum install docker -y
#install docker

systemctl enable docker --now

题外话：k8s不会放弃docker， docker会调整自己的兼容性。

```

### 部署方式 & 要求：：：

{% embed url="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/" %}

 按照官方要求安装

```bash
hostnamectl set-hostname k8s-master
# set host name --> instance 1 (10.128.0.3)
# need to reconnect to execute

free -m
# swap row: must be 0 0 0

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
# forbid linux settings

swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab
# turn off swap mounting???

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 从ipv6调节到ipv4，网卡，方便统计

sudo sysctl --system
# apply all the changes, 让命令生效

==== 安装三大件kubelet、kubeadm、kubectl ====

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
# set up mirror and download link

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet

sudo systemctl status kubelet
# check status： 应当是闪亮模式，一会儿开，一会儿关



==== 使用kubeadmin引导和创建集群 ====

sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
# 批量docker pull 处理

==== 将master变成主节点 ====
使用kubeadmin工具

master节点也是入口节点

#所有机器添加master域名映射，以下需要修改为自己的, 注册总部
echo "10.128.0.3  cluster-endpoint" >> /etc/hosts

==== master node init ====
#主节点初始化
kubeadm init \
--apiserver-advertise-address=10.128.0.3 \
--control-plane-endpoint=cluster-endpoint \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
#所有网络范围不重叠

# check and stop firewall
sudo firewall-cmd --state
sudo systemctl stop firewalld

# get nodes
kubectl get nodes
#NAME         STATUS     ROLES                  AGE   VERSION
#k8s-master   NotReady   control-plane,master   18m   v1.22.2


#查看集群所有节点
kubectl get nodes

#根据配置文件，给集群创建资源
kubectl apply -f xxxx.yaml

#查看集群部署了哪些应用？
docker ps   ===   kubectl get pods -A
# 运行中的应用在docker里面叫容器，在k8s里面叫Pod
kubectl get pods -A
kubectl get pods -A -w
kubectl get nodes -o wide

# after init sucessfully, execute the following:::
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

==== 需要部署网络节点-搭建网络 ====
curl https://docs.projectcalico.org/manifests/calico.yaml -O

# 给k8s装东西，直接下载yaml config file即可
kubectl apply -f calico.yaml

# check and find
cat calico.yaml |grep 192.168

==== any worker can add to the main network ====
kubeadm reset
# Now you need to set ip_forward content with 1 by following command:
echo 1 > /proc/sys/net/ipv4/ip_forward

kubeadm join cluster-endpoint:6443 --token j62pdj.3cahir8cm3sr3qz9 \ --discovery-token-ca-cert-hash sha256:3cc44d1eecb453ada4b0be39e6a9a02aeb8ad38867c0a2ab2ffa59d5ce452969

kubeadm token create --print-join-command
# create new token, because 24 hrs expires

```

{% hint style="info" %}
Docker里面叫container， K8s里面叫Pod
{% endhint %}

{% hint style="info" %}
Need to enable **`icmp`** protocol in VPC to make sure they can ping each other
{% endhint %}

```bash
# after install the kube master
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
You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:
  kubeadm join cluster-endpoint:6443 --token j62pdj.3cahir8cm3sr3qz9 \
        --discovery-token-ca-cert-hash sha256:3cc44d1eecb453ada4b0be39e6a9a02aeb8ad38867c0a2ab2ffa59d5ce452969 \
        --control-plane 
Then you can join any number of worker nodes by running the following on each as root:
kubeadm join cluster-endpoint:6443 --token j62pdj.3cahir8cm3sr3qz9 \
        --discovery-token-ca-cert-hash sha256:3cc44d1eecb453ada4b0be39e6a9a02aeb8ad38867c0a2ab2ffa59d5ce452969 
[root@k8s-master leoatud6]# Connected, host fingerprint: ssh-rsa 0 CC:E4:CD:7D:48:B2:E0:AC:D0:22:E8:D9:3F:8D:40:61:
28:64:FF:90:A7:91:1C:33:20:5E:4A:0A:D1:0B:86:D4
```

```bash
==== 安装UI: dashboard ====
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

# 设置访问端口
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
# type: ClusterIP 改为 type: NodePort

kubectl get svc -A |grep kubernetes-dashboard
# 找到端口，在安全组放行

kubernetes-dashboard   kubernetes-dashboard        NodePort    10.96.99.247   <none>        443:31940/TCP          
port=31940, must be open

访问： https://集群任意IP:端口      https://https://34.135.247.55:31940/

systemctl restart kubelet

```

### 操作使用：：：

 定义不同的namespace, dev, sandbox, prod 

![](<../.gitbook/assets/image (362).png>)

```bash
kubtctl get ns
# get namespace

kubectl delete ns default
# all pods will be created into "default"

kubectl create ns hello
kubectl delete ns hello
```

![ pod(k8s) --> container (docker)](<../.gitbook/assets/image (363).png>)

一个pod之内有多个containers

 

```bash
kubectl get pod
# 查看default底下的pod
kubectl describe pod xxx
# 描述具体内容

docker ps|grep mynginx 
# check if mynginx start

kubectl delete pod xxx
# 删除default里面的xxx


vi pod.yaml
kubectl apply -f pod.yaml
# 创建并apply配置文件

Kubectl delete -f pod.yaml
# 配置文件删除pod

kubectl logs -f xxx
# 查看logs

kubectl get pod -owide
# 每个pod都有分配一个ip

kubectl exec -it mynginx -- /bin/bash 
# 进入到control panel， 类比docker


```

```
// 配置文件标准
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep
  template:
    metadata:
      labels:
        app: my-dep
    spec:
      containers:
      - image: nginx
        name: nginx
```

{% hint style="info" %}
真正运行program的是pod，分配的IP都是192.168.xx， 集群内任意的机器都可以随便访问彼此
{% endhint %}

 pod里面包含n个docker containers-->多容器pod

####  同一个pod内的不同container之间可以通过127.0.0.1互相访问。（nginx80, tomcat8080）

`同一个pod里面`**`不可以有两个相同端口`**`的东西。`

![](<../.gitbook/assets/image (365).png>)

### 部署Deployment：：：

```bash
kubectl run mynginx --image=nginx
kubectl create deployment mytomcat --image=tomcat:8:5:68
# 使用deployment 部署的，可以自愈，哪怕删了delete也无所谓，也会自愈。

kubectl delete deploy mytomcat
# 删除整个部署，不会自愈

kubectl create deployment mytomcat --image=tomcat:8:5:68 --replicas-3
# 同一个部署可以创建多次

```

![](<../.gitbook/assets/image (370).png>)

### 扩缩容：：：

 当遇到流量高峰的时候，可以直接扩容，之后再下线。

```bash
kubectl scale deploy/my-dep --replicas=5
# 增加到5个container

kubectl scale deploy/my-dep --replicas=2
# 缩容到2

```

### 自愈 & 故障转移：：：

 如果发生问题，会自动重启。

如果这个pod下线/停机/服务器炸了了，内容会转移到别的pod上面继续之前的内容进行。（会有一个故障转移的时间，超过这个时间才会执行。）

### 滚动更新：：：

 更新版本的时候使用： v1 --> v2

 非停机更新，先动v2，然后准备好了，v1会升级到v2

```bash
kubectl get pod -w
kubectl set image nginx=nginx:1.16 deploy/my-dev --record
# 升级的时候只需要重新set version number即可

watch -n 1 kubectl get pod
# 监控，并且每秒打印
```

![](<../.gitbook/assets/image (372).png>) 

### 版本回退：：：

通过历史记录可以查看revision，直接回到相应的revision number即可： --revision=n

```bash
#历史记录
kubectl rollout history deployment/my-dep

#查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2

#回滚(回到上次)
kubectl rollout undo deployment/my-dep

#回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2
```

![statefulSet:  must save the data (mount to another new pod)](<../.gitbook/assets/image (367).png>)

{% hint style="info" %}
不会直接部署pod，而是通过整体的部署进行添加和删除

除了Deployment，k8s还有 `StatefulSet` 、`DaemonSet` 、`Job` 等 类型资源。我们都称为 `工作负载`。

有状态应用使用 `StatefulSet` 部署，无状态应用使用 `Deployment` 部署

[https://kubernetes.io/zh/docs/concepts/workloads/controllers/](https://kubernetes.io/zh/docs/concepts/workloads/controllers/)
{% endhint %}



## Service  （ClusterIP & NodePort）

### Pod的服务发现与负载均衡：：：（类比zookeeper）

Service：将一组pod公开为网络服务的抽象方法

前端只需要配置service的端口。

![](<../.gitbook/assets/image (368).png>)

```bash
kubectl expose deploy my-dep --port=8000 --target-port=80
# 统一暴露一个端口给前端访问，集群内任意访问 

kubectl delete my-dep
# 删除my-dep service

kubtctl get service 
kubtctl get pod --show-labels
# 每个pod都有一个label, app=my-dev
```

> #### Cluster IP （集群内部访问使用的域名）
>
>
>
> > 服务名.所在名称空间.svc ==> my-dep.default.svc  (另一种统一访问域名的方式，只能在pod内部访问)
>
> #### NodePort 模式 （集群外也可以访问）
>
> NodePort随机暴露范围：30000-32767

![](<../.gitbook/assets/image (369).png>)

## Ingress (入口,所有流量的统一入口，gateway )

Service的**统一网关入口**。（Traffic --> Ingress --> Service --> Pod --> Container）

【核心】：网络分层，有inner & outer，方便内部网络互相访问调用

![](<../.gitbook/assets/image (366).png>)

```bash
# 需要手动安装ingress，default是没有的
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml

#修改镜像
vi deploy.yaml
#将image的值改为如下值：
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0

# 检查安装的结果
kubectl get pod,svc -n ingress-nginx

# 最后别忘记把svc暴露的端口要放行

kubectl get svc -A
# check all services
31405 & 32401 ingress暴露端口，需要手动打开
```

官网地址：[https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

就是nginx做的，配置文件也是类似Nginx

[https://139.198.163.211:32401/](https://139.198.163.211:32401)    (-->port:433)

[http://139.198.163.211:31405/](https://139.198.163.211:32401)     (-->port:80)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000yaml
```

```
kubectl apply -f ingress-rule.yaml
# 使用上面的配置文件。给Ingress

kubectl get ingress

# 测试的时候别忘了host更改。test use only
```

#### 路径重写， Re-write (add annotation)

#### 流量现值（flow control)





