# 集成Log

Author: Huston

* 简介
* Usage
 - maven配置
 - logback-spring
 - debug,error,info
 - xml
 - permance
 - query 

## 1. 简介

utech的日志运行设施是基于ELK架构的。

每个微服务系统如何将日志打到 ELK ？ 那就是靠**一个**配置文件。想了解更多，可以咨询 Jason。

大致流程：

Slf4j (log.info/log.trace/log.error) --> Appender --> log api --> kafka --> es.

## 2. Usage

### 2.1 maven 配置

```xml
<dependency>
    <groupId>com.sigma</groupId>
    <artifactId>sigma-core</artifactId>
    <version>1.0.1</version>
</dependency>

<dependency>
    <groupId>com.sigma</groupId>
    <artifactId>sigma-logging</artifactId>
    <version>1.0.1</version>
</dependency>
```

repository：
```xml
<repositories>
        <!-- 配置nexus远程仓库 -->
        <repository>
            <id>nexus</id>
            <name>Nexus Snapshot Repository</name>
            <url>http://172.18.21.224:8081/repository/maven-public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

### 2.2 resources下面创建logback-spring.xml

coupon系统的示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <appender name="SigmaAppender" class="com.sigma.logging.SigmaLoggingAppender">
        <systemCode>coupon</systemCode>
        <sourceCode>coupon-ms</sourceCode>
        <logServiceUrl>http://172.18.21.228:40003/log/</logServiceUrl>
    </appender>

    <!-- 异步输出 -->
    <appender name="SigmaAppenderAsync" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="SigmaAppender"/>
    </appender>

    <springProfile name="default,test,dev,prod">
        <logger name="com.sigma" level="DEBUG">
            <appender-ref ref="SigmaAppenderAsync"/>
        </logger>

        <logger name="com.utech" level="DEBUG">
            <appender-ref ref="SigmaAppenderAsync"/>
        </logger>
    </springProfile>

</configuration>
```

解释：
* systemCode：系统代码
* source：模块
* logServiceUrl：log api地址。


### 2.3 logging as usual

```java
@Service
@Slf4j
public class AirShopServiceImpl implements AirShopService {
}

@Override
public LowFareSearchResponse lowFareSearch(LowFareSearchRequest lowFareSearchRequest) throws AirFaultMessage, JAXBException {

    LowFareSearchResponse result = null;
    try {
        var request = AirShopFactory.createLowFareSearchReq(lowFareSearchRequest);
        var response = airLowFareSearchPortType.service(request, null);

        log.trace("LFS请求", new XmlLoggingModel(JsonUtils.serialize(lowFareSearchRequest), JaxbUtil.toXmlDocument(request), JaxbUtil.toXmlDocument(response)));

        result = new LowFareSearchResponse();
        result.setOriginApiData(response);

        redisTemplate.opsForValue().set(RedisCacheKeyConfig.getLowfareSearchKey(lowFareSearchRequest.getTraceId()), JsonUtils.serialize(response));

    } catch (IOException e) {
        log.error("IOException occurred", e);
    }
    return result;
}
```
* 首先引入logger，如@Slf4j
* 其次，直接调用日志api，log.trace，log.debug，log.info，log.error。

### 2.4 xml日志如何打？

1. 使用com.sigma.logging.XmlLogAppender
2. log.trace("message", new XmlLoggingModel());

参考配置：http://gitlab.utech.com/utech-online/travel-port-ms/blob/feature/1.1/src/main/resources/logback-spring.xml

### 2.5 性能日志如何打？

目前还没有集成，等待Email。。。

### 2.6 怎么查询日志

DEV环境：http://172.18.21.222:5601/app/kibana

![](/assets/Log_sample.png)