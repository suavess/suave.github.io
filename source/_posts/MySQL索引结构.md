---
title: MySQL索引结构
date: 2022-02-22 10:38:57
tags: 
  - MySQL 
  - 索引
  - 数据结构
category:
  - MySQL
---

`索引是在MySQL的存储引擎层中实现的，而不是在服务器层实现的`。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型的。MySQL目前提供了以下4种索引：

- BTREE 索引 ： 最常见的索引类型，大部分索引都支持 B 树索引。
- HASH 索引：只有Memory引擎支持 ， 使用场景简单 。
- R-tree 索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少，不做特别介绍。
- Full-text （全文索引） ：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从Mysql5.6版本开始支持全文索引。

**MyISAM、InnoDB、Memory三种存储引擎对各种索引类型的支持**

| 索引        | InnoDB引擎      | MyISAM引擎 | Memory引擎 |
| ----------- | --------------- | ---------- | ---------- |
| BTREE索引   | `支持`          | `支持`     | `支持`     |
| HASH 索引   | 不支持          | 不支持     | `支持`     |
| R-tree 索引 | 不支持          | 支持       | 不支持     |
| Full-text   | 5.6版本之后支持 | 支持       | 不支持     |

我们平常所说的索引，如果没有特别指明，都是指B+树（多路搜索树，并不一定是二叉的）结构组织的索引。其中聚集索引、复合索引、前缀索引、唯一索引默认都是使用 B+tree 索引，统称为 索引。

### BTree索引

BTree又叫多路平衡搜索树，一颗m叉的BTree特性如下：

- 树中每个节点最多包含m个孩子。
- 除根节点与叶子节点外，每个节点至少有[(m/2)]个孩子。
- 若根节点不是叶子节点，则至少有两个孩子。
- 所有的叶子节点都在同一层。
- 每个非叶子节点由n个key与n+1个指针组成，其中[ceil(m/2)-1] <= n <= m-1

以5叉BTree为例，key的数量：公式推导[ceil(m/2)-1] <= n <= m-1。所以 2 <= n <=4 。当n>4时，中间节点分裂到父节点，两边节点分裂。

插入 C N G A H E K Q M F W L T Z D P R X Y S 数据为例。

演变过程如下：

1). 插入前4个字母 C N G A

![1555944126588](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944126588.png)

2). 插入H，n>4，中间元素G字母向上分裂到新的节点

![1555944549825](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944549825.png)

3). 插入E，K，Q不需要分裂

![1555944596893](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944596893.png)

4). 插入M，中间元素M字母向上分裂到父节点G

![1555944652560](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944652560.png)

5). 插入F，W，L，T不需要分裂

![1555944686928](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944686928.png)

6). 插入Z，中间元素T向上分裂到父节点中

![1555944713486](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944713486.png)

7). 插入D，中间元素D向上分裂到父节点中。然后插入P，R，X，Y不需要分裂

![1555944749984](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944749984.png)

8). 最后插入S，NPQR节点n>5，中间节点Q向上分裂，但分裂后父节点DGMT的n>5，中间节点M向上分裂

![1555944848294](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555944848294.png)

到此，该BTREE树就已经构建完成了， `BTREE树 和 二叉树 相比， 查询数据的效率更高， 因为对于相同的数据量来说，BTREE的层级结构比二叉树小，因此搜索速度快。`

### B+TREE 结构

B+Tree为BTree的变种，B+Tree与BTree的区别为：

>1). n叉B+Tree最多含有n个key，而BTree最多含有n-1个key。
>
>2). B+Tree的叶子节点保存所有的key信息，依key大小顺序排列。
>
>3). 所有的非叶子节点都可以看作是key的索引部分。

![1555906287178](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/00001.jpg)

由于B+Tree只有叶子节点保存key信息，查询任何key都要从root走到叶子。所以B+Tree的查询效率更加稳定。

#### MySQL中的B+Tree

`MySql索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能。`

MySQL中的 B+Tree 索引结构示意图:

![1555906287178](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1555906287178.png)

