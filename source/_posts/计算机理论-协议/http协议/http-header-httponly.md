---
title: http-header-httponly
abbrlink: 4194410282
date: 2018-06-06 15:53:09
tags:
categories:
---

# 属性说明：
1 secure属性
当设置为true时，表示创建的 Cookie 会被以安全的形式向服务器传输，也就是只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证，如果是 HTTP 连接则不会传递该信息，所以不会被窃取到Cookie 的具体内容。
2 HttpOnly属性
为了解决XSS（跨站脚本***）的问题，从IE6开始支持cookie的HttpOnly属性，这个属性目前已被大多数浏览器（IE、FF、Chrome、Safari）


如果在Cookie中设置了"HttpOnly"属性，那么通过程序(JS脚本、Applet等)将无法读取到Cookie信息，这样能有效的防止XSS攻击。
 
对于以上两个属性，
首先，secure属性是防止信息在传递的过程中被监听捕获后信息泄漏，HttpOnly属性的目的是防止程序获取cookie后进行攻击。
其次，GlassFish2.x支持的是servlet2.5，而servlet2.5不支持Session Cookie的"HttpOnly"属性。不过使用Filter做一定的处理可以简单的实现HttpOnly属性。GlashFish3.0(支持servlet3.0)默认开启Session Cookie的HttpOnly属性。
也就是说两个属性，并不能解决cookie在本机出现的信息泄漏的问题(FireFox的插件FireBug能直接看到cookie的相关信息)。
 