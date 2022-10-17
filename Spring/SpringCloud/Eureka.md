# Eureka

## 一、Eureka引入

若部署多个服务提供者，此时服务消费者发起远程调用，如何得知服务提供者的IP和Port呢？消费者如何选择提供者呢？消费者如何确定某个提供者是否健康呢？Eureka解决这些

![1665999960719](C:\Users\cloud\Desktop\我的\my\SpringCloud\assets\1665999960719.png)

## 二、Eureka架构

### 1.架构图

![1665998864021](C:\Users\cloud\Desktop\我的\my\SpringCloud\assets\1665998864021.png)

### 2.流程图

![1666002068789](C:\Users\cloud\Desktop\我的\my\SpringCloud\assets\1666002068789.png)

### 3.Eureka Server

Eureka Server是注册中心的服务端，提供三个功能

①**服务注册**：服务提供者启动后向Eureka Server注册自己，Eureka Server内存在两层缓存机制来维护此注册表

②

③










2.提供注册表：服务消费者调用服务时，Eureka Client未缓存注册表，则会从Eureka Server获取最新的注册表
3.同步状态：Eureka Client通过注册、心跳机制和Eureka Server同步当前客户端的状态
	心跳机制：Eureka Client每隔30秒发送一次心跳来续约(服务续约)，通过续约告知注册中心服务端该服务提供者是否正常运行，默认90秒未收到续约信号则会将该服务提供者从注册列表中删除(服务剔除)
		续约间隔配置：eureka.instance.lease-renewal-interval-in-seconds=30
		服务失效时间：eureka.instance.lease-expiration-duration-in-seconds=90





eureka-server保存服务名称到服务实例地址列表的映射关系

order-service根据服务名称，拉取实例地址列表。这个叫服务发现或服务拉取



问题3：order-service如何得知某个user-service实例是否依然健康，是不是已经宕机？

user-service会每隔一段时间（默认30秒）向eureka-server发起请求，报告自己状态，称为心跳

当超过一定时间没有发送心跳时，eureka-server会认为微服务实例故障，将该实例从服务列表中剔除

order-service拉取服务时，就能将故障实例排除了

注意：一个微服务，既可以是服务提供者，又可以是服务消费者，因此eureka将服务注册、服务发现等功能统一封装到了eureka-client端

因此，接下来我们动手实践的步骤包括：
————————————————
版权声明：本文为CSDN博主「安赫'」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_55849362/article/details/126739643













