# 一、Hystrix Dashboard应用

## Hystrix Dashboard服务

创建SpringBoot工程，SpringBoot版本2.0.3.RELEASE，SpringCloud版本Finchley.RELEASE

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

启动类

```
@EnableHystrixDashboard
@SpringBootApplication
public class DashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(DashboardApplication.class, args);
    }
}
```

配置文件

```
spring:
  application:
    name: dashboard
server:
  port: 11000
```

## 服务实例

通过访问服务的/actuator/hystrix.stream接口来实现监控，需要服务实例添加这个endpoint。

添加依赖

```

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

启动类

通过添加```@EnableCircuitBreaker```或者```@EnableHystrix```开启断路器功能


配置文件，添加配置

```
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

访问 Hystrix Dashboard 并开启对 http://localhost:9000/actuator/hystrix.stream 的监控，就可以看到该服务的监控信息。

# 二、Turbine应用

单独的使用Hystrix Dashboard只能监控单一服务，想要监控多个服务的话得结合turbine一块使用，Hystrix Dashboard通过监控turbine获取到所有服务的聚合监控数据。

## Http收集监控信息

创建SpringBoot工程，SpringBoot版本2.0.3.RELEASE，SpringCloud版本Finchley.RELEASE

添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
```

启动类

```
@EnableTurbine
@SpringBootApplication
public class TurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class, args);
    }
}
```

配置文件

需要将turbine注册到服务注册中心

```
spring:
  application:
    name: turbine
server:
  port: 8080
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
turbine:
  app-config: consumer,consumer1
  cluster-name-expression: new String("default")
  combine-host-port: true
```

参数说明

* turbine.app-config参数指定了需要收集监控信息的服务名，多个使用逗号分隔
* turbine.cluster-name-expression 参数指定了集群名称为 default，当我们服务数量非常多的时候，可以启动多个 Turbine 服务来构建不同的聚合集群，而该参数可以用来区分这些不同的聚合集群，同时该参数值可以在 Hystrix 仪表盘中用来定位不同的聚合集群，只需要在 Hystrix Stream 的 URL 中通过 cluster 参数来指定；
* turbine.combine-host-port参数设置为true，可以让同一主机上的服务通过主机名与端口号的组合来进行区分，默认情况下会以 host 来区分不同的服务，这会使得在本地调试的时候，本机上的不同服务聚合成一个服务来统计。

访问 Hystrix Dashboard 并开启对 http://localhost:8081/actuator/hystrix.stream 的监控，就可以看到所有服务的监控信息。

## 消息代理收集监控信息

使用消息代理还需要修改服务实例的POM文件

引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-hystrix-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Turbine服务，修改POM

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

启动类，使用```@EnableTurbineStream```注解

```
@EnableTurbineStream
@SpringBootApplication
public class TurbineStreamApplication {

    public static void main(String[] args) {
        SpringApplication.run(TurbineStreamApplication.class, args);
    }
}
```

配置文件把以下段去掉

```
turbine:
  app-config: consumer,consumer1
  cluster-name-expression: new String("default")
  combine-host-port: true
```

踩坑：http://localhost:8080/turbine.stream 一直处于 ```data:{"type":"ping"}``` 的状态，但当消费者请求提供者之后，数据就出来了。

注：以上使用的是本地Rabbitmq，指定其他地址使用下面配置

```
spring:
  rabbitmq:
    host: ${RABBIT_MQ_HOST:localhost}
      port:  ${RABBIT_MQ_PORT:5672}
      username: ${RABBIT_MQ_USER:guest}
      password: ${RABBIT_MQ_PASS:guest}
```


示例：http://gitlab.utech.com/Vick.Zeng/spring-cloud-demo.git

# 三、参考

[Spring Cloud（五）：Hystrix 监控面板](https://windmt.com/2018/04/16/spring-cloud-5-hystrix-dashboard/)

[Spring Cloud（六）：Hystrix 监控数据聚合 Turbine](https://windmt.com/2018/04/17/spring-cloud-6-turbine/)
