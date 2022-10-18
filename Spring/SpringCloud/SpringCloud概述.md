# SpringCloud概述

## 一、SpringCloud是什么？

前面我们已经学习过微服务理论部分，SOA面向服务的架构思想是一种思想，但这种思想该怎么落地？选用什么样的技术栈？全球的互联网公司都在积极尝试自己的微服务落地方案，其中在Java领域最引人注目的就是SpringCloud提供的方案了

[SpringCloud](https://spring.io/projects/spring-cloud)集成了各种微服务功能组件，并**基于SpringBoot实现了这些组件的自动装配**，从而提供了良好的开箱即用体验，其中常见的组件如下

![1665994563206](assets\1665994563206.png)

> 注意SpringCloud与SpringBoot二者需要版本兼容

## 二、HelloSpringCloud

### 1.数据准备

数据库建立`dept`表，数据如下

| `dno` | `dname` | `dsource` |
| :---: | :-----: | :-------: |
|   2   | 人事部  |   test    |
|   3   | 技术部  |   test    |
|   4   | 核心部  |   test    |
|   5   | 滚动部  |   test    |

### 2.父模块依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springcloud</artifactId>
    <version>1.0-SNAPSHOT</version>

	<!--子模块名称-->
    <modules>
        <module>microcloud-api</module>
        <module>microcloud-provider-dept-8001</module>
        <module>microcloud-consumer-dept-80</module>
    </modules>

	<!--只管理依赖关系，不打成jar包-->
    <packaging>pom</packaging>

    <!--配置文件中的常量设定-->
    <properties>
        <jdk.version>1.8</jdk.version>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <project.build.sourceEncoding>utf-8</project.build.sourceEncoding>

        <!--版本问题-->
        <!--<spring.cloud.version>2021.0.1</spring.cloud.version>-->
        <!--<spring.boot.version>2.6.3</spring.boot.version>-->
        <spring.cloud.version>Hoxton.SR12</spring.cloud.version>
        <spring.boot.version>2.2.1.RELEASE</spring.boot.version>

        <!--<spring.cloud.version>Finchley.SR4</spring.cloud.version>-->
        <!--<spring.boot.version>2.0.9.RELEASE</spring.boot.version>-->
    </properties>

    <!--管理依赖配置，而不真正导入jar-->
    <!--由子项目真正导入jar-->
    <!--子项目导入与此处定义的相同jar时，无须写版本号-->
    <dependencyManagement>
        <dependencies>
            <!--dependencyManagement标签中需要导入另一个pom中的dependencyManagement时，必须同时使用<scope>import</scope>和 <type>pom</type>
                 此时，该pom中dependencyManagement会包含导入的spring-boot-dependencies中的所有dependencyManagement
                 解决pom类型父工程单继承的问题，导入各种其他父工程的dependencyManagement
                 注意：dependencyManagement只在父工程中声明，子工程中定义无dependencyManagement是没有意义的
            -->

            <!--进行SpringCloud依赖包的导入处理 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!--SpringCloud离不开SpringBoot，所以必须要配置此依赖包-->
            <!--SpringCloud与SpringBoot的版本号需要严格对应(官网查)-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

			<!--正常导包工作(子项目中依赖的包最好在父项目中也写份带版本的，方便管理版本，子就不需要写版本)-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.49</version>
            </dependency>

            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.2.8</version>
            </dependency>

            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>3.4.2</version>
            </dependency>

            <!--日志测试~-->
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>

        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>microcloud</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <delimiters>
                        <delimiter>$</delimiter>
                    </delimiters>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${jdk.version}</source><!-- 源代码使用的开发版本 -->
                    <target>${jdk.version}</target><!-- 需要生成的目标class文件的编译版本 -->
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3.microcloud-api

microcloud-api是通用模块，所有通用代码放在此处

#### 3.1.目录结构

![1665996616957](assets\1665996616957.png)

#### 3.2.pom.xml

```java
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
    </dependency>
</dependencies>
```

#### 3.3.Entity

Entity表示与数据库中表对应的实体类

```java
@Data
@NoArgsConstructor//无参构造
@Accessors(chain = true)//set函数链式写法-->dept.setDno(1).set...
public class Dept implements Serializable {
    @TableId(type = IdType.AUTO)//指定主键
    private Long dno;
    private String dname;
    //数据库名称DATABASE()，一个服务对于一个数据库，同一信息可能存在不同数据库
    private String dsource;

    public Dept(String dname) {
        this.dname = dname;
    }
}
```

### 4.microcloud-provider-dept-8001

microcloud-provider-dept-8001是服务的提供方，负责提供服务处理业务

#### 4.1.目录结构

![1665996918515](assets\1665996918515.png)

#### 4.2.pom.xml

```java
<!--实现两个子模块间通信-->
<dependency>
    <groupId>org.example</groupId>
    <!--通信子模块的名字-->
    <artifactId>microcloud-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<!--日志门面-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>

<!--热部署：写完代码不用重启，刷新即可-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

#### 4.3.application.yml

```java
server: #端口号配置
  port: 8001

#mybatis-plus配置
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #开启SQL语句打印

#spring配置
spring:
  application:
    name: microcloud-provider-dept
  datasource: #数据库配置
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/test?serverTimezone=UTC #test:数据库名称
    username: root
    password: xbzz7789
    driver-class-name: com.mysql.jdbc.Driver
```

#### 4.4.mapper

与mybatis-plus技术相关

```java
public interface DeptMapper extends BaseMapper<Dept> {}
```

#### 4.5.dao

```java
public interface DeptDao {
    public int addDept(Dept dept);
    public Dept findById(Long id);
    public List<Dept> fingAll();
}
```

```java
@Repository
public class DeptDaoImpl implements DeptDao {
    @Autowired
    private DeptMapper deptMapper;

    @Override
    public int addDept(Dept dept) {
        return deptMapper.insert(dept);
    }
    @Override
    public Dept findById(Long id) {
        return deptMapper.selectById(id);
    }
    @Override
    public List<Dept> fingAll() {
        return deptMapper.selectList(null);
    }
}
```

#### 4.6.service

```java
public interface DeptService {
    public boolean addDept(Dept dept);
    public Dept findById(Long id);
    public List<Dept> fingAll();
}
```

```java
@Service
public class DeptServiceImpl implements DeptService {
    @Autowired
    private DeptDao deptDao;

    @Override
    public boolean addDept(Dept dept) {
        return deptDao.addDept(dept) > 0;
    }
    @Override
    public Dept findById(Long id) {
        return deptDao.findById(id);
    }
    @Override
    public List<Dept> fingAll() {
        return deptDao.fingAll();
    }
}
```

#### 4.7.controller

```java
@RestController
@RequestMapping("/dept")
public class DeptController {
    @Autowired
    private DeptService deptService;

    @RequestMapping("/addDept")
    public JsonModel addDept(@RequestBody Dept dept, JsonModel jsonModel) {
        this.deptService.addDept(dept);
        jsonModel.setCode(1);
        return jsonModel;
    }

    @RequestMapping("/findById/{id}")
    public JsonModel findById(@PathVariable("id") Long id, JsonModel jsonModel) {
        Dept dept = this.deptService.findById(id);
        jsonModel.setCode(1);
        jsonModel.setData(dept);
        return jsonModel;
    }

    @RequestMapping("/findAll")
    public JsonModel fingAll(JsonModel jsonModel) {
        List<Dept> list = this.deptService.fingAll();
        jsonModel.setCode(1);
        jsonModel.setData(list);
        return jsonModel;
    }
}
```

#### 4.8.启动类

```java
@SpringBootApplication
@MapperScan("com.yc.mapper")
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 5.microcloud-consumer-dept-80

microcloud-consumer-dept-80是服务的使用方，负责调用服务处理业务

#### 5.1.目录结构

![1665997288753](assets\1665997288753.png)

#### 5.2.pom.xml

```java
<dependency>
    <groupId>org.example</groupId>
    <artifactId>microcloud-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--热部署：写完代码不用重启，刷新即可-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

#### 5.3.application.yml

```java
server:
  port: 80
```

#### 5.4.配置类

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

#### 5.5.controller

```java
/**
 - 1.JDK原生的URLConnection
 - 2.Spring封装好的RestTemplate对象(最易)
 - 3.Apache的Http Client
 - 4.Netty的异步Http Client
 - 5.Feign(最优)
 */
@RestController
@RequestMapping("/consumer")
public class DeptConsumerController {

    //请求地址url
    private final String ADD_DEPT_URL = "http://localhost:8001/provider/addDept/";
    private final String FIND_ID_URL = "http://localhost:8001/provider/findById/";
    private final String FIND_ALL_URL = "http://localhost:8001/provider/findAll/";


    //消息转换器,消费者端不拥有service层
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/addDept")
    public JsonModel addDept(@RequestBody Dept dept, JsonModel jsonModel) {
        return jsonModel = this.restTemplate.exchange(ADD_DEPT_URL, HttpMethod.POST, new HttpEntity<>(dept), JsonModel.class).getBody();
    }

    @RequestMapping("/findById/{id}")
    public JsonModel findById(@PathVariable("id") Long id, JsonModel jsonModel) {
        return jsonModel = this.restTemplate.postForObject(FIND_ID_URL + id, null, JsonModel.class);
    }

    @RequestMapping("/findAll")
    public JsonModel fingAll(JsonModel jsonModel) {
        return jsonModel = this.restTemplate.postForObject(FIND_ALL_URL, null, JsonModel.class);
    }
}
```

#### 5.6.启动类

```java
/**
 * 错误
 * 1.Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could
 * 原因：导入jdbc包，但是没有配置url-->jdbc包在microcloud-api的mybatis-plus中导入
 * 解决：https://blog.csdn.net/qq_34322008/article/details/89954934
 * -> SpringBoot启动时，排除jdbc的自动装配机制即可，@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
 */
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

## 三、实现远程调用

HelloSpringCloud中我们学习到服务提供端和消费端间的通讯使用的是Rest风格，其实不只有这一种通讯方式，此处插播一则题外话

### 1.Rest风格接受请求

```java
@RestController
@RequestMapping("/product")
public class ProductController {
    @Autowired
    private ProductBiz productBiz;

	//1.映射地址传入参数
    @RequestMapping("/get/{pid}")
    public JsonModel get(@PathVariable("pid") Integer pid, JsonModel jsonModel) {
        Product product = this.productBiz.findById(pid);
        jsonModel.setCode(1);
        jsonModel.setData(product);
        return jsonModel;
    }

	//2.input表单接收参数
    @RequestMapping("/add")
    public JsonModel add(@RequestBody Product product, JsonModel jsonModel) {
        this.productBiz.add(product);
        jsonModel.setCode(1);
        return jsonModel;
    }

	//3.不接收参数
    @RequestMapping("/list")
    public JsonModel list(JsonModel jsonModel) {
        List<Product> list = this.productBiz.findByList();
        jsonModel.setCode(1);
        jsonModel.setData(list);
        return jsonModel;
    }
}
```

### 2.发送请求(远程调用)

#### 2.1.URLConnection

URLConnection是JDK原生的发送请求的方式，实现方式如下

```java
private final String BASE_URL_GET = "http://localhost:8081/product/get/";

@RequestMapping("/product/get/{pid}")
public JsonModel get(@PathVariable("pid") Integer pid, JsonModel jsonModel) throws IOException {
    String url = BASE_URL_GET + pid;
    jsonModel = sendRequest(url);
    return jsonModel;
}

private JsonModel sendRequest(String urlstr) throws IOException {
 JsonModel jsonModel = new JsonModel();
    String result = "";
    StringBuffer sb = new StringBuffer();
    URL url = new URL(urlstr);
    HttpURLConnection con = (HttpURLConnection) url.openConnection();
    InputStream iis = con.getInputStream();
    byte[] bs = new byte[100 * 1024];
    int length = -1;
    while ((length = iis.read(bs, 0, bs.length)) != -1) {
        result = new String(bs, 0, length);
        sb.append(result);
    }
    Gson g = new Gson();
    jsonModel = g.fromJson(sb.toString(), JsonModel.class);
    iis.close();
    return jsonModel;
}
```

#### 2.2.RestTemplate

RestTemplate是Spring已经封装好的Rest风格的对象，实现方式如下

```java
//配置类
@Configuration
public class WebConfigs {
    @Bean
    public RestTemplate  restTemplate(){
        return new RestTemplate();
    }
}
```

```java
private final String BASE_URL_GET = "http://localhost:8081/product/get/";

@Autowired
private RestTemplate restTemplate;

@RequestMapping("/product/get/{pid}")
public JsonModel get(@PathVariable("pid") Integer pid, JsonModel jsonModel) throws IOException {
    String url = BASE_URL_GET + pid;

    //(请求路径，响应数据类型)
    jsonModel = this.restTemplate.getForObject(url, JsonModel.class);
    //postForObject、exchange、delete
    return jsonModel;
}
```

#### 2.3.其他

其他方式还有Apache的HttpClient、Netty的异步HttpClient、Feign，其中Feign会在后续部分讲解

## 四、SpringCloud生态介绍

外部或内部的非SpringCloud项目都统一通过**Zuul网关**来访问内部服务，网关接收到请求后从**注册中心Eureka**获取可用服务，由**Ribbon均衡负载**后分发到后端的具体实例，微服务之间通过**Feign**进行通信处理业务，**Hystrix**负责处理服务超时熔断，**Turbine**监控服务间的调用和熔断相关指标

生态部分提到的关键技术在后续文章中介绍！





















































