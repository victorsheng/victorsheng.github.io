---
title: study-nginx-statistics
date: 2018-03-20 14:02:25
tags:
categories:
---
根据nginx日志,查询访问最频繁的IP


1.根据访问IP统计UV
```
awk '{print $1}'  /var/log/nginx/access.log|sort | uniq -c |wc -l
```

2.统计访问URL统计PV
```
awk '{print $7}' /var/log/nginx/access.log|wc -l
```

3.查询访问最频繁的URL
```
awk '{print $7}' /var/log/nginx/access.log|sort | uniq -c |sort -n -k 1 -r|more
```

4.查询访问最频繁的IP
```
awk '{print $1}' /var/log/nginx/access.log|sort | uniq -c |sort -n -k 1 -r|more
```
