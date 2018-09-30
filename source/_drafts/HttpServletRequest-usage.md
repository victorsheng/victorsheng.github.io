---
title: HttpServletRequest-usage
abbrlink: 2247303554
tags:
categories:
---
# 思考:HttpServletRequest和 HttpServletRequestWrapper 用法上有什么区别
- 自己的类继承HttpServletRequestWrapper,同时添加自己的方法
- 要对HttpServletRequest添加方法或者进行增强时,会在filter 中进行拦截,将原HttpServletRequest 接口对象传入HttpServletRequestWrapper的构造方法中

# 举例 
- MultiReadHttpServletResponse
- MultiReadHttpServletRequest
