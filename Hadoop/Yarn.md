# Yarn

## 一、HDFS架构

![1665299496854](assets\1665299496854.png)

## 二、单节点RM的Yarn配置

### 1.关闭Hadoop集群

关闭Hadoop集群命令：`stop-dfs.sh`

> 以下操作都在node1下进行

### 2.修改mapred-site.xml

```java
cd /usr/local/hadoop-2.7.1/etc/hadoop
mv mapred-site.xml.template mapred-site.xml	//移除目录
vim mapred-site.xml
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
```

### 3.修改yarn-site.xml

```java
vim yarn-site.xml
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
```

### 4.检测yarn是否配置成功

启动Hadoop集群`start-yarn.sh`，发现node1的进程jps上新增ResourseManager，且node234上则新增NodeManager

查看Web`http://node1:8088/`，发现可以正常访问

## 三、RM的HA配置

### 1.HA架构图

![1665299959964](assets\1665299959964.png)

### 2.配置图解

![1665299997809](assets\1665299997809.png)

### 3.高可用配置

#### 3.1.node1上修改yarn-site.xml

```java
vim yarn-site.xml
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yc2yarn</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>node3</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>node4</value>
    </property>
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>
```

#### 3.2.node1上同步配置文件到node234

```java
cd /usr/local/hadoop-2.7.1/etc/hadoop
scp ./*.xml   node2:`pwd`
scp ./*.xml   node3:`pwd`
scp ./*.xml   node4:`pwd`
```

#### 3.3.确保node123节点的ZK启动

命令：`zkServer.sh start`

#### 3.4.node1上关闭Hadoop集群

命令：`stop-dfs.sh`

#### 3.5.node34上运行如下命令

命令：`yarn-daemon.sh start resourcemanager`

#### 3.6.检测

①jps检查进程

| ![1665300175099](assets\1665300175099.png) | ![1665300189732](assets\1665300189732.png) |
| :----------------------------------------: | :----------------------------------------: |
| ![1665300202520](assets\1665300202520.png) | ![1665300211957](assets\1665300211957.png) |

②node34中检查端口占用情况：`netstat -npl`

![1665300256229](assets\1665300256229.png)

③Web访问`http://node3:8088/`，发现可以正常访问

④将Active状态的RM关闭，看Standby状态的节点是否接管`yarn-daemon.sh stop resourcemanager`，关闭后之前Active状态的节点通过web无法访问，另一个可以

> Active/Standby状态手工转移方案
>
> * 查看RM状态：`yarn rmadmin -getServiceState rm1`
> * Active变Standby：`yarn rmadmin -transitionToStandby -forcemanual rm1`
> * Standby变Active：`yarn rmadmin -transitionToStandby -forcemanual rm2`











