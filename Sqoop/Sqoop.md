# Sqoop

## 一、Sqoop概述

**Sqoop用于在Hadoop与传统数据库之间传递数据**，可将传统数据库的数据导入HDFS，反过来也可将HDFS的数据导入传统数据库

## 二、Sqoop安装

### 1.下载解压重命名

请前往[官网](https://sqoop.apache.org/)下载*sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz*文件，下载完成后上传到Linux并解压重命名文件

```java
tar -xvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
mv sqoop-1.4.7.bin__hadoop-2.6.0 sqoop147
```

### 2.配置环境变量

```java
vim /etc/profile
    JAVA_HOME=/usr/java/jdk1.8.0_152
    JRE_HOME=/usr/java/jdk1.8.0_151/jre
    HADOOP_HOME=/usr/local/hadoop-2.7.1
    ZOOKEEPER_HOME=/usr/local/zookeeper-3.3.6
    FLUME_HOME=/usr/local/flume-1.9.0
    NGINX_HOME=/usr/local/nginx
    TOMCAT_HOME=/usr/local/tomcat-8.5.81
    MYSQL8_HOME=/usr/local/mysql8
    SQOOP_HOME=/usr/local/sqoop147

    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$FLUME_HOME/bin:$NGINX_HOME/sbin:$TOMCAT_HOME/bin:$MYSQL8_HOME/bin:$SQOOP_HOME/bin
    export PATH CLASSPATH JAVA_HOME JRE_HOME HADOOP_HOME ZOOKEEPER_HOME FLUME_HOME NGINX_HOME TOMCAT_HOME MYSQL8_HOME SQOOP_HOME
source /etc/profile
```

### 3.配置sqoop-env.sh

```java
cd /usr/local/sqoop147/conf
cp sqoop-env-template.sh sqoop-env.sh
```

![在这里插入图片描述](assets\011d1d70453745aab3f900f5045ed24b.png)

### 4.导Mysql8驱动文件到Sqoop的lib

![在这里插入图片描述](assets\f5de4c3907a040cda77b19fc154632a5.png)

### 5.Sqoop的JAR放入Hadoop的lib

![在这里插入图片描述](assets\fed6e16bfb694db49b71c45da05e1d46.png)

![在这里插入图片描述](assets\c607e896976447ef96585cba62c92e1d.png)

### 6.测试是否安装成功

成功标志如下

![1665459415978](assets\1665459415978.png)

上图中，出现一个警告，根据以下步骤解决警告

![1665459483857](assets\1665459483857.png)

![在这里插入图片描述](assets\a2aa81aa94654683b1ab6825d7806539.png)

![在这里插入图片描述](assets\146a2503637c4678b63ddc76c88ae91e.png)

## 三、Sqoop命令

### 1.建立测试数据

①project数据库

```mysql
create database sqooptest1
use sqooptest1
create table project(
	id int not null auto_increment primary key,
	name varchar(100) not null,
	type tinyint(4) not null default 0,
	description varchar(500) default null,
	create_at date default null,
	update_at timestamp not null default current_timestamp on update current_timestamp,
	status tinyint(4) not null default 0
);
insert into project( name,type,description,create_at,status) values( 'project1',1,'project1 zy','2019-07-27',0);
insert into project( name,type,description,create_at,status) values( 'project2',1,'project2 zy','2019-07-26',0);
insert into project( name,type,description,create_at,status) values( 'project2',2,'project2 zy','2019-07-25',0);
```

输入sqoop命令测试

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1 --username root --password a --table project
```

![1665463164683](assets\1665463164683.png)

②students数据库

```mysql
create database sqooptest1
use sqooptest1
create table students(
	id int not null primary key,
	name varchar(100) not null,
	age varchar(100) not null
);
```

数据量比较大，文件位置在`E:\JAVA课程\...\11.Hadoop\12.Sqoop\a.txt`，使用Java代码导入，等待数据插入完成即可

```java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class AddBatchMysql {
	public static void main(String[] args) {
		//1.用户输入文件位置
		Scanner sc = new Scanner(System.in);
		System.out.println("文件位置：");
		String path = sc.nextLine();
		//2.以流的形式读取文件中所有数据，按行读，按\t切，分出id,name,age
		List<String> list = new ArrayList<String>();
		try (BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(new File(path))))){
			String str;
			while((str=br.readLine())!=null){
				list.add(str);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		//3.批量插入数据
		System.out.println("数据总条数："+list.size());
		
		String sql = "insert into students values(?,?,?)";
		
		Connection con=null;
		PreparedStatement pstmt=null;
		try {
			con = DriverManager.getConnection("jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC","root","a");
			con.setAutoCommit(false);//设置成手动提交事务
			pstmt = con.prepareStatement(sql);
			
			int total = 0;
			String s;
			String[] ss;
			for(int i=0;i<list.size();i++){
				s = list.get(i);
				ss = s.split("\t");
				pstmt.setString(1, ss[0]);
				pstmt.setString(2, ss[1]);
				pstmt.setString(3, ss[2]);
				
				//加入批处理操作
				//将当前要执行的操作添加到批缓存
				pstmt.addBatch();
				if((i+1)%1000==0){//1000条数据处理一次
					int[] res = pstmt.executeBatch();
					total+=sum(res);
					con.commit();
					pstmt.clearBatch();
				}
			}
			
			int[] res = pstmt.executeBatch();
			System.out.println(res);
			
			total+=sum(res);
			con.commit();
			pstmt.clearBatch();
			System.out.println("实际插入数据条数："+total);
		} catch (Exception e) {
			e.printStackTrace();
			try {
				con.rollback();
			} catch (SQLException e1) {
				e1.printStackTrace();
			}
		}finally {
			if(con!=null){
				try {
					con.setAutoCommit(true);
				} catch (SQLException e) {
					e.printStackTrace();
				}
				try {
					con.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	private static int sum(int[] res){
		int total = 0;
		if(res==null&&res.length<=0){
			return 0;
		}
		for(int i=0;i<res.length;i++){
			total+=res[i];
		}
		return total;
	}
}
```

具体命令详解在[官网](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html)查看

### 2.sqoop-list

①列库，`--verbose`表示打印更多信息

```java
sqoop-list-databases --connect jdbc:mysql://localhost:3306/mysql?serverTimezone=UTC --username root --password a --verbose
```

②列表

```java
sqoop-list-tables --connect jdbc:mysql://localhost:3306/mysql?serverTimezone=UTC --username root --password a --verbose
```

### 3.sqoop import

①指定输出路径`–target-dir`，指定目录是`/sqooptest/input1/project`，实际目录是`/sqooptest/input1/project`

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --target-dir /sqooptest/input1/project
```

![1665463613504](assets\1665463613504.png)

②指定表名当成数据仓库名`–warehouse-dir`，意思是在指定的目录下再创建名称为表名的目录，指定目录是`/sqooptest/input2`，实际目录是`/sqooptest/input2/project`

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --warehouse-dir /sqooptest/input2
```

![1665463749200](assets\1665463749200.png)

③指定要查询的列`--columns`与查询条件`--where`，其中`-m`表示只用到一个Mapper，一个Mapper对应一个切片，一个切片对应一个输出文件

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --warehouse-dir /sqooptest/input3 --columns  'id,name,type'  --where 'id>2'  -m 1
```

![1665463899807](assets\1665463899807.png)

④`--query`可以指定SQL语句进行操作，`--query`不能与`--table`和`--columns`一起使用，其中`--split-by`用于指定拆分工作单元的列，不能与`--autoreset-to-one-mapper`一起使用，`$CONDITIONS`表明分区列

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --target-dir /sqooptest/input4/project --query 'select id,name,type from project where id>2 and  $CONDITIONS'  --split-by project.id -m 1
```

![1665464147886](assets\1665464147886.png)

⑤**增量导入**仅检索最新更新的行，使用到的参数如下

|          命令          |           说明           |
| :--------------------: | :----------------------: |
|    `--check-column`    |         检索的列         |
| `--incremental append` | 确定哪些值是最新值的规则 |
|     `--last-value`     |       最后一次修改       |

------

新增4条数据

```mysql
insert into project( name,type,description,create_at,status) values( 'project5',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project6',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project7',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project8',5,'project5 zy','2019-07-25',0);
```

新增数据后的数据如下

![1665464653760](assets\1665464653760.png)

输入Sqoop命令

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --warehouse-dir /sqooptest/input6 -m 1 --check-column id  --incremental append --last-value 3
```

结果如下

![1665464669563](assets\1665464669563.png)

------

新增4条数据

```mysql
insert into project( name,type,description,create_at,status) values( 'project9',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project10',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project11',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project12',5,'project5 zy','2019-07-25',0);
```

新增数据后的数据如下

![1665464742404](assets\1665464742404.png)

输入Sqoop命令

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --warehouse-dir /sqooptest/input6 -m 1 --check-column id  --incremental append --last-value 3
```

结果如下

![1665464953658](assets\1665464953658.png)

------

输入Sqoop命令，`--append`表示将数据追加到HDFS的现有数据集

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --warehouse-dir /sqooptest/input8 -m 1 --check-column update_at  --incremental lastmodified --last-value "2022-06-29 15:46:12" --append
```

结果如下

![1665465075572](assets\1665465075572.png)

### 4.sqoop job

#### 4.1.创建任务

创建任务导入`sqooptest1`库中`project`表所有内容到Hadoop，原来不创建任务时的命令如下

```java
sqoop import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC --username root --password a --table project --target-dir /sqooptest/input9/project -m 1
```

------

创建命令的命令如下

```java
sqoop job --create yc-job1 -- import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC  --username root --password a --table project --target-dir /sqooptest/input10/project -m 1
```

查看任务的两种方法

```java
sqoop job --list
sqoop job --show yc-job1
```

执行任务

```java
sqoop job --exec yc-job1
```

查看执行结果

![1665465455888](assets\1665465455888.png)

#### 4.2.使用密码文件登录数据库

原来创建任务时会提示输入数据库密码，这阻碍了自动化运行，查看官方文档7.2.1发现可以配置密码文件

![1665465608813](assets\1665465608813.png)

创建密码隐藏文件的命令如下

```java
echo -n "a" >/root/.mysql.password
chmod 400 /root/.mysql.password
```

之后使用Sqoop命令的方式如下

```java
sqoop job --create yc-job2 -- import --connect  jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC  --username root --password-file file:////root/.mysql.password --table project --target-dir /sqooptest/input11/project -m 1
```

#### 4.3.创建增量导入任务

查看数据库中最大的id值

![1665465824070](assets\1665465824070.png)

插入一些新数据

```mysql
insert into project( name,type,description,create_at,status) values( 'project12',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project13',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project14',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project15',5,'project5 zy','2019-07-25',0);
```

创建任务并执行

```java
sqoop job --create yc-job3 -- import --connect jdbc:mysql://node3:3306/sqooptest1?serverTimezone=UTC  --username root --password-file file:////root/.mysql.password --table project --target-dir /sqooptest/input12/project -m 1 --check-column id  --incremental append --last-value 11
sqoop job --exec yc-job3
```

查看执行结果

![1665465899878](assets\1665465899878.png)

再插入一些新数据

```mysql
insert into project( name,type,description,create_at,status) values( 'project16',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status) values( 'project17',5,'project5 zy','2019-07-25',0);	
```

再次执行任务

```java
sqoop job --exec yc-job3
```

再次查看执行结果

![1665465968960](assets\1665465968960.png)

以上输出结果中表明**Job底层有MetaStore元数据库存储当前id的最新值 ，以便下次从此id处导入**，这方便了定时任务，不用程序员自己记录更新到那一条数据了

#### 4.4.定时作业

执行定时任务有三种方式，即Oozie、Azkaban等框架，编写定时程序，CentOS自带的crontab三种，接下来展示CentOS自带的crontab实现定时任务的方式

①`/usr/local/bin`下创建`sqoop_incremental.sh`定时任务脚本文件

```java
cd /usr/local/bin
vim sqoop_incremental.sh
	#! /bin/bash
	/usr/local/sqoop147/bin/sqoop  job --exec yc-job3>>/usr/local/sqoop147/myjob.out 2>&1 &
		#解释
		#/usr/local/sqoop147/bin/sqoop：sqoop命令全路径，防止找不到 
		#/usr/local/sqoop147/myjob.out：命令的结果输出到myjob.out
		#2>&1：错误日志也当成正确日志
		#&：后台进程
```

②创建crontab定时任务

```java
crontab -e
	#每5分钟执行一次
	*/5 * * * * /usr/bin/bash /usr/local/sqoop147/bin/sqoop_incremental.sh
	#格式:分 时 日 月 周 命令
```

③再插入一些新数据

```mysql
insert into project( name,type,description,create_at,status) values( 'project18',5,'project5 zy','2019-07-25',0);
```

④等待五分钟左右，查看日志文件`/usr/local/sqoop147/myjob.out`

![在这里插入图片描述](assets\590b85eee1aa485e82710a92a8832dec.png)

⑤查看执行结果

![1665466490287](assets\1665466490287.png)

### 5.小结

Sqoop底层运行的任务是什么？Sqoop中只有Map阶段，无Reduce，且默认4个MapTask

Sqoop在导入数据时发生数据倾斜怎么避免？首先`num-mappers`用于设置MapTask的数量，默认4个；其次`split-by`用于限定MapTask中分配的切片依据哪个列



























