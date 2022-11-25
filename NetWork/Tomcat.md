# Tomcat

## 一、什么是Tomcat？

Web服务器用于接收客户端的请求并做出响应，而**Tomcat是最常见的免费Web服务器**，因为免费，不是，因为好用所以广受初学者喜爱

## 二、Tomcat安装

下载网址：[https://apache.org/](https://apache.org/)

![1658710242671](assets/1658710242671.png)

![1658710318352](assets/1658710318352.png)

![1658710332518](assets/1658710332518.png)

![1658710356917](assets/1658710356917.png)

![1658710510571](assets/1658710510571.png)

## 三、Tomcat配置

### 1.配置环境变量

![1658710575651](assets/1658710575651.png)

![1658710589652](assets/1658710589652.png)

![1658710612333](assets/1658710612333.png)

![1658710646853](assets/1658710646853.png)

![1658710676398](assets/1658710676398.png)

![1658710695685](assets/1658710695685.png)

![1658710729050](assets/1658710729050.png)

![1658710768245](assets/1658710768245.png)

### 2.启动

![1658710801944](assets/1658710801944.png)

![1658710815617](assets/1658710815617.png)

![1658710839090](assets/1658710839090.png)

### 3.加入用户以及角色，并登录后台管理页面

![1658710867472](assets/1658710867472.png)

![1658710908443](assets/1658710908443.png)

![1658710922849](assets/1658710922849.png)

![1658710966063](assets/1658710966063.png)

![1658710979433](assets/1658710979433.png)

`shutdown.bat`关闭服务后重启，刷新网站，输入用户名和密码，可进入即可

### 4.修改端口

![1658711052275](assets/1658711052275.png)

![1658711064237](assets/1658711064237.png)

`shutdown.bat`关闭服务后重启，将原端口号8080修改为新设置，刷新网站

**端口被占用怎么办？？**

- `netstat -ano|findstr “80”`：查出占用80端口的应用
- `taskill /f /pid 4`：强制结束

* 杀不死则将端口号改成别的

## 四、Tomcat目录结构

### 1.conf目录下哪里可以修改核心配置？哪些核心配置可以修改？

![1658711422553](assets/1658711422553.png)

![1658711488957](assets/1658711488957.png)

关于以上的配置，我想提出**网站到底是如何访问的？**

![1658711570455](assets/1658711570455.png)

### 2.webapps目录下发布项目

放一个程序到`webapps`目录下

![1658711653125](assets/1658711653125.png)

配置`web.xml`文件

![1658711686938](assets/1658711686938.png)

```java
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1"
  metadata-complete="true">

</web-app>
```

重启页面访问，本机访问通过`http://localhost:8080/ddw/index.html`，别人访问通过`IP地址/ddw/index.html`













