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