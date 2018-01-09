title: study-skip-list
date: 2018-01-05 11:16:40
tags:
categories:
---
# 跳跃表
- 一种基于有序链表的扩展
- 利用类似索引的思想,找到关键点
- <img src="http://pic.victor123.cn/18-1-4/93090030.jpg">


# 删除
- 自上而下，查找第一次出现节点的索引，并逐层找到每一层对应的节点。O（logN）
- 删除每一层查找到的节点，如果该层只剩下1个节点，删除整个一层（原链表除外）。O（logN）

# 区别
- 跳跃表 维持平衡的成本较低,完全靠随机
- 二叉树需要靠rebalance来重新调整结构

# 应用
Redis当中的Sorted-set
