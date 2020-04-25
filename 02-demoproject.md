# 图书管理与购物项目

demo级别，使用servlet+tomcat+mySQL+maven(管理依赖)，部署到远程服务器上

## 部署环境

软件环境：IDEA	Java-11.0.5	Tomcat-9.0.29	Maven-3.6.1

建立Maven项目，根据已有archtype中的webapp。设置目录格式、使用MVC模型。Maven出了点小问题，需要调整到3.6.1才可以正常导入依赖包。

<img src="C:\Users\Liu Ruiping\AppData\Roaming\Typora\typora-user-images\image-20191207164946260.png" alt="image-20191207164946260" style="zoom: 80%;" />

## 写前端jsp页面

使用了frameset 虽然过时了但是简单应用一下，出了问题再改，前端最好还是用react。

这里用framset分帧管理的办法构建网页：每个网页由标题、边栏、主体等各个部分组成，每个部分使用一个jsp文件，后用一个总的jsp文件使用frameset调用。

## 写数据库接口

使用c3p0连接池，写dao类，抽接口，使用test运行测试。

## 写后端处理

使用Servlet做controller。