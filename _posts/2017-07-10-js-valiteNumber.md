---
layout: post
title: obj.length === +obj.length 
categories: [Language]
description: some word here
date:2017-07-10 10:58:30 +0800
keywords: [javascript, length]
---
->js
#obj.length === +obj.length
obj.length === +obj.length 等同于 type obj.length === 'number' && !isNaN(obj.length)
