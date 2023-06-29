---
title: Java Points
author: Travis <Hongxu Wei>
date: 2023-01-01 22:18:20 +0800
categories: [Java Learning Space]
tags: [java]
math: false
---

## 1.编译型语言和解释型语言

**编译型语言**

使用专门的编译器，针对特定的平台，将高级语言源代码一次性的编译成可被该平台硬件执行的机器码，并包装成该平台所能识别的可执行性程序的格式。

在编译型语言写的程序执行之前，需要一个专门的编译过程，把源代码编译成机器语言的文件，如exe格式的文件，以后要再运行时，直接使用编译结果即可，如直接运行exe文件。因为只需编译一次，以后运行时不需要编译，所以编译型语言执行效率高。

总结：

1. 一次性的编译成平台相关的机器语言文件，运行时脱离开发环境，运行效率高；
2. 与特定平台相关，一般无法移植到其他平台；
3. 现有的C、C++、Objective等都属于编译型语言。

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081608630.png" style="zoom:50%"/>

**解释型语言**

使用专门的解释器对源程序逐行解释成特定平台的机器码并立即执行。

解释型语言不需要事先编译，其直接将源代码解释成机器码并立即执行，所以只要某一平台提供了相应的解释器即可运行该程序。

总结：

1. 解释型语言每次运行都需要将源代码解释称机器码并执行，效率较低；
2. 只要平台提供相应的解释器，就可以运行源代码，所以可以方便源程序移植；
3. Python等属于解释型语言。

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081608060.png" style="zoom:40%"/>

## 2.JAVA内存块 (深入学习JVM)

电子书待看





## 3.张量中的数据类型：float32、float64、uint8区别

- ndim属性（纬度）
- dtype（数据类型属性）

## 4.JDK、JRE、JVM、javac关系

基本介绍： 

**JDK（Java Development Kit）**是针对Java开发员的产品，是整个Java的核心，包括了Java运行环境JRE、Java工具和Java基础类库。
**Java Runtime Environment（JRE）**是运行JAVA程序所必须的环境的[集合](https://so.csdn.net/so/search?q=集合&spm=1001.2101.3001.7020)，包含JVM标准实现及Java核心类库。
**JVM是Java Virtual Machine（Java虚拟机）**的缩写，是整个java实现跨平台的最核心的部分，能够运行以Java语言写作的软件程序。

1、**JDK**
JDK是java开发工具包，在其安装目录下面有六个文件夹、一些描述文件、一个src压缩文件。bin、include、lib、 jre这四个文件夹起作用，demo、sample是一些例子。可以看出来JDK包含JRE，而JRE包含[JVM](https://so.csdn.net/so/search?q=JVM&spm=1001.2101.3001.7020)。
bin:最主要的是编译器(javac.exe)
include:java和JVM交互用的头文件
lib：类库
jre:java运行环境（注意：这里的bin、lib文件夹和jre里的bin、lib是不同的）
总的来说JDK是用于java程序的开发,而jre则是只能运行class而没有编译的功能。 
JDK是提供给Java开发人员使用的，其中包含了java的开发工具，也包括了JRE。所以安装了JDK，就不用在单独安装JRE了。 其中的开发工具包括编译工具(javac.exe)打包工具(jar.exe)等


2、**JRE**
JRE是指java运行环境。光有JVM还不能成class的执行，因为在解释class的时候JVM需要调用解释所需要的类库lib。在JDK的安装目录里你可以找到jre目录，里面有两个文件夹bin和lib,在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib和起来就称为jre。所以，在你写完java程序编译成.class之后，你可以把这个.class文件和jre一起打包发给朋友，这样你的朋友就可以运行你写程序了。 
包括Java[虚拟机](https://so.csdn.net/so/search?q=虚拟机&spm=1001.2101.3001.7020)(JVM Java Virtual Machine)和Java程序所需的核心类库等， 
如果想要运行一个开发好的Java程序，计算机中只需要安装JRE即可。



3、**JVM**
JVM就是我们常说的java虚拟机，它是整个java实现跨平台的最核心的部分，所有的java程序会首先被编译为.class的类文件，这种类文件可以在虚拟机上执行，也就是说class并不直接与机器的操作系统相对应，而是经过虚拟机间接与操作系统交互，由虚拟机将程序解释给本地系统执行。 
可以理解为是一个虚拟出来的计算机，具备着计算机的基本运算方式，它主要负责将java程序生成的字节码文件解释成具体系统平台上的机器指令。让具体平台如window运行这些Java程序。

**简单而言：使用JDK开发完成的java程序，交给JRE去运行。**  

三者之间关系 
JDK 包含JRE,    JRE包含JVM。

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081608748.png" style="zoom:40%"/>

我们开发的实际情况是：我们利用JDK（调用JAVA API）开发了属于我们自己的JAVA程序后，通过JDK中的编译程序（javac）将我们的文本java文件编译成JAVA字节码，在JRE上运行这些JAVA字节码，JVM解析这些字节码，映射到CPU指令集或OS的系统调用。

## 5.abstract关键字

**（1）抽象类**

包含抽象方法的类，就一定是抽象类。
但不表示抽象类必须包含抽象方法。
如果子类继承抽象类，就必须重写所有抽象方法，如果不想全部重写，那就把子类定义成抽象类。

写做：

```java
(public) abstract class Demo{}
```

不能实例化，就是说new不了。它是代表一种类型，而创建对象是具体的。
但有自己的构造方法；（可能因为毕竟是继承吧，继承的话就需要父类属性跟实例方法，所以它需要给子类提供构造方法，但它本身不能实例化。）
可以包含普通方法（非抽象方法），也就是一般带方法体的方法；
不能用final修饰，因为抽象类需要通过继承来实现抽象方法。

**（2）抽象方法**

没有方法实现，不能直接调用；
不能用private修饰，因为子类要实现它；
不能用static修饰，因为static会分配内存，但它没有方法体呀，这样做没有意义。

**（3）this**

this代表当前发起调用的类。

**（4）super**

子类不能调用抽象父类的抽象方法。

**好处**

- 不能被实例化，通过方法的覆盖实现多态，即运行时绑定；
- 事物共性提出来，子类通过继承实现，代码易维护、扩展。

## 6.extends和implements区别

首先需要记住的是extends表示继承关系、implements表示实现关系

那么extends用于哪些情形、implements又用于哪些情形呢？

extends：

子类（class） extends   父类（class）===>继承类只能是单继承，也就是如果父亲属于类（class），那么父亲只能有一个

类（class）/ 接口（interface）    extends     接口1（interface，接口2（interface）===>继承接口可以是多继承，也就是如果父亲是接口（interface）那么可以有多个父亲 

[CSDN讲解](https://cr121.blog.csdn.net/article/details/86664925)

## 7. Java中堆内存和栈内存的区别

Java把内存分成两种，一种叫做栈内存，一种叫做堆内存。

在函数中定义的**一些基本类型的变量和对象的引用变量都是在函数的栈内存中分配**。当在一段代码块中定义一个变量时，java就在栈中为这个变量分配内存空间，当超过变量的作用域后，java会自动释放掉为该变量分配的内存空间，该内存空间可以立刻被另作他用。

**堆内存用于存放由new创建的对象和数组**。在堆中分配的内存，由java虚拟机自动垃圾回收器来管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，在栈中的这个特殊的变量就变成了数组或者对象的引用变量，以后就可以在程序中使用栈内存中的引用变量来访问堆中的数组或者对象，引用变量相当于为数组或者对象起的一个别名，或者代号。

引用变量是普通变量，定义时在栈中分配内存，引用变量在程序运行到作用域外释放。而数组＆对象本身在堆中分配，即使程序运行到使用new产生数组和对象的语句所在地代码块之外，数组和对象本身占用的堆内存也不会被释放，**数组和对象在没有引用变量指向它的时候，才变成垃圾，不能再被使用，但是仍然占着内存，在随后的一个不确定的时间被垃圾回收器释放掉。这个也是java比较占内存的主要原因，实际上，栈中的变量指向堆内存中的变量，这就是 Java 中的指针!**

示例1如下：

```java
Person per = new Person();
//这其实是包含了两个步骤，声明和实例化

Person per = null;//声明一个名为Person类的对象per
per = new Person(); // 实例化这个per对象
```

**声明** 指的是创建类的对象的过程；
**实例化** 指的是用关键词new来开辟内存空间。
它们在内存中的划分是这样的：

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081608237.png" style="zoom:40%"/>



**那什么是栈内存（heap）和栈内存（heap）呢？**

#### **栈内存：**

​       在函数中定义的一些基本类型的变量和对象的引用变量都在函数的栈内存中分配。栈内存主要存放的是基本类型类型的数据 如( int, short, long, byte, float, double, boolean, char) 和对象句柄。注意：并没有String基本类型、在栈内存的数据的大小及生存周期是必须确定的、其优点是寄存速度快、栈数据可以共享、缺点是数据固定、不够灵活。

#### 栈的共享：

```java
 1 String str1 = "myString";
 2  
 3 String str2 = "myString";
 4  
 5 System.out.println(str1 ==str2 );
 6  
 7 //注意：这里使用的是str1 ==str2，而不是str1.equals(str2)的方式。
 8  
 9 //因为根据JDK的说明，==号只有在两个引用都指向了同一个对象时才返回真值
10  
11 //而str1.equals(str2)，只是比较两个字符串是否相等
```

结果为True，这就说明了str1和str2其实指向的是同一个值。

​       上述代码的原理是，首先在栈中创建一个变量为str1的引用，然后查找栈中是否有myString这个值，如果没找到，就将myString存放进来，然后将str1指向myString。接着处理String str2 = "myString";；在创建完str2 的引用变量后，因为在栈中已经有myString这个值，便将str2 直接指向myString。这样，就出现了str1与str2 同时指向myString。

​       特别注意的是，这种字面值的引用与类对象的引用不同。假定两个类对象的引用同时指向一个对象，如果一个对象引用变量修改了这个对象的内部状态，那么另一个对象引用变量也即刻反映出这个变化。相反，通过字面值的引用来修改其值，不会导致另一个指向此字面值的引用的值也跟着改变的情况。如上例，我们定义完str1与str2 的值后，再令str1=yourString；那么，str2不会等于yourString，还是等于myString。在编译器内部，遇到str1=yourString；时，它就会重新搜索栈中是否有yourString的字面值，如果没有，重新开辟地址存放yourString的值；如果已经有了，则直接将str1指向这个地址。因此str1值的改变不会影响到str2的值。

#### 堆内存：

堆内存用来存放所有new 创建的对象和 数组的数据

```java
 1 String str1 = new String ("myString");
 2  
 3 String str2 = "myString";
 4  
 5 System.out.println(str1 ==str2 ); //False
 6  
 7 String str1 = new String ("myString");
 8  
 9 String str2 = new String ("myString");
10  
11 System.out.println(a==b); //False
```

创建了两个引用，创建了两个对象。两个引用分别指向不同的两个对象。以上两段代码说明，只要是用new()来新建对象的，都会在堆中创建，而且其字符串是单独存值的，即使与栈中的数据相同，也不会与栈中的数据共享。

为了深入理解，添加了实例2：

```java
 1 public class Demo1 {
 2     
 3     public static void main(String[] args) {
 4         
 5         String str1 = "hello";
 6         String str2 = "hello";
 7         String str3 = new String("hello");
 8         String str4 = new String("hello");
 9         System.out.println("str1==str2?"+(str1==str2));  // true  
10         System.out.println("str2==str3?"+(str2==str3));  //false
11         System.out.println("str3==str4?"+(str3==str4));  // false
12         System.out.println("str3.equals(str2)?"+(str3.equals(str4))); //true
13         //是String类重写了Object的equals方法，比较的是两个字符串对象 的内容 是否一致。
14         // "=="用于比较 引用数据类型数据的时候比较的是两个对象 的内存地址，equals方法默认情况下比较也是两个对象 的内存地址。   
16         test(null);
17     }
18 }
```

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081608277.png" style="zoom:60%"/>

## 8.C、A、P定义

C（Consistency）、A（Availability）、P（Partition tolerance）

-   C：一致性：对于客户端的每次读操作，要么读到的是最新的数据，要么读取失败。换句话说，一致性是站在分布式系统的角度，对访问本系统的客户端的一种承诺：要么我给您返回一个错误，要么我给你返回绝对一致的最新数据，不难看出，其强调的是数据正确。
-   A：可用性：任何客户端的请求都能得到响应数据，不会出现响应错误。换句话说，可用性是站在分布式系统的角度，对访问本系统的客户的另一种承诺：我一定会给您返回数据，不会给你返回错误，但不保证数据最新，强调的是不出错。
-   P：分区容忍性：由于分布式系统通过网络进行通信，网络是不可靠的。当任意数量的消息丢失或延迟到达时，系统仍会继续提供服务，不会挂掉。换句话说，分区容忍性是站在分布式系统的角度，对访问本系统的客户端的再一种承诺：我会一直运行，不管我的内部出现何种数据同步问题，强调的是不挂掉。

## 9.zookeeper和nacos对比

[zookeeper和nacos对比]:https://www.cnblogs.com/syq816/p/16332417.html

zookeeper是CP系统，强一致性，(集群leader挂了会重新选举，此时暂停对外服务)。Zookeeper是通过TCP的心跳判断服务是否可用。

nacos保证了P，能够根据策略实现AP或者CP，官方推荐使用A，即AP，以保证高可用性。

## 10.tomcat、springMVC

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081614662.png"/>

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081614472.png"/>

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303081614978.png"/>



## 11.缓存穿透、缓存击穿（一个key）、缓存雪崩（大量的key）

## 12.redis使用lua脚本为什么能保证原子性？

脚本的原子性：Redis使用（支持）相同的Lua解释器，来运行所有的命令。Redis还保证脚本以原子方式执行：在执行脚本时，不会执行其他脚本或Redis命令。这个语义类似于MULTI（开启事务）/EXEC（触发事务，一并执行事务中的所有命令）。从所有其他客户端的角度来看，脚本的效果要么仍然不可见，要么已经完成。

## 13.@Retention生命周期

![image-20230316221327387](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303162213467.png)

-   周期长度是SOURCE < CLASS < RUNTIME,所以前者能作用到的地方后者一定能作用到
-   @Override是让编译器检查当前方法是否在覆盖父类方法，@SupressWarnings也只是为了抑制代码警告，在代码编译后就都没有什么作用，因此不需要写入.class文件。
-   @ButterKnife会在编译时生成辅助代码，所以用RetentionPolicy.CLASS修饰。
-   **RetentionPolicy.CLASS是默认生命周期**，没有被@Retention修饰的注解的生命周期都是这种策略。
-   RetentionPolicy.RUNTIME修饰的注解处理器可以通过**反射**获取该注解的属性值，从而做一些运行时的逻辑处理。



## 14.为什么 HashMap 是线程不安全？

