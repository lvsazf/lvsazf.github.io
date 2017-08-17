---
layout: post
title: ssh ubuntu
categories: [Linux]
date: 2017-08-17 10:58:30 +0800
description: 远程ssh报Host key verification
keywords: ubuntu, ssh
---

>远程ssh的时候报Host key verification faild解决办法：
vi /etc/ssh/ssh_config添加
StrictHostKeyChecking no
UserKnownHostsFile /dev/null
