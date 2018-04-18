### Servlet 至 JSP 的简单转换
设置响应类型
```
Servlet 代码:
response.setContentType("text/html;charset=UTF-8");
JSP 代码：
<%@page contentType="text/html" pageEncoding="UTF-8"%>
```
获取输出对象
```
Servlet 代码：
PrintWriter out = response.getWriter();
JSP 代码：
可以直接删除，因为 JSP 中有隐式对象，out 就是 JSP 的一个隐式对象名称
```
获取属性 (JSP 中的 application 对象相当于 Servlet 中的 ServletContext 对象)
```
Servlet 代码：
BookmarkService bookmarkService = (BookmarkService) getServletContext().getAttribute("bookmarkService");
for(Bookmark bookmark : bookmarkService.getBookmarks()) {
JSP 代码：
<%
    BookmarkService bookmarkService = (BookmarkService) application.getAttribute("bookmarkService");
    for(Bookmark bookmark : bookmarkService.getBookmarks()) {
%>
```
import 语句
```
Servlet 代码：
import cc.openhome.model.Bookmark;
import cc.openhome.model.BookmarkService;
import java.io.*;
import java.util.*;
import javax.servlet.*;
import javax.servlet.http.*;
JSP 代码：
<%@page import="cc.openhome.model.*", java.util.*" %>
```
Java 取值操作对应的 JSP 代码
```
<%= bookmark.getTitle() %>
```
Java 声明
```
<%! 类的声明或方法声明 %>
```

### 指示元素
<%@ 指示类型 [属性="值"]* %>
常用的指示类型：
- page
    - import
    - contentType
    - pageEncoding
    - info
    - autoFlush
    - buffer
    - errorPage
    - extends
    - isErrorPage
    - language
    - session
    - isELIgnored
    - isThreadSafe
- include
    - file
- taglib

### 隐式对象
- out           转译后对应 JspWriter 对象，其内部关联一个 PrintWriter 对象
- request       转译后对应 HttpServletRequest 对象
- response      转译后对应 HttpServletResponse 对象
- config        转译后对应 ServletConfig 对象
- application   转译后对应 ServletContext 对象
- session       转译后对应 HttpSession 对象
- pageContext   转译后对应 PageContext 对象
- exception     转译后对应 Throwable 对象
- page          转译后对应 this

### 标准标签
- <jsp:include> 底层是 RequestDispatcher 的 include()
- <jsp:forward> 底层是 RequestDispatcher 的 forward()
- <jsp:param name="名字" value="值" />
- <jsp:useBean>
- <jsp:getProperty>
- <jsp:setProperty>

### EL 简介
EL 即 Expression Language，EL 是使用 ${} 来包含所要进行处理的表达式。
代码区别：
```
原代码：
<%= ((HttpServletRequest) pageContext.getRequest()).getMethod() %><br />
<%= ((HttpServletRequest) pageContext.getRequest()).getQueryString() %><br />
<%= ((HttpServletRequest) pageContext.getRequest()).getRemoteAddr() %><br />
EL代码：
${pageContext.request.method}<br/>
${pageContext.request.queryString}<br/>
${pageContext.request.remoteAddr}<br/>
```
- 若使用 . 运算符，左边可以是 JavaBean 或 Map 对象
- 若使用 [] 运算符，左边可以使 JavaBean、Map、数组或 List 对象

#### EL 运算符
- 算术运算符 + - * / % mod
- 逻辑运算符 and or not
- 关系运算符 < <= > >= == !=

#### 自定义 EL 函数
1. 编写一个标签程序库描述 (Tag Library Descriptor, TLD) 文件
2. 在 JSP 中使用 taglib 指示元素来指示如何寻找 TLD 文件
3. 在 JSP 中使用自定义的 EL 函数

### 使用 JSTL
#### JSTL 简介
JSTL 提供的跟页面呈现相关的逻辑判断标签，有五大类：
- 核心标签库：提供条件判断、属性访问、URL 处理以及错误处理等标签
- I18N 兼容格式标签库：提供数字、日期等的格式化功能，以及区域 (Locale)、信息、编码处理等国际化功能的标签。
- SQL 标签库：提供基本的数据库查询、更新、设置数据源 (DataSource) 等功能的标签。
- XML 标签库：提供 XML 解析、流程控制、转换等功能的标签。
- 函数标签库：提供常用字符串处理的自定义 EL 函数标签库。
JSTL 是 JSP 外的规范标准，如果希望使用，则需要下载 JSTL 相关 jar 包
要使用 JSTL 标签库，必须在 JSP 网页上使用 taglib 指示元素定义前置名称与 uri 参考
#### JSTL 示例
```
<c:if test="此处放置 EL 表达式">
    输出内容
</c:if>
```
```
<c:choose>
    <c:when test="此处放置 EL 表达式">
        输出内容
    </c:when>
    <c:otherwise>
        输出内容
    </c:otherwise>
</c:choose>
```
```
<c:forEach var="遍历的变量名" items="EL表达式">
    输出内容
</c:forEach>
```
- <c:catch>
- <c:redirect>
- <c:import>
- <c:set>
- <c:url>
- <c:out>