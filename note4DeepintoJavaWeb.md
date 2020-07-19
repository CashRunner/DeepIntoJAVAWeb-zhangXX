![image-20200718223525083](E:\git\gitLearn\Note4DeepintoJAVAWeb\note4DeepintoJavaWeb.assets\image-20200718223525083.png)

参考书籍张孝祥的深入体验Java Web。

# 1 文件上传

关于当前项目仓库创建的问题

在github上创建了内容为空的仓库，没有readme文档。github提示在本地创建仓库然后执行`git remote add origin [当前仓库SSH地址] `。该命令只是在本地仓库添加了一个远程仓库的标签，并没有将本地仓库的分支与远程仓库的分支绑定，所以在执行`git push`时出现如下错误提示：

![image-20200719002755295](E:\git\gitLearn\Note4DeepintoJAVAWeb\note4DeepintoJavaWeb.assets\image-20200719002755295.png)

文件上传浏览器端：form表单 enctype:multipart/form-data。服务器端需要额外组件响应客户端的文件传输。

Tomcat环境搭建：https://blog.csdn.net/gyshun/article/details/80920227

Tomcat上web app目录结构：
<img src="E:\git\gitLearn\Note4DeepintoJAVAWeb\note4DeepintoJavaWeb.assets\image-20200718234742376.png" alt="image-20200718234742376" style="zoom:80%;" />

关于文件上传jar包处理上传文件原理的感悟：
这个jar包或者这个类要实现什么功能，具体由哪些步骤，各个步骤涉及哪些类，类之间传递的数据格式是什么样的。这样看源代码才行，一来就逮着源代码一行一行看，强撸灰飞烟灭。

当前状态 PDF 23页，Tomcat无法启动。打开Tomcat启动程序，查看最近一次启动日志，解决Tomcat无法启动的问题。

需要解决的问题是，将最新的一次add并入最近一次commit如何操作。

公司修改后的代码没有同步到GitHub上，不想浪费时间在重新写代码上了。在网上找找源代码，或者看看后面内容是否用到现在的文件上传web app

# Filter

Tomcat自动加载修改后的servlet方法：
修改Tomcat安装目录下的config下的server.xml在`<Host>`标签添加`<Context>`标签说明，具体内容如下
`<Context path="/filter" docBase="filter" reloadable="true" />`

下面从以下几个方面进行归纳：使用Filter组件能够实现什么功能，工作原理，实现功能步骤和过程分析，

## 2.2.1  Filter工作原理

Tomcat为Servlet容器，调用Servlet处理客户端请求。用户请求到达Tomcat时候被封装成请求对象Request，可以在用户请求对象和Servlet之间插入Filter对用户请求进行预处理，以及对处理结果进行额外处理。因此Filter是对具体Servlet功能的扩充，需要将Filter在web.xml中具体Servlet进行注册【推测】。当Tomcat需要调用Servlet时候如果有注册Filter，会先调用Filter对用户请求对象进行处理，然后再调用Servlet。具体原理如下

<img src="E:\git\gitLearn\Note4DeepintoJAVAWeb\note4DeepintoJavaWeb.assets\image-20200719223813723.png" alt="image-20200719223813723" style="zoom:50%;" />

在Filter中可以决定是否进入Servlet中，同时可以对Servlet处理结果进行再处理。需要注意的是Filter调用Servlet逻辑是通过Filter.doFilter()方法中的FilterChain.doFilter()实现的，如果在Filter.doFilter()中缺省FilterChain.doFilter()方法，Servlet逻辑将不会被执行。

Filter和Servlet可以是多对多的关系，执行目标Servlet前根据在web.xml中Servlet中注册的Filter顺序逐次调用多个Filter。第一个Filter.doFilter()中调用FilterChain.doFilter()方法将激活下一个Filter，从而组成Filter链。

### 2.2.3 Filter接口

1 init(FilterConfig config)
该方法实现Filter对象的初始化工作，在Tomcat启动Web app时候，会根据web.xml文件生成Filter对象然后调用该方法完成Filter初始化工作，该方法在Filter生命周期中只调用一次。

2 doFilter(ServletRequest reqeust ,ServletResponse response,FilterChain chain)
该方法中实现Filter逻辑，由Tomcat调用。

3 destroy()
回收Filter资源。

### 2.2.4 FilterChain接口

是Filter程序与Tomcat进行通讯的接口，通过doFilter(ServletRequest reqeust ,ServletResponse response)方法告诉Tomcat调用下一个Filter或者Servlet。

### 2.2.5 FilterConfig接口

Tomcat在创建和初始化Filter对象时候，将web.xml中关于Filter相关的配置参数以及上下文信息封装在了FilterConfig对象中，通过系统系统的FilterConfig接口可以获取其中信息，通过接口系统规定了只能调用哪些方法，从而保证了对象的安全性。

### 2.2.7 Filter注册和映射

在web.xml中注册Filter，Tomcat在启动时根据这个配置文件到WEB-INF下的classes文件夹或者lib文件夹下找。可以在classes下创建子目录，但是web.xml中声明Filter时候需要在`<filter-class>`中以`/`开始指定Filter编译文件相对于classes目录的位置。典型Filter注册格式

```xml
<filter>
        <filter-name>Timing Filter</filter-name>   <!--必须-->
        <filter-class>filters.ExampleFilter</filter-class>   <!--必须-->
        <init-param>   <!--可选-->
            <param-name>attribute</param-name>
            <param-value>filters.ExampleFilter</param-value>
        </init-param>
</filter>
```

在Tomcat中资源被分为两类，静态资源和动态资源（Servlet程序）。静态资源最终还是由Servlet进行处理，所以在Tomcat中只有一种资源就是Servlet。所以Filter映射配置时可以指定请求资源路径或者servlet。在Servlet 2.4规范中增加了资源请求方式出发Filter（include和forward，猜测意思是当以上述两种方式请静态求资源或者Servlet时会调用Filter)。因此在配置Filter映射时有以下3中配置方式：

```xml
<filter-mapping>
    <filter-name></filter-name>
    <url-pattern></url-pattern><!--静态资源或者Servlet的URL-->
</filter-mapping>

<filter-mapping>
    <filter-name></filter-name>
    <servlet-name></servlet-name>
</filter-mapping>

<filter-mapping>
    <filter-name></filter-name>
    <servlet-name></servlet-name>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>REQUEST</dispatcher><!--对所在资源缺省访问方式，可以设置多个资源请求方式-->
    <dispatcher>FORWADR</dispatcher>
    <dispatcher>ERROR</dispatcher>
</filter-mapping>
```

在配置映射时使用`<filter-mapping>`将资源和Filter分离，这样解耦Filter和资源。filter和资源（静态资源和Servlet）是多对多关系，如果请求资源设置多个`<filter-mapping>`根据在web.xml中出现顺序顺序访问Filter。	

当前进度73页，2小时完成10页。。。。沃妮马。。。。





