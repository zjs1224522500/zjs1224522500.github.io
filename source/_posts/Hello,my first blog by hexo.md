---
title: 'Hello,my first blog by hexo'
date: 2017-04-30 00:55:13
tags: JavaSE
---
# Java 基本常识

### Java体系架构
> JavaSE : Java Standard Edition  
> JavaEE : Java Enterprise Edition  
> JavaME : Java Micro Edition  

### Java语言的特点
* 面向对象  
* 跨平台（提供了在不同平台下运行的解释环境）
* 健壮性（吸收了C/C++的特点）
* 安全性较高（自动回收垃圾，强制类型检查，取消指针）

### Java跨平台的原理
* Java源代码 .java 编译为 Java字节码 .class（跨平台）
* Java字节码 .class 执行在 Java虚拟机 JVM
* 虚拟机JVM 基于不同操作系统（MAC、Linux、Windows)
* .class 文件由不同操作系统的虚拟机解释给操作系统。

### JVM Java虚拟机
![image](http://img.my.csdn.net/uploads/201212/21/1356070838_2499.png)
* .class 对应的字节码在虚拟机上解释器再次进行“解释”和即使编译器上进行即时编译，再到达运行期系统，再到操作系统，再到硬件
* JVM 可以理解成一个可运行Java字节码的虚拟计算系统，包含一个解释器组件，可以实现Java字节码和计算机操作系统之间的通信，对于不同的运行平台，有不同的JVM
* JVM屏蔽了底层运行平台的差别，实现了“一次编译，随处运行”。

### 垃圾回收器
* 不再使用的内存空间应当进行回收 - 垃圾回收
* JVM 提供了一种系统线程跟踪存储空间的分配情况。
* 并在JVM的空闲时，检查并释放那些可以被释放的存储空间，自动运行。

### JavaSE 的组成概念图
![image](http://img.blog.csdn.net/20151030225424963?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<!-- more -->
### 数组内存结构分析
* 栈内存（先进后出，栈内存大小是固定的）
> 存储确定大小的数据类型：临时变量，基本数据类型，引用类型。由于固定大小，所以存取速度较快。
* 堆内存 
> 存储不确定大小的值（譬如值可能发生改变），需要临时地动态地进行分配，速度相对于栈较慢。  

``` 
    譬如:  
    String[] name = {"AA","bb","CC"};
    name 这个数组名会在栈内存中开辟空间，name 对应的是数组的地址
    而 name 数组所对应的值则会在堆内存中开辟空间
```

### 多维数组
