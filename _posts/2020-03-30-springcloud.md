---
title: SpringCloud入门
tag:
  - springcloud
---

更新中。。。<!--more-->

## 微服务架构理论入门

将单一应用划分成一组小的服务，服务之间相互协调配合。

### 基本包含

* 服务注册与发现
* 服务调用
* 服务熔断
* 负载均衡
* 服务降级
* 服务消息队列
* 配置中心管理
* 服务网关
* 服务监控
* 全链路追踪
* 自动化构建部署
* 服务定时任务调度操作
* 。。。

强在于一个整体

核心（时至2020-03）：

* 服务注册与发现 `Nacos` `ZK` `Consul`
* 服务负载与调用 `Netflix OSS` `ribbon` `OpenFeign` `LoadBalancer`
* 熔断降级 `× Hystix` `resilience4j` `Alibaba Sentinel`
* 网关 `gateway` `× Zuul`
* 分布式配置 `Nacos`
* 总线 `Nacos`
* 开发 `boot`

### SpringCloud

分布式架构的一站式解决方案，是多种微服务架构落地技术的几何体

## Maven

### 父POM

样例

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

<!--  GAV maven坐标-->
  <groupId>com.vwmin.springcloud</groupId>
  <artifactId>cloud2020</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

<!--  统一jar包和版本号的管理-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <lombok.version>1.16.18</lombok.version>
    <mysql.version>5.1.47</mysql.version>
    <druid.version>1.1.16</druid.version>
    <mybatis.srping.boot.version>1.3.0</mybatis.srping.boot.version>
  </properties>

<!--  子模块继承后，提供作用：锁定版本 子module不用谢groupID和version；该过程只声明依赖而不引入-->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-project-info-reports-plugin</artifactId>
        <version>3.0.0</version>
      </dependency>
      <!--spring boot 2.2.2-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.2.5.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud Hoxton.SR1-->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR3</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud 阿里巴巴-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>2.1.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--mysql-->
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
        <scope>runtime</scope>
      </dependency>
      <!-- druid-->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid.version}</version>
      </dependency>
      <!--mybatis-->
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.spring.boot.version}</version>
      </dependency>
      <!--junit-->
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
      </dependency>
      <!--log4j-->
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

## IDEA调整

### 自动热部署

引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

添加插件（可直接放到父POM

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

开启自动编译选项

`build, Exception, Deployment` 启用 `build project automatically` `compile independent modules`

~~算了，这玩意根本不好用~~

## Nacos

服务注册和配置中心

[下载](https://github.com/alibaba/nacos/releases)

单例启动

```shell
bash startup.sh -m standalone
```

### 服务注册

yaml配置

```yml
spring:
  application:
    name: cloud-payment-service
  cloud:
    nacos:
      discovery:
        server-addr: ip:8848

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

主启动类 `@EnableDiscoveryClient`

#### CP AP

todo

### 配置中心

#### 基础配置

```yaml
cloud:
    nacos:
      discovery:
        server-addr: ip:8848
      config:
        server-addr: ip:8848
        file-extension: yaml
```

设置`dataId`: `${prefix}-${spring.profile-actice}.${file-extension}`

`prefix`默认为 `spring.application.name`，也可配置为 `spring.cloud.nacos.config.prefix`

通过 `@RefreshScope`实现自动更新

进入Nacos编写配置

#### 分类配置

Nacos默认的namespace是`public`，namespace主要用来实现隔离，比方说生产环境、测试环境、开发环境之间

默认的group是`DEFAULT_GROUP`，group可以吧不同的微服务划分到同一个分组里面去

service就是微服务，一个service可以包含多个cluster，Nacos默认的cluster是`DEFAULT`，cluster是对指定微服务的一个虚拟划分；比方说为了容灾，将service微服务分别部署在杭州机房和广州机房，这时就可以给杭州机房的service微服务起集群名称`HZ`

instance是微服务的实例



case

DataId方案：默认namespace+默认group，使用`spring.profile.active`来进行多环境下配置文件的读取

Group方案：`spring.cloud.nacos.config.group`

Namespace方案: `spring.cloud.nacos.config.namespace`

### 切换数据库

`conf/nacos-mysql.sql` 复制运行

`conf/application.properties`修改db相关

#### 集群

`unable to find local peer`: 需要保证`cluster.conf`里的配置与`hostname -i`的结果一致

修改hostname ：`hostnamectl set-hostname xxx`

以上都是废话，用这个 `JAVA_OPT="${JAVA_OPT} -Dnacos.server.ip=ip"`

#### nginx配置

`nginx -t`检查conf正确性

```nginx
upstream cluster{
      server 203..233:8848;
      server 120..118:8848;
      server 121..118:8848;
  }
  
server {
      listen 8847;
      server_name 121..118:8847;
  
      location / {
          proxy_pass http://cluster;
      }
 }

```





