---
title: nginx-proxy_pass-path
abbrlink: 2351374196
date: 2018-09-17 14:14:22
tags:
categories:
---
## 配置upstream
```
upstream demo {
    server     172.11.11.11:9302;
    keepalive 200;
}
```


## 第一种相对路径(原始转发)
```
location /proxy/ {
         proxy_pass http://demo;
    }
```
当访问 http://nginx_ip/proxy/cuffs/css/toosimple.txt时，nginx匹配到/proxy/路径，把请求转发给demo服务，实际请求代理服务器的路径为
http://demo/proxy/cuffs/css/toosimple.txt 此时nginx会把匹配的proxy也代理给代理服务器


## 第二种绝对路径
```
location /proxy/ {
         proxy_pass http://demo/;
    }
```
当访问 http://nginx_ip/proxy/cuffs/css/toosimple.txt时，nginx匹配到/proxy/路径，把请求转发给demo服务，实际请求代理服务器的路径为
http://demo/cuffs/css/toosimple.txt


## 第三种
```
location /proxy/ {
         proxy_pass http://demo/static01/;
    }
```
当访问 http://nginx_ip/proxy/cuffs/css/toosimple.txt时，nginx匹配到/proxy/路径，把请求转发给demo服务，实际请求代理服务器的路径为
http://demo/static01/cuffs/css/toosimple.txt

## 第四种
```
location /proxy/ {
         proxy_pass http://demo/static01;
    }
```
当访问 http://nginx_ip/proxy/cuffs/css/toosimple.txt时，nginx匹配到/proxy/路径，把请求转发给demo服务，实际请求代理服务器的路径为
http://demo/static01cuffs/css/toosimple.txt
