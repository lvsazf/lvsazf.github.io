---
layout: post
title:  maven 打包跳过测试
categories: [maven]
date: 2020-06-05 10:58:30 +0800
keywords: [maven,skip]
---

> maven打包的时候，有时候需要跳过测试，maven该如何跳过单元测试呢？
maven在打包的时候跳过单元测试有两种方式-DskipTests和-Dmaven.test.skip=true，那么有什么区别呢？
- -DskipTests
跳过测试用例，但是编译测试用例到target/test-classes下  
- -Dmaven.test.skip
跳过测试用例，也不编译测试用例

上面两种可以在执行maven命令的时候使用，如
```code
maven package -DskipTests
maven package -Dmaven.test.skip=true
```
也可以直接配置在pom.xml中
-DskipTests
```code
<plugin>  
    <groupId>org.apache.maven.plugin</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>2.1</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin>  
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin> 
```
-Dmaven.test.skip

```code
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skipTests>true</skipTests>  
    </configuration>  
</plugin> 
```
