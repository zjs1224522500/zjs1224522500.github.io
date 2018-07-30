---
title: '深入剖析Tomcat读书笔记（一）'
date: 2017-10-19 21:28:52
tags: 服务器
---
# 前言
## 1、Servlet容器是如何工作的？
- 创建一个`Request`对象，用可能会在`Servlet`中使用的信息填充该`Request`对象;
- 创建一个调用`Servlet`的`response`对象，用来向WEB客户端发送响应；
- 调用`Servlet`的`service()`方法,将`Request`对象和`Response`对象。


> ### 复习Servlet的生命周期
> - Servlet 通过调用 `init ()` 方法进行初始化。
> - Servlet 调用 `service()` 方法来处理客户端的请求。
> - Servlet 通过调用 `destroy()` 方法终止（结束）。
> - 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

## 2、Catalina
- `Catalina`是一个成熟的软件，设计和开发的十分优雅，功能结构也是模块化的，主要分为以下两个模块：
    - `连接器` :负责将请求和容器相关联，为每个接收到的HTTP请求创建一个`request`对象和一个`response`，然后将处理过程交给容器。
    - `容器`：从连接器中接受到`request`和`response`对象，并调用相应的`Servlet`的`service()`方法。


# 第一章 简单的Web服务器
- 通过复习`Java套接字编程`，搭建一个简单的`Web服务器`，同时复习了计算机网络的相关知识以及`JavaIO`中的`输入输出流`相关操作。

## WEB服务器的工作原理
- 服务器利用相应的`服务器套接字(ServerSocket)`（又称`连接套接字,欢迎套接字(WelcomeSocket)`）对相应的IP地址和端口进行监听，等待客户端发送相关连接请求;
- `客户端的套接字(ClientSocket)`提出连接请求，要连接的目标是`ServerSocket`。为此，`CllientSocket`必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址`Host`和端口号`port`，然后就向`服务器端套接字`提出连接请求;
- 当`服务器端套接字(ServerSocket)`监听到或者说接收到`客户端套接字(ClientSocket)`的连接请求时，就响应`客户端套接字`的请求，建立一个新的线程，把`服务器端套接字`的描述发 给客户端，一旦客户端确认了此描述，双方就正式建立连接。而`服务器端套接字`继续处于监听状态，继续接收其他`客户端套接字`的连接请求;`服务器端套接字(ServerSocket)`在收到请求时会调用`accept()`方法为客户端创建一个新的套接字`连接套接字（connection socket）`，实质上和客户端建立连接之后进行通信的是`连接套接字`


## 在编写过程中遇到的问题：(待解决)
### 编写代码时抛出异常：
> java.net.SocketException: Software caused connection abort: socket write error.

- 通过debug初步分析的结果是在浏览器生成发送给客户端响应时的输出流的`write()`函数出现了问题,查阅资料显示为**在write()时输出流被提前关闭**

### 测试过程中存在的问题：
> IE浏览器测试成功，谷歌浏览器测试失败.

<!-- more -->
# 第二章 一个简单的Servlet容器
## Servlet容器的搭建
- 基于第一章WEB服务器的搭建，（第一章的WEB服务器只能访问服务器端的静态资源），在能访问静态资源`static resources`的基础之上，能够处理`Servlet`对应的相关请求并调用`Servlet`对应的类的方法。

- **采用类加载器加载URI中对应的Servlet类名对应的类 + newInstance() 实例化的方法来实例化Servlet类**


### new关键字 和 newInstance实例化的区别
> **new关键字 和 newInstance实例化的区别**
> - 创建对象的方式不一样，前者是创建一个新对象,后者是使用类加载机制.
> - new创建一个类的时候，这个类可以没有被加载。但是使用newInstance()方法的时候，就必须保证：
>    - 1、这个类已经加载；
>    - 2、这个类已经连接了。
> - newInstance: 弱类型。低效率。只能调用无参构造。  
 new: 强类型。相对高效。能调用任何public构造。
> ---
> [**参考博客：newInstance() 和 new 有什么区别?**](http://blog.csdn.net/gudu1289/article/details/46638495)

### ServletServerTwo相比ServletSeverOne
- **使用了外观模式Facade（结构型）**
#### 外观模式
> - 概述：我们通过外观的包装，使应用程序只能看到外观对象，而不会看到具体的细节对象，这样无疑会降低应用程序的复杂度，并且提高了程序的可维护性。
> - 为子系统中的一组接口提供一个一致的界面， Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。引入外观角色之后，用户只需要直接与外观角色交互，用户与子系统之间的复杂关系由外观角色来实现，从而降低了系统的耦合度
***
> [参考博客：设计模式（九）外观模式Facade（结构型）](http://blog.csdn.net/hguisu/article/details/7533759)

[**简单web服务器对应的源码demo的Github地址**](https://github.com/zjs1224522500/demo)