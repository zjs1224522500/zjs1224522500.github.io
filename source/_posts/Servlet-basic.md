---
title: 'Servlet基础知识'
date: 2017-10-28 21:28:52
tags: JavaWeb
---
# 1、什么是`Servlet`？
> `Java Servlet` 是运行在 `Web` 服务器或应用服务器上的程序，它是作为来自 `Web` 浏览器或其他 `HTTP` 客户端的请求和 `HTTP` 服务器上的数据库或应用程序之间的中间层。
-----
> 个人见解:`Servlet`本质属于`MVC`架构模式中的`C（controller）`层，充当控制器的角色
<!--more-->
# 2、`Servlet`架构
![image](http://www.runoob.com/wp-content/uploads/2014/07/servlet-arch.jpg) ![image](http://www.runoob.com/wp-content/uploads/2014/07/Servlet-LifeCycle.jpg)
# 3、`Servlet`声明周期
- `Servlet` 通过调用 `init ()` 方法进行初始化。
    - `init` 方法**被设计成只调用一次**。它在第一次创建 `Servlet` 时被调用，在后续每次用户请求时不再调用。当用户调用一个 `Servlet` 时，就会创建一个 `Servlet` 实例，**每一个用户请求都会产生一个新的线程**，适当的时候移交给 `doGet` 或 `doPost` 方法。`init()` 方法简单地创建或加载一些数据，这些数据将被用于 `Servlet` 的**整个生命周期**。
    > [博文推荐：关于的servlet的单例模式(单实例多线程)的解释
](http://blog.csdn.net/wgyscsf/article/details/50129367?ref=myread)
- `Servlet` 调用 `service()` 方法来处理客户端的请求。
    - `service()` 方法是执行实际任务的主要方法。`Servlet` 容器（即 `Web` 服务器）调用 `service()` 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。
    > [扩展阅读：Http协议中的方法](http://blog.csdn.net/macroway/article/details/1428541)
- `Servlet` 通过调用 `destroy()` 方法终止（结束）。
    - destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。
- 最后，`Servlet` 是由 `JVM` 的垃圾回收器进行垃圾回收的。

# 4、`Servlet`获取请求头信息
方法 | 描述
---|---
Cookie[] getCookies()|返回一个数组，包含客户端发送该请求的所有的 Cookie 对象。  
Enumeration getAttributeNames()|返回一个枚举，包含提供给该请求可用的属性名称。
Enumeration getHeaderNames()|返回一个枚举，包含在该请求中包含的所有的头名。
Enumeration getParameterNames()|返回一个 String 对象的枚举，包含在该请求中包含的参数的名称。
HttpSession getSession()|返回与该请求关联的当前 session 会话，或者如果请求没有 session 会话，则创建一个。
HttpSession getSession(boolean create)|返回与该请求关联的当前 HttpSession，或者如果没有当前会话，且创建是真的，则返回一个新的 session 会话。
Locale getLocale()|基于 Accept-Language 头，返回客户端接受内容的首选的区域设置。
Object getAttribute(String name)|以对象形式返回已命名属性的值，如果没有给定名称的属性存在，则返回 null。
ServletInputStream getInputStream()|使用 ServletInputStream，以二进制数据形式检索请求的主体。
String getAuthType()|返回用于保护 Servlet 的身份验证方案的名称，例如，"BASIC" 或 "SSL"，如果JSP没有受到保护则返回 null。
String getCharacterEncoding()|返回请求主体中使用的字符编码的名称。
String getContentType()|返回请求主体的 MIME 类型，如果不知道类型则返回 null。
String getContextPath()|返回指示请求上下文的请求 URI 部分。
String getHeader(String name)|以字符串形式返回指定的请求头的值。
String getMethod()|返回请求的 HTTP 方法的名称，例如，GET、POST 或 PUT。
String getParameter(String name)|以字符串形式返回请求参数的值，或者如果参数不存在则返回 null。
String getPathInfo()|当请求发出时，返回与客户端发送的 URL 相关的任何额外的路径信息。
String getProtocol()|返回请求协议的名称和版本。
String getQueryString()|返回包含在路径后的请求 URL 中的查询字符串。
String getRemoteAddr()|返回发送请求的客户端的互联网协议（IP）地址。
String getRemoteHost()|返回发送请求的客户端的完全限定名称。
String getRemoteUser()|如果用户已通过身份验证，则返回发出请求的登录用户，或者如果用户未通过身份验证，则返回 null。
String getRequestURI()|从协议名称直到 HTTP 请求的第一行的查询字符串中，返回该请求的 URL 的一部分。
String getRequestedSessionId()|返回由客户端指定的 session 会话 ID。
String getServletPath()|返回调用 JSP 的请求的 URL 的一部分。
String[] getParameterValues(String name)|返回一个字符串对象的数组，包含所有给定的请求参数的值，如果参数不存在则返回 null。
boolean isSecure()|返回一个布尔值，指示请求是否使用安全通道，如 HTTPS。
int getContentLength()|以字节为单位返回请求主体的长度，并提供输入流，或者如果长度未知则返回 -1。
int getIntHeader(String name)|返回指定的请求头的值为一个 int 值。
int getServerPort()|返回接收到这个请求的端口号。
int getParameterMap()|将参数封装成 Map 类型。
- 示例代码
```java
    Enumeration headerNames = request.getHeaderNames();

    while(headerNames.hasMoreElements()) {
        String paramName = (String)headerNames.nextElement();
        out.print("<tr><td>" + paramName + "</td>\n");
        String paramValue = request.getHeader(paramName);
        out.println("<td> " + paramValue + "</td></tr>\n");
    }
```

# 5、设置`HTTP`响应报头的方法

方法 | 描述
---|---
String encodeRedirectURL(String url) |为 sendRedirect 方法中使用的指定的 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。
String encodeURL(String url)|对包含 session 会话 ID 的指定 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。
boolean containsHeader(String name) |返回一个布尔值，指示是否已经设置已命名的响应报头。
boolean isCommitted() | 返回一个布尔值，指示响应是否已经提交。
void addCookie(Cookie cookie) | 把指定的 cookie 添加到响应。
void addDateHeader(String name, long date) |添加一个带有给定的名称和日期值的响应报头。
void addHeader(String name, String value) |添加一个带有给定的名称和值的响应报头。
void addIntHeader(String name, int value) |添加一个带有给定的名称和整数值的响应报头。
void flushBuffer() |强制任何在缓冲区中的内容被写入到客户端。
void reset() |清除缓冲区中存在的任何数据，包括状态码和头。
void resetBuffer() |清除响应中基础缓冲区的内容，不清除状态码和头。
void sendError(int sc) |使用指定的状态码发送错误响应到客户端，并清除缓冲区。
void sendError(int sc, String msg) |使用指定的状态发送错误响应到客户端。
void sendRedirect(String location) |使用指定的重定向位置 URL 发送临时重定向响应到客户端。
void setBufferSize(int size) |为响应主体设置首选的缓冲区大小。
void setCharacterEncoding(String charset) |设置被发送到客户端的响应的字符编码（MIME 字符集）例如，UTF-8。
void setContentLength(int len) |设置在 HTTP Servlet 响应中的内容主体的长度，该方法设置 HTTP Content-Length 头。
void setContentType(String type) |如果响应还未被提交，设置被发送到客户端的响应的内容类型。
void setDateHeader(String name, long date) |设置一个带有给定的名称和日期值的响应报头。
void setHeader(String name, String value) |设置一个带有给定的名称和值的响应报头。
void setIntHeader(String name, int value) |设置一个带有给定的名称和整数值的响应报头。
void setLocale(Locale loc) |如果响应还未被提交，设置响应的区域。
void setStatus(int sc) |为该响应设置状态码。

# 6、`Servlet`中的`HTTP`状态码
## 请求报文和响应报文的格式
- 初始状态行 + 回车换行符（回车+换行）
- 零个或多个标题行+回车换行符
- 一个空白行，即回车换行符
- 一个可选的消息主体，比如文件、查询数据或查询输出  

##### 例子如下：
```html
HTTP/1.1 200 OK            // HTTP版本 状态码 对应状态码的短消息
Content-Type: text/html    // 报文头中包含的各个属性信息:键值对的形式
Header2: ...
...
HeaderN: ...
  (Blank Line)
<!doctype ...>
<html>
<head>...</head>
<body>
...
</body>
</html>
```
## 设置 HTTP 状态代码的方法
- 由于状态代码的产生都是服务端对于客户端发起的请求进行的相应的回应的过程中产生的，所以我们通过调用`HttpServletResponse`中的相关方法来进行对状态码的设置。

方法 | 描述
---|---
public void setStatus ( int statusCode ) | 该方法设置一个任意的状态码。setStatus 方法接受一个 int（状态码）作为参数。如果您的反应包含了一个特殊的状态码和文档，请确保在使用 PrintWriter 实际返回任何内容之前调用 setStatus。
public void sendRedirect(String url) | 该方法生成一个 302 响应，连同一个带有新文档 URL 的 Location 头。
public void sendError(int code, String message) | 该方法发送一个状态码（通常为 404），连同一个在 HTML 文档内部自动格式化并发送到客户端的短消息。

