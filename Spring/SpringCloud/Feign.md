# Feign

## 一、Feign概述

### 1.Feign是什么？

Feign是基于Netflix Feign实现的，是基于REST服务调用提供的更高级别的抽象，支持可插拔的编码器和解码器，用于**远程调用**

### 2.Feign远程调用流程

原来无Feign的流程图如下

![1666084852369](assets\1666084852369.png)

加入Feign的流程图如下

![1666084896311](assets\1666084896311.png)

## 二、HelloFeign

HelloFeign在HelloRibbon基础上进行改进

### 1.公共模块配置Feign接口

①目录结构

![1666085076156](assets\1666085076156.png)

②pom.xml

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

③Feign接口

```java
@FeignClient(name = "MICROCLOUD-PROVIDER-DEPT")
public interface DeptFeignService {

    @RequestMapping("/provider/addDept") //映射到服务提供方的地址
    public JsonModel addDept(@RequestBody Dept dept);

    @RequestMapping("/provider/findById/{id}")
    public JsonModel findById(@PathVariable("id") Long id);

    @RequestMapping("/provider/findAll")
    public JsonModel fingAll();
}
```

### 2.新建服务消费者模块

Feign和原来的RestTemplate是两种不同风格的远程调用，所以新建服务消费者模块是为了部分二者

①目录结构

![1666085283430](assets\1666085283430.png)

②pom.xml

```java
<!--feign声明式接口所在模块-->
<dependency>
    <groupId>org.example</groupId>
    <artifactId>microcloud-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

③使用Feign接口调用服务

```java
@RestController
@RequestMapping("/consumer")
public class DeptConsumerServlet {

    @Autowired
    private DeptFeignService deptFeignService;

    @RequestMapping("/addDept")
    public JsonModel addDept(@RequestBody Dept dept) {
        return this.deptFeignService.addDept(dept);
    }

    @RequestMapping("/findById/{id}")
    public JsonModel findById(@PathVariable("id") Long id) {
        return this.deptFeignService.findById(id);
    }

    @RequestMapping("/findAll")
    public JsonModel fingAll() {
        return this.deptFeignService.fingAll();
    }
}
```

④启动类添加注解**@EnableFeignClients**

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableEurekaClient
@EnableFeignClients("com.yc.feign")//feign接口所在的包
public class FeignConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignConsumerApplication.class, args);
    }
}
```

### 3.网络数据压缩配置

启用请求与响应压缩机制配置

```java
feign:
  compression:
    request:
      enabled: true #开启压缩配置
      mime-types: #压缩类型
        - text/xml
        - application/xml
        - application/text
        - application/json
        - application/msword
        - text/plain
      min-request-size: 1024 #超过1024字节就开始压缩(消耗CPU性能)
```

































