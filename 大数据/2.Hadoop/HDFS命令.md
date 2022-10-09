# HDFS命令

## 一、概述

### 1.hadoop xxx

`hadoop fs`命令可用于其他文件系统，不止是hdfs文件系统内，也就是说该命令的使用范围更广，可以用于HDFS、Local FS等不同的文件系统

### 2.hdfs xxx

`hdfs dfs`命令只用于hdfs文件系统

## 二、命令

### 1.用户命令

`fsck`：磁盘检查

`dfs`：这个命令很重要，单独讲

`version`：版本

`classpath`：类路径

`jar`

`job`：运行mapreduce任务

### 2.管理命令

`namenode -format`：格式化namenode

`secondarynamenode`：运行secondarynamenode

`namenode`：运行namenode

`journalnode`：运行journalnode

`zkfc`：故障迁移

`dfsadmin`：登录DFS客户端

`datanode`：运行datanode

`haadmin`：登录DFS HA客户端

`balancer`：run a cluster balancing utility

`daemonlog`

`jobtracker`

### 3.其他运维命令

不重要，先不展示了，需要思维导图内有，也可以百度

## 三、用户命令之dfs

### 1.查看文件结构

没有cd命令：`hdfs dfs -cd /`

![1665295015805](assets\1665295015805.png)

查看hdfs根目录下文件：`hdfs dfs -ls /`

![1665295056115](assets\1665295056115.png)

查看hdfs某个目录下的**所有**文件：`hdfs dfs -ls -R /`

![1665295087238](assets\1665295087238.png)

### 2.创建文件

在根目录下创建文件：`hdfs dfs -mkdir /test`

![1665295121229](assets\1665295121229.png)

递归创建多级目录：`hdfs dfs -mkdir -p /test/test1/…`

![1665295170503](assets\1665295170503.png)

### 3.移动文件

本地文件移动上传hdfs某个目录：`hdfs dfs -moveFromLocal /本地文件 /hdfs文件`

![1665295211002](assets\1665295211002.png)

hdfs文件移动到本地：`hdfs dfs -moveToLocal /hdfs文件/ /本地目录/`

hdfs内部进行文件移动：`hdfs dfs -mv /hdfs文件1 /hdfs文件2`

![1665295262521](assets\1665295262521.png)

### 4.上传文件

本地文件放到hdfs某个目录：`hdfs dfs -put /本地文件/ /hdfs目录/`

![1665295312484](assets\1665295312484.png)

### 5.下载文件

将hdfs文件下载到linux本地：`hadoop dfs -get /hdfs文件/`

### 6.文件内容

> 如果数据量比较大，不能使用

查看hello.txt文件内容：`hdfs dfs -cat /test/hello.txt`

![1665295389474](assets\1665295389474.png)

查看文件末尾，一直等待查看：`hdfs dfs -tail -f /path`

查看文件的大小：`hdfs dfs -du -h /path`

追加一个或者多个文件到hdfs指定文件中：`hdfs dfs -appendToFile /test/aa.txt /bb.txt /test/hello.txt`

![1665295433340](assets\1665295433340.png)

### 7.复制文件

本地文件复制到hdfs某个目录：`hdfs dfs -copyFromLocal /本地文件 /hdfs文件`

hdfs文件复制到本地：`hdfs dfs -copyToLocal /hdfs文件/ /本地目录/`

hdfs间文件拷贝，可以覆盖，可以保留原有权限信息

![1665295676997](assets\1665295676997.png)

### 8.删除文件

删除文件或者目录：`hdfs dfs -rmr /test/a`

![1665295709496](assets\1665295709496.png)

### 9.安全模式

查看当前hadoop安全模式的开关状态：`hdfs dfsadmin -safemode get`

打开安全模式：`hdfs dfsadmin -safemode enter`

离开安全模式：`hdfs dfsadmin -safemode leave`









