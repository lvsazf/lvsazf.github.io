---
layout: post
title:  Cassandra idea配置
categories: [Distribution]
date: 2020-04-13 10:58:30 +0800
keywords: [Cassandra, idea]
---

>本文主要描述Cassandra在idea环境下的配置

Cassandra的源码开发环境还是比较简单的，基本按照官方文档一步步走下来就可以了。本文基于Cassaandra 4介绍。

### 环境准备
1. java1.8 
2. ant 

### 步骤

* 下载
  ``` bash
  git clone https://gitbox.apache.org/repos/asf/cassandra.git cassandra-trunk
  ```

* 选择合适的分支

  ``` bash
  git checkout cassandra-3.0
  ```
* 使用ant编译

  ``` bash
  ant
  ```

* IntelliJ IDEA 设置

  从2.1.5版本开始，采用新的ant target ：<code>generate-idea-files</code>

    1. 生成idea 文件

        ``` bash
        ant generate-idea-files
        ```

    2. 启动idea

    3. 打开项目

    <code>generate-idea-files</code> 任务生成的工程几乎包含所有debug cassandra和junit单元测试

    * Run/debug defaults for JUnit
    * Run/debug configuration for Cassandra daemon
    * License header for Java source files
    * Cassandra code style
    * Inspections

Cassandra支持的ide还包含Apache NetBeans、Eclipse，介于笔者目前没需求，就不做介绍了，感兴趣的童鞋移步 [Cassandra构建及IDE配置](http://cassandra.apache.org/doc/latest/development/ide.html#)



   
    