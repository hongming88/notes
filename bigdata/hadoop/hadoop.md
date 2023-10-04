# Hadoop概述

## 大数据概念

大数据（Big Data）：指无法在一定时间范围内用常规软件工具进行捕捉、管理和
处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化
能力的海量、高增长率和多样化的信息资产。

## Hadoop是什么
1）Hadoop是一个==由Apache基金会所开发的**分布式系统基础架构**==。
2）==主要解决，海量数据的**存储**和海量数据的分析**计算**问题==。
3）是大数据技术中的基石。用户可以在不了解分布式底层细节的情况下，开发分布式程序，用户可以==轻松地在Hadoop上开发和**运行处理海量数据的应用程序**==。

## Hadoop 优势（4 高）

1. ==高可靠性==：Hadoop底层==维护多个数据副本==，所以即使Hadoop**某个计算元**
   **素或存储出现故障，也不会导致数据的丢失**。
2. ==高扩展性==：在==集群间分配任务数据==，可方便的扩展数以千计的节点。
3. ==高效性==：在MapReduce的思想下，==Hadoop是**并行工作的==**，以加快任务处
   理速度。
4. ==高容错性==：能够自动将失败的任务重新分配。

## Hadoop组成（面试重点）

### HDFS架构概述

`Hadoop Distributed File System`，简称HDFS，是一个==分布式文件系统==。

1. `NameNode（nn）`：==存储文件的元数据==，如**文件名，文件目录结构，文件属性**（生成时间、副本数、文件权限），以及每个文件的==块列表==和==块所在的DataNode==等。
2. `DataNode(dn)`：在本地文件==系统存储文件块数据==，以及块数据的校验和。
3. `Secondary NameNode(2nn)`：每隔一段时间==对NameNode元数据备份==。

### YARN架构概述

`Yet Another Resource Negotiator`简称YARN ，另一种资源协调者，==是Hadoop的资源管理器==。

1. `ResourceManager（RM）`：整个集群资源（内存、CPU等）的老大
2. `ApplicationMaster（AM）`：单个任务运行的老大
3. `NodeManager（NM）`：单个节点服务器资源老大
4. `Container`：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如==内存、CPU、磁盘、网络等。==



- ==客户端可以有多个==
- 集群上可以运行==多个ApplicationMaster==
- ==每个NodeManager上可以有多个Container==

### MapReduce 架构概述

MapReduce 将计算过程分为两个阶段：`Map` 和`Reduce`

1. `Map` 阶段并行处理输入数据
2. `Reduce`阶段对Map 结果进行汇总

### HDFS、YARN、MapReduce 三者关系



![](..\..\pics\bigdata\hadoop\hadoopRelate.PNG)

