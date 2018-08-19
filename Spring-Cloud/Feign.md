# 一、Ribbon负载均衡

在一个分布式的应用当中，一般有三种角色，服务注册中心、服务提供者、服务消费者，这些服务都可以水平扩展，那么当一个消费者对应多个提供者时，就必定需要一种负载均衡策略。

## 服务提供者

创建个服务，提供接口```/producer/test```，返回```producer```

引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

启动类
添加```@SpringBootApplication```注解即可，无需添加```@EnableDiscoveryClient```注解（那么如果我引入了相关的 jar 包又想禁用服务注册与发现怎么办？设置eureka.client.enabled=false）

```
@SpringBootApplication
public class ProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }
}
```

配置文件

```
spring:
  application:
    name: producer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
server:
  port: 8000
```

复制上面的服务，修改端口，接口返回```producer1```

## 服务消费者

创建个服务，提供接口```/consumer/test```，调用服务提供者的```/producer/test```接口

有两种服务调用方式，一种是使用RestTemplate，一种是使用Feign

## RestTemplate

基于Ribbon，spring-cloud-starter-netflix-eureka-client 里边包含了 spring-cloud-starter-netflix-ribbon


依赖、启动类、配置文件与服务提供者一致

需要初始化RestTemplate Bean在Spring上下文中

```
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

调用

```
@GetMapping("/restTemplateTest")
public String restTemplateTest() {
    return restTemplate.getForObject("http://producer/producer/test", String.class);
}
```

## Fegin

在上面的基础上添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

启动类添加```@EnableFeignClients```注解

调用实现

```
@FeignClient(value = "producer", fallback = ProducerClientFallBack.class)
public interface ProducerClient {
    @GetMapping("/producer/test")
    String test();
}
```

```
@GetMapping("/feignTest")
public String feignTest() {
    return producerClient.test();
}
```

启动注册中心、两个提供者、消费者，多次访问消费者测试接口，结果在```producer```、```producer1```交替出现，这是因为Ribbon默认的负载均衡策略是轮询的。

### Ribbon本身提供了下面几种负载均衡策略：

* RoundRobinRule: 轮询策略，Ribbon以轮询的方式选择服务器，这个是默认值。所以示例中所启动的两个服务会被循环访问;
* RandomRule: 随机选择，也就是说Ribbon会随机从服务器列表中选择一个进行访问;
* BestAvailableRule: 最大可用策略，即先过滤出故障服务器后，选择一个当前并发请求数最小的;
* WeightedResponseTimeRule: 带有加权的轮询策略，对各个服务器响应时间进行加权处理，然后在采用轮询的方式来获取相应的服务器;
* AvailabilityFilteringRule: 可用过滤策略，先过滤出故障的或并发请求大于阈值一部分服务实例，然后再以线性轮询的方式从过滤后的实例清单中选出一个;
* ZoneAvoidanceRule: 区域感知策略，先使用主过滤条件（区域负载器，选择最优区域）对所有实例过滤并返回过滤后的实例清单，依次使用次过滤条件列表中的过滤条件对主过滤条件的结果进行过滤，判断最小过滤数（默认1）和最小过滤百分比（默认0），最后对满足条件的服务器则使用RoundRobinRule(轮询方式)选择一个服务器实例。
* RetryRule：重试策略，在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server。

通过在消费者服务添加配置指定策略

```
producer: #服务提供者名
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

可以通过继承ClientConfigEnabledRoundRobinRule，来实现自己负载均衡策略。

看网上的博客跟官网都说RoundRobinRule是默认的策略，可是在DEBUG的时候发现并没有走```RoundRobinRule-chose```方法，走的是```PredicateBasedRule-AbstractServerPredicate-chooseRoundRobinAfterFiltering```，看源码也是轮询选择一个服务器。

# 二、Hystrix服务熔断

对于查询操作，我们可以实现一个 fallback 方法，当请求后端服务出现异常的时候，可以使用 fallback 方法返回的值。fallback 方法的返回值一般是设置的默认值或者来自缓存。

在原本的consumer项目添加配置

```
feign:
  hystrix:
    enabled: true
```

调用实现添加fallback

```
@FeignClient(value = "producer", fallback = ProducerClientFallBack.class)
public interface ProducerClient {
    @GetMapping("/producer/test")
    String test();
}
```

实现类

```
@Service
public class ProducerClientFallBack implements ProducerClient {
    @Override
    public String test() {
        return "consumer";
    }
}
```

启动注册中心、消费者，访问消费者接口，返回consumer，启动服务提供者，访问消费者接口，返回producer。


示例：http://gitlab.utech.com/Vick.Zeng/spring-cloud-demo.git

# 三、参考

[Spring Cloud（三）：服务提供与调用 Eureka【Finchley 版】](https://windmt.com/2018/04/15/spring-cloud-3-service-producer-and-consumer/)

[Spring Cloud（四）：服务容错保护 Hystrix【Finchley 版】](https://windmt.com/2018/04/15/spring-cloud-4-hystrix/)

[spring cloud中Ribbon自定义负载均衡策略](https://blog.csdn.net/liuchuanhong1/article/details/54693124)

[Spring Cloud入门教程(二)：客户端负载均衡(Ribbon)](https://www.jianshu.com/p/df9393755a05)

[Ribbon Working with load balancers
](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers)
