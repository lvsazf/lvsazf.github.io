---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---
f = open('D:/svn.hw/trunk/basecode.txt','r')
f.read()
运行报错：
    UnicodeDecodeError: 'gbk' codec can't decode byte 0xba in position 146: illegal multibyte sequence

修改方式一：
    读取文件带上字符集
    f = open('D:/svn.hw/trunk/basecode.txt','r',encoding='utf-8')
修改方式二:
    f = open('D:/svn.hw/trunk/basecode.txt','rb')
