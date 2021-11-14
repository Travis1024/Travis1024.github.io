---
title: 渗透问题-Percolation（XDU算法实验）
author: Travis <Hongxu Wei>
date: 2021-07-18 18:32:00 +0800
categories: [Algorithm, Coding]
tags: [Java, Course, 算法实验]
math: true
---



## 一、问题描述

使用合并-查找（union-find）数据结构，编写程序通过蒙特卡罗模拟（Monte Carlo simulation）来估计渗透阈值的值。

**安装Java编程环境**。按照以下各步指令，在你的计算机上（操作系统[Mac OS X](http://algs4.cs.princeton.edu/mac) （http://algs4.cs.princeton.edu/mac）· [Windows](http://algs4.cs.princeton.edu/windows) （http://algs4.cs.princeton.edu/windows）· [Linux](http://algs4.cs.princeton.edu/linux) （http://algs4.cs.princeton.edu/linux）安装Java编程环境。执行这些指令后，在你的Java classpath下会有[stdlib.jar](http://algs4.cs.princeton.edu/code/stdlib.jar) and [algs4.jar](http://algs4.cs.princeton.edu/code/algs4.jar)。前者包含库：从标准输入读数据、向标准输出写数据以及向标准绘制绘出结果，产生随机数、计算统计量以及计时程序；后者包含了教科书中的所有算法。

给定由随机分布的绝缘材料和金属材料构成的组合系统：金属材料占多大比例才能使组合系统成为电导体？ 给定一个表面有水的多孔渗水地形（或下面有油），水将在什么条件下能够通过底部排出（或油渗透到表面）？ 科学家们已经定义了一个称为渗透（**percolation**）的抽象过程来模拟这种情况。

**模型**。 我们使用**N**×**N**网格点来模型一个渗透系统。 每个格点或是**open**格点或是**blocked**格点。 一个**full** site是一个**open**格点，它可以通过一连串的邻近（左，右，上，下）**open**格点连通到顶行的一个**open**格点。如果在底行中有一个**full** site格点，则称系统是渗透的。（对于绝缘/金属材料的例子，**open**格点对应于金属材料，渗透系统有一条从顶行到底行的金属路径，且**full** sites格点导电。对于多孔物质示例，**open**格点对应于空格，水可能流过，从而渗透系统使水充满**open**格点，自顶向下流动。）

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image@main//20210718172140.png" style="zoom:80%;" />

**问题**。 在一个著名的科学问题中，研究人员对以下问题感兴趣：如果将格点以空置概率**p**独立地设置为**open**格点（因此以概率1-**p**被设置为**blocked**格点），系统渗透的概率是多少？ 当**p** = 0时，系统不会渗出; 当**p**=1时，系统渗透。 下图显示了20×20随机网格（左）和100×100随机网格（右）的格点空置概率**p**与渗滤概率。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image@main//20210718172220.png" style="zoom:67%;" />

当**N**足够大时，存在阈值**p***，使得当**p** <**p***，随机**N**´ **N**网格几乎不会渗透，并且当**p**> **p***时，随机**N**´ **N**网格几乎总是渗透。 尚未得出用于确定渗滤阈值**p***的数学解。你的任务是编写一个计算机程序来估计**p***。

**Percolation数据类型**。模型化一个Percolation系统，创建含有以下API的数据类型Percolation。

```java
public class Percolation {
  public Percolation(int N)      // create N-by-N grid, with all sites blocked
  public void open(int i, int j)      // open site (row i, column j) if it is not already
  public boolean isOpen(int i, int j) // is site (row i, column j) open?
  public boolean isFull(int i, int j) // is site (row i, column j) full?
  public boolean percolates()     // does the system percolate?
  public static void main(String[] args)  // test client, optional
}
```

约定行**i**列**j**下标在1和**N**之间，其中(1, 1)为左上格点位置：如果open(), isOpen(), or isFull()不在这个规定的范围，则抛出IndexOutOfBoundsException例外。如果**N** ≤ 0，构造函数应该抛出IllegalArgumentException例外。构造函数应该与**N**2成正比。所有方法应该为常量时间加上常量次调用合并-查找方法union(), find(), connected(), and count()。

**蒙特卡洛模拟（Monte Carlo simulation）**. 要估计渗透阈值，考虑以下计算实验：

· 初始化所有格点为**blocked**。

· 重复以下操作直到系统渗出：

o 在所有**blocked**的格点之间随机均匀选择一个格点 (row **i**, column **j**)。

o 设置这个格点(row **i**, column **j**)为**open**格点。

*·* **open**格点的比例提供了系统渗透时渗透阈值的一个估计。

例如，如果在20×20的网格中，根据以下快照的**open**格点数，那么对渗滤阈值的估计是204/400 = 0.51，因为当第204个格点被**open**时系统渗透。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image@main//20210718172424.png" style="zoom:67%;" />

通过重复该计算实验**T**次并对结果求平均值，我们获得了更准确的渗滤阈值估计。 令xt是第t次计算实验中open格点所占比例。 样本均值m提供渗滤阈值的一个估计值； 样本标准差s测量阈值的灵敏性。

## 二、实验报告+源代码
[算法实验报告+代码开源地址（GitHub）](https://github.com/Travis1024/Course_Code/tree/main/Algorithm_Experiment)

