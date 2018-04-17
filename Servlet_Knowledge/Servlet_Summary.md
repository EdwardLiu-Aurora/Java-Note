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
- 定义默认的区域与编码的对应关系(例如默认将中国大陆对应使用 UTF-8 )
```
<locale-encoding-mapping-list>
    <locale-encoding-mapping>
        <locale>zh_CN</locale>
        <encoding>UTF-8</encoding>
    </local-encoding-mapping>
<locale-encoding-mapping-list>
```
- 设置文件后缀名与 MIME 类型的对应关系 (可以通过 servletContext 的 getMimeType() 方法获得)
```
<mime-mapping>
    <extension>pdf</extension>
    <mime-type>application/pdf</mime-type>
</mime-mapping>
```
- 设置 HttpSession 的失效时间 (单位为秒)
```
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```
- 在 web.xml 中设定 SessionCookieConfig 信息
```
<session-config>
    <session-timeout>30</session-timeout>
    <cookie-config>
        <name>sid-caterpillar</name>
        <http-only>true</http-only>
    </cookie-config>
</session-config>
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
getLocale|java.util.Locale getLocale()|获取地区信息


- 若使用 Servlet 处理上传的文件，可以使用 Java IO 结合 getInputStream()。判断上传的文件的开始位置和结束位置，然后用字节流写为文件。
- 若使用 Servlet 处理上传的文件，也可以使用 getPart()、getParts() 方法。
- 使用 RequestDispatcher.include() 可以把其它 servlet 的操作流程包含到当前的 servlet 操作流程之中。


### HttpServletResponse
代表 Http 响应报文的类

方法|方法签名|方法功能
---|---|---
getWriter|java.io.PrintWriter getWriter()|取得响应输出对象
setContentType|void setContentType(java.land.String type)|设置响应内容类型
setHeader|void setHeader(java.lang.String name, java.lang.String value)|设置特定响应标头的值
addHeader|void addHeader(java.lang.String name, java.lang.String value)|添加响应标头的名称和值
setIntHeader|void setIntHeader(java.lang.String name, int value)|如果对应标头的值是整数，可用此方法设置
setDateHeader|void setDateHeader(java.lang.String name, long date)|如果对应标头的值是日期，可用此方法设置
addIntHeader|void addIntHeader(java.lang.String name, int value)|如果对应标头的值是整数，可用此方法添加
addDateHeader|void addDateHeader(java.lang.String name, long date)|如果对应标头的值是日期，可用此方法添加
setLocale|void setLocale(java.util.Locale loc)|设置地区以及编码行为
setCharacterEncoding|void setCharacterEncoding(java.lang.String charset)|设置字符编码
setContentType|void setContentType(java.lang.String type)|设置内容类型
getOutputStream|ServletOutputStream getOutputStream()|取得输出的流对象
sendRedirect|void sendRedirect(java.lang.String location)|在响应中设置状态码 301 以及 Location 标头，浏览器会根据新的 URL 进行重定向
sendError|void sendError(int sc)|向浏览器返回错误状态码
sendError|void sendError(int sc, java.lang.String msg)|向浏览器返回错误状态码和相关错误信息

- 容器可以对响应进行缓冲，可以操作 HttpServletResponse 以下有关缓冲的几个方法
    - getBufferSize()   获取缓冲区大小
    - setBufferSize()   设置缓冲区大小，必须在调用 getWriter 之前调用
    - isCommitted()     查看响应是否已经确认
    - reset()           重置所有响应信息，包括标头信息
    - resetBuffer()     重置相应信息，不包括标头信息
    - flushBuffer()     清除所有缓冲区已设置的信息到客户端
- 容器响应关闭的时机点 (容器关闭会清楚所有的响应内容)：
    - Servlet 的 service() 方法已结束，响应的内容长度超过 HttpServletResponse 的 setContentLength() 所设置的长度
    - 调用了 sendRedirect() 方法
    - 调用了 sendError() 方法
    - 调用了 AsyncContext 的 complete() 方法
- 使用 ServletContext 对象获取当前 Web 应用程序目录内的文件
    - 使用 HttpServlet 的 getServletContext() 获取 ServletContext 对象
    - 使用 ServletContext 的 getResourceAsStream() 方法以串流读取文件
    - 使用 HttpServletResponse 的 getOutputStream() 获取 ServletOutputStream 对象并进行响应输出

### 会话管理
1. 使用隐藏域：在 html 表单中使用 type='hidden' 来将数据写入隐藏域
```
<form action='XXX' method='post'>
    隐藏域 <input type='text' name='first' /><br />
    <input type='hidden' name='second' value='1' />
    <input type='hidden' name='third' value='2' />
    <input type='submit' name='page' value='完成' />
</form>
```
使用这种方法，可以用 request.getParameters() 等方法获取之前页面中的数据
2. 使用 Cookie
```
// 新建 Cookie 对象
Cookie cookie = new Cookie("name", "data");
// 设置 Cookie 有效期 (单位为秒)
cookie.setMaxAge(7 * 24 * 60 * 60);
// 将 cookie 添加到响应中
response.addCookie(cookie);
```
Cookie 可以使用 request.getCookies() 来取得
3. 使用 URL 重写，即在超链接中包含相关信息，以 url 方式传给 servlet
```
<a href='search?start=1' />
```

#### 使用 HttpSession
在 Servlet/JSP 中，如果想进行会话管理，可以使用 HttpServletRequest 的 getSession() 方法取得 HttpSession 对象
```
HttpSession session = request.getSession();
```
HttpSession 常用方法及其用途：
- setAttribute()                设置属性
- getAttribute()                取得属性
- invalidate()                  注销
- setMaxInactiveInternal()      设置浏览器多久没有请求应用程序，session 会失效 (单位为秒)

可以使用 ServletContext 的 getSessionCookieConfig() 来取得 SessionCookieConfig 接口的实现

HttpSession 与 URL 重写
> 因为 HttpSession 的原理是使用 Cookie 存储 Session ID，所以如果用户关掉浏览器接收 Cookie 的功能，就无法使用 Cookie 在浏览器存储 Session ID。如果在用户禁用 Cookie 的情况下，仍然打算使用 HttpSession 来进行会话管理，那么可以搭配 URL 重写。

如果要使用 URL 重写的方式来发送 Session ID，可以使用 HttpServletResponse 的 encodeURL() 协助产生所需的 URL 重写。当容器尝试取得 HttpSession 实例时，如果能从 Http 请求中获得带有 Session ID 的 Cookie ，那么 encodeURL() 将不对 URL 做修改。而如果容器无法取得 Session ID 时 (一般意味着用户禁用了 cookie )，encodeURL() 将会自动产生带有 Session ID 的 URL 重写。

encodeRedirectURL() 可以在浏览器重定向时，在 URL 上显示 Session ID

## Servlet 进阶 API

### Servlet
- init()        启动时
- service()     传入请求和响应对象
- destroy()     销毁时

### ServletConfig
使用 getServletConfig() 获取 ServletConfig 对象
- getInitParameter(String name)     获取对应的 Servlet 设置参数
- getInitParameters()               获取全体的 Servlet 设置参数
> 可以使用 @WebServlet 中的 initParam 属性设置
```
    initParams={
        @WebInitParam{name="PARAM1",value="VALUE1"),
        @WebInitParam{name="PARAM2",value="VALUE2")
    }
```

### ServletContext
使用 getServletContext() 获取 ServletContext 对象

ServletContext 接口定义了运行 Servlet 应用程序环境的一些行为与观点，可以使用 ServletContext 实现对象来取得请求资源的 URL、设置与存储属性、应用程序初始参数以及动态设置 Servlet 实例等。

1. getRequestDispatcher()   获得请求分发器
2. getResourcePaths()       获得 Web 应用程序某个目录中有哪些文件
3. getResourceAsStream()    获得 Web 应用程序某个文件的内容，返回 InputStream

- ServletContextListener
    - 可以实现这个接口，来在应用程序初始化或销毁时进行自定义操作
    - 可以准备好数据库连接，读取应用程序设置等
    - 使用 @WebListener 标注
    - contextInitialized()
    - contextDestroyed()

- ServletContextAttributeListener
    - attributeAdded()
    - attributeRemoved()
    - attributeReplaced()

- HttpSession 事件、监听器
    - HttpSessionListener
        - SessionCreated()
        - SessionDestroyed()
    - HttpSessionAttributeListener
        - attributeAdded()
        - attributeRemoved()
        - attributeReplaced()
    - HttpSessionBindingListener
        - valueBound()
        - valueUnbound()
    - HttpSessionActivationListner
        - sessionWillPassivate()
        - sessionDidActivate()

- HttpServletRequest 事件、监听器
    - ServletRequestListener
        - requestDestroyed()
        - requestInitialized()
    - ServletRequestAttributeListener()
        - attributeAdded()
        - attributeRemoved()
        - attributeReplaced()

- 过滤器
    - 可以通过实现 Filter 接口，并使用 @WebFilter 标注或在 web.xml 中定义过滤器。
    - Filter 接口有三个要实现的方法：
        - public void init(FilterConfig filterConfig)
        - public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        - public void destroy()
    - chain.doFilter(request, response); 前执行的代码为 service() 前执行的代码，后执行的代码为 service() 后执行的代码
    - @WebFilter(filterName="XXX", urlPatterns={"XXX"})
    ```
    <filter>
        <filter-name>performance</filter-name>
        <filter-class>cc.openhome.PerformanceFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>performance</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```
    - 可以继承 HttpServletRequestWrapper 来对 HttpServletRequest 类进行处理，然后再再 doFilter 中传入 service()
    - 可以继承 HttpServletResponseWrapper 来对 HttpServletResponse 类进行处理。一般会在压缩输出中用到。

- 异步处理 AsyncContext
    - 要使用 @WebServlet 注解告诉容器此 Servlet 支持异步处理 asyncSupported=true

刘丰璨 写于 2018.04.15 9:00 简书: https://www.jianshu.com/p/d6e8b1483083
