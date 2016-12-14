---
layout: post
title:  kubeadm快速安装kubernetes
categories: [docker,Cloud]
date: 2016-12-14 10:58:30 +0800
keywords: [docker,云计算,kubernetes]
---

>Kubernetes 1.5.0刚刚发布，添加了众多的新特性，我们的云平台也计划从Mesos和Rancher迁移到kubernetes，所以迫不及待地想尝试一下，Google出品，必属精品，无奈GFW的层层阻挡，让原本简单的部署步骤变得异常复杂，所以写下此文，供各位参考。

### 1. 环境准备
准备了三台机器作安装测试工作，机器信息如下: 

|      IP      |   Name   |    Role    |           OS           |
|--------------|----------|------------|------------------------|
| 172.16.1.101 | Master01 | Controller | Centos7.2+docker1.12.4 |
| 172.16.1.106 | Minion01 | Compute    | Centos7.2+docker1.12.4 |
| 172.16.1.107 | Minino02 | Compute    | Centos7.2+docker1.12.4 |

```bash
#!/bin/bash
images=(kube-proxy-amd64:v1.5.0 kube-discovery-amd64:1.0 kubedns-amd64:1.7 kube-scheduler-amd64:v1.5.0 kube-controller-manager-amd64:v1.5.0 kube-apiserver-amd64:v1.5.0 etcd-amd64:2.2.5 kube-dnsmasq-amd64:1.3 exechealthz-amd64:1.1 pause-amd64:3.0 kubernetes-dashboard-amd64:v1.5.0)
for imageName in ${images[@]} ; do
  docker pull cloudnil/$imageName
  docker tag cloudnil/$imageName gcr.io/google_containers/$imageName
  docker tag cloudnil/$imageName hub.cloudx5.com/$imageName
  docker push hub.cloudx5.com/$imageName
  docker rmi cloudnil/$imageName
  docker rmi hub.cloudx5.com/$imageName
done
```