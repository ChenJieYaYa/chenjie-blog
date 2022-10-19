# Eureka

## 一、Eureka引入

若部署多个服务提供者，此时服务消费者发起远程调用，如何得知服务提供者的IP和Port呢？消费者如何选择提供者呢？消费者如何确定某个提供者是否健康呢？Eureka解决这些

![1665999960719](assets\1665999960719.png)

## 二、Eureka架构

### 1.架构图

![1665998864021](assets\1665998864021.png)

### 2.流程图

![1666002068789](assets\1666002068789.png)

### 3.Eureka Server

Eureka Server是注册中心的服务端，提供三个功能

①**服务注册**：服务提供者启动后向Eureka Server注册自己，Eureka Server内存在两层缓存机制来维护此注册表

②**提供注册表**：服务消费者调用服务时会从Eureka Server拉取注册表缓存到本地

③**维护心跳**：Eureka Client每隔30秒发送心跳到Eureka Server用于服务续约，通过心跳告知Eureka Server该Eureka Client是健康的正常运行的，90秒未收到续约信号则将该服务提供者从注册列表中删除，续约时间与失效时间可配置

|                           配置                            |   说明   |
| :-------------------------------------------------------: | :------: |
|  `eureka.instance.lease-renewal-interval-in-seconds=30`   | 续约间隔 |
| `eureka.instance.lease-expiration-duration-in-seconds=90` | 失效时长 |

### 4.Eureka Client

Eureka Client是注册中心的客户端，包括服务提供者和服务消费者

Eureka Client会拉取注册表且缓存到本地，那么这也将带来一个问题，当Eureka Server更新注册表后与Eureka Client本地缓存的注册表不一致，拉取注册表时间间隔可配置

|                        配置                        |            说明             |
| :------------------------------------------------: | :-------------------------: |
|        `eureka.client.fetch-registry=true`         | Eureka Client启用拉取注册表 |
| `eureka.client.registry-fetch-interval-seconds=30` |    拉取注册表的时间间隔     |

### 5.自我保护机制

微服务架构中服务之间通常是跨进程调用，网络通信面临各种问题，Eureka Server在90秒内未收到续约信号则会注销该服务，但**大量的服务注销严重威胁整个微服务架构的可用性**，由此推出自我保护机制

自我保护机制指**Eureka Server在运行期间统计心跳失败比例15分钟内是否低于85%🙃**，如果低于85%则Eureka Server即会进入自我保护机制

进入自我保护机制后**Eureka Server不再从注册表中剔除注销的服务**，新的注册和查询请求依然被接收，但**不再同步到其他节点**，网络稳定以后新的注册信息才会被同步

自我保护机制说白了就是**当个别客户端出现心跳失联时则认为是客户端的问题，剔除掉该客户端即可；当大量客户端心跳失联时则认为可能是网络问题，进入自我保护机制，当大部分客户端心跳恢复时自动退出自我保护机制**

自我保护机制通过`eureka.server.enable-self-preservation=true`配置开启

## 三、HelloEureka

HelloEureka在HelloSpringCloud基础上进行改进

### 1.EurekaServer

#### 1.1.基础配置

①目录结构

![1666062159951](assets\1666062159951.png)

②pom.xml

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<!--热部署：写完代码不用重启，刷新即可-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

③application.yml

```java
server:
  port: 7001

#eureka配置
eureka:
  instance:
    hostname: localhost #服务端实例名称，可通过c:\windows\system32\drivers\etc修改域名映射
  client:
	fetch-registry: false #为false表示自己为服务注册中心
    register-with-eureka: false #表示是否向eureka注册中心注册自己
    serviceUrl: #eureka源码看到defaultZone是其默认地址，此时修改这个默认地址 http://服务端实例名称:端口/eureka/
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: eureka-server
```

④启动类

```java
@SpringBootApplication
@EnableEurekaServer//启动Eureka服务器
public class EurekaServer {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer.class, args);
    }
}
```

#### 1.2.关闭自我保护机制

当自我保护机制开启时，会出现如下红字提示，开启自我保护机制可以保证可用性

![1666063193587](assets\1666063193587.png)

①application.yml

```java
#eureka配置
eureka:
  instance:
    hostname: localhost #服务端实例名称，可通过c:\windows\system32\drivers\etc修改域名映射
  client:
	fetch-registry: false #为false表示自己为服务注册中心
    register-with-eureka: false #表示是否向eureka注册中心注册自己
    serviceUrl: #eureka源码看到defaultZone是其默认地址，此时修改这个默认地址 http://服务端实例名称:端口/eureka/
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    eviction-interval-timer-in-ms: 1000 #清理服务注册列表的间隔时间(清除掉线的，默认情况60s)
    enable-self-preservation: false #是否启用自我保护模式(默认关闭)
```

### 2.Provider

修改microcloud-provider-dept-8001的相关内容

#### 2.1.基础配置

①pom.xml

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

②application.yml

```java
#eureka配置,注册服务到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7001/eureka/
  instance:
    instance-id: microcloud-provider-dept-8001 #服务名
```

③启动类

```java
@SpringBootApplication
@MapperScan("com.yc.mapper")
@EnableEurekaClient //服务启动后自动注册到eureka
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

#### 2.2.启动Acurator

Acurator是监控器，启动后可提供监控信息

①pom.xml

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

②application.yml

```java
#actuator配置,打开actuator中的端口,默认情况springboot2后这些端点都没有开放
management:
  endpoints:
    web:
      exposure:
        include: "*" #yml中*有关键字,所以加要加""
```

#### 2.3.配置心跳间隔和失效时间

改变application.yml

```java
#eureka配置,注册服务到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7001/eureka/
  instance:
    instance-id: microcloud-provider-dept-8001 #服务名
    lease-renewal-interval-in-seconds: 2 #设置客户端向服务端发送心跳信息的时间间隔(默认:30s)
    lease-expiration-duration-in-seconds: 6 #设置服务器若6秒都没有接到心跳则将此客户端租约设为超期
```

### 3.Consumer

修改microcloud-provider-dept-8001的相关内容

①pom.xml

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

②可通过**EurekaClient对象**获取注册表

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableEurekaClient
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

```java
@Autowired
private EurekaClient eurekaClient;

@RequestMapping("/eurekaClient")
public Object discovery(){
    System.out.println("客户端配置信息:" + this.eurekaClient.getEurekaClientConfig());
    System.out.println("服务端应用列表:"+   this.eurekaClient.getApplications());
    return this.eurekaClient.getAllKnownRegions();//["us-east-1"]
}
```

③可通过**DiscoveryClient对象**获取注册表

```java
@SpringBootApplication
@EnableDiscoveryClient//启用springcloud的discoveryClient
public class ProviderProductApp {
    public static void main(String[] args) {
        SpringApplication.run(  ProviderProductApp.class, args);
    }
}
```

```java
@Autowired
private DiscoveryClient discoveryClient;

@RequestMapping("/discoveryClient")
public Object springCloudDiscoveryClient(){
    System.out.println(this.discoveryClient.description());
    System.out.println("注册的服务名:"+this.discoveryClient.getServices());
    return this.discoveryClient;
}
```

## 四、Eureka集群

### 1.CAP理论

CAP分别指的是一致性、可用性和分区容错性，其中分区容错只微服务架构内服务通常是跨进程调用，跨进程调用存在不可预知的网络问题，当由于网络造成单台服务不可用时不能导致整个集群服务不可用，当网络恢复时要保证服务间的一致性，一致性和可用性顾名思义。

一致性一般要求节点越少越好，因为某个节点更新后需要与其他节点同步数据，节点越少同步的消耗越小，但节点少又导致可用性降低，反之可用性就是增加节点，由此可见**CA是互斥的**。

分布式CAP理论中三者不能同时满足，P是必须满足的，CA根据实际情况取舍，但不是分布式系统的话CA可同时满足

------

学习过Eureka的自我保护机制可知，当自我保护机制开启后其注册和查询请求还是可以处理，只是不再剔除不可用服务，也不在同步新注册的服务，这保证了可用性，但放弃了一致性，所以**Eureka保证的是AP**

在Zookeeper中事务请求被转发到Leader中，Leader只有一个，一旦Leader宕机就需要重新选取，选取Leader的时间是较长的，且在此过程中没有Leader处理事务请求，从而导致整个集群不可用，Zookeeper为了保证一致性存在原子广播机制等，所以**Zookeeper保证的是CP**

### 2.集群搭建

#### 2.1.添加域名映射

修改域名文件`C:\\Windows\System32\drivers\etc\hosts`，添加如下映射

```java
127.0.0.1 eureka1
127.0.0.1 eureka2
127.0.0.1 eureka3
```

#### 2.2.EurekaServer添加配置文件

①目录结构

![1666080023402](assets\1666080023402.png)

②application.yml

```java
#eureka配置 --> 添加映射地址后修改hostname，defaultZone修改集群地址，name修改
eureka:
  instance:
    hostname: eureka1 #服务端实例名称，已经通过c:\windows\system32\drivers\etc修改域名映射
  client:
	fetch-registry: false #为false表示自己为服务注册中心
    register-with-eureka: false #表示是否向eureka注册中心注册自己
    serviceUrl: #eureka源码看到defaultZone是其默认地址，此时修改这个默认地址 http://服务端实例名称:端口/eureka/  -->配置集群地址
      defaultZone: http://eureka1:7001/eureka/,http://eureka2:7002/eureka/,http://eureka3:7003/eureka/
  server:
    eviction-interval-timer-in-ms: 1000 #清理服务注册列表的间隔时间(清除掉线的，默认情况60s)
    enable-self-preservation: false #是否启用自我保护模式(默认关闭)

spring:
  application:
    name: eureka-server1
```

③application-second.yml

```java
server:
  port: 7002

eureka:
  instance:
    hostname: eureka2

spring:
  application:
    name: eureka-server2
```

④application-third.yml

```java
server:
  port: 7003

eureka:
  instance:
    hostname: eureka3

spring:
  application:
    name: eureka-server3
```

⑤为每个配置文件都分配一台服务器，步骤如下

![1666080217606](assets\1666080217606.png)

![1666080274384](assets\1666080274384.png)

![在这里插入图片描述](assets\d7f5bd0326ef43e988794b0ba7452f02.png)

#### 2.3.Provider修改配置

```java
#eureka配置,注册服务到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka1:7001/eureka/,http://eureka2:7002/eureka/,http://eureka3:7003/eureka/
  instance:
    instance-id: microcloud-provider-dept-8001 #实例名，应用下的多个微服务的名字
    lease-renewal-interval-in-seconds: 2 #设置客户端向服务端发送心跳信息的时间间隔(默认:30s)
    lease-expiration-duration-in-seconds: 6 #设置服务器如果6秒都没有接到心跳，则将此客户端的租约设为超期
```





























