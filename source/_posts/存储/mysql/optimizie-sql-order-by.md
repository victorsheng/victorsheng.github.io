---
title: 优化mysql排序
tags:
  - mysql
categories:
  - mysql
abbrlink: 4014084071
date: 2018-03-24 15:34:50
---
# mysql排序方式:
## 常规排序
-  a 从表t1中获取满足WHERE条件的记录
-  b 对于每条记录，将记录的主键+排序键(id,col2)取出放入sort buffer
-  c 如果sort buffer可以存放所有满足条件的(id,col2)对，则进行排序；否则sort buffer满后，进行排序并固化到临时文件中。(排序算法采用的是快速排序算法)
-  d 若排序中产生了临时文件，需要利用归并排序算法，保证临时文件中记录是有序的
-  e 循环执行上述过程，直到所有满足条件的记录全部参与排序
-  f 扫描排好序的(id,col2)对，并利用id去捞取SELECT需要返回的列(col1,col2,col3)
-  g 将获取的结果集返回给用户。

从上述流程来看，是否使用文件排序主要看sort buffer是否能容下需要排序的(id,col2)对，这个buffer的大小由sort_buffer_size参数控制。此外一次排序需要两次IO，一次是捞(id,col2),第二次是捞(col1,col2,col3)，**由于返回的结果集是按col2排序，因此id是乱序的，通过乱序的id去捞(col1,col2,col3)时会产生大量的随机IO**。对于第二次MySQL本身一个优化，即在捞之前首先将id排序，并放入缓冲区，这个缓存区大小由参数read_rnd_buffer_size控制，然后有序去捞记录，将随机IO转为顺序IO。

## 优化排序
    常规排序方式除了排序本身，还需要额外两次IO。优化的排序方式相对于常规排序，减少了第二次IO。主要区别在于，放入sort buffer不是(id,col2),而是(col1,col2,col3)。由于sort buffer中包含了查询需要的所有字段，因此排序完成后可以直接返回，无需二次捞数据。这种方式的代价在于，同样大小的sort buffer，能存放的(col1,col2,col3)数目要小于(id,col2)，如果sort buffer不够大，可能导致需要写临时文件，造成额外的IO。当然MySQL提供了参数max_length_for_sort_data，只有当排序元组小于max_length_for_sort_data时，才能利用优化排序方式，否则只能用常规排序方式。


## 优先队列排序
     为了得到最终的排序结果，无论怎样，我们都需要将所有满足条件的记录进行排序才能返回。那么相对于优化排序方式，是否还有优化空间呢？5.6版本针对Order by limit M，N语句，在空间层面做了优化，加入了一种新的排序方式:优先队列，这种方式采用堆排序实现。堆排序算法特征正好可以解limit M，N 这类排序的问题，虽然仍然需要所有元素参与排序，但是只需要M+N个元组的sort buffer空间即可，对于M，N很小的场景，基本不会因为sort buffer不够而导致需要临时文件进行归并排序的问题。对于升序，采用大顶堆，最终堆中的元素组成了最小的N个元素，对于降序，采用小顶堆，最终堆中的元素组成了最大的N的元素。



# 对比页数比较大的情况
低效率:

![upload successful](/images/pasted-086.png)

高效率:

![upload successful](/images/pasted-087.png)

# 针对limit 优化有很多种方式
1 前端加缓存、搜索，减少落到库的查询操作。比如海量商品可以放到搜索里面，使用瀑布流的方式展现数据，很多电商网站采用了这种方式。
2 优化SQL 访问数据的方式，直接快速定位到要访问的数据行。
3 使用**书签方式** ，记录上次查询最新/大的id值，向后追溯 M行记录。

-----
## 快速定位
对于第二种方式 我们推荐使用"延迟关联"的方法来优化排序操作，何谓"延迟关联" ：通过使用覆盖索引查询返回需要的主键,再根据主键关联原表获得需要的数据。
3.1 延迟关联
```
root@xxx 12:33:48>explain SELECT id, cu_id, name, info, biz_type, gmt_create, gmt_modified,start_time, end_time, market_type, back_leaf_category,item_status,picuture_url FROM relation where biz_type ='0' AND end_time >='2014-05-29' ORDER BY id asc LIMIT 149420 ,20;

explain SELECT a.* FROM relation a, (select id from relation where biz_type ='0' AND end_time >='2014-05-29' ORDER BY id asc LIMIT 149420 ,20 ) b where a.id=b.id;

```

## 书签方式
首先要获取复合条件的记录的最大 id和最小id(默认id是主键)
```
select max(id) as maxid ,min(id) as minid from t where kid=2333 and type=1;
```

其次 根据id 大于最小值或者小于最大值 进行遍历。

```
select xx,xx from t where      kid=2333 and type=1 and id >=min_id order by id asc  limit 100;

select xx,xx from t where      kid=2333 and type=1 and id <=max_id order by id  desc limit 100;
```

使用延迟关联查询数据510ms ，使用基于书签模式的解决方法减少到10ms以内 绝对是一个质的飞跃。
