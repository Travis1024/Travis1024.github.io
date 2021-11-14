﻿---
title: An architectural approach to the design and analysis of cyber-physical systems（CPS）
author: Travis <Hongxu Wei>
date: 2021-07-24 18:32:00 +0800
categories: [Paper Summary]
tags: [CPS, Course]
math: true
---



# 《信息物理系统》报告

**论文：[28]《An architectural approach to the design and analysis of cyber-physical systems.》**

## 一、文章是否具备新颖性？是否正确，包括思路、推导、语法、表达等？有哪些比较有趣的现象和发现？

- **新颖性：**

文章是具有新颖性的，文章介绍了一种新的支持替代架构的原理设计和评估的CPS架构风格。新的CPS架构风格使用了带插件的有限状态流程（FSP）或者带插件的线性混合自动机(LHA)，并且使用带标记的转换系统分析器(LTSA)或多面体混合自动机验证者(PHAVER)来执行行为分析。

- **现象与发现：**

文章比较形象的是提出了一种模型（物理家庭温度控制系统的建筑建模）来对CPS架构风格的架构进行了建模演示，并对其中的体系结构，运行机理，组件连接与功能，建模框架继续了分析。

在模型运行时发现了一个错误，每当炉子在本地通电时，它会默认进入Poweredon状态。假设恒温器只向炉子向炉子发送命令时会打开（关闭），并且不会交叉检查，看看炉子是否实际响应。如果恒温器向炉子发出命令以在炉子局部断电的同时打开热量，则炉子会偏离命令。如果炉子接通电源，则默认进入Poweredon状态，直到它接收到从恒温器接收到下一个热量的下一个热量。但是，恒温器不会发出另一个命令，因为它已经发出了热量的命令并且正在等待温度增加到需要发出散热指令的点。

这显然是一个错误的过程，文章中也提出了一种补救措施。文章使用带标记的转换系统分析器（LTSA）分析了活动特性，成功捕捉到了这个错误，同时也提供了一个反例跟踪，最终发现了活动性需求失败的原因—恒温器未能成功判断捕获熔炉的状态。

最终文章中就是使用带插件的线性混合自动机(LHA)，在多面体混合自动机验证者(PHAVER)中进行LHA分析可以在加热和冷却时发现最小和最大值的变化率的良好范围，成功解决了发现的问题。

文章将理论与实践应用进行了结合，将理论方法应用在实际模型出现的问题中，并成功解决了模型出现的问题，以实际的模型展示了文章提出的理论方法的有效性，更具说服力。

## 二、系统模型是什么？要解决的问题是什么？

### 1.系统模型

#### 1）总体目标

创建一个可扩展的框架，可以创建一套全面的设计工具，创建一个一般的一般组件和连接器，可以作为本广域的特定于应用程序特定样式的基础。

#### 2）创新之处

a) 本文提出了一种新的CPS架构风格和行为注释和用于验证的插件。

b) 我们定义了与网络域，物理域及其互连有关的三个相关家族。

c) 构建能够从属性中生成可分析文本文件的插件。

d) 开发FSP插件：可以产生一个由标记的转换系统分析器分析的文件

e) 开发LHA插件：可以通过多面体混合自动机验证器分析的文件

### 2.要解决的问题

当今的信息物理系统(CPS)的分析和设计模型和方法典型地沿着不同的数学形式主义和工程和计算机科学中不同的方法定义的路线分散。虽然为了便于处理，需要将关注点分离，但这种分析方法往往在系统设计的网络和物理特征之间施加早期分离，使得很难评估跨越这些领域边界的替代方案的影响和权衡。

本文介绍了软件架构描述的扩展，以包含包含网络物理系统的全系列元素。我们的目标是创建一个可扩展的框架，可以创建一套全面的设计工具。作为这方面的第一步，本文提出了一种新的CPS架构风格和行为注释和用于验证的插件。

体系结构通常在比仿真模型更高的层次上表示系统，仿真模型表示特定系统实现的细节。尽管体系结构建模已经被用于特定领域，以将物理元素作为组件合并，但目前还没有一种方法以一般的方式平等地对待网络元素和物理元素。这是因为软件体系结构风格中的组件和连接器不足以表示CPS中的物理组件类型及其与网络实体的交互。

## 三、假设是否成立，推导是否正确，用到了哪些具体的技术？

文章中的假设定义，模型构建，文本插件都具有一定的现实意义与正确性。

### 1.用到的技术：

#### 1）软件体系结构：

软件体系结构已成为大规模软件系统的严格工程设计的主要技术之一。软件体系结构通常将系统建模为组件和连接器的图形，其中组件表示系统运行时结构的主要计算元素，而连接器表示组件之间的通信路径。这些元素带有属性，这些属性描述了它们的抽象行为，并有助于进行系统级设计权衡的推理。

#### 2）Acme ADL

它对定义灵活的建筑风格提供了强大的支持。在Acme中，建筑风格表示为一系列元素类型，这些元素类型遵循某些规则和结构，涉及系统中可以存在的组件和连接器的类型以及它们之间相互连接的方式。通过添加其他元素和规则，一般用户可以将其简化为特定于应用程序的系列。

#### 3）两种网络连接器

一个是表示一对一通信的调用-返回连接器，另一个是表示一对多的发布-订阅连接器。每种类型都可以进一步专门化，以表示特定的通信协议和网络特征，表示系统中的通信元素对于解释软件元素之间的时序以及这如何影响整个系统的物理行为是很重要的。

## 四、结果是什么？是否存在亮点和启示？未来的扩展方向是什么？

### 1.结论

本文介绍了用于在架构级别对网络物理系统进行建模和分析的工具。提出了CPS架构样式，以及用于使用行为描述建模为有限状态过程和线性混合自动机的工具来注释CPS架构的工具。本文介绍的CPS体系结构样式提供了一个统一的框架，用于开发针对性能度量（该行为表征系统行为的重要特征）来优化设计的新方法。

### 2.未来的扩展方向：

未来可以开发提供CPS体系结构的不同视图的功能，其中每个视图可以对应于特定类型的数学抽象，并且不同的性能度量可以与不同的视图相关联。视图的收集和相关的性能度量导致设计优化问题网络，这些问题通过共享变量和约束相互关联。也可以开发公式化和解决这些相互联系的设计问题的方法，以此作为综合方法来分析和设计网络物理系统的基础。

## 五、文章的组织结构和表达如何？

文章在摘要部分描述了文章的整体工作，第一部分为文章介绍部分，主要介绍了文章的整体结构和内容分布；第二部分主要介绍了之前已经完成的工作；第三部分提出了CPS的架构风格，作为为特定应用程序领域开发框架构造的基础；第四部分通过一个简单的示例说明了CPS体系机构样是中的组件和连接器的应用；第五部分介绍了如何在注释和插件的帮助下进行行为分析；第六部分说明了第四部分中介绍的示例的行为验证；结论部分总结了本文的贡献，并讨论了此研究的后续步骤。

文章的表达采用文字+图形化图像相结合的方式，能够更加形象的将文章中的理论模型与应用模型展示出来，使得文章整体具有良好的可读性。