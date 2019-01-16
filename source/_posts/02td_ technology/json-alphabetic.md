---
title: json-alphabetic
abbrlink: 1319210670
date: 2018-10-30 11:41:26
tags:
categories:
---
# 类注解
https://stackoverflow.com/questions/27577701/jackson-objectmapper-specify-serialization-order-of-object-properties

在model上添加注解
```
// ensure that "id" and "name" are output before other properties
@JsonPropertyOrder({ "id", "name" })

// order any properties that don't have explicit setting using alphabetic order
@JsonPropertyOrder(alphabetic=true)
```
然而经过测试只针对这一层级,确认的字段,下一层级,如果用Object存储,无法指定顺序,同下面的问题

https://stackoverflow.com/questions/33186004/jackson-jsonpropertyorder-is-ignored

# objectMapper参数
objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY, true);

https://www.ibm.com/developerworks/cn/java/jackson-advanced-application/index.html

最后发现依然没有效果