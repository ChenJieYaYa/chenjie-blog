Azkaban

## 一、Azkaban是什么？

Azkaban**是批量工作流任务调度器**，主要适用于在工作流中以特定顺序运行某组工作的情景，**解决任务单元之间的依赖关系和定时执行问题**

* 依赖关系：完整的数据分析系统通常由大量任务单元组成，例如Shell脚本、Java程序、MapReduce程序、Hive脚本等，各个任务单元之间存在时间先后依赖关系，为了很好的组织这些任务单元，需要工作流调度系统来调度这些任务单元

  ![1665467704413](assets\1665467704413.png)

* 定时调度：每个任务单元什么时候开始需要人工开启并监视任务进度，但大多数数据分析任务都在半夜执行，所以需要定时任务来定时调度这些任务单元

Azkaban的配置通过简单的键值对的方式，Dependencies设置依赖关系，Job配置文件建立任务之间的依赖关系，并提供易于使用的web用户界面维护和跟踪你的工作流

## 二、Azkaban的特点

①Azkaban兼容任何版本的Hadoop

②Azkaban有Web用户界面，方便简单傻瓜化操作

③Azkaban有模块化和可插拔的插件机制

④Azkaban有认证/授权，权限相关工作

⑤Azkaban能够杀死并重新启动工作流

⑥Azkaban有有关失败和成功的电子邮件提醒

## 三、Ooize&Azkaban

![1665467671529](assets\1665467671529.png)

## 四、Azkaban的架构

![1665467823140](assets\1665467823140.png)

**AzkabanWebServer**是整个Azkaban工作流系统的主要管理者，它有用户登录认证、项目管理、定时执行工作流、跟踪工作流执行进度等功能

**AzkabanExecutorServer**负责具体的工作流的提交、执行，它们通过MySQL数据库来协调任务的执行

关系型数据库(MySQL)存储大部分执行流状态，AzkabanWebServer和AzkabanExecutorServer都需要访问数据库

## 五、Azkaban安装

### 1.准备工作

请前往[官网](http://azkaban.github.io/downloads.html)下载压缩文件，将依赖的JAR放入到`/usr/lcoal/azkabanjar`目录

![1665468743886](assets\1665468743886.png)

解压重命名相关文件

```java
tar -xvf azkaban-web-server-2.5.0.tar.gz -C /usr/local/azkaban/
tar -xvf azkaban-executor-server-2.5.0.tar.gz -C /usr/local/azkaban/
tar -xvf azkaban-sql-script-2.5.0.tar.gz -C /usr/local/azkaban/

mv azkaban-web-2.5.0/ web
mv azkaban-executor-2.5.0/ executor
```

进入MySQL创建Azkaban数据库，并将解压后的脚本文件导入到该库

```java
mysql -uroot -p
create database azkaban;
use azkaban;
source /usr/local/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql
```

### 2.生成密钥库

生成keystore的密码及相应信息的密钥库

```java
cd /usr/local/azkaban
keytool -keystore keystore -alias jetty -genkey -keyalg RSA
    /*Keytool是java数据证书的管理工具，使用户能够管理自己的公/私钥对及相关证书
        -keystore：指定密钥库的名称及位置(产生的各类信息将不在.keystore文件中)
        -genkey：在用户主目录中创建一个默认文件".keystore" 
        -alias：对我们生成的.keystore 进行指认别名；如果没有默认是mykey
        -keyalg：指定密钥的算法 RSA/DSA 默认是DSA*/
```

密钥库的密码至少必须6个字符，可以是纯数字或者字母或者数字和字母的组合等等，密钥库的密码最好和jetty的密钥相同，方便记忆

```java
[root@node1 azkaban]# keytool -keystore keystore -alias jetty -genkey -keyalg RSA
输入密钥库口令:  
再次输入新口令: 
您的名字与姓氏是什么?
  [Unknown]:  
您的组织单位名称是什么?
  [Unknown]:  
您的组织名称是什么?
  [Unknown]:  
您所在的城市或区域名称是什么?
  [Unknown]:  
您所在的省/市/自治区名称是什么?
  [Unknown]:  
该单位的双字母国家/地区代码是什么?
  [Unknown]:  
CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown是否正确?
  [否]:  y

输入 <jetty> 的密钥口令
        (如果和密钥库口令相同, 按回车):  
再次输入新口令:
```

将keystore拷贝到azkaban web服务器根目录中

```java
mv keystore web/
```

### 3.时间同步

![1665240201418](https://chenjieyaya.github.io/chenjie-blog/%E5%A4%A7%E6%95%B0%E6%8D%AE/2.Hadoop/assets/1665240201418.png)

![1665240206289](https://chenjieyaya.github.io/chenjie-blog/%E5%A4%A7%E6%95%B0%E6%8D%AE/2.Hadoop/assets/1665240206289.png)

### 4.替换Mysql驱动

Azkaban2.5默认使用MySQL5.1驱动，我这里使用的是MySQL8.0，所以要更换驱动

![1665469454634](assets\1665469454634.png)

![1665469474952](assets\1665469474952.png)

### 5.Web Server配置

#### 5.1.azkaban.properties配置文件

```java
cd /usr/local/azkaban/web/conf
vim azkaban.properties
    #Azkaban Personalization Settings
    #服务器UI名称,用于服务器上方显示的名字
    azkaban.name=Test
    #描述
    azkaban.label=My Local Azkaban
    #UI颜色
    azkaban.color=#FF3601
    azkaban.default.servlet.path=/index
    #默认web server存放web文件的目录
    web.resource.dir=/usr/local/azkaban/web/web/
    #默认时区,已改为亚洲/上海 默认为美国
    default.timezone.id=Asia/Shanghai

    #Azkaban UserManager class
    user.manager.class=azkaban.user.XmlUserManager
    #用户权限管理默认类（绝对路径）
    user.manager.xml.file=/usr/local/azkaban/web/conf/azkaban-users.xml

    #Loader for projects
    #global配置文件所在位置（绝对路径）
    executor.global.properties=/usr/local/azkaban/executor/conf/global.properties
    azkaban.project.dir=projects

    #数据库类型
    database.type=mysql
    #端口号
    mysql.port=3306
    #数据库连接IP
    mysql.host=node3
    #数据库实例名
    mysql.database=azkaban?useSSL=false&serverTimezone=UTC&characterEncoding=utf-8
    #数据库用户名
    mysql.user=root
    #数据库密码
    mysql.password=a
    #最大连接数
    mysql.numconnections=100

    # Velocity dev mode
    velocity.dev.mode=false

    # Azkaban Jetty server properties.
    # Jetty服务器属性.
    #最大线程数
    jetty.maxThreads=25
    #Jetty SSL端口
    jetty.ssl.port=8443
    #Jetty端口
    jetty.port=8081
    #SSL文件名（绝对路径）
    jetty.keystore=/usr/local/azkaban/web/keystore
    #SSL文件密码
    jetty.password=xbzz7789
    #Jetty主密码与keystore文件相同
    jetty.keypassword=xbzz7789
    #SSL文件名（绝对路径）
    jetty.truststore=/usr/local/azkaban/web/keystore
    #SSL文件密码
    jetty.trustpassword=xbzz7789

    # Azkaban Executor settings
    executor.port=12321

    # mail settings
    mail.sender=
    mail.host=
    job.failure.email=
    job.success.email=

    lockdown.create.projects=false

    cache.directory=cache
```

#### 5.2.azkaban-users.xml配置文件

增加管理员用户，配置两种角色admin和metrics

```java
cd /usr/local/azkaban/web/conf
vim azkaban-users.xml
    <azkaban-users>
        <user username="azkaban" password="azkaban" roles="admin" groups="azkaban" />
        <user username="metrics" password="metrics" roles="metrics"/>
        <user username="admin" password="admin" roles="admin,metrics" />
        <role name="admin" permissions="ADMIN" />
        <role name="metrics" permissions="METRICS"/>
    </azkaban-users>
```

### 6.Executor Server配置

azkaban.properties配置文件

```java
cd /usr/local/azkaban/executor/conf
vim azkaban.properties
    #Azkaban
    #时区
    default.timezone.id=Asia/Shanghai

    # Azkaban JobTypes Plugins
    #jobtype 插件所在位置
    azkaban.jobtype.plugin.dir=plugins/jobtypes

    #Loader for projects
    executor.global.properties=/usr/local/azkaban/executor/conf/global.properties
    azkaban.project.dir=projects

    database.type=mysql
    mysql.port=3306
    mysql.host=node3
    mysql.database=azkaban?useSSL=false&serverTimezone=UTC&characterEncoding=utf-8
    mysql.user=root
    mysql.password=a
    mysql.numconnections=100

    # Azkaban Executor settings
    #最大线程数
    executor.maxThreads=50
    #端口号(如修改,请与web服务中一致)
    executor.port=12321
    #线程数
    executor.flow.threads=30
```

### 7.启动服务

先执行Executor，再执行Web，避免Web Server会因为找不到执行器启动失败

```java
cd /usr/local/azkaban/executor/bin
./azkaban-executor-start.sh

cd /usr/local/azkaban/web/bin
./azkaban-web-start.sh
```

### 8.查看启动情况

![在这里插入图片描述](assets\15eb7d69b1404473bf4794af50bdb87b.png)

### 9.Web Server访问

![在这里插入图片描述](assets\04ab66a11f7848e085759090fea80d0e.png)

![在这里插入图片描述](assets\ec1b479334bf47b7b7f3fe617eafe1fd.png)

## 六、Azkaban案例

### 1.单个Job

①创建Job文件

```java
vim job1.job
    #first.job
    type=command
    command=echo 'this is my first job'
```

②打包Job文件成zip，Azkaban上传的工作流文件只支持xxx.zip文件，xxx.zip文件应包含运行作业所需的文件xxx.job

* 安装zip,unzip命令：`yum install -y unzip zip`
* 压缩：`zip job1.zip job1.job`

③通过Web Server创建项目并上传Job

![在这里插入图片描述](assets\6b7af6f221054a1a8dde9aef551e77db.png)

![在这里插入图片描述](assets\ea0b8e6b81f4499693d5b414d8349e7e.png)

④启动上传的Job

![1665470250286](assets\1665470250286.png)

![1665470262746](assets\1665470262746.png)

![1665470271201](assets\1665470271201.png)

⑤Job执行成功

![1665470305728](assets\1665470305728.png)

⑥点击查看Job日志

![1665470335868](assets\1665470335868.png)

### 2.多个Job

①创建Job文件

* `start.job`

```java
vim start.job
	#start.job
	type=command
	command=touch /usr/local/azkaban/jobs/t2test.txt
```

* `step1.job`依赖`start.job`

```java
vim step1.job
	#step1.job
	type=command
	dependencies=start
	command=echo "this is step1 job"
```

* `step2.job`依赖`start.job`

```java
vim step2.job
	#step2.job
	type=command
	dependencies=start
	command=echo "this is step2 job"
```

* `finish.job`依赖`step1.job`和`step2.job`

```java
vim finish.job
	#finish.job
	type=command
	dependencies=step1,step2
	command=echo "this is finish job"
```

②打包Job文件成zip：`zip jobs.zip start.job step1.job step2.job finish.job`

③通过Web Server创建项目并上传Job同时启动

![1665470544280](assets\1665470544280.png)

④点击查看Job日志

![1665470594286](assets\1665470594286.png)

### 3.Java&Job

①编写Java程序

```java
import java.io.FileOutputStream;
import java.io.IOException;

public class AzkabanTest {
	public void run() throws IOException {
		FileOutputStream fos = new FileOutputStream("/usr/local/azkaban/t3test.txt");
		fos.write("this is a java progress".getBytes());
		fos.close();
    }

	public static void main(String[] args) throws IOException {
		AzkabanTest azkabanTest = new AzkabanTest();
		azkabanTest.run();
	}
}
```

②Java程序打包并移入Linux

* 编译运行生成`.class`文件并打包`jar -cvf AzkabanTest.jar AzkabanTest.class`

![1665470742503](assets\1665470742503.png)

* 上传至Linux

![1665470773054](assets\1665470773054.png)

* 试运行`java -jar AzkabanTest.jar`

③创建Job文件

```java
vim java.job
    type=javaprocess
    java.class=AzkabanTest
    classpath=/usr/local/azkaban/jobs/t3/jar/*
```

④打包Job文件成zip：`zip java.zip java.job`

⑤通过Web Server创建项目并上传Job同时启动

![1665470904087](assets\1665470904087.png)

### 4.HDFS&Job

①创建Job文件

```java
vim fs.job
    type=command
    command=/usr/local/hadoop-2.7.1/bin/hadoop fs -mkdir /azkaban
```

②打包Job文件成zip：`zip fs.zip fs.job`

③通过Web Server创建项目并上传Job同时启动

![1665470967540](assets\1665470967540.png)

④HDFS中查看结果

![1665470992700](assets\1665470992700.png)

### 5.MapReduce&Job

①创建Job文件

```java
vim mapreduce.job
    #mapreduce job
    type=command
    command=/usr/local/hadoop-2.7.1/bin/hadoop jar /usr/local/hadoop-2.7.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar wordcount /wordcountinput /wordcountinputazkaban
```

②打包Job文件成zip：`zip mapreduce.zip mapreduce.job`

③通过Web Server创建项目并上传Job同时启动

![1665471080907](assets\1665471080907.png)

④HDFS中查看结果

![1665471092665](assets\1665471092665.png)















