# CentOS安装与配置

前提需要先安装好Vmware，然后下载[CentOS7](https://pan.baidu.com/share/init?surl=W5OsfNnXyEEc_t6Styq2ww)，提取码5l27

## 一、创建虚拟机

![1665237976462](../Hadoop/assets/1665237976462.png)

![1665237983505](../Hadoop/assets/1665237983505.png)

![1665237988812](../Hadoop/assets/1665237988812.png)

![1665237992868](../Hadoop/assets/1665237992868.png)

![1665237998913](../Hadoop/assets/1665237998913.png)

![1665238003693](../Hadoop/assets/1665238003693.png)

![1665238009429](../Hadoop/assets/1665238009429.png)

![1665238014302](../Hadoop/assets/1665238014302.png)

![1665238019271](../Hadoop/assets/1665238019271.png)

![1665238023217](../Hadoop/assets/1665238023217.png)

![1665238028079](../Hadoop/assets/1665238028079.png)

![1665238033084](../Hadoop/assets/1665238033084.png)

![1665238038637](../Hadoop/assets/1665238038637.png)

![1665238044750](../Hadoop/assets/1665238044750.png)

![1665238125480](../Hadoop/assets/1665238125480.png)

![1665238131366](../Hadoop/assets/1665238131366.png)

![1665238135691](../Hadoop/assets/1665238135691.png)

![1665238139499](../Hadoop/assets/1665238139499.png)

![1665238143629](../Hadoop/assets/1665238143629.png)

![1665238154807](../Hadoop/assets/1665238154807.png)

![1665238159038](../Hadoop/assets/1665238159038.png)

![1665238163377](../Hadoop/assets/1665238163377.png)

![1665238167217](../Hadoop/assets/1665238167217.png)

![1665238171777](../Hadoop/assets/1665238171777.png)

![1665238177166](../Hadoop/assets/1665238177166.png)

![1665238182295](../Hadoop/assets/1665238182295.png)

![1665238186919](../Hadoop/assets/1665238186919.png)

![1665238192267](../Hadoop/assets/1665238192267.png)

![1665238196629](../Hadoop/assets/1665238196629.png)

![1665238203039](../Hadoop/assets/1665238203039.png)

![1665238208802](../Hadoop/assets/1665238208802.png)

![1665238213822](../Hadoop/assets/1665238213822.png)

![1665238217751](../Hadoop/assets/1665238217751.png)

![1665238222596](../Hadoop/assets/1665238222596.png)

![1665238228763](../Hadoop/assets/1665238228763.png)

![1665238233061](../Hadoop/assets/1665238233061.png)

![1665238237722](../Hadoop/assets/1665238237722.png)

![1665238242912](../Hadoop/assets/1665238242912.png)

## 二、打开终端

也可以切换成[命令行界面](https://blog.csdn.net/zhangyingchengqi/article/details/103559276)，命令行效率会更高，不切也没关系

![1665238384798](../Hadoop/assets/1665238384798.png)

## 三、配置虚拟机启用网卡，并设置固定IP地址

### 1.配置原理

配置为固定IP后，不管什么情况下不受虚拟机影响，只要笔记本主机可以正常上网，那么启动虚拟机中的CentOS 7系统就可以正常访问外网，无需再进行任何设置

![1665238680619](../Hadoop/assets/1665238680619.png)

### 2.设置虚拟机IP

①设置虚拟机的网络连接方式

![1665238735609](../Hadoop/assets/1665238735609.png)

②这一步部分计算机不需要设置

  ![1665238816933](../Hadoop/assets/1665238816933.png)

③修改子网IP，实现自由设定固定IP，**1网段无法成功(第三位IP段)**

![1665238823364](../Hadoop/assets/1665238823364.png)

![1665238948465](../Hadoop/assets/1665238948465.png)

### 3.配置笔记本主机具体VMnet8本地地址参数

![1665238993967](../Hadoop/assets/1665238993967.png)

![1665238998864](../Hadoop/assets/1665238998864.png)

![1665239005740](../Hadoop/assets/1665239005740.png)

### 4.修改虚拟机中的CentOS7系统为固定IP的配置文件

①输入`su root`指令切换到root用户

②ifcfg-ens33是虚拟机的网卡，配置连接VMnet8的网关，通过网关连接主机的网卡

![1665239113010](../Hadoop/assets/1665239113010.png)

![1665239156440](../Hadoop/assets/1665239156440.png)

```java
cd /etc/sysconfig/network-scripts
vim ifcfg-ens33	//i编辑，esc退出编辑，:wq退出保存
	DNS1=114.114.114.114
	IPADDR=192.168.10.100
	NETMASK=255.255.255.0
	GATEWAY=192.168.10.2                    
service network restart
```

### 5.检验配置是否成功

①查看修改后的固定IP

![1665239264111](../Hadoop/assets/1665239264111.png)

②测试虚拟机中的CentOS 7系统是否能连外网

![1665239289727](../Hadoop/assets/1665239289727.png)

③测试本机是否能ping通虚拟机的固定IP

![1665239295403](../Hadoop/assets/1665239295403.png)

## 四、修改主机名

![1665239336663](../Hadoop/assets/1665239336663.png)

![1665239340940](../Hadoop/assets/1665239340940.png)

## 五、在CentOS中安装sftp服务

**sftp安全文件传送协议**是传输文件的安全网络加密方法，为了上传文件到虚拟机系统

①确认已安装好ssh服务

![1665239437171](../Hadoop/assets/1665239437171.png)

②配置sftp

![1665239443208](../Hadoop/assets/1665239443208.png)

![1665239478187](../Hadoop/assets/1665239478187.png)

![1665239447235](../Hadoop/assets/1665239447235.png)

```java
Match Group sftp
ChrootDirectory /usr/sftp
ForceCommand internal-sftp
AllowTcpForwarding no
X11Forwarding no
```

## 六、远程连接检测是否配置成功

![1665239482213](../Hadoop/assets/1665239482213.png)

![1665239571889](../Hadoop/assets/1665239571889.png)

![1665239577296](../Hadoop/assets/1665239577296.png)

![1665239581194](../Hadoop/assets/1665239581194.png)

## 七、关闭防火墙

取消开机自启动防火墙`systemctl disable firewalld.service`

查看防火墙状态`systemctl status firewalld`














