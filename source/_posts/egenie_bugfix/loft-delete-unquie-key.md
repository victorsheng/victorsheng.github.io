---
title: 软删除唯一索引解决
abbrlink: 2053783284
date: 2018-03-24 09:30:51
tags:
categories:
---
软件中使用软删除is_usable=1改为0来代表删除，但是遇到唯一索引时，
例如：username+is_usable的唯一索引
第二次删除则会失败，解决办法，将is_usable 改为其他不重复的数字及可