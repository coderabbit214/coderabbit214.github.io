# Kubernetes


# Kubernetes

## 命令

```bash
# 查看节点状态
kubectl get nodes

# ---后缀---
-oyaml 获取yaml

# ---pod---
# 创建pod
kubectl run mynginx --image=nginx
# 跟根据yaml添加pod
kubectl apply -f calico.yaml
# 查看default名称空间的Pod
kubectl get pod
watch -n 1 kubectl get pods
#使用标签检索Pod
kubectl get pod -l app=my-dep
# 查看集群部署了哪些应
kubectl get pods -A
# 根据命名空间
kubectl get pods -n kubernetes-dashboard
# 描述(可以看到具体执行以及在哪个节点上运行)
kubectl describe pod 你自己的Pod名字
# 删除
kubectl delete pod Pod名字
# 查看Pod的运行日志
kubectl logs Pod名字
# 获取详细信息
kubectl get pod -owide
# 进入容器内部
kubectl exec -it myapp -- /bin/bash

# ---命名空间---
# 查看命名空间
kubectl get ns
# 创建命名空间
kubectl create ns a
# 删除命名空间
kubectl delete ns a

# ---deployment---
# 创建
kubectl create deployment mytomcat --image=tomcat:8.5.68
# 查看
kubectl get deploy
# 删除
kubectl delete deploy mytomcat
# 修改
kubectl edit deployment my-dep

# 多副本 --replicas=3
kubectl create deployment my-dep --image=nginx --replicas=3
# 扩缩容
kubectl scale --replicas=5 deployment/my-dep
# 版本更新
kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
kubectl set image deployment/[deployment名字] [容器名字]=nginx:1.16.1 --record
# 查看容器名字
kubectl get deploy -oyaml
# --版本回退--
# 查看状态
kubectl rollout status deployment/my-dep
# 历史记录
kubectl rollout history deployment/my-dep
# 查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2
# 回滚(回到上次)
kubectl rollout undo deployment/my-dep
# 回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2


# service,ingress 等资源都是可以用get，edit，-oyaml等命令
```

## yaml快速编写

- 使用`kubectl create`生成yaml文件

  ```bash
  # 生成文件内容到控制台
  kubectl create deployment web --image=nginx -o yaml --dry-run
  # 输出内容到文件
  kubectl create deployment web --image=nginx -o yaml --dry-run > my.yaml
  ```

- 使用`kubectl get`命令导出yaml文件

  ```bash
  kubectl get deploy nginx -o=yaml --export > my2.yaml
  ```

  

## 定义

- **服务发现和负载均衡**
  Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。
- **存储编排**
  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。
- **自动部署和回滚**
  你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。
- **自动完成装箱计算**
  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。
- **自我修复**
  Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。
- **密钥与配置管理**
  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

### 架构

### 工作方式

Kubernetes **Cluster** **=** N **Master** Node **+** N **Worker** Node：N主节点+N工作节点； N>=1

## 组件架构

![components-of-kubernetes](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/05/20220705-163057.svg)

### 控制平面组件（Control Plane Components） 

控制平面的组件对集群做出全局决策(比如调度)，以及检测和响应集群事件（例如，当不满足部署的 `replicas` 字段时，启动新的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。

控制平面组件可以在集群中的任何节点上运行。 然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件， 并且不会在此计算机上运行用户容器。 请参阅[使用 kubeadm 构建高可用性集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/) 中关于多 VM 控制平面设置的示例。

#### kube-apiserver

API 服务器是 Kubernetes [控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件公开了 Kubernetes API。 API 服务器是 Kubernetes 控制面的前端。

Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/)。 kube-apiserver 设计上考虑了水平伸缩，也就是说，它可通过部署多个实例进行伸缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

#### etcd

etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

您的 Kubernetes 集群的 etcd 数据库通常需要有个备份计划。

要了解 etcd 更深层次的信息，请参考 [etcd 文档](https://etcd.io/docs/)。

#### kube-scheduler

控制平面组件，负责监视新创建的、未指定运行[节点（node）](https://kubernetes.io/zh/docs/concepts/architecture/nodes/)的 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)，选择节点让 Pod 在上面运行。

调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

#### kube-controller-manager

在主节点上运行 [控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/) 的组件。

从逻辑上讲，每个[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。

这些控制器包括:

- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应
- 任务控制器（Job controller）: 监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌

#### cloud-controller-manager

云控制器管理器是指嵌入特定云的控制逻辑的 [控制平面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)组件。 云控制器管理器允许您链接集群到云提供商的应用编程接口中， 并把和该云平台交互的组件与只和您的集群交互的组件分离开。

`cloud-controller-manager` 仅运行特定于云平台的控制回路。 如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的环境中不需要云控制器管理器。

与 `kube-controller-manager` 类似，`cloud-controller-manager` 将若干逻辑上独立的 控制回路组合到同一个可执行文件中，供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

下面的控制器都包含对云平台驱动的依赖：

- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器

### Node 组件 

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。

#### kubelet

一个在集群中每个[节点（node）](https://kubernetes.io/zh/docs/concepts/architecture/nodes/)上运行的代理。 它保证[容器（containers）](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/#why-containers)都 运行在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

#### kube-proxy

[kube-proxy](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个节点上运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh/docs/concepts/services-networking/service/) 概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它来实现网络规则。否则， kube-proxy 仅转发流量本身。

## kubeadm创建集群

![截屏2022-07-05 17.13.38](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/05/20220705-171345.png)

### 环境准备

#### 配置yum源

```
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### Docker安装

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo yum remove docker*
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
# 启动
systemctl enable docker --now

# 配置加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://lu9qvzo1.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### linux设置

- 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版以及一些不提供包管理器的发行版提供通用的指令
- 每台机器 2 GB 或更多的 RAM （如果少于这个数字将会影响你应用的运行内存)
- 2 CPU 核或更多
- 集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)

- - **设置防火墙放行规则**

- 节点之中不可以有重复的主机名、MAC 地址或 product_uuid。请参见[这里](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address)了解更多详细信息。

- - **设置不同hostname**

- 开启机器上的某些端口。请参见[这里](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports) 了解更多详细信息。

- - **内网互信**

- 禁用交换分区。为了保证 kubelet 正常工作，你 **必须** 禁用交换分区。

- - **永久关闭**

```bash
#各个机器设置自己的域名
hostnamectl set-hostname xxxx


# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### 安装kubelet、kubeadm、kubectl（每台服务器）

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

> 查看kubelet状态
>
> systemctl status kubelet

### 使用kubeadm引导集群

#### 下载各个机器需要的镜像

```bash
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
```

#### 初始化主节点

```bash
#所有机器添加master域名映射，以下需要修改为自己的
echo "172.16.1.3  cluster-endpoint" >> /etc/hosts



#主节点初始化
kubeadm init \
--apiserver-advertise-address=172.16.1.3 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16

#所有网络范围不重叠


```

成功后提示

```bash
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

  kubeadm join cluster-endpoint:6443 --token 9o3q1b.z2sxmoppiqf9yh5j \
    --discovery-token-ca-cert-hash sha256:f5cee157412f35851006863d1d26478ae692c3eaa03486604c4493118e47d39a \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token 9o3q1b.z2sxmoppiqf9yh5j \
    --discovery-token-ca-cert-hash sha256:f5cee157412f35851006863d1d26478ae692c3eaa03486604c4493118e47d39a 
```

根据提示操作

1. 创建 .kube

   ```
    mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

2. 配置网络插件

   ```bash
   # 注意版本
   curl https://docs.projectcalico.org/manifests/calico.yaml -O
   curl https://docs.projectcalico.org/v3.18/manifests/calico.yaml -O
   
   kubectl apply -f calico.yaml
   ```

3. 在其他节点加入

   > 新令牌
   >
   > kubeadm token create --print-join-command

   - master

     ```
       kubeadm join cluster-endpoint:6443 --token 9o3q1b.z2sxmoppiqf9yh5j \
         --discovery-token-ca-cert-hash sha256:f5cee157412f35851006863d1d26478ae692c3eaa03486604c4493118e47d39a \
         --control-plane 
     ```

   - node

     ```
     kubeadm join cluster-endpoint:6443 --token 9o3q1b.z2sxmoppiqf9yh5j \
         --discovery-token-ca-cert-hash sha256:f5cee157412f35851006863d1d26478ae692c3eaa03486604c4493118e47d39a 
     ```

4. 验证

   ```
     kubectl get nodes
   ```

### 部署dashboard

- 启动

  ```
  curl https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml -O
  kubectl apply -f  recommended.yaml
  ```

- 设置访问端口

  ```bash
  # type: ClusterIP 改为 type: NodePort
  kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
  
  ## 找到端口，在安全组放行
  kubectl get svc -A |grep kubernetes-dashboard
  ```

- 创建访问账号

  ```bash
  #创建访问账号，准备一个yaml文件； vi dash.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: admin-user
    namespace: kubernetes-dashboard
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: admin-user
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
  ```

  ```bash
  kubectl apply -f dash.yaml
  ```

- 获取访问令牌

  ```bash
  kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
  ```

  > 隐私错误 ，直接输入thisisunsafe

### 安装文件系统

1. 安装nfs

   ```bash
   # 在每个机器。
   yum install -y nfs-utils
   
   
   # 在master 执行以下命令 
   echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
   
   
   # 执行以下命令，启动 nfs 服务;创建共享目录
   mkdir -p /nfs/data
   
   
   # 在master执行
   systemctl enable rpcbind
   systemctl enable nfs-server
   systemctl start rpcbind
   systemctl start nfs-server
   
   # 使配置生效
   exportfs -r
   
   
   #检查配置是否生效
   exportfs
   ```

2. 从节点

   ```bash
   showmount -e [masterip]
   
   mkdir -p /nfs/data
   
   mount -t nfs [masterip]:/nfs/data /nfs/data
   ```

3. 配置默认存储

   > 注意修改ip

   ```yaml
   ## 创建了一个存储类
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: nfs-storage
     annotations:
       storageclass.kubernetes.io/is-default-class: "true"
   provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
   parameters:
     archiveOnDelete: "true"  ## 删除pv的时候，pv的内容是否要备份
   
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nfs-client-provisioner
     labels:
       app: nfs-client-provisioner
     # replace with namespace where provisioner is deployed
     namespace: default
   spec:
     replicas: 1
     strategy:
       type: Recreate
     selector:
       matchLabels:
         app: nfs-client-provisioner
     template:
       metadata:
         labels:
           app: nfs-client-provisioner
       spec:
         serviceAccountName: nfs-client-provisioner
         containers:
           - name: nfs-client-provisioner
             image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/nfs-subdir-external-provisioner:v4.0.2
             # resources:
             #    limits:
             #      cpu: 10m
             #    requests:
             #      cpu: 10m
             volumeMounts:
               - name: nfs-client-root
                 mountPath: /persistentvolumes
             env:
               - name: PROVISIONER_NAME
                 value: k8s-sigs.io/nfs-subdir-external-provisioner
               - name: NFS_SERVER
                 value: 172.31.0.4 ## 指定自己nfs服务器地址
               - name: NFS_PATH  
                 value: /nfs/data  ## nfs服务器共享的目录
         volumes:
           - name: nfs-client-root
             nfs:
               server: 172.31.0.4
               path: /nfs/data
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: nfs-client-provisioner
     # replace with namespace where provisioner is deployed
     namespace: default
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: nfs-client-provisioner-runner
   rules:
     - apiGroups: [""]
       resources: ["nodes"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["persistentvolumes"]
       verbs: ["get", "list", "watch", "create", "delete"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims"]
       verbs: ["get", "list", "watch", "update"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["storageclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["events"]
       verbs: ["create", "update", "patch"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: run-nfs-client-provisioner
   subjects:
     - kind: ServiceAccount
       name: nfs-client-provisioner
       # replace with namespace where provisioner is deployed
       namespace: default
   roleRef:
     kind: ClusterRole
     name: nfs-client-provisioner-runner
     apiGroup: rbac.authorization.k8s.io
   ---
   kind: Role
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: leader-locking-nfs-client-provisioner
     # replace with namespace where provisioner is deployed
     namespace: default
   rules:
     - apiGroups: [""]
       resources: ["endpoints"]
       verbs: ["get", "list", "watch", "create", "update", "patch"]
   ---
   kind: RoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: leader-locking-nfs-client-provisioner
     # replace with namespace where provisioner is deployed
     namespace: default
   subjects:
     - kind: ServiceAccount
       name: nfs-client-provisioner
       # replace with namespace where provisioner is deployed
       namespace: default
   roleRef:
     kind: Role
     name: leader-locking-nfs-client-provisioner
     apiGroup: rbac.authorization.k8s.io
   ```

### 安装metrics-server

> 集群指标监控组件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  - configmaps
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --kubelet-insecure-tls
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/metrics-server:v0.4.3
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 4443
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          periodSeconds: 10
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

## 核心

### 总结

![截屏2022-07-14 09.54.38](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/14/20220714-095445.png)

![截屏2022-07-09 19.57.28](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/09/20220709-195734.png)

> 资源创建两种方式
>
> - 命令行
> - yaml文件

### Namespace

- 命令行

  ```bash
  # 查看命名空间
  kubectl get ns
  # 创建命名空间
  kubectl create ns a
  # 删除命名空间
  kubectl delete ns a
  # 如果是使用文件创建的，还可以治直接使用文件删除
  kubectl delete -f hello.yaml 
  ```

- yaml文件

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: hello
  ```

### Pod

>  运行中的一组容器，Pod是kubernetes中应用的最小单位

- 命令行

  ```bash
  kubectl run mynginx --image=nginx
  
  # 查看default名称空间的Pod
  kubectl get pod 
  # 描述(可以看到具体执行以及在哪个节点上运行)
  kubectl describe pod 你自己的Pod名字
  # 删除
  kubectl delete pod Pod名字
  # 查看Pod的运行日志
  kubectl logs Pod名字
  
  # 每个Pod - k8s都会分配一个ip
  kubectl get pod -owide
  # 进入容器内部
  kubectl exec -it myapp -- /bin/bash
  
  # 集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod
  ```

- 配置文件

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: mynginx
    name: mynginx
  #  namespace: default
  spec:
    containers:
    - image: nginx
      name: mynginx
  ```

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: myapp
    name: myapp
  spec:
    containers:
    - image: nginx
      name: nginx
    - image: tomcat:8.5.68
      name: tomcat
  ```

#### 特性

- 同一个pod中的容器共享网络(同ip，端口)
- 同一个pod中的容器共享存储

#### 镜像拉取策略

> - IfNotPresent:默认值，镜像不存在时拉取
> - Always:每次都拉取
> - Never:永远不主动拉取

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - image: nginx
    name: nginx
    imagePullPolicy: Always
```

#### 资源限制

```yaml
sepc:
	containers:
	- name: db
		image: mysql
		resources:
		# 调度资源限制(最大)
			requests:
				memory: "64Mi"
				# 1核 = 1000m
				cpu: "250m"
		# 运行时资源限制(最大)
			limits:
				memory: "128Mi"
				cpu: "500m"
	restartPolicy: Always
```

#### 重启机制

> Always:总是重启(默认)
>
> OnFailure:异常退出（状态码非0），才重启
>
> Never:从不重启

```yaml
sepc:
	containers:
	- name: db
		image: mysql
	restartPolicy: Always
```

#### 健康检查

##### 检查方式

- livenessProbe(存活检查)
- readinessProbe(就绪检查)

##### 检查方法

![截屏2022-07-13 10.36.00](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/13/20220713-103645.png)

- httpGet
- exec
- topSocket

#### 调度策略

影响调度的属性

- 资源限制

- 节点选择器

  1. 对节点打标签

     ```bash
     # 打标签
     kubectl label node node01 env_role=prod
     # 查看标签
     kubectl get node --show-labels
     ```

  2. 使用

     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: myapp
     spec:
     	nodeSelector: 
     		env_role: prod
       containers:
       - image: nginx
         name: nginx
         imagePullPolicy: Always
     ```

- 节点亲和性

  - 硬亲和性(约束条件必须满足)
  - 软亲和性(尝试满足)

  ![17-1-Pod调度-节点亲和性](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/13/20220713-105827.png)

- 污点和污点容忍(节点属性)

  1. 查看污点情况

     ```bash
     kubectl describe node k8s-master01 | grep Taint
     ```

  2. 三种情况

     - NoSchedule:一定不被调度
     - PreferNoSchdule:尽量不被调度
     - NoExecute:不会被调度，而且还会驱逐Node已有Pod

  3. 创建污点

     ```bash
     kubeclt taint node [node] [key]=[value]:[污点的三种情况]
     ## 举例
     kubeclt taint node k8s-master01 env_role=yes:NoSchedule
     ```

  4. 删除污点

     ```bash
     kubeclt taint node [node] key:[污点的三种情况]-
     ## 举例
     kubeclt taint node k8s-master01 key:NoSchedule-
     ```

- 污点容忍

  ```yaml
  spec: 
  	tolerations:
  	- key: "key1"
  	  operator: "Equal"
  	  value: "value1"
  	  effect: "NoSchedule"
  	- key: "key1"
  	  operator: "Equal"
  	  value: "value1"
  	  effect: "NoExecute"
  	- key: "node.alpha.kubernetes.io/unreachable"
  	 	operator: "Exists"
   	  effect: "NoExecute"
   	  tolerationSeconds: 6000
  ```

### Deployment(一种工作负载)

![截屏2022-07-07 17.39.12](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/07/07/20220707-173917.png)

> 控制Pod，使Pod拥有多副本，自愈，扩缩容等能力

#### 操作

- 命令行

  ```bash
  # 创建
  kubectl create deployment mytomcat --image=tomcat:8.5.68
  # 查看
  kubectl get deploy
  # 删除
  kubectl delete deploy mytomcat
  # 修改
  kubectl edit deployment my-dep
  
  # 多副本 --replicas=3
  kubectl create deployment my-dep --image=nginx --replicas=3
  # 扩缩容
  kubectl scale --replicas=5 deployment/my-dep
  # 版本更新
  kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
  kubectl set image deployment/[deployment名字] [容器名字]=nginx:1.16.1 --record
  # 查看容器名字
  kubectl get deploy -oyaml
  # --版本回退--
  # 查看状态
  kubectl rollout status deployment/my-dep
  # 历史记录
  kubectl rollout history deployment/my-dep
  # 查看某个历史详情
  kubectl rollout history deployment/my-dep --revision=2
  # 回滚(回到上次)
  kubectl rollout undo deployment/my-dep
  # 回滚(回到指定版本)
  kubectl rollout undo deployment/my-dep --to-revision=2
  ```

- yaml

  ```yaml
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

#### 特性

- 自愈
- 多副本
- 扩缩容
- 滚动更新
- 版本回退

### Service

> 统一多个pod对外统一暴露端口，负载均衡
>
> 使用ip或者域名访问
>
> 域名只能在pod(可以不在一个service中)中使用，不可以在主机上使用
>
> 域名规则：服务名.所在空间名称.svc(my-dep.default.svc)

- 命令

  ```bash
  #创建 默认 --type=ClusterIP
  kubectl expose deployment my-dep --port=8000 --target-port=80
  #对外暴露端口
  kubectl expose deployment my-dep --port=8000 --target-port=80 --type=NodePort
  # 获取列表
  kubectl get service
  
  # 删除
  kubectl delete svc my-dep
  ```

- yaml

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: my-dep
    name: my-dep
  spec:
    selector:
      app: my-dep
    ports:
    - port: 8000
      protocol: TCP
      targetPort: 80
  ```

#### ClusterIP

> 不对外暴露，只有集群内部可以访问

#### NodePort

> 对外暴露端口

### Ingress

[官网](https://kubernetes.github.io/ingress-nginx/)

#### 安装

```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml
kubectl apply deploy.yaml
```

#### 使用

```bash
#获取网关端口
kubectl get svc -A
```

创建测试环境

```bash
vi test.yaml
kubectl apply -f test.yaml
```

`test.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
```

需求： 访问`sit.jsh.com`转给服务`nginx-demo`,访问`uat.jsh.com`转给服务`hello-server`

实现：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "uat.jsh.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "sit.jsh.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
```

路径重写

```yaml
  - host: "sit.jsh.com"
		http:
      paths:
      - pathType: Prefix
        path: "/nginx(/|$)(.*)"  # 
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
```

### 存储

#### 环境准备

1. 所有节点

   ```bash
   #所有机器安装
   yum install -y nfs-utils
   ```

2. 主节点:暴露目录

   ```bash
   #nfs主节点
   echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
   
   mkdir -p /nfs/data
   systemctl enable rpcbind --now
   systemctl enable nfs-server --now
   #配置生效
   exportfs -r
   ```

3. 从节点

   ```bash
   showmount -e 172.16.1.3
   
   #执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
   mkdir -p /nfs/data
   
   mount -t nfs 172.16.1.3:/nfs/data /nfs/data
   # 写入一个测试文件
   echo "hello nfs server" > /nfs/data/test.txt
   ```

4. 原生方式数据挂载

   > Pod 删除后数据不会自动删除

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: nginx-pv-demo
     name: nginx-pv-demo
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nginx-pv-demo
     template:
       metadata:
         labels:
           app: nginx-pv-demo
       spec:
         containers:
         - image: nginx
           name: nginx
           volumeMounts:
           - name: html
             mountPath: /usr/share/nginx/html
         volumes:
           - name: html
             nfs:
               server: 172.16.1.3
               path: /nfs/data/nginx-pv
   ```

#### PV&PVC

> *PV：持久卷（Persistent Volume），将应用需要持久化的数据保存到指定位置*
>
> *PVC：持久卷申明（**Persistent Volume Claim**），申明需要使用的持久卷规格*

1. 创建pv池

   准备

   ```bash
   #nfs主节点
   mkdir -p /nfs/data/01
   mkdir -p /nfs/data/02
   mkdir -p /nfs/data/03
   ```

   创建

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv01-10m
   spec:
     capacity:
       storage: 10M
     accessModes:
       - ReadWriteMany
     storageClassName: nfs
     nfs:
       path: /nfs/data/01
       server: 172.16.1.3
   ---
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv02-1gi
   spec:
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteMany
     storageClassName: nfs
     nfs:
       path: /nfs/data/02
       server: 172.16.1.3
   ---
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv03-3gi
   spec:
     capacity:
       storage: 3Gi
     accessModes:
       - ReadWriteMany
     storageClassName: nfs
     nfs:
       path: /nfs/data/03
       server: 172.16.1.3
   ```

2. 创建pvc

   ```yaml
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: nginx-pvc
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 200Mi
     storageClassName: nfs
   ```

3. 绑定pvc和pod

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: nginx-deploy-pvc
     name: nginx-deploy-pvc
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nginx-deploy-pvc
     template:
       metadata:
         labels:
           app: nginx-deploy-pvc
       spec:
         containers:
         - image: nginx
           name: nginx
           volumeMounts:
           - name: html
             mountPath: /usr/share/nginx/html
         volumes:
           - name: html
             persistentVolumeClaim:
               claimName: nginx-pvc
   ```

#### ConfigMap

> 主要存放配置，可以实时更新

1. 创建配置文件`redis.conf`

   ```
   appendonly yes
   ```

2. 创建配置集

   ```bash
   kubectl create cm redis-conf --from-file=redis.conf
   # 查看详情
   kubectl get cm redis-conf -oyaml
   ```

3. 创建pod

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: redis
   spec:
     containers:
     - name: redis
       image: redis
       command:
         - redis-server
         - "/redis-master/redis.conf"  #指的是redis容器内部的位置
       ports:
       - containerPort: 6379
       volumeMounts:
       - mountPath: /data
         name: data
       - mountPath: /redis-master
         name: config
     volumes:
       - name: data
         emptyDir: {}
       - name: config
         configMap:
           name: redis-conf
           items:
           - key: redis.conf
             path: redis.conf
   ```

4. 进入pod验证

   ```bash
   kubectl exec -it redis -- redis-cli
   
   CONFIG GET appendonly
   ```

#### Secret

> Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 secret 中比放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的定义或者 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 中来说更加安全和灵活。

##### 定义

```bash
kubectl create secret docker-registry my-docker \
--docker-username=xxx \
--docker-password=xxx \
--docker-email=xxx@xxxx

##命令格式
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```

##### 使用

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx
spec:
  containers:
  - name: private-nginx
    image: xxxxxxxx
  imagePullSecrets:
  - name: my-docker
```

### 安全机制

[官网](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/)

### 监控

Prometheus+Grafana

## 附录

## 微服务java`DockerFile`

```bash
docker build -t loca-dup:1.0 .
```

```dockerfile
FROM openjdk:8-jdk
LABEL maintainer=coderabbit


#docker run -e PARAMS="--server.port 9090"
ENV PARAMS="--server.port=8080 --spring.profiles.active=prod --spring.cloud.nacos.discovery.server-addr=his-nacos.his:8848 --spring.cloud.nacos.config.server-addr=his-nacos.his:8848 --spring.cloud.nacos.config.namespace=prod --spring.cloud.nacos.config.file-extension=yml"
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

COPY target/*.jar /app.jar
EXPOSE 8080

#
ENTRYPOINT ["/bin/sh","-c","java -Dfile.encoding=utf8 -Djava.security.egd=file:/dev/./urandom -jar app.jar ${PARAMS}"]
```

```bash
$ docker login --username=xxxx registry.cn-hangzhou.aliyuncs.com

#把本地镜像，改名，成符合阿里云名字规范的镜像。
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/lfy_ruoyi/镜像名:[镜像版本号]
## docker tag 393f2ccae8fb registry.cn-hangzhou.aliyuncs.com/jsh_aliyun/f-dup:1.2.6

$ docker push registry.cn-hangzhou.aliyuncs.com/lfy_ruoyi/镜像名:[镜像版本号]
## docker push registry.cn-hangzhou.aliyuncs.com/lfy_ruoyi/ruoyi-visual-monitor:v1
```


