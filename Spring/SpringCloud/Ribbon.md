# Ribbon

## 一、Ribbon概述

### 1.Ribbon是什么？

Ribbon是基于Netflix Ribbon实现的、基于HTTP和TCP协议的**客户端负载均衡工具**，服务消费者获取到注册表后到底选取那台服务提供者呢？Ribbon解决此问题。

Ribbon几乎存在于每个SpringCloud构建的微服务和基础设施中，所以一般是不需要独立部署的。

### 2.负载均衡

负载均衡是**缓解网络压力**和**扩容处理能力**的重要手段之一，当客户端发送请求到负载均衡设备时，负载均衡设备按某种算法从注册表中取出一台服务提供者地址，然后进行转发，常见的负载均衡算法有轮询、权重、流量等

> 注册表中缓存的服务提供者一般都是健康的，因为心跳机制会剔除不健康的服务

负载均衡流程图如下

![1666081229717](assets\1666081229717.png)

## 二、HelloRibbon

HelloRibbon在HelloEureka基础上进行改进，由于Ribbon是客户端的负载均衡，所以只需改变服务消费者代码即可

### 1.目录结构

![1666081428832](assets\1666081428832.png)

### 2.pom.xml

Eureka Client中是自带Ribbon依赖的，所以无需额外添加

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 3.application.yml

```java
#eureka配置,注册服务到eureka
eureka:
  client:
    register-with-eureka: false #表示是否向eureka注册中心注册自己 --> 消费端不用注册自己
    serviceUrl:
      defaultZone: http://eureka1:7001/eureka/,http://eureka2:7002/eureka/,http://eureka3:7003/eureka/
```

### 4.配置负载均衡

#### 4.1.全局负载均衡配置

```java
@Configuration
public class AppConfig {
    @Bean
    @LoadBalanced//配置负载均衡
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    //修改访问策略：直接托管对应的类,配置的是全局负载均衡策略
    @Bean
    public IRule ribbonRule(){
        //随机访问策略
        return new com.netflix.loadbalancer.RandomRule();
    }
}
```

#### 4.2.局部负载均衡配置

①添加负载均衡注解

```java
@Configuration
public class AppConfig {
    @Bean
    @LoadBalanced//配置负载均衡
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

②创建负载均衡策略配置类，该类不用托管，否则还是全局负载均衡策略配置

```java
public class RibbonConfig {
    @Bean
    public IRule ribbonRule() {
        //随机访问策略
        return new com.netflix.loadbalancer.RandomRule();
    }
}
```

③启动类添加注解，意思是针对`name`服务采用RibbonConfig.class配置项

```java
@RibbonClient(name="MICROCLOUD-PROVIDER-DEPT",configuration=RibbonConfig.class)
```

### 5.修改请求地址

加入负载均衡后是不知道服务的具体IP和Port的，所以需要将该部分配置为RibbonClient的`name`

```java
@RestController
@RequestMapping("/consumer")
public class DeptConsumerServlet {
    //请求地址url
    /*private final String ADD_DEPT_URL = "http://localhost:8001/provider/addDept/";
    private final String FIND_ID_URL = "http://localhost:8001/provider/findById/";
    private final String FIND_ALL_URL = "http://localhost:8001/provider/findAll/";*/

    //Ribbon后请求的地址应该是变量，通过服务名访问，具体使用有负载均衡访问策略决定
    private final String RIBBON_URL = "http://MICROCLOUD-PROVIDER-DEPT/provider/";

    //消息转换器,消费者端不拥有service层
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/addDept")
    public JsonModel addDept(@RequestBody Dept dept, JsonModel jsonModel) {
        return jsonModel = this.restTemplate.exchange(RIBBON_URL + "addDept/", HttpMethod.POST, new HttpEntity<>(dept), JsonModel.class).getBody();
    }

    @RequestMapping("/findById/{id}")
    public JsonModel findById(@PathVariable("id") Long id, JsonModel jsonModel) {
        return jsonModel = this.restTemplate.postForObject(RIBBON_URL + "findById/" + id, null, JsonModel.class);
    }

    @RequestMapping("/findAll")
    public JsonModel fingAll(JsonModel jsonModel) {
        return jsonModel = this.restTemplate.postForObject(RIBBON_URL + "findAll/", null, JsonModel.class);
    }
}
```

### 6.测试负载均衡

①启动1个及以上Eureka

②启动2个及以上Provider

③启动1个Consumer

④浏览器中输入`http://localhost/consumer/findAll`

⑤刷新一次则观察一个该请求被发送到哪个服务提供端

### 7.获取请求到的提供者信息

```java
@Autowired
private LoadBalancerClient loadBalancerClient;

@RequestMapping("/findAll")
public JsonModel fingAll(JsonModel jsonModel) {
    //获取服务提供方信息
    ServiceInstance serviceInstance = this.loadBalancerClient.choose("MICROCLOUD-PROVIDER-DEPT");
    System.out.println("[服务提供实例信息] - host:" + serviceInstance.getHost() + " - port:" + serviceInstance.getPort() + " serviceid:" + serviceInstance.getServiceId());

    return jsonModel = this.restTemplate.postForObject(RIBBON_URL + "findAll/", null, JsonModel.class);
}
```





























