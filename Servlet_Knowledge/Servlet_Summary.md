# Servlet 知识点汇总

### @WebServlet 注解
注解配置样例 @WebServlet(name=“HelloServlet”,urlPatterns={"/hello"})

属性名|类型|属性描述
---|---|---
name|String|Servlet的名称。等价于<Servlet-name>。如没有显式指定，则该取值为类的全限定名.
value|String[]|等价于urlPatterns，二者不能共存。
urlPatterns|String[]|指定一组servlet的url的匹配模式，等价于<url-pattern>标签。
loadOnStartup|int|指定servlet的加载顺序，数字大于0则表示启动时初始化，数字越小初始化越早，等价于<load-on-startup>标签。
initParams|WebInitParam[]|指定一组初始化参数，等价于<init-param>标签。
asyncSupported|boolean|声明servlet是否支持异步操作模式，等价于<async-supported>标签。
displayName|String|servlet的显示名，等价于<display-name>标签。
description|String|servlet的描述信息，等价于<description>标签。

### @MultipartConfig 注解
注解配置样例 @MultipartConfig(location="c:/workspace")

属性名|类型|属性描述
---|---|---
fileSizeThreshold|int|若上传文件大小超过设置门槛，则会先写入缓存文件，默认值为 0
location|String|设置写入文件时的目录，可搭配 Part 的 write() 方法用，默认为空字符
maxFileSize|long|限制上传文件大小，默认值为 -1L，表示不限大小。
maxRequestSize|long|限制 multipart/form-data 请求个数，默认值为 -1L，表示不限个数。

### 使用 web.xml
文件位于 WEB-INF 文件夹内。web.xml 中的设置会覆盖 Servlet 中的标注设置。
- 定义 Servlet 的属性
```
<servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>cc.openhome.HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
<servlet>
```
- 定义 Servlet 的映射
```
<servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/HelloUser.view</url-pattern>
</servlet-mapping>
```
- 定义欢迎页面
```
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>default.jsp</welcome-file>
</welcome-file-list>
```
- 定义 @MultipartConfig 对应的信息，将 <multipart-config> 加到对应的 <servlet> 标签中
```
<multipart-config>
    <location>c:/workspace</location>
</multipart-config>
```

### URL 模式设置
一个请求 URI 实际上由三个部分组成：
> requestURI = contextPath + servletPath + pathInfo

URL 模式 (如果模式匹配重复，则默认按照最严格的模式优先)：
- 路径映射          如 "/guest/*
- 扩展映射          如 "*.view"
- 环境根目录映射    如 "/"
- 预设 Servlet      当找不到合适的 URL 模式对应时，就会使用预设 Servlet。
- 完全匹配          如 "/guest/test.view"

### 请求对象编码详解
- Web 容器默认编码处理是 ISO-8859-1
- POST 方式的请求，可以使用 request.setCharacterEncoding() 方法指定编码
- GET 方式的请求，可以以下方式来指定编码 ( 因为 setCharacterEncoding 只对 body 有效，对 URI 无效 )
```
    String name = request.getParameter("name");
    String name = new String(name.getBytes("ISO-8859-1"),"UTF-8");
```
- 和前端协商好编码的类型，以此确定服务器端的解码类型。

### HttpServletRequest
代表 Http 请求报文的类

方法|方法签名|方法功能
---|---|---
getParameter|java.lang.String getParameter(java.lang.String name)|返回请求参数，如果不存在则返回 null
getRequestURI|java.lang.String getRequestURI()|获取请求 URI
getContextPath|java.lang.String getContextPath()|获取环境路径 contextPath
getServletPath|java.lang.String getServletPath()|获取 Servlet 路径
getPathInfo|java.lang.String getPathInfo()|获取路径信息
getParameterValues|java.lang.String[] getParameterValues(java.lang.String name)|获取同一请求名称的多个值
getParameterMap|java.util.Map<java.lang.String,java.lang.String[]> getParameterMap()|将请求参数以 Map 形式返回
getHeader|java.lang.String getHeader(java.lang.String name)|返回 Http 请求首部
getHeaders|java.util.Enumeration<java.lang.String> getHeaders(java.lang.String name)|返回同一首部参数的多个值
getHeaderNames|java.util.Enumeration<java.lang.String> getHeaderNames()|返回所有 Http 首部参数名
getDateHeader|long getDateHeader(java.lang.String name)|将 Http 首部特定参数的值转换为 Date 的值
getIntHeader|int getIntHeader(java.lang.String name)|将 Http 首部特定参数的值转换为 int
getReader|java.io.BufferedReader getReader()|取得 BufferedReader 来读取请求的 Body 数据
getInputStream|ServletInputStream getInputStream()|返回请求报文主体的二进制字节数据
getRequestDispatcher|RequestDispatcher getRequestDispatcher(java.lang.String path)|获得请求分配器，可以转发或包含相对的 URL 地址


- 若使用 Servlet 处理上传的文件，可以使用 Java IO 结合 getInputStream()。判断上传的文件的开始位置和结束位置，然后用字节流写为文件。
- 若使用 Servlet 处理上传的文件，也可以使用 getPart()、getParts() 方法。
- 使用 RequestDispatcher.include() 可以把其它 servlet 的操作流程包含到当前的 servlet 操作流程之中。


### HttpServletResponse
代表 Http 响应报文的类

方法|方法签名|方法功能
---|---|---
getWriter|java.io.PrintWriter getWriter()|取得响应输出对象
setContentType|void setContentType(java.land.String type)|设置响应内容类型

刘丰璨 写于 2018.04.15 9:00 简书: https://www.jianshu.com/p/d6e8b1483083
