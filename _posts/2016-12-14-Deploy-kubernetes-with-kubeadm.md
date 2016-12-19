---
layout: post
title:  kubeadm快速安装kubernetes
categories: [docker,Cloud]
date: 2016-12-14 10:58:30 +0800
keywords: [docker,云计算,kubernetes]
---

>Kubernetes 1.5.0刚刚发布，添加了众多的新特性，我们的云平台也计划从Mesos和Rancher迁移到kubernetes，所以迫不及待地想尝试一下，Google出品，必属精品，无奈GFW的层层阻挡，让原本简单的部署步骤变得异常复杂，所以写下此文，供各位参考。

### 1 环境准备

准备了三台机器作安装测试工作，机器信息如下: 

|      IP      |   Name   |    Role    |     OS    |
|--------------|----------|------------|-----------|
| 172.16.1.101 | Master01 | Controller | Centos7.2 |
| 172.16.1.106 | Minion01 | Compute    | Centos7.2 |
| 172.16.1.107 | Minino02 | Compute    | Centos7.2 |

### 2 安装docker

```bash
tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF

yum update -y && yum upgrade -y
yum install docker-engine -y
systemctl start docker
systemctl enable docker.service
```

### 3 安装k8s工具包

三种方式：官方源安装、非官方源安装和release工程编译，yum方式因为不能直接使用google提供的源，非官方源中提供的版本比较老（mritd提供的源很不错，版本很新），如果要使用新版本，可以尝试release工程编译的方式。

**官方源安装**

跨越GFW方式不细说，你懂的。

建议使用`yumdownloader`下载rpm包，不然那下载速度，会让各位对玩k8s失去兴趣的。

```bash
yum install -y yum-utils

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yumdownloader kubelet kubeadm kubectl kubernetes-cni
rpm -ivh *.rpm
systemctl enable kubelet && systemctl start kubelet
```

**非官方源安装**

```bash
#感谢mritd维护了一个yum源
tee /etc/yum.repos.d/mritd.repo << EOF
[mritdrepo]
name=Mritd Repository
baseurl=https://rpm.mritd.me/centos/7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://cdn.mritd.me/keys/rpm.public.key
EOF
yum makecache
yum install -y kubelet kubectl kubernetes-cni kubeadm
systemctl enable kubelet && systemctl start kubelet
```

**relese编译**

```bash
git clone https://github.com/kubernetes/release.git

cd release/rpm

./docker-build.sh
```

编译完成后生成rpm包到：`/output/x86_64`，进入到该目录后安装rpm包，注意选择amd64的包（相信大多数同学都是64bit环境，如果是32bit或者arm架构请自行选择安装）。

### 4 下载docker镜像

kubeadm方式安装kubernetes集群需要的镜像在docker官方镜像中并未提供，只能去google的官方镜像库：`gcr.io` 中下载，GFW咋办？翻墙!也可以使用docker hub做跳板自己构建，这里针对k8s-1.5.0我已经做好镜像，各位可以直接下载。

kubernetes-1.5.0所需要的镜像：

- etcd-amd64:2.2.5
- kubedns-amd64:1.7
- kube-dnsmasq-amd64:1.3 
- exechealthz-amd64:1.1 
- pause-amd64:3.0
- kube-discovery-amd64:1.0
- kube-proxy-amd64:v1.5.0 
- kube-scheduler-amd64:v1.5.0
- kube-controller-manager-amd64:v1.5.0 
- kube-apiserver-amd64:v1.5.0 
- kubernetes-dashboard-amd64:v1.5.0

偷下懒吧，直接执行以下脚本：

```bash
#!/bin/bash
images=(kube-proxy-amd64:v1.5.0 kube-discovery-amd64:1.0 kubedns-amd64:1.7 kube-scheduler-amd64:v1.5.0 kube-controller-manager-amd64:v1.5.0 kube-apiserver-amd64:v1.5.0 etcd-amd64:2.2.5 kube-dnsmasq-amd64:1.3 exechealthz-amd64:1.1 pause-amd64:3.0 kubernetes-dashboard-amd64:v1.5.0)
for imageName in ${images[@]} ; do
  docker pull cloudnil/$imageName
  docker tag cloudnil/$imageName gcr.io/google_containers/$imageName
  docker rmi cloudnil/$imageName
done
```

### 5 安装master节点

由于kubeadm和kubelet安装过程中会生成`/etc/kubernetes`目录，而`kubeadm init`会先检测该目录是否存在，所以我们先使用kubeadm初始化环境。

```bash
kubeadm reset && systemctl start kubelet
kubeadm init --use-kubernetes-version v1.5.0 --pod-network-cidr=10.244.0.0/16
```

>说明：`--pod-network-cidr=10.244.0.0/16`是Flannel使用的网段，如果不打算使用Flannel的可以去掉该属性。如果有多网卡的，请根据实际情况配置`--api-advertise-addresses=<ip-address>`。

如果出现`ebtables not found in system path`的错误，要先安装`ebtables`包，我安装的过程中未提示，该包系统已经自带了。
```bash
yum install -y ebtables
```

安装过程大概2-3分钟，输出结果如下：

```bash
[kubeadm] WARNING: kubeadm is in alpha, please do not use it for production clusters.
[preflight] Running pre-flight checks
[init] Using Kubernetes version: v1.5.0
[tokens] Generated token: "064158.548b9ddb1d3fad3e"
[certificates] Generated Certificate Authority key and certificate.
[certificates] Generated API Server key and certificate
[certificates] Generated Service Account signing keys
[certificates] Created keys and certificates in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[apiclient] Created API client, waiting for the control plane to become ready
[apiclient] All control plane components are healthy after 61.317580 seconds
[apiclient] Waiting for at least one node to register and become ready
[apiclient] First node is ready after 6.556101 seconds
[apiclient] Creating a test deployment
[apiclient] Test deployment succeeded
[token-discovery] Created the kube-discovery deployment, waiting for it to become ready
[token-discovery] kube-discovery is ready after 6.020980 seconds
[addons] Created essential addon: kube-proxy
[addons] Created essential addon: kube-dns

Your Kubernetes master has initialized successfully!

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node:

kubeadm join --token=de3d61.504a049ec342e135 172.16.1.101
```

### 6 安装minion节点

Master节点安装好了Minoin节点就简单了。

```bash
kubeadm reset && systemctl start kubelet
kubeadm join --token=de3d61.504a049ec342e135 172.16.1.101
```

输出结果如下：

```bash
[kubeadm] WARNING: kubeadm is in alpha, please do not use it for production clusters.
[preflight] Running pre-flight checks
[preflight] Starting the kubelet service
[tokens] Validating provided token
[discovery] Created cluster info discovery client, requesting info from "http://172.16.1.101:9898/cluster-info/v1/?token-id=f11877"
[discovery] Cluster info object received, verifying signature using given token
[discovery] Cluster info signature and contents are valid, will use API endpoints [https://172.16.1.101:6443]
[bootstrap] Trying to connect to endpoint https://172.16.1.101:6443
[bootstrap] Detected server version: v1.5.0
[bootstrap] Successfully established connection with endpoint "https://172.16.1.101:6443"
[csr] Created API client to obtain unique certificate for this node, generating keys and certificate signing request
[csr] Received signed certificate from the API server:
Issuer: CN=kubernetes | Subject: CN=system:node:yournode | CA: false
Not before: 2016-12-15 19:44:00 +0000 UTC Not After: 2017-12-15 19:44:00 +0000 UTC
[csr] Generating kubelet configuration
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

安装完成后可以查看下状态：
```bash
[root@master ~]# kubectl get nodes
NAME       STATUS         AGE
master     Ready,master   6m
minion01   Ready          2m
minion02   Ready          2m
```

### 7 安装Flannel网络

网络组件选择很多，可以根据自己的需要选择calico、weave、flannel，calico性能最好，weave和flannel差不多。[Addons](http://kubernetes.io/docs/admin/addons/)中有配置好的yaml。

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

检查各节点组件运行状态：

```bash
[root@master ~]# kubectl get po -n=kube-system -o wide
NAME                                    READY     STATUS    RESTARTS   AGE       IP             NODE
dummy-2088944543-md22p                  1/1       Running   0          17m        172.16.1.101   master
etcd-master                             1/1       Running   0          19m        172.16.1.101   master
kube-apiserver-master                   1/1       Running   0          19m        172.16.1.101   master
kube-controller-manager-master          1/1       Running   0          17m        172.16.1.101   master
kube-discovery-2220406887-jqsdz         1/1       Running   0          17m        172.16.1.101   master
kube-dns-654381707-sxjsj                3/3       Running   0          17m        10.244.0.3     master
kube-flannel-ds-8dmw4                   2/2       Running   0          10m        172.16.1.106   minion01
kube-flannel-ds-gljhj                   2/2       Running   0          10m        172.16.1.107   minion02
kube-flannel-ds-kpwjd                   2/2       Running   0          10m        172.16.1.101   master
kube-proxy-sx5l9                        1/1       Running   0          17m        172.16.1.106   minion01
kube-proxy-59x41                        1/1       Running   0          17m        172.16.1.101   master
kube-proxy-pn1j9                        1/1       Running   0          17m        172.16.1.107   minion02
kube-scheduler-master                   1/1       Running   0          17m        172.16.1.101   master
```
>说明：kube-dns需要等flannel配置完成后才是running状态。