### @SpringBootApplication

申明让Spring boot 自动给程序进行必要的配置，这个配置等于：@Configuration ，@EnableAutoConfiguration 和 @ComponentScan

---

### @Controller

用于定义控制类，在Spring项目中将用户发来的url请求转发到对应的服务器接口（Service层）

---

### @RestController

用于标注控制层组件，等同于@ResponseBody+@Controller

---

### @RequestMapping

提供路由信息，负责映射url到具体函数，其组合注解如下：

* @GetMapping

* @PostMapping

* @DeleteMapping

* @PutMapping

---

### @Autowired

自动导入依赖的bean

---

### @Value

注入application中配置的属性值

`@Value(value = “#{message}”)`

`private String message;`

---

### @Bean

用@Bean标注方法等价于XML中配置的bean。

---

### @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

---

### @PathVariable

获取url中的数据

`RequestMapping(“user/get/mac/{macAddress}”)`

`public String getByMacAddress(@PathVariable String macAddress){`

`//do something;`

`}`

---

### @RequestParam

获取请求中传过来的值

`@RequestParam(Value = "id", required = false, defaultValue =0)`

---

### @ConfigurationProperties

将配置文件转成对象，yml配置：

`spring:`

`redis:`

`dbIndex: 0`

`hostName: 192.168.58.133`

`password: nmamtf`

`port: 6379`

`timeout: 0`

`poolConfig:`

`- maxIdle: 8`

`- minIdle: 0`

`- maxActive: 8`

`- maxWait: -1`

定义转换对象：

`@Component`

`@ConfigurationProperties(prefix="spring.redis")`

`public class RedisProps {`

`private int dbIndex;`

`@NotNull`

`private String hostname;`

`private String password;`

`@NotNull`

`private int port;`

`private long timeout;`

`private List<Map<String,String>> poolConfig;`

`  
`

`public int getDbIndex() {`

`return dbIndex;`

`}`

`public void setDbIndex(int dbIndex) {`

`this.dbIndex = dbIndex;`

`}`

`public String getHostname() {`

`return hostname;`

`}`

`public void setHostname(String hostname) {`

`this.hostname = hostname;`

`}`

`public String getPassword() {`

`return password;`

`}`

`public void setPassword(String password) {`

`this.password = password;`

`}`

`public int getPort() {`

`return port;`

`}`

`public void setPort(int port) {`

`this.port = port;`

`}`

`public long getTimeout() {`

`return timeout;`

`}`

`public void setTimeout(long timeout) {`

`this.timeout = timeout;`

`}`

`public List<Map<String, String>> getPoolConfig() {`

`return poolConfig;`

`}`

`public void setPoolConfig(List<Map<String, String>> poolConfig) {`

`this.poolConfig = poolConfig;`

`}`

`}`

---

### @Transactional

数据库的事物管理，若是同时插入两条信息，失败则会同时回滚（查询的时候不用加）\#\# @SpringBootApplication

申明让Spring boot 自动给程序进行必要的配置，这个配置等于：@Configuration ，@EnableAutoConfiguration 和 @ComponentScan

---

### @Controller

用于定义控制类，在Spring项目中将用户发来的url请求转发到对应的服务器接口（Service层）

---

### @RestController

用于标注控制层组件，等同于@ResponseBody+@Controller

---

### @RequestMapping

提供路由信息，负责映射url到具体函数，其组合注解如下：

* @GetMapping

* @PostMapping

* @DeleteMapping

* @PutMapping

---

### @Autowired

自动导入依赖的bean

---

### @Value

注入application中配置的属性值

`@Value(value = “#{message}”)`

`private String message;`

---

### @Bean

用@Bean标注方法等价于XML中配置的bean。

---

### @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

---

### @PathVariable

获取url中的数据

`RequestMapping(“user/get/mac/{macAddress}”)`

`public String getByMacAddress(@PathVariable String macAddress){`

`//do something;`

`}`

---

### @RequestParam

获取请求中传过来的值

`@RequestParam(Value = "id", required = false, defaultValue =0)`

---

### @ConfigurationProperties

将配置文件转成对象，yml配置：

```
spring:
  redis:
    dbIndex: 0

    hostName: 192.168.58.133

    password: nmamtf

    port: 6379

    timeout: 0

    poolConfig: 

      - maxIdle: 8

      - minIdle: 0

      - maxActive: 8

      - maxWait: -1
```

定义转换对象：

```
@Component
@ConfigurationProperties(prefix="spring.redis")
public class RedisProps {
        private int dbIndex;

        @NotNull

        private String hostname;

        private String password;

        @NotNull

        private int port;

        private long timeout;

        private List&lt;Map&lt;String,String&gt;&gt; poolConfig;



        public int getDbIndex\(\) {

            return dbIndex;

        }

        public void setDbIndex\(int dbIndex\) {

            this.dbIndex = dbIndex;

        }

        public String getHostname\(\) {

            return hostname;

        }

        public void setHostname\(String hostname\) {

            this.hostname = hostname;

        }

        public String getPassword\(\) {

            return password;

        }

        public void setPassword\(String password\) {

            this.password = password;

        }

        public int getPort\(\) {

            return port;

        }

        public void setPort\(int port\) {

            this.port = port;

        }

        public long getTimeout\(\) {

            return timeout;

        }

        public void setTimeout\(long timeout\) {

            this.timeout = timeout;

        }

        public List&lt;Map&lt;String, String&gt;&gt; getPoolConfig\(\) {

            return poolConfig;

        }

        public void setPoolConfig\(List&lt;Map&lt;String, String&gt;&gt; poolConfig\) {

            this.poolConfig = poolConfig;

        }
}
```

---

### @Transactional

数据库的事物管理，若是同时插入两条信息，失败则会同时回滚（查询的时候不用加）

