# Java入门

## 一、CMD

### 1.人机交互

现代计算机大多使用图形画界面，鼠标双击即可打开相应的软件；但是最初电脑是通过命令行的方式操作的，那么图形画界面由谁创造呢？

![1661753812643](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661753812643.png)

图形化界面交互感强，但是起运行速度相对较慢，且需要加载更多图片等，消耗内存

在Windows中，可通过CMD使用命令行的方式操作计算机

### 2.打开CMD

`Start+R`，输入`cmd`，后回车，弹出的小黑窗口就是CMD窗口

![1661744596637](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661744596637.png)

### 3.常见CMD命令

盘符切换：`盘符名称 :`

![1661754213846](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754213846.png)

列出当前目录下的文件及文件夹：`dir`

![1661754301367](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754301367.png)

进入指定目录：`cd xxx/yyy`	

![1661754548969](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754548969.png)

退回上级目录：`cd ..`

![1661754609952](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754609952.png)

退回当前目录：`cd .`

![1661754679256](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754679256.png)

退回根目录：`cd /`

![1661754727377](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754727377.png)

创建目录：`md xxx`

![1661754872010](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754872010.png)

删除目录：`rd xxx`

![1661754918867](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661754918867.png)

清屏：`cls`

退出CMD：`exit`

### 4.练习

#### 4.1.利用CMD打开QQ

①找到QQ启动程序所在位置

![1661755451224](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661755451224.png)

②`Win+R`打开CMD命令行

![1661755182099](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661755182099.png)

③切换盘符后，进入`QQ.exe`所在路径

![1661755694584](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661755694584.png)

#### 4.2.实现任意目录下输入QQ.exe都能启动QQ

QQ是频繁使用的软件，每次打开都需要切换盘符、进入多层目录，太麻烦了，但是在其他目录下输入`QQ.exe`会报错，因为其他路径下并没有`QQ.exe`文件，那如何解决？

![1661756015238](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661756015238.png)

**只需要将QQ的路径记在电脑的某个位置，即环境变量**，当输入`QQ.exe`时，首先在当前路径下找，没找到则到环境变量中找，环境变量具体如何配置？请参考JDK的配置

## 二、HelloJava

### 1.JDK下载

> 下载建议：最好将所有开发工具放在同一文件夹下

下载地址：[https://www.oracle.com/](https://www.oracle.com/)

![1661757990964](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661757990964.png)

![1661758200905](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661758200905.png)

![1661758392680](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661758392680.png)

![1661758416995](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661758416995.png)

![1661758760652](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661758760652.png)

### 2.JDK安装

> 安装建议：安装路径最好不要有中文

双夹下载好的安装包开始安装

![1661761998776](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761998776.png)

![1661762261435](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661762261435.png)

![1661762080565](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661762080565.png)

![1661762153552](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661762153552.png)

### 3.JDK目录

|  目录   |               说明                |
| :-----: | :-------------------------------: |
| **bin** | **可执行文件，如`javac`、`java`** |
| include |            包含C头文件            |
|   jre   |          Java运行时环境           |
|   lib   |             Java类库              |
| src.zip |             资源文件              |

### 4.配置环境变量

CMD练习中，曾经就QQ.exe无法在其他路径下运行提出过解决方案，即配置环境变量，那么Java可执行文件也是需要频繁使用的，故最好也是配上环境变量

> 为什么要配置环境变量？因为想要在任意目录下都可以打开指定软件

①右键[此电脑]，点击[属性]，点击[高级系统设置]

![1661765013292](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661765013292.png)

②新建`JAVA_HOME`

![1661765241512](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661765241512.png)

③双击[Path]

![1661765485790](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661765485790.png)

④点击[确定]

### 5.HelloJava

①用记事本编写程序`HelloJava.java`

![1661759849178](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661759849178.png)

```java
class HelloJava{//Java程序的最基本单位是类，所以我们要定义一个类，要求文件名和类名需要一致
	public static void main(String [] args){//Java程序要想执行，必须有main方法
		System.out.println("Hello Java");//做一个简单的输出
	 }
}
```

> 后缀名隐藏的修改办法
>
> ![1661759945501](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661759945501.png)

> 使用记事本看代码不太方便，可下载高级记事本`Notepad++`打开Java代码，一些相关设置请自行百度

②编译程序：`javac 文件名.java`

![1661765851144](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661765851144.png)

③运行程序：`java 文件名`

![1661765902758](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661765902758.png)

> 编译和运行如何理解？首先Windows不认识Java代码，从而无法运行Java代码，那怎么办？
>
> ![1661759640907](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661759640907.png)
>
> 既然操作系统不认识Java代码，那么首先就将Java代码翻译成操作系统能看得懂的内容，即编译
>
> ![1661759742987](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661759742987.png)
>
> 运行的是翻译以后的文件

> 学以致用：每次编译运行Java程序都需要进入指定目录，太麻烦，怎么解决呢？相信你已经想到了解决方案，即配置环境变量

## 三、Java概述

### 1.Java版本

#### 1.1.Java SE

**Java语言的标准版**，是其他两个版本的基础，**用于桌面应用的开发**，桌面应用指的是打开电脑就使用的应用，如计算器、蜘蛛纸牌、扫雷等，但Java开发桌面应用并不占优势，较合适的语言为C、C++，那么为什么还需要学习呢？打基础！培养逻辑思维！

#### 1.2.Java ME

**Java语言的小型版**，**用于嵌入式电子设备或者小型移动设备**，嵌入式电子设备指的是嵌入到打印机、电视、相机等设备的系统，小型移动设备指手机，Java ME逐渐被Android、IOS取代，市面上的招聘岗位也较少

#### 1.3.Java EE

**Java语言的企业版**，**用于Web方向的网站开发**(No1)，网站开发包括浏览器+服务器

### 2.Java作用

![1661761265518](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761265518.png)

### 3.Java为什么火？

> 语言火不火请看：用户量、适用范围、与时俱进、自身明显特点

#### 3.1.用户量

![1661761480078](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761480078.png)

#### 3.2.适用范围

![1661761265518](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761265518.png)

#### 3.3.与时俱进

![1661761533655](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761533655.png)

#### 3.4.自身特点

面向对象、安全性(Java写出来的代码安全，漏洞较少)、多线程(同时做多件事)、简单易用、开源、**跨平台(跨OS)**

![1661761872899](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661761872899.png)

> 现在不理解没关系！请具体学习过后再来体会

### 4.Java跨平台

#### 4.1.高级语言的执行过程

**编程**：编写相应语言的代码，即`.java`、`.c`、`.py`

**编译**：机器只认识机器语言，该过程将语言代码翻译成机器认识的机器语言

**运行**：机器执行编译后的指令

#### 4.2.高级语言的编译运行方式

**编译型**：**通过编译器一次性将源代码翻译成机器码**，其执行效率高，开发效率低，每个平台不同，那么编译后的指令也可能不同，所以导致指令无法复用，对于不同OS都要重新编译；**编译型语言会产生编译后的文件，将该文件交给不同的平台运行**

![1661766847643](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661766847643.png)

**解释型**：**通过解释器逐行的解释源代码为机器码后执行**，执行速度慢，开发效率高；**解释型语言不会产生其他文件，其天生开源，将源代码发送给不同平台，然后逐行解释运行**

![1661767083043](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661767083043.png)

**混合型(半编译半解释)**：通过编译器使Java代码整体生成字节码文件，字节码文件发送到不同机器，通过解释器逐行解释字节码文件，然后在虚拟机中执行

![1661767392930](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661767392930.png)

> 字节码是JVM可理解的代码，后缀为`class`的文件；字节码只面向JVM，不面向特定的处理机，即在一定程度上解决解释型语言效率低的问题，又保留解释型语言可移植的特点，由于字节码不面向特定虚拟机，所以无需重新编译可在不同OS计算机上运行

#### 4.3.Java虚拟机

实际上解释后的字节码并不是直接运行于机器电脑上，而是虚拟机中，**实现跨平台的关键就是针对不同的平台提供不同的虚拟机**，虚拟机Java已经实现好，只需我们在不同平台安装相应虚拟机即可

![1661767728617](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661767728617.png)

只需要满足JVM规范可自定义JVM，HotSpot VM是JVM的实现

### 5.JVM&JRE&JDK

通过以上的学习，请思考![img](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\4827431.PNG)我们的程序从开发到运行需要使用到哪些工具呢？？

* 首先是**JVM(Java虚拟机)**，运行Java程序的地方
* 其次是**Java核心类库**，即Java已经写好的内容，如`Sysstem.out.println`
* 最后是**Java开发工具**，如`javac`、`java`

以上三部分实际就组成了JDK(Java Development Kit)，即Java开发工具包

![1661768279634](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661768279634.png)

------

现在我只需要运行Java程序，那么想想我还需要安装一个完整的JDK吗？如果不需要去掉那个部分呢？

* 无需完整JDK，但安装完整JDK也没问题，只是JDK内有的东西用不到，会占用空间
* 注意是运行Java程序，那么当然是不在需要开发工具，只保留运行工具，如`java`，而无需`javac`

所以JDK去掉部分开发工具后就组成了JRE(Java Runtime Environment)，即Java的运行环境

![1661768563957](C:\Users\cloud\Desktop\我的\chenjie-blog\2.Java基础\assets\1661768563957.png)

------

**JDK > JRE > JVM**













