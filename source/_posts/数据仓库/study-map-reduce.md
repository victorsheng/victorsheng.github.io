---
title: study_map_reduce
tags:
  - MapReduce
categories:
  - BI
abbrlink: 4036242772
date: 2018-01-31 15:16:00
---


# 处理流程
- 在正式执行 Map 前，需要将输入数据进行 分片。所谓分片，就是将输入数据切分为大小相等的数据块，每一块作为单个 Map Worker 的输入被处理，以便于多个 Map Worker 同时工作。
< !– more –>

- 分片完毕后，多个 Map Worker 便可同时工作。每个 Map Worker 在读入各自的数据后，进行计算处理，最终输出给 Reduce。Map Worker 在输出数据时，需要为每一条输出数据指定一个 Key，这个 Key 值决定了这条数据将会被发送给哪一个 Reduce Worker。Key 值和 Reduce Worker 是多对一的关系，具有相同 Key 的数据会被发送给同一个 Reduce Worker，单个 Reduce Worker 有可能会接收到多个 Key 值的数据。

- 在进入 Reduce 阶段之前，MapReduce 框架会对数据按照 Key 值排序，使得具有相同 Key 的数据彼此相邻。如果您指定了 合并操作（Combiner），框架会调用 Combiner，将具有相同 Key 的数据进行聚合。Combiner 的逻辑可以由您自定义实现。与经典的 MapReduce 框架协议不同，在 MaxCompute 中，Combiner 的输入、输出的参数必须与 Reduce 保持一致，这部分的处理通常也叫做 <b>洗牌（Shuffle）</b>。

- 接下来进入 Reduce 阶段。相同 Key 的数据会到达同一个 Reduce Worker。同一个 Reduce Worker 会接收来自多个 Map Worker 的数据。每个 Reduce Worker 会对 Key 相同的多个数据进行 Reduce 操作。最后，一个 Key 的多条数据经过 Reduce 的作用后，将变成一个值。


# WordCount

![upload successful](/images/pasted-49.png)

操作步骤：

- 输入数据：对文本进行分片，将每片内的数据作为单个 Map Worker 的输入。

- Map 阶段：Map 处理输入，每获取一个数字，将数字的 Count 设置为 1，并将此<Word, Count>对输出，此时以 Word 作为输出数据的 Key。

- Shuffle > 合并排序：在 Shuffle 阶段前期，首先对每个 Map Worker 的输出，按照 Key 值（即 Word 值）进行排序。排序后进行 Combiner 操作，即将 Key 值（Word 值）相同的 Count 累加，构成一个新的<Word, Count>对。此过程被称为合并排序。

- Shuffle > 分配 Reduce：在 Shuffle 阶段后期，数据被发送到 Reduce 端。Reduce Worker 收到数据后依赖 Key 值再次对数据排序。

- Reduce 阶段：每个 Reduce Worker 对数据进行处理时，采用与 Combiner 相同的逻辑，将 Key 值（Word 值）相同的 Count 累加，得到输出结果。

- 输出结果数据。


# 学习资料
https://help.aliyun.com/document_detail/27875.html