# HTTP协议

## 一、什么是HTTP协议？

HTTP全称Hyper Text Transfer Protocol，译为**超文本传输协议**，是一种应用层协议，基于**TCP/IP协议**传输数据，即C-S之间需要经过三次握手四次挥手，主要解决如何包装数据的问题，主要功能是在服务器与客户端之间传输超文本，默认端口号80，具有以下特点

* **无状态**：同一个客户端发送的两次请求是无关联的，对于HTTP服务器来说，并不知道这两个请求来自同一个客户端，之后，为解决引入Cookie机制
* **无连接**：每次只能处理一个请求，即短链接，但HTTP2.0后支持长连接
* **任何类型的数据都可以通过HTTP发送**，Content-Type指定数据类型

**HTTP与HTTPS的区别？**

* HTTP是超文本传输协议，而HTTPS是以安全为目标的HTTP通道，具有安全性的SSL加密传输协议
* HTTP默认端口80，HTTPS默认端口443

## 二、HTTP报文

### 1.什么是HTTP报文？

HTTP报文是HTTP协议在客户端和服务端之间传送的**数据块**，**HTTP报文由起始行、头域、实体组成**，起始行是对报文的描述，头域包含报文的一些属性，实体就是报文的数据

### 2.HTTP报文分类

请求报文：`Client->Server`，由请求头、头域、实体组成，**请求头由请求方法、URI、协议版本组成**，例`GET http://www.baidu.com/index.html HTTP/1.0`

响应报文：`Server->Client`，由响应头、头域、实体组成，**响应头由协议版本、状态码、描述信息组成**

## 三、请求方法

### 1，常见请求方法

|  方法  |                   描述                   | 是否包含主体 |
| :----: | :--------------------------------------: | :----------: |
|  GET   |           从服务端获取指定数据           |      否      |
|  POST  |         向服务端发送待处理的数据         |      是      |
|  HEAD  |        从服务端获取指定信息的头部        |      否      |
|  PUT   | 向服务端发送数据并替换服务端上指定的数据 |      是      |
| DELETE |           从服务端删除指定数据           |      否      |
| TRACE  |    沿着目标资源的路径执行消息环回测试    |      否      |

### 2.Get与Post之间的区别？

* Get从服务器获取数据，Post向服务器发送数据
* Get不安全，因为URL公开，Post比Get更安全
* Get将字段通过`&`拼接与URL的`?`后，该过程对用户可见，Post则将字段封存于请求实体中发送给服务端，该过程对用户不可见
* Get传输量小，因为受URL长度限制，但高效，Post可传输大量数据，所以上传文件只可通过Post方式
* Get只支持ASCII字符，Post支持标准字符集，可以正确传递中文字符

## 四、状态码

| 整体范围 | 已定义范围 |    分类    |
| :------: | :--------: | :--------: |
| 100~199  |  100~101   |  信息提示  |
| 200~299  |  100~206   |    成功    |
| 300~399  |  300~305   |   重定向   |
| 400~499  |  400~415   | 客户端错误 |
| 500~599  |  500~505   | 服务端错误 |

## 五、头域

### 1.请求头域

|      Header       |             含义             |                             示例                             |
| :---------------: | :--------------------------: | :----------------------------------------------------------: |
|      Accept       |      浏览器可接收的类型      |           Accept: text/html,application/xhtml+xml            |
|  Accept-Charset   |   浏览器可接收的字符编码集   |                     Accept-Charset: GBK                      |
|  Accept-Encoding  |  浏览器可接收的压缩编码类型  |                Accept-Encoding: gzip, deflate                |
|  Accept-Language  | 浏览器可接收的语言和国家类型 |                  Accept-Language: zh-CN,zh                   |
|       Host        |    浏览器请求的主机和端口    |                     Host: www.lks.cn:80                      |
|      Referer      |       请求来自哪个页面       |                Referer: http://onemore.study                 |
|    User-Agent     |        浏览器相关信息        | User-Agent: Mozilla/5.0 (Windows NT 6.2; Win64; x64; rv:65.0) |
| If-Modified-Since |      某个页面的缓存时间      |       If-Modified-Since: Mon, 16 Mar 2020 11:11:11 GMT       |
|      Cookie       |  浏览器暂存服务器发送的信息  |                   Cookie: onemore=呃呃呃;                    |
|    Connection     |           连接方式           |                    Connection: keep-alive                    |
|       Date        |     请求网站的日期和时间     |              Date: Tue, 11 Jul 2000 18:23:51GMT              |
|       Allow       |          请求的方法          |                       Allow: GET, POST                       |
|    Keep-Alive     |          长连接时间          |                        Keep-Alive：5                         |
|   Cache-Control   |      请求遵循的缓存机制      |                   Cache-Control: no-cache                    |

### 2.响应头域

|       Header        |                  含义                  |                       示例                        |
| :-----------------: | :------------------------------------: | :-----------------------------------------------: |
|      Location       |         控制浏览器显示哪个页面         |      Location: http://www.lks.cn/index.html       |
|       Server        |               服务器类型               |                Server:apache nginx                |
|  Content-Encoding   |        服务器发送的压缩编码方式        |              Content-Encoding: gzip               |
|   Content-Length    |          服务器发送的内容长度          |               Content-Length: 1024                |
|  Content-Language   |      服务器发送的内容的语言和国家      |            Content-Language: zh-CN,zh             |
|    Content-Type     |     服务器发送的内容类型和编码类型     |      Content-Type: image/jpeg; charset=UTF-8      |
|    Last-Modified    |          服务器最后修改的时间          |   Last-Modified: Mon, 16 Mar 2020 11:11:11 GMT    |
|       Refresh       |      控制浏览器1s后转发到指定页面      |         Refresh: 1;url=http://www.lks.cn          |
| Content-Disposition | 服务器控制浏览器方以下载的方式打开文件 | Content-Disposition: attachment; filename=lks.jpg |
|  Transfer-Encoding  |       服务器分发数据传递到客户端       |             Transfer-Encoding:chunked             |
|       Expires       |          响应过期的日期和时间          |      Expires: Mon, 16 Mar 2020 11:11:11 GMT       |
|    Cache-Control    |           响应遵循的缓存机制           |              Cache-Control: no-cache              |
|       Pragma        |      服务器控制浏览器不要缓存网页      |                 Pragma: no-cache                  |
|     Set-Cookie      |       服务器发送Cookie相关的信息       |            Set-Cookie: onemore=呃呃呃;            |
|     Connection      |                连接方式                |              Connection: keep-alive               |
|        Date         |          响应网站的日期和时间          |        Date: Tue, 11 Jul 2000 18:23:51GMT         |
|        ETag         |             资源实体的标识             |     ETag: "306073f04224cbd114f14693c272f6a0"      |

## 六、协议案例

### 1.需求

从`https://pm.myapp.com/invc/xfspeed/qqpcmgr/download/QQPCDownload1530.exe`下载指定文件，获取文件大小，并在本地用户目录下创建相同大小的空文件，以时间作为文件名，后缀名就为原来下载的文件的后缀名

### 2.代码

```java
public class t1_DownLoadPrepare {
    public static void main(String[] args) throws Exception {
        String url = "https://pm.myapp.com/invc/xfspeed/qqpcmgr/download/QQPCDownload1530.exe";

        URL u = new URL(url);
        HttpURLConnection con = (HttpURLConnection) u.openConnection();

        //请求头Head
        con.setRequestMethod("HEAD");
        //设置超时时间
        con.setConnectTimeout(3000);
        //连接
        con.connect();

        //文件大小
        long fileSize = con.getContentLength();
        //文件名
        Date d = new Date();
        SimpleDateFormat s = new SimpleDateFormat("yyyyMMddHHmmss");
        String fileName = s.format(d);
        //后缀名
        String suffix = url.substring(url.lastIndexOf("."));
        String newFileName = fileName + suffix;

        //在用户(有权限)目录中创建
        String userHome = System.getProperty("user.home");
        //File f = new File(userHome, newFileName);//路径 名字
        /*if (f.exists() == false) {
            file.setLength(fileSize);
            file.close();
            System.out.println("文件已创建");
        }*/

        //建一个有大小的空文件
        //File.separator == /
        RandomAccessFile file = new RandomAccessFile(userHome + File.separator + newFileName, "rw");
        file.setLength(fileSize);
        file.close();
        System.out.println("文件已创建:" + file.getFD());
    }
}

```



```
普通post请求参数在实体中：name=a&pwd=aa
html===============================
<!--方法必须为post  加上参数enctype-->
<form action="xx" method="post" enctype="multipart/form-data">
	头像：<input type="file" name="head" /><br />
	名称：<input type="text" name="name" /><br />
	<input type="submit" />
</form>
==============================================================
此时不用&分割，用分割线分割

-----------------------------121066605317290619602852684163
Content-Disposition: form-data; name="head"; filename="1.png"
Content-Type: image/png

‰PNG（魔术字）
skdjfldsjfdskfj
-----------------------------121066605317290619602852684163
Content-Disposition: form-data; name="uname"

æ£®æž— 
-----------------------------121066605317290619602852684163--
==============================================================
以魔术字来判断文件类型，而不是文件后缀名
不能再用request.getParameter("username");接收参数
	
```

