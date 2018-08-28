# 一、zipkin

## 快速启动

zipkin分为服务器端和客户端，客户端就是微服务中的应用.

在 Spring Cloud Sleuth 中对 Zipkin 的整合进行了自动化配置的封装，我们可以很轻松的引入和使用它。

### 服务器端

首先搭建个Zipkin的工程

POM文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.utech.trace</groupId>
    <artifactId>trace</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.5.0</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <version>2.5.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

启动类

```
package com.utech.trace;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import zipkin.server.EnableZipkinServer;

@SpringBootApplication
@EnableZipkinServer
public class TraceApplication {
    public static void main(String[] args) {
        SpringApplication.run(TraceApplication.class,args);
    }
}
```

配置文件

```
spring:
  application:
    name: trace
   # 表示当前程序不使用sleuth
  sleuth:
    enabled: false

server:
  port: 9411
```

启动后访问 [http://localhost:9411/zipkin/](http://localhost:9411/zipkin/) 就能看到界面

### 客户端

默认我们已经有了基本的微服务工程，想要微服务之间的调用被zipkin发现，需要在服务工程的POM中引入依赖、修改配置文件

添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

添加配置

```
spring:
  sleuth:
    web:
      client:
        enabled: true
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    base-url: http://localhost:9411/ # 指定 Zipkin 服务器的地址
```

> Spring Cloud Sleuth 有一个 Sampler 策略，可以通过这个实现类来控制采样算法。采样器不会阻碍 span 相关 id 的产生，但是会对导出以及附加事件标签的相关操作造成影响。 Sleuth 默认采样算法的实现是 Reservoir sampling，具体的实现类是 PercentageBasedSampler，默认的采样比例为: 0.1\(即 10%\)。不过我们可以通过spring.sleuth.sampler.percentage来设置，所设置的值介于 0.0 到 1.0 之间，1.0 则表示全部采集。

## 存储方式

Zipkin 提供了可插拔数据存储方式：In-Memory、MySql、Cassandra 以及 Elasticsearch。  
上面示例的存储方式就是In-Memory，下面我们介绍下MySql的存储方式。

只需要添加服务端依赖、配置，不需要修改客户端

```
<!--保存到数据库需要如下依赖-->
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-storage-mysql</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

配置文件

```
spring:
  datasource:
    url: jdbc:mysql://${MYSQL_HOST:localhost}:${MYSQL_PORT:3306}/zipkin?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: 
    password: 
    driver-class-name: com.mysql.jdbc.Driver
    continue-on-error: true
  application:
    name: trace
  # 表示当前程序不使用sleuth
  sleuth:
    enabled: false

# zipkin数据保存到数据库中需要进行如下配置
zipkin:
  # 表示zipkin数据存储方式是mysql
  storage:
    type: mysql
server:
  port: 9411
```

## 发送方式

客户端发送链路调用的方式主要有两种，一种是 HTTP 报文的方式，还有一种是消息总线的方式如 RabbitMQ。

上面示例的发送方式就是HTTP的，下面我们介绍下RabbitMQ的存储方式。

修改服务端POM、配置文件

添加依赖

```
<!-- 使用消息的方式收集数据（使用rabbitmq） -->
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-collector-rabbitmq</artifactId>
    <version>2.5.0</version>
</dependency>
```

相关配置

```
# zipkin数据保存到数据库中需要进行如下配置
zipkin:
  # 表示zipkin数据存储方式是mysql
  storage:
    type: mysql
  collector:
    rabbitmq:
      addresses: ${RABBIT_MQ_HOST:127.0.0.1}
      port: ${RABBIT_MQ_PORT:5672}
      password: 
      username: 
      queue: zipkin2
```

修改客户端POM、配置文件

添加依赖

```
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
```

相关配置

```
spring:
  zipkin:
    rabbitmq:
      # 默认队列名为zipkin，需要修改队列名可以修改此配置
      queue: zipkin2
```

示例：[http://gitlab.utech.com/Vick.Zeng/trace.git](http://gitlab.utech.com/Vick.Zeng/trace.git)

# 二、SpringBoot2.0后版本zipkin

关于 Zipkin 的服务端，在使用 Spring Boot 2.x 版本后，官方就不推荐自行定制编译了，反而是直接提供了编译好的 jar 包来给我们使用，详情请看 [upgrade to Spring Boot 2.0 NoClassDefFoundError UndertowEmbeddedServletContainerFactory · Issue \#1962 · openzipkin/zipkin · GitHub](https://github.com/openzipkin/zipkin/issues/1962)

并且以前的`@EnableZipkinServer`也已经被打上了`@Deprecated`

## 启动

在Linux下中使用以下脚本下载jar包，如下\(PS：并没有找到直接下载jar包的链接，只能在Linux上进行\)

`curl -sSL https://zipkin.io/quickstart.sh | bash -s`

把jar包下载下来之后使用命令启动，以下是收集使用rabbitmq、存储方式使用mysql的示例

```
java -jar zipkin.jar --RABBIT_ADDRESSES=172.18.21.219:5672 --RABBIT_PASSWORD=admin --RABBIT_USER=admin --STORAGE_TYPE=mysql --MYSQL_HOST=172.18.21.217 --MYSQL_USER=utech --MYSQL_PASS=Utech2018 --MYSQL_DB=utech_zipkin
```

linux 启动也可以吧参数放前面（PS：为什么？）

```
RABBIT_ADDRESSES=172.18.21.219:5672 RABBIT_PASSWORD=admin RABBIT_USER=admin STORAGE_TYPE=mysql MYSQL_HOST=172.18.21.217 MYSQL_USER=utech MYSQL_PASS=Utech2018 MYSQL_DB=utech_zipkin  java -jar zipkin.jar
```

如果是Docker的话

`docker run -d -p 9411:9411 openzipkin/zipkin`

全部配置信息

```
zipkin:
  self-tracing:
    # Set to true to enable self-tracing.
    enabled: ${SELF_TRACING_ENABLED:false}
    # percentage to self-traces to retain
    sample-rate: ${SELF_TRACING_SAMPLE_RATE:1.0}
    # Timeout in seconds to flush self-tracing data to storage.
    message-timeout: ${SELF_TRACING_FLUSH_INTERVAL:1}
  collector:
    # percentage to traces to retain
    sample-rate: ${COLLECTOR_SAMPLE_RATE:1.0}
    http:
      # Set to false to disable creation of spans via HTTP collector API
      enabled: ${HTTP_COLLECTOR_ENABLED:true}
    kafka:
      # Kafka bootstrap broker list, comma-separated host:port values. Setting this activates the
      # Kafka 0.10+ collector.
      bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS:}
      # Name of topic to poll for spans
      topic: ${KAFKA_TOPIC:zipkin}
      # Consumer group this process is consuming on behalf of.
      group-id: ${KAFKA_GROUP_ID:zipkin}
      # Count of consumer threads consuming the topic
      streams: ${KAFKA_STREAMS:1}
    rabbitmq:
      # RabbitMQ server address list (comma-separated list of host:port)
      addresses: ${RABBIT_ADDRESSES:}
      concurrency: ${RABBIT_CONCURRENCY:1}
      # TCP connection timeout in milliseconds
      connection-timeout: ${RABBIT_CONNECTION_TIMEOUT:60000}
      password: ${RABBIT_PASSWORD:guest}
      queue: ${RABBIT_QUEUE:zipkin}
      username: ${RABBIT_USER:guest}
      virtual-host: ${RABBIT_VIRTUAL_HOST:/}
      useSsl: ${RABBIT_USE_SSL:false}
      uri: ${RABBIT_URI:}
  query:
    enabled: ${QUERY_ENABLED:true}
    # 1 day in millis
    lookback: ${QUERY_LOOKBACK:86400000}
    # The Cache-Control max-age (seconds) for /api/v2/services and /api/v2/spans
    names-max-age: 300
    # CORS allowed-origins.
    allowed-origins: "*"

  storage:
    strict-trace-id: ${STRICT_TRACE_ID:true}
    search-enabled: ${SEARCH_ENABLED:true}
    type: ${STORAGE_TYPE:mem}
    mem:
      # Maximum number of spans to keep in memory.  When exceeded, oldest traces (and their spans) will be purged.
      # A safe estimate is 1K of memory per span (each span with 2 annotations + 1 binary annotation), plus
      # 100 MB for a safety buffer.  You'll need to verify in your own environment.
      # Experimentally, it works with: max-spans of 500000 with JRE argument -Xmx600m.
      max-spans: 500000
    cassandra:
      # Comma separated list of host addresses part of Cassandra cluster. Ports default to 9042 but you can also specify a custom port with 'host:port'.
      contact-points: ${CASSANDRA_CONTACT_POINTS:localhost}
      # Name of the datacenter that will be considered "local" for latency load balancing. When unset, load-balancing is round-robin.
      local-dc: ${CASSANDRA_LOCAL_DC:}
      # Will throw an exception on startup if authentication fails.
      username: ${CASSANDRA_USERNAME:}
      password: ${CASSANDRA_PASSWORD:}
      keyspace: ${CASSANDRA_KEYSPACE:zipkin}
      # Max pooled connections per datacenter-local host.
      max-connections: ${CASSANDRA_MAX_CONNECTIONS:8}
      # Ensuring that schema exists, if enabled tries to execute script /zipkin-cassandra-core/resources/cassandra-schema-cql3.txt.
      ensure-schema: ${CASSANDRA_ENSURE_SCHEMA:true}
      # 7 days in seconds
      span-ttl: ${CASSANDRA_SPAN_TTL:604800}
      # 3 days in seconds
      index-ttl: ${CASSANDRA_INDEX_TTL:259200}
      # the maximum trace index metadata entries to cache
      index-cache-max: ${CASSANDRA_INDEX_CACHE_MAX:100000}
      # how long to cache index metadata about a trace. 1 minute in seconds
      index-cache-ttl: ${CASSANDRA_INDEX_CACHE_TTL:60}
      # how many more index rows to fetch than the user-supplied query limit
      index-fetch-multiplier: ${CASSANDRA_INDEX_FETCH_MULTIPLIER:3}
      # Using ssl for connection, rely on Keystore
      use-ssl: ${CASSANDRA_USE_SSL:false}
    cassandra3:
      # Comma separated list of host addresses part of Cassandra cluster. Ports default to 9042 but you can also specify a custom port with 'host:port'.
      contact-points: ${CASSANDRA_CONTACT_POINTS:localhost}
      # Name of the datacenter that will be considered "local" for latency load balancing. When unset, load-balancing is round-robin.
      local-dc: ${CASSANDRA_LOCAL_DC:}
      # Will throw an exception on startup if authentication fails.
      username: ${CASSANDRA_USERNAME:}
      password: ${CASSANDRA_PASSWORD:}
      keyspace: ${CASSANDRA_KEYSPACE:zipkin2}
      # Max pooled connections per datacenter-local host.
      max-connections: ${CASSANDRA_MAX_CONNECTIONS:8}
      # Ensuring that schema exists, if enabled tries to execute script /zipkin2-schema.cql
      ensure-schema: ${CASSANDRA_ENSURE_SCHEMA:true}
      # how many more index rows to fetch than the user-supplied query limit
      index-fetch-multiplier: ${CASSANDRA_INDEX_FETCH_MULTIPLIER:3}
      # Using ssl for connection, rely on Keystore
      use-ssl: ${CASSANDRA_USE_SSL:false}
    elasticsearch:
      # host is left unset intentionally, to defer the decision
      hosts: ${ES_HOSTS:}
      pipeline: ${ES_PIPELINE:}
      max-requests: ${ES_MAX_REQUESTS:64}
      timeout: ${ES_TIMEOUT:10000}
      index: ${ES_INDEX:zipkin}
      date-separator: ${ES_DATE_SEPARATOR:-}
      index-shards: ${ES_INDEX_SHARDS:5}
      index-replicas: ${ES_INDEX_REPLICAS:1}
      username: ${ES_USERNAME:}
      password: ${ES_PASSWORD:}
      http-logging: ${ES_HTTP_LOGGING:}
      legacy-reads-enabled: ${ES_LEGACY_READS_ENABLED:true}
    mysql:
      host: ${MYSQL_HOST:localhost}
      port: ${MYSQL_TCP_PORT:3306}
      username: ${MYSQL_USER:}
      password: ${MYSQL_PASS:}
      db: ${MYSQL_DB:zipkin}
      max-active: ${MYSQL_MAX_CONNECTIONS:10}
      use-ssl: ${MYSQL_USE_SSL:false}
  ui:
    enabled: ${QUERY_ENABLED:true}
    ## Values below here are mapped to ZipkinUiProperties, served as /config.json
    # Default limit for Find Traces
    query-limit: 10
    # The value here becomes a label in the top-right corner
    environment:
    # Default duration to look back when finding traces.
    # Affects the "Start time" element in the UI. 1 hour in millis
    default-lookback: 3600000
    # When false, disables the "find a trace" screen
    search-enabled: ${SEARCH_ENABLED:true}
    # Which sites this Zipkin UI covers. Regex syntax. (e.g. http:\/\/example.com\/.*)
    # Multiple sites can be specified, e.g.
    # - .*example1.com
    # - .*example2.com
    # Default is "match all websites"
    instrumented: .*
    # URL placed into the <base> tag in the HTML
    base-path: /zipkin

server:
  port: ${QUERY_PORT:9411}
  use-forward-headers: true
  compression:
    enabled: true
    # compresses any response over min-response-size (default is 2KiB)
    # Includes dynamic json content and large static assets from zipkin-ui
    mime-types: application/json,application/javascript,text/css,image/svg

spring:
  jmx:
     # reduce startup time by excluding unexposed JMX service
     enabled: false
  mvc:
    favicon:
      # zipkin has its own favicon
      enabled: false
  autoconfigure:
    exclude:
      # otherwise we might initialize even when not needed (ex when storage type is cassandra)
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
info:
  zipkin:
    version: "@project.version@"

logging:
  pattern:
    level: "%clr(%5p) %clr([%X{traceId}/%X{spanId}]){yellow}"
  level:
    # Silence Invalid method name: '__can__finagle__trace__v3__'
    com.facebook.swift.service.ThriftServiceProcessor: 'OFF'
#     # investigate /api/v2/dependencies
#     zipkin2.internal.DependencyLinker: 'DEBUG'
#     # log cassandra queries (DEBUG is without values)
#     com.datastax.driver.core.QueryLogger: 'TRACE'
#     # log cassandra trace propagation
#     com.datastax.driver.core.Message: 'TRACE'
#     # log reason behind http collector dropped messages
#     zipkin2.server.ZipkinHttpCollector: 'DEBUG'
#     zipkin2.collector.kafka.KafkaCollector: 'DEBUG'
#     zipkin2.collector.kafka08.KafkaCollector: 'DEBUG'
#     zipkin2.collector.rabbitmq.RabbitMQCollector: 'DEBUG'
#     zipkin2.collector.scribe.ScribeCollector: 'DEBUG'

management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
# Disabling auto time http requests since it is added in Undertow HttpHandler in Zipkin autoconfigure
# Prometheus module. In Zipkin we use different naming for the http requests duration
  metrics:
    web:
      server:
        auto-time-requests: false
```

# 三、参考

[Spring Cloud（十二）：分布式链路跟踪 Sleuth 与 Zipkin【Finchley 版】](https://windmt.com/2018/04/24/spring-cloud-12-sleuth-zipkin/)

[Sleuth with Zipkin over RabbitMQ or Kafka](http://cloud.spring.io/spring-cloud-static/Finchley.RELEASE/multi/multi__introduction.html#_sleuth_with_zipkin_over_rabbitmq_or_kafka)

[Cloud-Admin](https://gitee.com/minull/ace-security)

