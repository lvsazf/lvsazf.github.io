---
layout: post
title:  openstack之trove（1）
categories: [openstack]
date: 2020-07-20 10:58:30 +0800
keywords: [openstack,trove]
---

> 本文简单了解trove的架构

### 高层描述

trove设计来支持在单个Nova实例上管理多租户数据库。因为Trove组件仅仅通过api别的组件进行交互，因此Nova如何配置对它没任何影响。

### trove-api

trove-api提供支持json数据格式的rest api定义和管理trove实例

* 提供rest-ful 的组件
* 入口-trove/cmd/api.py
* 通过etc/trove/api-paste.ini配置WSGI启动器
* 定义过滤器链，用户鉴权、限流等
* 为trove应用定义app_factory作为trove.common.api:app_factory
* api类将rest路径定义到适当的controller中
* 在service.py中实现相关模块（versions/instance/backup/configuration）中的controller
* controller通常将实现重定向到models.py模块中的一个类中
* 就这点而言，别的组件（TaskManager, GuestAgent等）的api模块用RabbitMQ发送请求

### trove-taskmanage

trove-taskmanage的重任是配置实例，管理实例的生命周期和在数据库上执行操作

* 监听RabbitMQ
* 入口-trove/cmd/taskmanager.py
* 通过etc/trove/trove.conf配置RpcService，这个配置文件定义了trove.taskmanager.manager.Manage，是管理员主要通过队列到达的切入点
* As described above, requests for this component are pushed to MQ from another component using the TaskManager’s api module using _cast() or _call() (sync/a-sync) and putting the method’s name as a parameter
* In module oslo.messaging, oslo_messaging/rpc/dispatcher.py - RpcDispatcher.dispatch() invokes the proper method in the Manager by some equivalent to reflection
* The Manager then redirect the handling to an object from the models.py module. It loads an object from the relevant class with the context and instance_id
* 实际在models.py中处理

### trove-guestagent

guestagent是运行在guest实例上的服务，负责在数据库自身上管理和执行操作。通过消息总线监听RPC消息和处理请求操作

* 作为监听MQ消息的服务的意义上来说，和taskManage类似
* 运行在每一个DB实例上，和独享一个rabbit topic
* 入口-trove/cmd/guest.py
* Runs as a RpcService configured by /etc/trove/conf.d/trove-guestagent.conf which defines trove.guestagent.datastore.manager.Manager as the manager - basically this is the entry point for requests arriving through the queue
* As described above, requests for this component are pushed to MQ from another component using the GuestAgent’s api module using _cast() or _call() (sync/a-sync) and putting the method’s name as a parameter
* In module oslo.messaging, oslo_messaging/rpc/dispatcher.py - RpcDispatcher.dispatch()invokes the proper method in the Manager by some equivalent to reflection
* The Manager then redirect the handling to an object (usually) from the dbaas.py module.
* The database service is running as a docker container inside the Nova VM.

### trove-conductor

conductor是一个运行在主机上的服务，负责从guest实例上接收消息更新信息。如实例状态和备份的当前状态。使用conductor，guest实例不需要重定向连接到主机上的数据库。conductor通过消息总线监听RPC消息和执行相关的操作。
* 和guest-agent类似，监听MQ，不同的是conductor运行在主机上，而不是agent上
* Guest agents communicate to conductor by putting messages on the topic defined in cfg as conductor_queue. By default this is “trove-conductor”.
* 入口 - trove/cmd/conductor.py
* Runs as RpcService configured by etc/trove/trove.conf which defines trove.conductor.manager.Manager as the manager. This is the entry point for requests arriving on the queue.
* As guestagent above, requests are pushed to MQ from another component using _cast() (synchronous), generally of the form {“method”: “<method_name>”, “args”: {<arguments>}}
* 实际数据库更新工作由by trove/conductor/manager.py完成
* The “heartbeat” method updates the status of an instance. This is used to report that instance has changed from NEW to BUILDING to ACTIVE and so on.
* The “update_backup” method changes the details of a backup, including its current status, size of the backup, type, and checksum.


