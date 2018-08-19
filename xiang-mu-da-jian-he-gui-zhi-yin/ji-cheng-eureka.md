# 集成Eureka

Author: Huston.Peng

注：这里说的Eureka通常都是指Eureka客户端。

## 1. maven依赖

```xml
<!-- 引入spring cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- eureka client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 2. 使用client

使用注解：@EnableEurekaClient

```java
@EnableEurekaClient
@SpringBootApplication(scanBasePackages = {"com.utech.*", "com.sigma.*"})
public class TravelPortMsApplication {
   //...
}
```

## 3. 注册到eureka-server

application.yml

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://172.18.21.232:20001/eureka/,http://172.18.21.233:20002/eureka/
```

如果还需要自定义配置的，可以参考官方文档。

