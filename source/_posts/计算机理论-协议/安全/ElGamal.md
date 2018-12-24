---
title: 非对称加密算法ElGamal
abbrlink: 3855446360
date: 2018-11-06 15:24:42
tags:
categories:
---
# 简介
ElGamal算法是一种常见加密算法, 与Diffie-Hellman算法有密切关联。
ElGamal算法，是由T.Elgamal于1984年提出的公钥密码体制和椭圆曲线加密体系。既能用于数据加密也能用于数字签名，其安全性依赖于计算有限域上离散对数这一难题即CDH假设。在加密过程中，生成的密文长度是明文的两倍，且每次加密后都会在密文中生成一个随机数K，其实现依据是求解离散对数是困难的，而其逆运算指数运算可以应用快速幂的方法有效地计算。也就是说，在适当的群G中，指数函数是单向函数。




# 参考
https://zh.wikipedia.org/wiki/ElGamal%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95
http://rangerway.com/way/public-key-one-elgamal
https://blog.mythsman.com/2015/10/07/3/