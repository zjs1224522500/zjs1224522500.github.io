---
title: 'Servlet常用用法（二）-文件上传下载'
date: 2017-11-2 21:28:52
tags: JavaWeb
---
# 3、`Servlet`实现文件上传下载
### 文件上传：
#### 原理（Servlet）
- 通过使用文件上传下载组件，通过读取请求中的文件的输入流，在预先协商好的路径下根据对应的文件名创建相应的文件对象，将提交的文件对象`FileItem`写入对应的文件中。
- 在`Servlet3.0`中可以使用注解`@MultiPartConfig`将一个`servlet`标识为**支持文件上传**，将`Multipart/form-data`的`POST`请求封装为成`Part`，通过`Part`对上传的文件进行操作。对应的在`Servlet`处理请求时使用`getPart()`和`getParts()`方法来获取对应的文件对象（`input`对应的`file`标签）
#### 框架对应的文件上传方法
- `SpringMVC`中用的是`MultipartFile`来进行文件上传 所以我们首先要配置`MultipartResolver`:**用于处理表单中的`file`**

<!-- more-->
```
<!-- 文件上传组件 -->
    <bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="maxUploadSize" value="500000"/>
    </bean>
```
- 传输过程中一直使用`MultipartFile`，而在磁盘对应的文件读写中则使用`java.io.file`；对应`MultipartFile`有几个关键方法：
    - **String getContentType()**//获取文件MIME类型
    - **InputStream getInputStream()**//后去文件流
    - **String getName()** //获取表单中文件组件的名字
    - **String getOriginalFilename()**//获取上传文件的原名
    - **boolean isEmpty()** //是否为空
    - **void transferTo(File dest)** //保存到一个目标文件中。

> 文件上传过程中可能存在**文件名中文乱码**的问题，常常是因为`request`的相关编码问题，为了能够统一解决，最好的方式时利用**字符过滤器**对所有请求进行整体的编码。其中`Spring`自带的字符过滤器就是比较好的选择。也可以使用自己定义的字符过滤器。

```xml
<!--start Spring自带字符过滤器 -->
    <filter>
        <filter-name>EncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>EncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--end Spring自带字符过滤器 -->
```

### 文件下载：
#### 原理（Servlet）
- 通过请求提交的指定的文件名，在服务器端根据路径和请求参数创建相应的文件对象和文件流，并利用字节数组将创建得到的文件流写入到`response`对应的输出流`ServletOutputStream`中，并指定`response`对应的一些头的属性，譬如`ContentType`中对应的`MIME类型`和`Content-Disposition`中对应的文件打开方式，然后随`response`发送到客户端。
- 在使用框架下载文件的过程中，通常和原生`Servlet`的处理方法保持一致，但往往需要构造`RestFul`风格的响应，故常常需要将下载文件作为一种特殊的响应类型（`SteamRestResponse`**流类型**），还是需要设置`response`对应的头的属性。
```
/**
 * 响应流返回结果
 *
 * @param file     文件信息
 * @param fileName 下载时显示的文件名称
 * @return 空
 * @throws IOException 将文件写入流中可能存在IO异常
 */
public RestResponse streamResponse(FileInfo file, String fileName) throws IOException {

	HttpServletResponse response = getResponseContext();

	//设置响应类型
	response.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
	response.setHeader("Accept-Ranges", "bytes");
	response.setIntHeader("Accept-Length", Math.toIntExact(file.getSize()));// 解决乱码问题

	// FireFox用ISO-8859-1
	String encodedFileName;
	if (Browser.IE.getName() == getClientBrowser()){
	    encodedFileName = URLEncoder.encode(fileName, "UTF-8");
	} else {
	    encodedFileName = new String(fileName.getBytes(), "ISO-8859-1");
	}
	response.setHeader("Content-Disposition","attachment; filename=\"" + encodedFileName + "\"; filename*=utf-8''" + encodedFileName);

	Path path = Paths.get(file.getPath());
	Files.copy(path, response.getOutputStream());

	return successResponse();
	}
```

### 文件上传下载过程中对应的文件路径的问题
- 由于开发和生产环境的不同，可能对应的需要保存文件上传下载的目录也有所不同，为了保证程序的的可移植性，常常需要采用相对路径来进行文件的相关存储，但又为了保证各个项目之间的数据能够独立，所以我们常把对应的文件存储在对应的项目目录下，并利用数据库持久化文件的相关数据，譬如`MIME类型`，`文件大小`，`MD5摘要`,`存放的路径`等等一系列信息。
- 为了实现上述功能，我们常常使用`Context`或`Config`来获取在`web.xml`中的定义的初始化参数（文件相对路径）。代码示例如下：
- web.xml
```
<!-- 通过使用ServletContext去获取 -->
<context-param>
    <param-name>uploadPath</param-name>
    <param-value>data</param-value>
</context-param>

<!-- 通过使用ServletConfig去获取 -->
<servlet>
    <init-param>
        <param-name>uploadPath</param-name>
        <param-value>data</param-value>
    </init-param>
</servlet>
```
- servlet或controller中
```java
ServletContext servletContext = req.getServletContext();
ServletConfig servletConfig = getServletConfig();
//使用servlet
String uploadPath = servletContext.getRealPath("/") + File.separator + servletConfig.getInitParameter("uploadPath");
				
//使用context				
String directory = servletContext.getRealPath("/") + File.separator + servletContext.getInitParameter("uploadPath");
```