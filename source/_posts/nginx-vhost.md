---
title: nginx_vhost
date: 2019-01-14 16:55:51
tags:
categories:
---
# 配置方式
## nginx.conf中添加:
```
include vhost.d/*.conf;
```

## vhost.d文件夹中
80.conf
```
server {
    listen       80;
    server_name  localhost;
    client_max_body_size 1G;

    //真实路径
    location /demoURL/ {
        //upstream的名字
        proxy_pass http://demo/url/;
        proxy_http_version 1.1;
        proxy_set_header   Connection          "";
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

```

upstream.conf
```
upstream demo {
    server     0.0.0.0:9303;
    keepalive 200;
}
```
