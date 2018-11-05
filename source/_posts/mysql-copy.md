---
title: mysql-copy
abbrlink: 3653935070
date: 2018-11-02 15:25:28
tags:
categories:
---
# 进行不同实例mysql之间的数据拷贝

```
mysqldump -h172.20.33.8 -uroot -p123456 -C --databases opal_data_share_demo --tables all_tag_score_format tag_info  > ~/tmp/d.sql

source ~/tmp/d.sql
```


# json聚合

SELECT
count(ifnull(tags->'$."3"',null)) as "a3",
from all_tag_score_format

