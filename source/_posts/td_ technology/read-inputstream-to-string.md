---
title: read-inputstream-to-string
abbrlink: 1376762376
date: 2018-08-01 16:26:53
tags:
categories:
---

https://blog.csdn.net/lmy86263/article/details/60479350

# 方法4:
Scanner s = new Scanner(inputStream).useDelimiter("\\A");
String str = s.hasNext() ? s.next() : "";

# 方法5:
String resource = new Scanner(inputStream).useDelimiter("\\Z").next();
return resource;

## 其中\A代表字符串的开始

```
^ Matches the beginning of a line.

$ Matches the end of a line.

\A Matches the beginning of the string.

\z Matches the end of the string.

\Z Matches the end of the string unless the string ends with a "\n", in which case it matches just before the "\n".
```