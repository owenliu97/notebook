# Servlet 学习笔记

前面一部分因为误操作删除了，以后再补上。

## 使用IDEA配置本地运行的web application

新建project时选择web application，选择本地的tomcat。在src中添加代码，在WEB-INF中的web.xml添加servlet信息。如果有其他文件需要显示或者下载，需要添加在web文件夹中，web文件夹下的子目录除WEB-INF外应该标注为resources。

如果有类更新则应点redeploy或者重启服务器。tomcat是服务器，一个项目建立一个运行在上面的artifact。

### 注意

1. **getWriter()和getOutputStream()两个方法不能同时调用**。如果同时调用就会出现异常
2. Servlet程序向ServletOutputStream或PrintWriter对象中**写入的数据将被Servlet引擎从response里面获取，Servlet引擎将这些数据当作响应消息的正文，然后再与响应状态行和各响应头组合后输出到客户端**。
3. Servlet的**serice()方法结束后【也就是doPost()或者doGet()结束后】**，Servlet引擎将检查getWriter或getOutputStream方法返回的输出流对象是否已经调用过close方法，**如果没有，Servlet引擎将调用close方法关闭该输出流对象.**

## Cookie技术

保存在浏览器上的数据，name, string成对存储。

## Session技术

保存在服务器端的数据，可以保存对象。SessionId唯一记录Session，保存在cookies中，若禁用cookies则需要url重写技术。

## 过滤器Filter的使用和配置

 **过滤器之间的执行顺序看在web.xml文件中mapping的先后顺序的，如果放在前面就先执行，放在后面就后执行！如果是通过注解的方式配置，就比较urlPatterns的字符串优先级** 

#### filter的三种典型应用：

1. 可以在filter中根据条件决定是否调用chain.doFilter(request, response)方法，即是否让目标资源执行

2. 在让目标资源执行之前，可以对request\response作预处理，再让目标资源执行

3. 在目标资源执行之后，可以捕获目标资源的执行结果，从而实现一些特殊的功能

