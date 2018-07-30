---
title: 'Servlet常用用法（一）'
date: 2017-11-1 21:28:52
tags: JavaWeb
---
# 1、`Servlet`编写过滤器(`Filter`)

> **`Servlet` 过滤器** 可以动态地拦截请求【**请求预处理**】和响应【**响应后处理**】，以变换或使用包含在请求或响应中的信息。可以将一个或多个 `Servlet` 过滤器附加到一个 `Servlet` 或一组 `Servlet`。`Servlet` 过滤器也可以附加到 `JavaServer Pages (JSP)` 文件和 `HTML` 页面。调用 `Servlet` 前调用所有附加的 `Servlet` 过滤器。从而实现一些特殊的功能。

#### 请求预处理：
- 在`HttpServletRequest`到达 `Servlet` 之前，拦截客户的`HttpServletRequest` 。根据需要检查`HttpServletRequest`，也可以修改`HttpServletRequest` 头和数据。

#### 响应后处理：
- 在`HttpServletResponse`到达客户端之前，拦截`HttpServletResponse` 。根据需要检查`HttpServletResponse`，也可以修改`HttpServletResponse`头和数据。
<!--more-->
### 分类
- 身份验证过滤器（Authentication Filters）。【**权限访问控制**】
- 数据压缩过滤器（Data compression Filters）。【**压缩响应信息**】
- 加密过滤器（Encryption Filters）。
- 触发资源访问事件过滤器。
- 图像转换过滤器（Image Conversion Filters）。
- 日志记录和审核过滤器（Logging and Auditing Filters）。
- MIME-TYPE 链过滤器（MIME-TYPE Chain Filters）。
- 标记化过滤器（Tokenizing Filters）。
- XSL/T 过滤器（XSL/T Filters），转换 XML 内容。

## Filter的生命周期
```
/*初始化，Filter的创建和销毁由WEB服务器负责。 web 应用程序启动时，web 服务器将创建Filter 的实例对象,  
并调用其init方法，读取web.xml配置，完成对象的初始化功能，  
从而为后续的用户请求作好拦截的准备工作（filter对象只会创建一次，init方法也只会执行一次）。  
开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。
*/
public void init(FilterConfig filterConfig) throws ServletException;

/*
拦截请求,完成实际的过滤操作。当客户请求访问与过滤器关联的URL的时候，
Servlet过滤器将先执行doFilter方法。FilterChain参数用于访问后续过滤器。
*/
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;

/*
销毁,Filter对象创建后会驻留在内存，当web应用移除或服务器停止时才销毁。  
在Web容器卸载 Filter 对象之前被调用。该方法在Filter的生命周期中仅执行一次。  
在这个方法中，可以释放过滤器使用的资源。
*/
public void destroy();
```

- 字符编码过滤器、静态资源过滤器、禁止缓存过滤器、非法字符过滤器 编码实现：
> [`JavaWeb`中常用的`Filter`对应我的Github仓库地址](https://github.com/zjs1224522500/JavaWebUtils/tree/master/CommonFilters)

# 2、`Servlet` 跟踪`Session`
### 维持客户端和服务器之间的`session`会话的方式：
- **`Cookies`**
- **隐藏的表单字段** `<input type="hidden" name="sessionid" value="12345">`
- **`URL`重写**：在每个`URL`末尾追加一些额外的数据来标识`session` 会话，服务器会把该`session`会话标识符与已存储的有关`session` 会话的数据相关联。`http://w3cschool.cc/file.htm;sessionid=12345`
- **`HttpSession`对象**：`Servlet`还提供了`HttpSession`接口，该接口提供了一种跨多个页面请求或访问网站时识别用户以及存储有关用户信息的方式。`Servlet`容器使用这个接口来创建一个`HTTP`客户端和 `HTTP`服务器之间的`session`会话。会话持续一个指定的时间段，跨多个连接或页面请求。
> `HttpSession session = request.getSession();`

方法 | 描述
---|---
public Object getAttribute(String name)|该方法返回在该 session 会话中具有指定名称的对象，如果没有指定名称的对象，则返回 null。
public Enumeration getAttributeNames()|该方法返回 String 对象的枚举，String 对象包含所有绑定到该 session 会话的对象的名称。
public long getCreationTime() | 该方法返回该 session 会话被创建的时间，自格林尼治标准时间 1970 年 1 月 1 日午夜算起，以毫秒为单位。
public String getId() | 该方法返回一个包含分配给该 session 会话的唯一标识符的字符串。
public long getLastAccessedTime() | 该方法返回客户端最后一次发送与该 session 会话相关的请求的时间自格林尼治标准时间 1970 年 1 月 1 日午夜算起，以毫秒为单位。
public int getMaxInactiveInterval() | 该方法返回 Servlet 容器在客户端访问时保持 session 会话打开的最大时间间隔，以秒为单位。
public void invalidate() | 该方法指示该 session 会话无效，并解除绑定到它上面的任何对象。
public boolean isNew() | 如果客户端还不知道该 session 会话，或者如果客户选择不参入该 session 会话，则该方法返回 true。
public void removeAttribute(String name) | 该方法将从该 session 会话移除指定名称的对象。
public void setAttribute(String name, Object value) | 该方法使用指定的名称绑定一个对象到该 session 会话。
public void setMaxInactiveInterval(int interval) | 该方法在 Servlet 容器指示该 session 会话无效之前，指定客户端请求之间的时间，以秒为单位。


### 删除`Session`会话数据
##### 当您完成了一个用户的 session 会话数据，您有以下几种选择：
- **移除一个特定的属性**：您可以调用 `public void removeAttribute(String name)` 方法来删除与特定的键相关联的值。
删除整个 `session` 会话：您可以调用 `public void invalidate()` 方法来丢弃整个 `session` 会话。
- **设置 `session` 会话过期时间**：您可以调用 `public void setMaxInactiveInterval(int interval)` 方法来单独设置 `session` 会话超时。
- **注销用户**：如果使用的是支持 `servlet 2.4` 的服务器，您可以调用 `logout` 来注销 `Web` 服务器的客户端，并把属于所有用户的所有 `session` 会话设置为无效。
- **`web.xml` 配置**：如果您使用的是 Tomcat，除了上述方法，您还可以在 web.xml 文件中配置 session 会话超时，如下所示：
```
  <session-config>
    <session-timeout>15</session-timeout>
  </session-config>
```
上面实例中的超时时间是以分钟为单位，将覆盖 `Tomcat` 中默认的 30 分钟超时时间。  

在一个 `Servlet` 中的 `getMaxInactiveInterval()` 方法会返回 `session` 会话的超时时间，以秒为单位。

> [session相关代码示例Github地址](https://github.com/zjs1224522500/JavaWebUtils/blob/master/CommonFilters/src/main/java/com/elvis/session/SessionTrack.java)
---
> `sessionId`:`sessionid`是一个会话的`key`，浏览器第一次访问服务器会在服务器端生成一个`session`，有一个`sessionid`和它对应。`tomcat`生成的`sessionid`叫做`jsessionid`。

- `session`在访问`tomcat`服务器`HttpServletRequest`的`getSession(true)`的时候创建，`tomcat`的`ManagerBase`类提供创建`sessionid`的方法：**随机数+时间+jvmid**；
> [参考博文：sessionid如何产生？由谁产生？保存在哪里？](http://www.cnblogs.com/woshimrf/p/5317776.html)