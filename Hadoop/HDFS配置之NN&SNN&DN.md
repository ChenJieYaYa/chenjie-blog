# HDFS配置之NN&SNN&DN

## 一、配置图解

![1665221758174](assets/1665221758174.png)

## 二、创建并配置虚拟机

下载[CentOS7](https://pan.baidu.com/share/init?surl=W5OsfNnXyEEc_t6Styq2ww)，提取码5l27

### 1.创建虚拟机

![1665237976462](assets/1665237976462.png)

![1665237983505](assets/1665237983505.png)

![1665237988812](assets/1665237988812.png)

![1665237992868](assets/1665237992868.png)

![1665237998913](assets/1665237998913.png)

![1665238003693](assets/1665238003693.png)

![1665238009429](assets/1665238009429.png)

![1665238014302](assets/1665238014302.png)

![1665238019271](assets/1665238019271.png)

![1665238023217](assets/1665238023217.png)

![1665238028079](assets/1665238028079.png)

![1665238033084](assets/1665238033084.png)

![1665238038637](assets/1665238038637.png)

![1665238044750](assets/1665238044750.png)

![1665238125480](assets/1665238125480.png)

![1665238131366](assets/1665238131366.png)

![1665238135691](assets/1665238135691.png)

![1665238139499](assets/1665238139499.png)

![1665238143629](assets/1665238143629.png)

![1665238154807](assets/1665238154807.png)

![1665238159038](assets/1665238159038.png)

![1665238163377](assets/1665238163377.png)

![1665238167217](assets/1665238167217.png)

![1665238171777](assets/1665238171777.png)

![1665238177166](assets/1665238177166.png)

![1665238182295](assets/1665238182295.png)

![1665238186919](assets/1665238186919.png)

![1665238192267](assets/1665238192267.png)

![1665238196629](assets/1665238196629.png)

![1665238203039](assets/1665238203039.png)

![1665238208802](assets/1665238208802.png)

![1665238213822](assets/1665238213822.png)

![1665238217751](assets/1665238217751.png)

![1665238222596](assets/1665238222596.png)

![1665238228763](assets/1665238228763.png)

![1665238233061](assets/1665238233061.png)

![1665238237722](assets/1665238237722.png)

![1665238242912](assets/1665238242912.png)

### 2.打开终端

也可以切换成[命令行界面](https://blog.csdn.net/zhangyingchengqi/article/details/103559276)，命令行效率会更高，不切也没关系

![1665238384798](assets/1665238384798.png)

### 3.配置虚拟机启用网卡，并设置固定IP地址

#### 3.1.配置原理

配置为固定IP后，不管什么情况下不受虚拟机影响，只要笔记本主机可以正常上网，那么启动虚拟机中的CentOS 7系统就可以正常访问外网，无需再进行任何设置

![1665238680619](assets/1665238680619.png)

#### 3.2.设置虚拟机IP

①设置虚拟机的网络连接方式

![1665238735609](assets/1665238735609.png)

②这一步部分计算机不需要设置

  ![1665238816933](assets/1665238816933.png)

③修改子网IP，实现自由设定固定IP，**1网段无法成功(第三位IP段)**

![1665238823364](assets/1665238823364.png)

![1665238948465](assets/1665238948465.png)

#### 3.3.配置笔记本主机具体VMnet8本地地址参数

![1665238993967](assets/1665238993967.png)

![1665238998864](assets/1665238998864.png)

![1665239005740](assets/1665239005740.png)

#### 3.4.修改虚拟机中的CentOS 7系统为固定IP的配置文件

①输入`su root`指令切换到root用户

②ifcfg-ens33是虚拟机的网卡，配置连接VMnet8的网关，通过网关连接主机的网卡

![1665239113010](assets/1665239113010.png)

![1665239156440](assets/1665239156440.png)

```java
cd /etc/sysconfig/network-scripts
vim ifcfg-ens33	//i编辑，esc退出编辑，:wq退出保存
	DNS1=114.114.114.114
	IPADDR=192.168.10.100
	NETMASK=255.255.255.0
	GATEWAY=192.168.10.2                    
service network restart
```

#### 3.5.检验配置是否成功

①查看修改后的固定IP

![1665239264111](assets/1665239264111.png)

②测试虚拟机中的CentOS 7系统是否能连外网

![1665239289727](assets/1665239289727.png)

③测试本机是否能ping通虚拟机的固定IP

![1665239295403](assets/1665239295403.png)

### 4.修改主机名

![1665239336663](assets/1665239336663.png)

![1665239340940](assets/1665239340940.png)

### 5.在CentOS中安装sftp服务

**sftp安全文件传送协议**是传输文件的安全网络加密方法，为了上传文件到虚拟机系统

①确认已安装好ssh服务

![1665239437171](assets/1665239437171.png)

②配置sftp

![1665239443208](assets/1665239443208.png)

![1665239478187](assets/1665239478187.png)

![1665239447235](assets/1665239447235.png)

```java
Match Group sftp
ChrootDirectory /usr/sftp
ForceCommand internal-sftp
AllowTcpForwarding no
X11Forwarding no
```

### 6.检测是否配置成功

![1665239482213](assets/1665239482213.png)

![1665239571889](assets/1665239571889.png)

![1665239577296](assets/1665239577296.png)

![1665239581194](assets/1665239581194.png)

### 7.关闭防火墙

取消开机自启动防火墙`systemctl disable firewalld.service`

查看防火墙状态`systemctl status firewalld`

## 二、完全分步式安装步骤

### 1.时间同步

#### 1.1.配置

![1665240201418](assets/1665240201418.png)

![1665240206289](assets/1665240206289.png)

#### 1.2.问题

![1665240247496](assets/1665240247496.png)

![1665240252903](assets/1665240252903.png)

### 2.安装JDK

#### 2.1.删除CentOS自带openjdk

![1665240302820](assets/1665240302820.png)

```java
rpm -qa | grep jdk
yum -y remove 
```

![1665240307150](assets/1665240307150.png)

#### 2.2.FinalShell连接虚拟机

![1665240355215](assets/1665240355215.png)

![1665240359415](assets/1665240359415.png)

#### 2.3.创建目录，上传JDK到Linux并解压

![1665240399250](assets/1665240399250.png)

![1665240421770](assets/1665240421770.png)

![1665240426243](assets/1665240426243.png)

若解压后的文件名字太长，最好**重命名**，自己百度以下相关命令

#### 2.4.配置环境变量

![1665240473260](assets/1665240473260.png)

![1665240477354](assets/1665240477354.png)

```java
JAVA_HOME=/usr/java/jdk1.8.0_151
JRE_HOME=/usr/java/jdk1.8.0_151/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin 
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib 
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

### 3.配置主机地址映射文件

#### 2.1.配置CentOS7中的/etc/hosts

![1665240596486](assets/1665240596486.png)

![1665240600473](assets/1665240600473.png)

```java
192.168.10.100 node1 master nn
192.168.10.101 node2 snn dn1
192.168.10.102 node3 dn2
192.168.10.103 node4 dn3
```

#### 2.2.配置windows中的C:\Windows\System32\drivers\etc\hosts

![1665240657356](assets/1665240657356.png)

### 4.克隆虚拟机

![1665240717779](assets/1665240717779.png)

![1665240722592](assets/1665240722592.png)

![1665240726467](assets/1665240726467.png)

![1665240731337](assets/1665240731337.png)

![1665240735365](assets/1665240735365.png)

![1665240740218](assets/1665240740218.png)

![1665240744593](assets/1665240744593.png)

克隆好后主机名和固定IP等是需要修改的，具体修改哪些我也忘记统计，可以自己试一试，若发现已经配置过了，直接跳过即可

出现克隆后无法同时开启四台虚拟机的问题

![1665240865711](assets/1665240865711.png)

![1665240869279](assets/1665240869279.png)

### 5.免密钥设置

#### 5.1.分析

node1生成公私密钥对后将公钥发送给node2，node1登录node2时，node2将之前的公钥再发送回node1，node1使用之前生成的私钥进行解密，解密成功则可以免密登录

#### 5.2.node1免密登录node1

①生成公私密钥对，存储在`~/.ssh/id_rsa`目录下：`ssh-keygen -t rsa -P '' -f    ~/.ssh/id_rsa`

②将本机公钥加到受信列表中，即将公钥追加到服务端(远程主机)：`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

③设置权限：`chmod 0600 ~/.ssh/authorized_keys`

④测试是否可以免密登录

![1665241033714](assets/1665241033714.png)

#### 5.3.node1将公钥~/.ssh/id_rsa.pub写入其他节点的tmp临时文件下保存

`scp ~/.ssh/id_rsa.pub     node2:/tmp/`、`scp ~/.ssh/id_rsa.pub     node3:/tmp/`、`scp ~/.ssh/id_rsa.pub     node4:/tmp/`

![1665241087819](assets/1665241087819.png)

#### 5.4.其他节点生成公私密钥对，并将tmp下的公钥信息保存到~/.ssh/authorized_keys

①生成公私密钥对，存储在`~/.ssh/id_rsa`目录下：`ssh-keygen -t rsa -P '' -f    ~/.ssh/id_rsa`

②将本机公钥加到受信列表中，即将公钥追加到服务端(远程主机)：`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

③将需要免密登录本机的主机的公钥加到受信列表中：`cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys`

④设置权限：`chmod 0600 ~/.ssh/authorized_keys`

⑤测试是否可以免密登录

#### 5.5.测试node1是否可以免密登录其他节点

![1665241125398](assets/1665241125398.png)

### 6.上传解压Hadoop

![1665241241659](assets/1665241241659.png)

`cd /usr/local`、`tar -xvf 压缩文件名称`，这两步没有截图

### 7.修改配置

#### 7.1.配置Hadoop环境变量

![1665241283282](assets/1665241283282.png)

![1665241290584](assets/1665241290584.png)

```java
vim /etc/profile
	HADOOP_HOME=/usr/local/hadoop-2.7.1
	PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	export PATH JAVA_HOME CLASSPATH HADOOP_HOME
	--------------------------------------------
	以上的配置方法与jdk环境变量的配置优点冗余，可以采取以下这种
	JAVA_HOME=/usr/java/jdk1.8.0_151
	JRE_HOME=/usr/java/jdk1.8.0_151/jre
	HADOOP_HOME=/usr/local/hadoop-2.7.1
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib 
	export JAVA_HOME JRE_HOME PATH CLASSPATH HADOOP_HOME
source /etc/profile
```

#### 7.2.修改hadoop-2.7.1/etc/hadoop/hadoop-env.sh中的JAVA_HOME

![1665241346154](assets/1665241346154.png)

![1665241349418](assets/1665241349418.png)

#### 7.3.配置hadoop-2.7.1/etc/hadoop/core-site.xml

![1665241375284](assets/1665241375284.png)

![1665241378664](assets/1665241378664.png)

```java
<property> 
	<name>fs.defaultFS</name> 
	<value>hdfs://node1:9000/</value> 
</property> 
<property> 
	<name>hadoop.tmp.dir</name>
	<value>/opt/hadoopdata</value> 
</property> 
//注意hadoopdata目录不存在，由hadoop生成
```

#### 7.4.配置hadoop-2.7.1/etc/hadoop/hdfs-site.xml

![1665241432851](assets/1665241432851.png)

![1665241436311](assets/1665241436311.png)

```java
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
<property>
    <name>dfs.namenode.secondary.http-address</name>
     <value>node2:50090</value>
</property>
//注意hadoopdata目录不存在，由hadoop生成
```

#### 7.5.hadoop-2.7.1/etc/hadoop/slaves中指定三台DN

![1665241481837](assets/1665241481837.png)

![1665241485832](assets/1665241485832.png)

#### 7.6.手动创建masters文件指定SNN

![1665241518709](assets/1665241518709.png)

![1665241513722](assets/1665241513722.png)

### 8.同步配置文件到其它节点

`scp -r hadoop-2.7.1/    node2:/usr/local/`、`scp -r hadoop-2.7.1/    node3:/usr/local/`、`scp -r hadoop-2.7.1/    node4:/usr/local/`

![1665241703324](assets/1665241703324.png)

### 9.格式化NN

#### 9.1.格式化

![1665241789912](assets/1665241789912.png)

![1665241793665](assets/1665241793665.png)

#### 9.2.多次格式化带来的问题

每次格式化NameNode会生成一个新的`clusterID`，而DataNode的`clusterID`还是旧的，`clusterID`不一致导致DataNode无法启动，只要把这两文件中的`clusterID`改一致就能正常启动，修改NameNode的`clusterID`或修改DataNode的`clusterID`都行，但在集群情况下DataNode的数量多，还是改NameNode的`clusterID`比较方便

* NameNode的`clusterID`写在`/opt/hadoopdata/dfs/name/current/VERSION`文件中
* DataNode的`clusterID`写在`/opt/hadoopdata/dfs/data/current/VERSION`文件中

![1665241979953](assets/1665241979953.png)

![1665241986386](assets/1665241986386.png)

![1665241991502](assets/1665241991502.png)

![1665241995148](assets/1665241995148.png)

------

![1665242008344](assets/1665242008344.png)

![1665242012137](assets/1665242012137.png)

### 10.启动HDFS

![1665242075565](assets/1665242075565.png)

启动信息如下

```
Starting namenodes on [node1]
node1: starting namenode, logging to /usr/local/hadoop271/logs/hadoop-root-namenode-node1.out
node2: starting datanode, logging to /usr/local/hadoop271/logs/hadoop-root-datanode-node2.out
node3: starting datanode, logging to /usr/local/hadoop271/logs/hadoop-root-datanode-node3.out
node4: starting datanode, logging to /usr/local/hadoop271/logs/hadoop-root-datanode-node4.out
Starting secondary namenodes [node2]
node2: starting secondarynamenode, logging to /usr/local/hadoop271/logs/hadoop-root-secondarynamenode-node2.out
```

启动成功后，查看进程

![1665242112492](assets/1665242112492.png)

### 11.最后测试是否配置完毕

#### 11.1.NameNode查看

查看地址http://node1:50070，注意查看防火墙是否开放50070端口

![1665242202331](assets/1665242202331.png)

#### 11.2.SNN查看

查看地址http://node2:50090

![1665242253784](assets/1665242253784.png)

#### 11.3.访问不到检查是否关闭防火墙

查看防火墙状态：`systemctl status firewalld.service`

关闭运行的防火墙：`systemctl stop firewalld.service`

禁止防火墙服务器：`systemctl disable firewalld.service`

------

防火墙中打开8080端口

```
1.firewall-cmd --zone=public --add-port=8080/tcp --permanent
2.firewall-cmd --reload
3.firewall-cmd --list-port #查看开放的端口列表
4.http://node1:8080/
```











