---
title: DeepLearning Points
author: Travis <Hongxu Wei>
date: 2023-01-01 22:20:41 +0800
categories: [DeepLearning Space]
tags: [DeepLearning]
math: false
---

## 1.图谱

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206271143379.png" style="zoom:40%"/>

### 【图像分类】

#### （1）LeNet

最简单的神经网络，在20世纪90年代提出，输入图像为单通道。

Lenet是一个 7 层的神经网络，包含 3 个卷积层，2 个池化层，1 个全连接层。其中所有卷积层的所有卷积核都为 5x5，步长 strid=1，池化方法都为全局 pooling，激活函数为 Sigmoid，网络结构如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206271123363.png"/>

[csdn lenet详解](https://blog.csdn.net/qq_42570457/article/details/81460807)

[简书 lenet详解](https://www.jianshu.com/p/cd73bc979ba9)

#### （2）AlexNet（分组卷积）

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206271137148.png"/>

- AlexNet为8层结构，其中前5层为卷积层，后面3层为全连接层；学习参数有6千万个，神经元有650,000个

Alexnet——2012年标志性网络

**该网络的亮点在于：**

（0）**分组卷积**[超链接source](#分组卷积)

（1）首次利用GPU进行网络加速训练。
（2）使用了ReLU激活函数，而不是传统的Sigmoid激活函数以及Tanh激活函数。（smoid求导比较麻烦而且当网路比较深的时候会出现梯度消失）
（3）使用了LRN(Local Response Normalization)局部响应归一化（后来发现具有一定的局限性，目前基本已经不再使用）。公式如下图所示：

​		<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206291148243.png" style="zoom:30%"/>

​		[LRN局部响应归一化详解](https://blog.csdn.net/qq_27825451/article/details/88745034)

​		在神经网络中，我们用激活函数将神经元的输出做一个非线性映射，但是tanh和sigmoid这些传统的激活函数的值域都是有范围的，但是ReLU激活函数得到的值域没有一个区间，所以要对ReLU得到的结果进行归一化。也就是Local Response Normalization。

（4）在全连接层的前两层中使用了Dropout随机失活神经元操作，以减少过拟合。dropout解释：使用dropout后，在每一层中随机失活一些神经元——减少训练参数从而减少over fitting.

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206271153973.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206271153344.png"/>

[csdn AlexNet详解](https://blog.csdn.net/luoluonuoyasuolong/article/details/81750190)

------

> 代码地址：https://github.com/WZMIAOMIAO/deep-learning-for-image-processing/tree/master/pytorch_classification/Test2_alexnet

------



#### （3）VGG-Series（小卷积核）

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206291151815.png"/>

**特点**：

- 结构简洁。VGG由5层卷积层、3层全连接层、softmax输出层构成，层与层之间使用max-pooling分开，所有隐层的激活单元都采用ReLU函数。
- 小卷积核和多卷积子层。VGG使用多个较小卷积核（3x3）的卷积层代替一个卷积核较大的卷积层，一方面可以减少参数，另一方面相当于进行了更多的非线性映射，可以增加网络的拟合/表达能力。VGG通过降低卷积核的大小（3x3），增加卷积子层数来达到同样的性能。
- 小池化核。相比AlexNet的3x3的池化核，VGG全部采用2x2的池化核。
- 通道数多。VGG网络第一层的通道数为64，后面每层都进行了翻倍，最多到512个通道，通道数的增加，使得更多的信息可以被提取出来。
- 层数更深、特征图更宽。使用连续的小卷积核代替大的卷积核，网络的深度更深，并且对边缘进行填充，卷积的过程并不会降低图像尺寸。
- **全连接转卷积**（测试阶段）。在网络测试阶段将训练阶段的三个全连接替换为三个卷积，使得测试得到的全卷积网络因为没有全连接的限制，因而可以接收任意宽或高为的输入。

**网络特点：**
<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206291154441.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206291157198.png"/>

------

> 代码地址：https://github.com/WZMIAOMIAO/deep-learning-for-image-processing/tree/master/pytorch_classification/Test3_vggnet

------



#### （4）GoogLeNet（引入Inception）

**亮点：引入了inception**

GoogLeNet是google推出的基于Inception模块的深度神经网络模型，在2014年的ImageNet竞赛中夺得了冠军，在随后的两年中一直在改进，形成了Inception V2、Inception V3、Inception V4等版本。我们会用一系列文章，分别对这些模型做介绍。本篇文章先介绍最早版本的GoogLeNet。

GoogLeNet是2014年Christian Szegedy提出的一种全新的深度学习结构，在这之前的AlexNet、VGG等结构都是通过增大网络的深度（层数）来获得更好的训练效果，但层数的增加会带来很多负作用，比如overfit、梯度消失、梯度爆炸等。inception的提出则从另一种角度来提升训练结果：能更高效的利用计算资源，在相同的计算量下能提取到更多的特征，从而提升训练结果。

**为什么要提出inception**：

一般来说，提升网络性能最直接的办法就是增加网络深度和宽度，但一味地增加，会带来诸多问题：
1）参数太多，如果训练数据集有限，很容易产生过拟合；
2）网络越大、参数越多，计算复杂度越大，难以应用；
3）网络越深，容易出现梯度弥散问题（梯度越往后穿越容易消失），难以优化模型。
我们希望在增加网络深度和宽度的同时减少参数，为了减少参数，自然就想到将全连接变成稀疏连接。但是在实现上，全连接变成稀疏连接后实际计算量并不会有质的提升，因为大部分硬件是针对密集矩阵计算优化的，稀疏矩阵虽然数据量少，但是计算所消耗的时间却很难减少。在这种需求和形势下，Google研究人员提出了Inception的方法。

**什么是inception：**

Inception就是把多个卷积或池化操作，放在一起组装成一个网络模块，设计神经网络时以模块为单位去组装整个网络结构。模块如下图所示：

![img](https://pic3.zhimg.com/80/v2-effa75269cba3c8038c49185e9e8368a_1440w.jpg)

在未使用这种方式的网络里，我们一层往往只使用一种操作，比如卷积或者池化，而且卷积操作的卷积核尺寸也是固定大小的。但是，在实际情况下，在不同尺度的图片里，需要不同大小的卷积核，这样才能使性能最好，或者或，对于同一张图片，不同尺寸的卷积核的表现效果是不一样的，因为他们的感受野不同。所以，我们希望让网络自己去选择，Inception便能够满足这样的需求，一个Inception模块中并列提供多种卷积核的操作，网络在训练的过程中通过调节参数自己去选择使用，同时，由于网络中都需要池化操作，所以此处也把池化层并列加入网络中。

**实际中需要什么样的inception：**

我们在上面提供了一种Inception的结构，但是这个结构存在很多问题，是不能够直接使用的。首要问题就是参数太多，导致特征图厚度太大。为了解决这个问题，作者在其中加入了1X1的卷积核，改进后的Inception结构如下图：

![img](https://pic1.zhimg.com/80/v2-39e361ad7cdbbb521f6be1bdb57452e0_1440w.jpg)

这样做有两个好处，首先是大大减少了参数量，其次，增加的1X1卷积后面也会跟着有非线性激励，这样同时也能够提升网络的表达能力。

**整体网络节后设计：**

GoogLeNet的整体网络结构如下图所示：

![img](https://pic3.zhimg.com/80/v2-766c3f59d3791da39ad805606d6445f6_1440w.jpg)

对上图说明如下：

1）GoogLeNet采用了模块化的结构（Inception结构），方便增添和修改；

2）网络最后采用了average pooling（平均池化）来代替全连接层，该想法来自NIN（Network in Network），事实证明这样可以将准确率提高0.6%。

3）虽然移除了全连接，但是网络中依然使用了Dropout ; 

4）为了避免梯度消失，网络额外增加了2个辅助的softmax用于向前传导梯度（辅助分类器）

对于前三点都很好理解，下面我们重点看一下第4点。这里的辅助分类器只是在训练时使用，在正常预测时会被去掉。辅助分类器促进了更稳定的学习和更好的收敛，往往在接近训练结束时，辅助分支网络开始超越没有任何分支的网络的准确性，达到了更高的水平。

------

> 代码地址：https://github.com/WZMIAOMIAO/deep-learning-for-image-processing/tree/master/pytorch_classification/Test4_googlenet

------



#### （5）ResNet（引入残差）

**1. 背景知识：**

- 为什么要构建深层网络？
  答：认为神经网络的每一层分别对应于提取不同层次的特征信息，有低层，中层和高层，而网络越深的时候，提取到的不同层次的信息会越多，而不同层次间的层次信息的组合也会越多。
- ResNets为什么能构建如此深的网络？
  答：深度学习对于网络深度遇到的主要问题是梯度消失和梯度爆炸，传统对应的解决方案则是数据的初始化(normlized initializatiton)和（batch normlization）正则化，但是这样虽然解决了梯度的问题，深度加深了，却带来了另外的问题，就是网络性能的退化问题，深度加深了，错误率却上升了，而残差用来设计解决退化问题，其同时也解决了梯度问题，更使得网络的性能也提升了。

**2. 什么是ResNet：**

ResNet 网络是在 2015年 由微软实验室中的何凯明等几位大神提出，斩获当年ImageNet竞赛中分类任务第一名，目标检测第一名。获得COCO数据集中目标检测第一名，图像分割第一名。

**3. ResNet网络亮点：**

1.超深的网络结构（超过1000层）。
2.提出residual（残差结构）模块。
3.使用Batch [Normalization](https://so.csdn.net/so/search?q=Normalization&spm=1001.2101.3001.7020) 加速训练（丢弃dropout）。

**4. 网络的深度为什么重要?**

我们知道，在CNN网络中，我们输入的是图片的矩阵，也是最基本的特征，整个CNN网络就是一个信息提取的过程，从底层的特征逐渐抽取到高度抽象的特征，网络的层数越多也就意味这能够提取到的不同级别的抽象特征更加丰富，并且越深的网络提取的特征越抽象，就越具有语义信息。

**5. 为什么不能简单的增加网络层数?**

对于传统的CNN网络，简单的增加网络的深度，容易导致梯度消失和爆炸。针对梯度消失和爆炸的解决方法一般是正则初始化(normalized initialization**)**和中间的正则化层(intermediate normalization layers**)，**但是这会导致另一个问题，退化问题，随着网络层数的增加，在训练集上的准确率却饱和甚至下降了。这个和过拟合不一样，因为过拟合在训练集上的表现会更加出色。

在我参考的博客中，作者针对“退化问题”做了实验并得出如下结论：

按照常理更深层的网络结构的解空间是包括浅层的网络结构的解空间的，也就是说深层的网络结构能够得到更优的解，性能会比浅层网络更佳。但是实际上并非如此，深层网络无论从训练误差或是测试误差来看，都有可能比浅层误差更差，这也证明了并非是由于过拟合的原因。导致这个原因可能是因为随机梯度下降的策略，往往解到的并不是全局最优解，而是局部最优解，由于深层网络的结构更加复杂，所以梯度下降算法得到局部最优解的可能性就会更大。

**6. ResNet中残差网络详解**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302328220.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302330330.png"/>

------

------

------

------

**7. ResNet残差网络**

**a. 背景和思考**

神经网络的深度（depth）和宽度（width）是表征网络复杂度的两个核心因素，不过深度相比宽度在增加网络的复杂性方面更加有效（网络越深，可表达的特征越丰富），这也是为什么VGG想要设法增加网络深度的一个原因。

但是神经网络层数叠加地越深，训练会变得更难，效果不一定会更好。当层数加到某种程度，会发生梯度消失和梯度爆炸（神经网络以反向传播为参数更新的基础的结果）。

由于人为的参数设置，梯度更倾向于消失而不是爆炸。

但是归一化BN层已经是解决梯度问题的方法了，resnet是要干什么呢？

回到神经网络上，按理说，当我们堆叠一个模型时，理所当然的会认为效果会越堆越好。因为，假设一个比较浅的网络已经可以达到不错的效果，那么即使之后堆上去的网络什么也不做，模型的效果也不会变差。

比如，现在把我送到清华，清华已经很好了，我去了什么都不做，清华也不会变差。

然而事实上，这却是问题所在。“什么都不做” 恰好是当前神经网络最难做到的东西之一。

MobileNet V2的论文也提到过类似的现象，由于非线性激活函数Relu的存在，每次输入到输出的过程都几乎是不可逆的（信息损失）。我们很难从输出反推回完整的输入。

也就是说，神经网络层不会闲着，它一定要搞点事情，经过它搞的事情，一切都会变化，很难保留原样。

所以网络层结构不会随着层数的增加而尽可能地维持效果，而我们本希望它什么都不做的本质叫做“恒等映射”（identity mapping）

因此，可以认为Residual Learning的初衷，其实是让模型的内部结构至少有恒等映射的能力。以保证在堆叠网络的过程中，网络至少不会因为继续堆叠而产生退化！

 **b. 原理和结构**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302341889.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302342405.png" style="zoom:40%" align="left"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302343702.png" style="zoom:22%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302347178.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302347226.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302348744.png"/>

5. 网络结构和维度

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302351048.png" style="zoom:60%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302354134.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206302359810.png"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207010000251.png" style="zoom:30%"/>

**c. 残差学习究竟在学习什么？**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207010002208.png"/>

**d. ResNet到底在解决什么问题？**

【参考链接】

[ResNet详解链接CSDN](https://blog.csdn.net/limonor/article/details/106252935)

**e. ResNet总结**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207010003043.png"/>

#### （6）ResNeXt（残差+Inception）

- **网络结构：**

​		ResNeXt是[ResNet](https://zhuanlan.zhihu.com/p/42706477)[2]和[Inception](https://zhuanlan.zhihu.com/p/42704781)[3]的结合体，不同于[Inception v4](https://zhuanlan.zhihu.com/p/42706477)[4]的是，ResNext不需要人工设计复杂的Inception结构细节，而是每一个分支都采用相同的拓扑结构。ResNeXt的本质是分组卷积（Group Convolution，通过变量**基数（Cardinality）**来控制组的数量。分组卷积是普通卷积和深度可分离卷积的一个折中方案，即每个分支产生的Feature Map的通道数为 ![[公式]](https://www.zhihu.com/equation?tex=n+%28n%3E1%29) 。

​    	ResNext是ResNet的增强版。该网络的提出主要吸收了VGG/ResNet和Inception家族网络的优点。VGG/ResNet的优点是网络结构是通多堆叠相同拓扑结构的模块而成，这样的话可以减少超参数的自由选择，网络深度称为最根本的超参数。而Inception家族的网络的Inception模块都是精心设计的，但是都遵循一个特性，那就是split-transform-merge。Inception网络的这种设计可以用最低的计算复杂性达到大的和深的网络的表征能力。但是Inception有一个缺点就是超参数太多了，因此不知如何调整这个网络去适应新的数据集。在这篇文章中，ResNeXt吸收了VGG/ResNet的堆叠重复网络层的优点以及Inception家族的split-transform-merge策略。ResNeXt模块包含一系列转换，每个转换都基于一部分特征图，最后这些转换的输出通过相加进行融合。我们追求一种简单的实现，那就是所有的转换都是相同的拓扑结构，因此可扩展性非常强，而不同特殊设计。出了网络层宽度和深度外，ResNeXt单元中引入了一种新的维度称为“cardinality”，它代一个ResNeXt单元内相同的转换的个数，即分组的大小。实验表明，其它条件不变的情况下，增大cardinality的大小可以增大分类的准确率。

![img](https://img-blog.csdnimg.cn/20181210172715678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01PVV9JVA==,size_16,color_FFFFFF,t_70)

​		如图所示，上面是论文提出的ResNeXt模块的三种的等价形式，第一种是输入的256通道特征图，通过32个1*1的卷积，生成32个通道数为4的特征图，然后每个组进行3*3的卷积，然后通过1*1的卷积升维到256通道，最后32个组的结果相加融合。第二种和第一种类似，只不过32组先进行concat，然后再通过1*1的卷积升维。第三种是先通过1*1的卷积降维为128通道的特征图，然后通过分组卷积后concat，最后用1*1的卷积进行升维。这里的分组卷积主要是来自于AlexNet。ResNeXt-50的整体网络结构如下（只是将ResNet单元换成了ResNeXt单元），而ResNeXt-101以及ResNeXt-152的结构也与此类推：

<div>
  <center>
  <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207031605820.png" style="zoom:40%"/>
  </center>
</div>

​		ResNeXt单元的设计遵循：1）如果生成相同尺寸的特征图，那么这些分组共享相同的超参数（卷积核尺寸和通道数）；2）当特征图的尺寸下采样2倍时，特征图的通道数需要增加为原来的两倍，比如对于第一阶段的残差块的通道数为256时，分为32组，每组通道为数为4；而对于第二阶段的残差块通道数为512时，分为32组，每组的通道就为8。依此类推，通道数逐渐翻倍。

- **实验结果：**

（1）下面两幅图为ablation实验，左图的实验表明，随着“cardinality”的增加，错误率在逐渐下降。右图的实验表明，通过增加ResNet的模型参数（即所有特征图的通道数加倍或者网络层数加深）带来的提升很微小，而通过增加“cardinality”，即增加ResNeXt单元的分组数带来的提升更大。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207031617612.png" style="zoom:50%"/>

(2）与其它state-of-the-art模型比较（**单模型single-crop测试**）

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207031618199.png" style="zoom:50%"/>

此外，对于**单模型多尺度或者多裁剪**，论文实现了top5/top1的错误率为：**17.7%/3.7%**。而对于**多模型融合**，论文的top5错误率为**3.03%**。

#### （7）MobileNet-Series（引入DW卷积）

mobilenet系列：mobilenetV1、mobilenetV2、mobilenetV3三个特征提取网络。MobileNet模型是Google针对手机等嵌入式设备提出的一种轻量级的深层神经网络，其使用的**核心思想**便是**depthwise separable convolution（深度可分离卷积）**

##### ---MobileNet v1

**（a）什么是深度可分离卷积(Mobilenetv1提出)？**

假设某一网络卷积层，其卷积核大小为3×3，输入通道为16，输出通道为32；

常规卷积操作是将32个3×3×16的卷积核作用于16通道的输入图像，则根据卷积层参数量计算公式，

得到所需参数为32*(3*3*16+1)= 4640个。

若先用16个、大小为3×3的卷积核(3*3*1)作用于16个通道的输入图像，得到了16个特征图，在做融合操作之前，接着用32个大小为1×1的卷积核(1*1*16)遍历上述得到的16个特征图，根据卷积层参数计算公式，所需参数为(3*3*1*16+16) + (1*1*16*32+32) = 706个。

上述即为深度可分离卷积的作用，通俗的讲，普通卷积层的特征提取与特征组合一次完成并输出，而深度可分离卷积先用厚度为1的3*3的卷积核（**depthwise分层卷积**），再用1*1的卷积核(**pointwise 卷积**)调整通道数，将特征提取与特征组合分开进行。

由此可以看出，深度可分离卷积可大大减少模型的参数，其具体结构如下图：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207041046329.png" style="zoom:30%"/>

- 在进行 **deepthwise(DW)** 卷积时只使用了一种维度为`in_channels`的卷积核进行**特征提取**（没有进行特征组合）；
- 在进行 **pointwise(PW)** 卷积时只使用了`output_channels `种维度为`in_channels` 1*1 的卷积核进行**特征组合。**

##### ---MobileNet v2

以上介绍了MobileNet V1，那么MobileNet V1是否有缺点呢？答案是肯定的。

- 结构问题：

V1结构过于简单，没有复用图像特征，即没有concat/eltwise+ 等操作进行特征融合，而后续的一系列的ResNet, DenseNet等结构已经证明复用图像特征的有效性。

- 逐深度卷积问题：

1. 在处理低维数据（比如逐深度的卷积）时，relu函数会造成信息的丢失。
2. DW 卷积由于本身的计算特性决定它自己没有改变通道数的能力，上一层给它多少通道，它就只能输出多少通道。所以如果上一层给的通道数本身很少的话，DW 也只能很委屈的在低维空间提特征，因此效果不够好。

原论文作者用了大量篇幅说明以上结论1，在此我们简单理解下（因为笔者只能”简单理解“，不得已为之）。

![img](https://pic1.zhimg.com/80/v2-9439f25683ede6e47c07d747a4fcf648_1440w.jpg)

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207041059286.png"/>

![img](https://pic3.zhimg.com/80/v2-7cdf75ec38bd4ed388a4f3bfd04a1df2_1440w.jpg)

V2使用了跟V1类似的深度可分离结构，不同之处也正对应着V1中逐深度卷积的缺点改进：

- **V2 去掉了第二个 PW 的激活函数改为线性激活**。
  论文作者称其为 Linear Bottleneck。原因如上所述是因为作者认为激活函数在高维空间能够有效的增加非线性，而在低维空间时则会破坏特征，不如线性的效果好。
- **V2 在 DW 卷积之前新加了一个 PW 卷积**。
  给每个 DW 之前都配备了一个 PW，专门用来升维，定义升维系数 ![[公式]](https://www.zhihu.com/equation?tex=t%3D6) ，这样不管输入通道数 ![[公式]](https://www.zhihu.com/equation?tex=C_%7Bin%7D) 是多是少，经过第一个 PW 升维之后，DW 都是在相对的更高维 ( ![[公式]](https://www.zhihu.com/equation?tex=t.C_%7Bin%7D) ) 进行更好的特征提取。



**倒残差**

![img](https://pic2.zhimg.com/80/v2-36ef7bdf7133cae681de3fc04a59d535_1440w.jpg)

MobileNet V2 借鉴 ResNet，都采用了 ![[公式]](https://www.zhihu.com/equation?tex=1%C3%971%E2%80%94%3E3%C3%973%E2%80%94%3E1%C3%971) 的模式。同样使用 Shortcut 将输出与输入相加（未在上式画出） .

但是ResNet **先降维** (0.25倍)、卷积、再升维，而 MobileNet V2 则是 **先升维** (6倍)、卷积、再降维。直观的形象上来看，ResNet 的微结构是**沙漏形**，而 MobileNet V2 则是**纺锤形**，刚好相反。因此论文作者将 MobileNet V2 的结构称为 Inverted Residual Block。这么做也是因为使用DW卷积而作的适配，希望特征提取能够在高维进行。

MobileNet V2的深度可分离模块与MobileNet V1对比：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207041102126.png"/>

![img](https://pic2.zhimg.com/80/v2-c6940447e39d1e2d2690e368234d05d9_1440w.jpg)

#### （8）ShuffleNet-Series

#### （9）EfficientNet-Series

#### （10）Vision Transformer（vit）

#### （11）Swin-Transformer

#### （12）ConvNeXt

### 【语义分割】

#### （1）FCN

#### （2）UNet

#### （3）SegNet

#### （4）PSPNet

#### （5）DeepLab-Series

#### （6）LR-ASPP

### 【目标检测】

#### （1）RCNN

#### （2）Faster-RCNN

#### （3）SSD算法

#### （4）RetinaNet

#### （5）YOLO-Series

### 【NLP】

## 2.常见的迁移学习方式

1. 载入权重后训练所有参数
2. 载入权重后只训练最后几层参数
3. 载入权重后在原网络基础上再添加一层全连接层，仅训练最后一层全连接层。

## 3.分类｜回归

- svm
- 决策树
- 随机森林
- 朴素贝叶斯分类



## 4.DL优化器分类

- 梯度下降优化器
  - SGD
  - BGD
- 动量优化器
  - Momentum
  - NAG
- 自适应优化器
  - AdaGrad
  - RMSProp
  - AdaDelta
  - Adam

https://zhuanlan.zhihu.com/p/393431989



## 5.DL损失函数

- MSELoss均方误差
- 交叉熵损失函数CrossEntropyLoss

https://pytorch.org/docs/stable/nn.html#loss-functions



- 二分类问题：二元交叉熵（binary crossentropy）
- 多分类问题：分类交叉熵（categorical crossentropy）
- 序列学习问题：CTC

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202205281630435.png"/>



## 6.张量中的数据类型：float32、float64、uint8区别

- ndim属性（纬度）
- dtype（数据类型属性）



## 7.sigmoid与softmax

- sigmoid常用于多标签分类问题，即最后的结果不排斥，可能具有多个类别标签
- softmax用于多类别分类，单标签问题，多个类别互斥



## 8.one-hot与标签整数编码

- one-hot：对应分类交叉熵
- 标签整数编码：对应sparse分类交叉熵



## 9.强化学习

待学习



## 10.神经网络的数据预处理

1. 向量化
2. 值标准化
3. 处理缺失值



## 11.正则化（降低过拟合）

- 获取更多的训练数据

- 减小网络大小
- 添加权重正则化：强制让模型权重只能取较小的值，从而限制模型的复杂度。
  - L1正则化
  - L2正则化
- 添加dropout正则化



## 12.小型数据集训练CNN的技巧

- 数据增强
- 用预训练的网络做特征提取
  - 卷积基概念
- 对预训练的网络进行微调



## 13.CNN可视化

- 可视化CNN的中间输出（中间激活）
- 可视化CNN的过滤器
- 可视化图像中类激活的热力图
  - 类激活图技术（CAM）
  - Grad-CAM算法

## 14.残差结构

可以说是建立在前身Highway Network上的，是对高速公路网络的一种简化。

深度学习对于网络深度遇到的主要问题是梯度消失和梯度爆炸，传统对应的解决方案则是数据的初始化(normlized initializatiton)和（batch normlization）正则化，但是这样虽然解决了梯度的问题，深度加深了，却带来了另外的问题，就是网络性能的退化问题，深度加深了，错误率却上升了，而残差用来设计解决退化问题，其同时也解决了梯度问题，更使得网络的性能也提升了。

普通网络（Plain network），类似VGG，没有残差，凭经验会发现随着网络深度的加深，训练错误会先减少，然后增多（并证明的错误的增加并不是由于过拟合产生，而是由于网络变深导致难以训练）。从理论上分析，网络深度越深越好。但实际上，如果没有残差网络，对于一个普通网络来说，深度越深意味着用优化算法越难训练。实际上，随着网络深度的增加，训练误差会越来越多，这被描述为网络退化。

参考：https://blog.csdn.net/limonor/article/details/106252935

参考：https://blog.csdn.net/weixin_44695969/article/details/102997574

## 15.超参数自动优化

待看

## 16.模型集成

待看

## 17.深度可分离卷积、扩张卷积（空洞卷积）、反卷积、分组卷积

1. 深度可分离卷积（Depthwise separable convolution）

主要分为两个过程

- 逐通道卷积（Depthwise Convolution）：缩减输入的Width与Height
- 逐点卷积（Pointwise Convolution）：扩展通道数

将得到相同大小的FeatureMap分为两个过程之后，可节省约三分之二的参数量。

计算量相同的情况下，Depthwise Separable Convolution可以将神经网络层数可以做的更深。

参考：https://blog.csdn.net/zml194849/article/details/117021815

2. 扩张卷积（空洞卷积）

扩张卷积引入另一个卷积层的参数被称为扩张率。这定义了内核中值之间的间距。扩张速率为2的3x3内核将具有与5x5内核相同的视野，而只使用9个参数。 想象一下，使用5x5内核并删除每个间隔的行和列。（如下图所示）

系统能以相同的计算成本，提供更大的感受野。扩张卷积在实时分割领域特别受欢迎。 在需要更大的观察范围，且无法承受多个卷积或更大的内核，可以才用它。
参考：https://blog.csdn.net/gwplovekimi/article/details/89890510

3、 分组卷积

<a name="分组卷积">超链接target</a>

（a）起源

分组卷积(Group Convolution) 起源于2012年的 AlexNet - 《ImageNet Classification with Deep Convolutional Neural Networks》。由于当时硬件资源的限制，因为作者将Feature Maps分给多个GPU进行处理，最后把多个GPU的结果进行融合。如下图：
![img](https://img-blog.csdnimg.cn/fc0e35a4b2984476988b44eb769a87ad.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAQWFyb25fbmVpbA==,size_20,color_FFFFFF,t_70,g_se,x_16)

（b）分组卷积介绍

我接下来用图来直观的展示普通2D卷积 和 分组卷积的区别：

**标准的 2D 卷积**步骤如下图所示，输入特征为 (H × W × C) ，然后应用 C' 个filters（每个filter的大小为 （h × w × c），输入层被转换为大小为 （H' × W' × C'） 的输出特征。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207031549409.png" style="zoom:40%"/>

分组卷积 的表示如下图(下图表示的是被拆分为 2 个filters组的分组卷积) ：

- 首先每个filters组，包含 C'/2个 数量的filter, 每个filter 的通道数为传统2D-卷积filter的一半。
- 每个filters组作用于原来 W × H × C 对应通道数的一半，也就是 W × H × C/2
- 最终每个filters组对应输出输出 C' / 2 个通道的特征。
- 最后将通道堆叠得到了最终的 C'个通道，实现了和上述标准2D 卷积一样的效果。
  <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202207031550292.png" style="zoom:40%"/>

（c）分组卷积的优势

根据上面的表述，既然能实现和传统卷积一样的效果，那这样做的目的是什么呢？重点来了！

- 我们先计算一下标准2D卷积 和 分组卷积的 参数量：

  标准2D卷积：w × h × C × C'

  分组卷积：w × h × C/2 × C'/2 × 2 

  好！看出来差别了吧！参数量减少到原来的1/2！当Group为4的时候，参数量减少到原来的1/4，这个我觉得是最主要的优势。

- 但是虽然得到了一样size的feature，参数量也降低了。那对于模型来说分组卷积的效果好不好呢？这篇文章给了一个非常满意的答复 https://blog.yani.ai/filter-group-tutorial/ 。

总结来说：在某些情况下，分组卷积能带来的模型效果确实要优于标准的2D 卷积，是因为组卷积的方式能够增加相邻层filter之间的对角相关性，而且能够减少训练参数，不容易过拟合，这类似于正则的效果。


## 18.1*1卷积作用

- 降维和升维：常见于残差结构中。

- 放缩channel数：通过控制卷积核的数量达到通道数大小的放缩。池化层只能改变高度和宽度，无法改变通道数。
- 增加非线性：1×1卷积核的卷积过程相当于全连接层的计算过程，并且还加入了非线性激活函数，从而可以增加网络的非线性，使得网络可以表达更加复杂的特征。
- 减少参数：减少计算量，减少参数。

## 19.Inception结构（GoogleNet引入）

inception模块的基本机构如图1，整个inception结构就是由多个这样的inception模块串联起来的。inception结构的主要贡献有两个：一是使用1x1的卷积来进行升降维；二是在多个尺寸上同时进行卷积再聚合。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206032056031.png"/>

- 多个尺寸上进行卷积再聚合

图2可以看到对输入做了4个分支，分别用不同尺寸的filter进行卷积或池化，最后再在特征维度上拼接到一起。这种全新的结构有什么好处呢？Szegedy从多个角度进行了解释：

解释1：在直观感觉上在多个尺度上同时进行卷积，能提取到不同尺度的特征。特征更为丰富也意味着最后分类判断时更加准确。

解释2：利用稀疏矩阵分解成密集矩阵计算的原理来加快收敛速度。举个例子图4左侧是个稀疏矩阵（很多元素都为0，不均匀分布在矩阵中），和一个2x2的矩阵进行卷积，需要对稀疏矩阵中的每一个元素进行计算；如果像图4右图那样把稀疏矩阵分解成2个子密集矩阵，再和2x2矩阵进行卷积，稀疏矩阵中0较多的区域就可以不用计算，计算量就大大降低。这个原理应用到inception上就是要在特征维度上进行分解！传统的卷积层的输入数据只和一种尺度（比如3x3）的卷积核进行卷积，输出固定维度（比如256个特征）的数据，所有256个输出特征基本上是均匀分布在3x3尺度范围上，这可以理解成输出了一个稀疏分布的特征集；而inception模块在多个尺度上提取特征（比如1x1，3x3，5x5），输出的256个特征就不再是均匀分布，而是相关性强的特征聚集在一起（比如1x1的的96个特征聚集在一起，3x3的96个特征聚集在一起，5x5的64个特征聚集在一起），这可以理解成多个密集分布的子特征集。这样的特征集中因为相关性较强的特征聚集在了一起，不相关的非关键特征就被弱化，同样是输出256个特征，inception方法输出的特征“冗余”的信息较少。用这样的“纯”的特征集层层传递最后作为反向计算的输入，自然收敛的速度更快。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202206032058771.png"/>

解释3：Hebbin赫布原理。Hebbin原理是神经科学上的一个理论，解释了在学习的过程中脑中的神经元所发生的变化，用一句话概括就是*fire togethter, wire together*。赫布认为“两个神经元或者神经元系统，如果总是同时兴奋，就会形成一种‘组合’，其中一个神经元的兴奋会促进另一个的兴奋”。比如狗看到肉会流口水，反复刺激后，脑中识别肉的神经元会和掌管唾液分泌的神经元会相互促进，“缠绕”在一起，以后再看到肉就会更快流出口水。用在inception结构中就是要把相关性强的特征汇聚到一起。这有点类似上面的解释2，把1x1，3x3，5x5的特征分开。因为训练收敛的最终目的就是要提取出独立的特征，所以预先把相关性强的特征汇聚，就能起到加速收敛的作用。

在inception模块中有一个分支使用了max pooling，作者认为pooling也能起到提取特征的作用，所以也加入模块中。注意这个pooling的stride=1，pooling后没有减少数据的尺寸。

## 20.梯度退化、梯度消失、梯度爆炸

- **网络退化：**在增加网络层数的过程中，training accuracy 逐渐趋于饱和，继续增加层数，training accuracy 就会出现下降的现象，而这种下降不是由过拟合造成的。实际上较深模型后面添加的不是恒等映射，而是一些非线性层。因此，退化问题也表明了：通过多个非线性层来近似恒等映射可能是困难的。

  恒等映射亦称恒等函数：是一种重要的映射，对任何元素，象与原象相同的映射
  解决方案：学习残差。Resnet正是基于此问题提出。

- **梯度消失/梯度爆炸：**二者问题问题都是因为网络太深,网络权值更新不稳定造成的。本质上是因为梯度反向传播中的连乘效应（小于1连续相乘多次）。
  反向传播更新参数w的更新过程，对于参数。梯度消失时，越靠近输入层的参数w越是几乎纹丝不动；梯度爆炸时，越是靠近输入层的参数w越是上蹿下跳。稳定和突变都不是我们想要的，我们想要的是参数w朝着误差减小的方向稳步变化。
  在反向传播过程中需要对激活函数进行求导，如果导数大于1，那么随着网络层数的增加梯度更新将会朝着指数爆炸的方式增加这就是梯度爆炸。同样如果导数小于1，那么随着网络层数的增加梯度更新信息会朝着指数衰减的方式减少这就是梯度消失。因此，梯度消失、爆炸，其根本原因在于反向传播训练法则，属于先天不足。

参考：https://blog.csdn.net/weixin_42620513/article/details/123120771

参考：https://blog.csdn.net/qq_36396104/article/details/89459267



## 21.SSD和Faster-RCNN

待看