##### **一、简单介绍**

##### Spring Cloud Zuul的作用就是路由转发和过滤， 即将请求转发到微服务或拦截请求； Zuul默认集成了负载均衡功能。

##### **二、简单使用**

##### **本项目基于Springboot2.0.3和Spring Cloud Finchley进行部署。**

1.添加依赖pom.xml

```java
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
```

2.配置文件application.yml

```
spring:
  application:
    name: zuul
server:
  port: 8040
```

3.启动类ZuulApplication.java

启动类添加@EnableZuulProxy，即可支持网关路由

```
@EnableZuulProxy
@SpringBootApplication
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }

}
```

4.测试

用浏览器访问localhost:8040，即可测试是否启动项目。

##### 三、服务化

##### Zuul注册到eureka server，通过url的映射转化为服务名ServiceId进行映射。

前提：

* 先启动eureka，端口号为8761
* 再启动微服务account，端口号为9090，注册到eureka，并存在请求localhost:9090/account/test

1.添加依赖pom.xml

添加组建服务eureka注册、发现

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

2.配置文件application.yml

注册到eureka,并且配置路由转发。

serviceId为微服务名称，path为路由地址

stripPrefix为false时，则保留account前缀，再转发url到微服务

```
eureka:
  client:
    service-url:
      defaultZone:   http://localhost:8761/eureka
zuul:
  routes:
    account:
      path: /account/**
      serviceId: account
      stripPrefix: false
```

3.测试

用浏览器访问localhost:8040/account/test，如果通过zuul路由，能访问account服务（端口是9090）的请求，则为成功。

网关zuul服务和account服务注册到eureka，如下图：

![](/assets/QQ截图20180807170054.jpg)

##### 四、跨域问题

##### 转发过滤器前，在HttpServletResponse添加跨域头

1.创建CorsFilter .java

```
@Component
public class CorsFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre"; // 可以在请求被路由之前调用
    }

    @Override
    public int filterOrder() {
        return 0;// filter执行顺序，通过数字指定 ,优先级为0，数字越大，优先级越低
    }

    @Override
    public boolean shouldFilter() {
        return true;// 是否执行该过滤器，此处为true，说明需要过滤
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
            HttpServletResponse res= ctx.getResponse();
            res.setHeader("Access-Control-Allow-Origin", "*");
            res.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE ,PUT");
            res.setHeader("Access-Control-Max-Age", "3600");
            res.setHeader("Access-Control-Allow-Headers", "Origin, No-Cache, X-Requested-With, If-Modified-Since,"
                + " Pragma, Last-Modified, Cache-Control, Expires, Content-Type, "
                + "X-E4M-With,userId,token,Authorization,deviceId,Access-Control-Allow-Origin,Access-Control-Allow-Headers,Access-Control-Allow-Methods");
            res.setHeader("Access-Control-Allow-Credentials", "true");
            return null;
    }
}
```

2.启动类ZuulApplication.java，添加如下

```
    @Bean
    public CorsFilter corsFilter(){
        return new CorsFilter();
    }
```

3.测试

重新启动，即可解决跨域问题。![](/assets/cors.png)

4.注意

* 如果zuul配置了cors跨域问题，则不能在微服务下面在配置cors跨域，不然会继续存在跨域问题。
* Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“pre”、“routing”、“post”、“error”。具体请看教程：[springcloud\(十一\)：服务网关Zuul高级篇](https://blog.csdn.net/u011820505/article/details/79373594)

##### 五、网关权限

核心框架为Spring Cloud security。所有微服务经过网关zuul，进行权限控制，其中包括登陆和注销。

account微服务的接口前提：

* GET /account/user/username/{username} 为根据用户名获取当前用户信息
* GET /account/menu/user/{id} 为根据用户id获取当前用户权限菜单
* POST /account/password/check 为根据输入密码和加密密码进行对比
* GET /account/token/{token} 为根据token，获取当前用户
* POST /account/token 为保存token和用户信息
* POST /account/token/check/{token} 为检测token是否存在
* DELETE /account/token/{token} 删除token

1.添加依赖pom.xml，feign用于访问微服务account

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

2.添加配置

创建全局权限类：WebSecurityConfig.java

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    }

    /**
    *不添加权限的url过滤请求，其中开启swagger
    **/
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .antMatchers(HttpMethod.POST, "/login", "/logout")
                .antMatchers("/swagger-ui.html", "/webjars/**", "/v2/**", "/swagger-resources/**")
                .antMatchers("/actuator/**");

    }    
    /**
    *security基础配置
    **/
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //配置customAccessDecisionManager
        http.authorizeRequests()
                //.requestMatchers(CorsUtils::isPreFlightRequest).permitAll()
                .anyRequest().authenticated()
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O fsi) {
                        fsi.setAccessDecisionManager(customAccessDecisionManager());
                        return fsi;
                    }
                });
        //替换传统的session，用token代替
        http.securityContext().securityContextRepository(tokenSecurityContextRepository());
        //异常信息处理
        http.exceptionHandling()//
                .authenticationEntryPoint(goAuthenticationEntryPoint())
                .accessDeniedHandler(goAccessDeniedHandler());
        //关闭csrf
        http.csrf().disable();
    }

    @Bean
    public AccessDecisionManager customAccessDecisionManager() {
        return new CustomAccessDecisionManager();
    }

    @Bean
    public GoAuthenticationEntryPoint goAuthenticationEntryPoint() {
        return new GoAuthenticationEntryPoint();
    }

    @Bean
    public GoAccessDeniedHandler goAccessDeniedHandler() {
        return new GoAccessDeniedHandler();
    }

    @Bean
    public TokenSecurityContextRepository tokenSecurityContextRepository() {
        return new TokenSecurityContextRepository();
    }

}
```

添加自定义验证类CustomAccessDecisionManager.java，用于对url过滤

```
/**
 * 自定义验证权限
 */
@Component
public class CustomAccessDecisionManager implements AccessDecisionManager {

    //decide 方法是判定是否拥有权限的决策方法
    @Override
    public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) 
            throws AccessDeniedException, InsufficientAuthenticationException {
        HttpServletRequest request = ((FilterInvocation) object).getHttpRequest();
        String url, method;
        AntPathRequestMatcher matcher;
        for (GrantedAuthority ga : authentication.getAuthorities()) {
            if (ga instanceof CustomGrantedAuthority) {
                CustomGrantedAuthority urlGrantedAuthority = (CustomGrantedAuthority) ga;
                url = urlGrantedAuthority.getUrl();
                method = MethodEnum.getNameByCode(urlGrantedAuthority.getMethod());
                matcher = new AntPathRequestMatcher(url);
                if (matcher.matches(request)) {
                    //当权限表权限的method为ALL时表示拥有此路径的所有请求方式权利。
                    if (method.equals(request.getMethod()) || "ALL".equals(method)) {
                        return;
                    }
                }
            }
        }
        throw new AccessDeniedException("denied.error.message");
    }


    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }
}
```

创建自定义权限实体类CustomGrantedAuthority.java，自定义权限url

```
/**
 * 自定义权限实体（Spring security）
 */
@Data
public class CustomGrantedAuthority implements GrantedAuthority {

    private String url;
    private Integer method;
    private String authority;


    @Override
    public String getAuthority() {
        return this.url + ";" + MethodEnum.getNameByCode(this.method);
    }
}
```

创建自定义用户实体（Spring Security\)

```
/**
 * 自定义用户实体（Spring Security）
 */
@Data
public class CustomUserDetails implements UserDetails {
    private Integer id;
    private String userName;
    private String password;
    private String email;
    private String fullName;
    private Integer usable;
    private List<GrantedAuthority> authorities;
    private String token;
    //重写
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.userName;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

创建token管理类TokenSecurityContextRepository.java，重写SecurityContext（Spring Security）,token代替传统session

```
@Component
public class TokenSecurityContextRepository implements SecurityContextRepository {

    @Autowired
    AccountFeignClient accountFeignClient;

    /**
     * 加载用户对象
     *
     * @param httpRequestResponseHolder
     * @return
     */
    @Override
    public SecurityContext loadContext(HttpRequestResponseHolder httpRequestResponseHolder) {
        String token = SystemUtil.getToken(httpRequestResponseHolder.getRequest());
        if (StringUtils.isEmpty(token)) {
            return this.generateNewContext();
        }
        SigmaResponse<CustomUser> response = accountFeignClient.getToken(token);
        CustomUser customUser = response.getData();
        if (customUser == null) {
            return this.generateNewContext();
        }
        CustomUserDetails user = new CustomUserDetails();
        user.setUserName(user.getUsername());
        BeanUtils.copyProperties(customUser, user);
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (MenuAuth menuAuth : customUser.getAuthorities()) {
            CustomGrantedAuthority authority = new CustomGrantedAuthority();
            authority.setMethod(menuAuth.getMethod());
            authority.setUrl(menuAuth.getUrl());
            authorities.add(authority);
        }
        UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(user, null,
             authorities);
        return new SecurityContextImpl(authentication);
    }

    /**
     * 保存用户对象
     *
     * @param securityContext
     * @param httpServletRequest
     * @param httpServletResponse
     */
    @Override
    public void saveContext(SecurityContext securityContext, HttpServletRequest httpServletRequest, 
        HttpServletResponse httpServletResponse) {
    }

    /**
     * 验证用户对象
     *
     * @param httpServletRequest
     * @return
     */
    @Override
    public boolean containsContext(HttpServletRequest httpServletRequest) {
        String token = SystemUtil.getToken(httpServletRequest);
        if (StringUtils.isEmpty(token)) {
            return false;
        }
        SigmaResponse<Boolean> response = accountFeignClient.checkToken(token);
        return response.getData();
    }

    /**
     * 初始化用户对象
     *
     * @return
     */
    protected SecurityContext generateNewContext() {
        return SecurityContextHolder.createEmptyContext();
    }
}
```

创建自定义创建返回异常权限类和未登录类GoAccessDeniedHandler.java和GoAuthenticationEntryPoint.java

```
/**
 * 授权异常，返回信息
 * 未登录，会返回
 */
@Component
public class GoAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Autowired
    MessageService messageService;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException exception) throws IOException {
        response.setHeader("Content-Type", "application/json;charset=utf-8");
        SigmaResponse<String> sigmaResponse = new SigmaResponse<>();
        sigmaResponse.setHeader(new SigmaResponseHeader("-1", messageService.getMessage("nologin.error.message", 
            request), null));
        response.getWriter().print(new ObjectMapper().writeValueAsString(sigmaResponse));
        response.getWriter().flush();
    }
}
```

```
/**
 * 权限异常，返回信息
 * 无菜单权限，跨角色的权限操作会报进入此类
 */
@Component
public class GoAccessDeniedHandler implements AccessDeniedHandler {
    @Autowired
    MessageService messageService;

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException exception) throws IOException {
        response.setHeader("Content-Type", "application/json;charset=utf-8");
        SigmaResponse<String> sigmaResponse = new SigmaResponse<>();
        sigmaResponse.setHeader(new SigmaResponseHeader("-2", 
            messageService.getMessage("denied.error.message", request), null));
        response.getWriter().print(new ObjectMapper().writeValueAsString(sigmaResponse));
        response.getWriter().flush();
    }
}
```

创建自定义登陆和注销类LoginController.java

```
@RestController
@RequestMapping
@Api(value = "login & logout", tags = "login")
public class LoginController {
    @Autowired
    MessageService messageService;
    @Autowired
    AccountFeignClient accountFeignClient;

    @ApiOperation(value = "user login")
    @PostMapping("/login")
    public SigmaResponse login(@RequestBody ReqUserLoginParam userLoginParam) {
        if (userLoginParam == null
                || StringUtil.isEmpty(userLoginParam.getUsername())
                || StringUtil.isEmpty(userLoginParam.getPassword())) {
            throw new MessageException("login.error.message");
        }
        SigmaResponse<UserFindResultBO> response = accountFeignClient.getUserByUsername(userLoginParam.getUsername());
        UserFindResultBO user = response.getData();
        //登陆验证
        if (user == null) {
            throw new MessageException("login.error.message");
        }
        SigmaResponse<Boolean> checkResult = accountFeignClient.checkPassword(
            new ReqPasswordCheckParam(userLoginParam.getPassword(), user.getPassword()));
        if (checkResult.getData() == null || checkResult.getData() == false) {
            throw new MessageException("login.error.message");
        }
        if (user.getUsable() != 1) {
            throw new MessageException("login.enableError.message");
        }
        SigmaResponse<List<MenuListForLoginBO>> menuListForLogin = accountFeignClient.getMenuListForLogin(user.getId());
        List<MenuListForLoginBO> menuList = menuListForLogin.getData();
        //菜单权限配置（Security框架）
        List<MenuAuth> menuAuths = new ArrayList<>();
        for (MenuListForLoginBO menu : menuList) {
            if (menu != null && menu.getName() != null && menu.getIsMenu() == 0) {
                MenuAuth authority = new MenuAuth();
                authority.setUrl(menu.getUrl());
                authority.setMethod(menu.getMethod());
                menuAuths.add(authority);
            }
        }
        //token生成
        String token = SystemUtil.createToken();
        //自定义保存redis的用户对象
        CustomUser customUser = CopyUtil.copyObject(user, CustomUser.class);
        customUser.setAuthorities(menuAuths);
        customUser.setToken(token);
        //保存到redis中
        accountFeignClient.saveToken(customUser);

        //自定义返回user对象
        RespUserLoginParam respUserLoginParam = CopyUtil.copyObject(customUser, RespUserLoginParam.class);
//        List<MenuListResultBO> boList = CopyUtil.copyList(menuList, MenuListResultBO.class);
//        List treeJson = MenuUtil.createTreeJson(boList, true);
//        respUserLoginParam.setMenus(treeJson);
        //返回response对象
        SigmaResponse<RespUserLoginParam> respUserFindParamSigmaResponse = new SigmaResponse<>();
        respUserFindParamSigmaResponse.setHeader(new SigmaResponseHeader("0", 
            messageService.getMessage("login.seccess.message"), null));
        respUserFindParamSigmaResponse.setData(respUserLoginParam);
        return respUserFindParamSigmaResponse;
    }

    @ApiOperation(value = "user logout")
    @PostMapping("/logout")
    public SigmaResponse<String> login(HttpServletRequest request) {
        String token = SystemUtil.getToken(request);
        accountFeignClient.deleteToken(token);
        SigmaResponse<String> respUserFindParamSigmaResponse = new SigmaResponse<>();
        respUserFindParamSigmaResponse.setHeader(new SigmaResponseHeader("0", 
            messageService.getMessage("logout.success.message"), null));
        return respUserFindParamSigmaResponse;
    }
}
```

创建Feign请求类：AccountFeignClient.java，用于访问account微服务

```
@FeignClient("account")
public interface AccountFeignClient {
    //登陆验证
    @RequestMapping(value = "/account/user/username/{username}", method = RequestMethod.GET)
    SigmaResponse<UserFindResultBO> getUserByUsername(@PathVariable("username") String username);

    @RequestMapping(value = "/account/menu/user/{id}", method = RequestMethod.GET)
    SigmaResponse<List<MenuListForLoginBO>> getMenuListForLogin(@PathVariable("id") Integer id);

    @RequestMapping(value = "/account/password/check", method = RequestMethod.POST)
    SigmaResponse<Boolean> checkPassword(@RequestBody ReqPasswordCheckParam reqPasswordCheckParam);

    //token验证
    @RequestMapping(value = "/account/token/{token}", method = RequestMethod.GET)
    SigmaResponse<CustomUser> getToken(@PathVariable("token") String token);

    @RequestMapping(value = "/account/token", method = RequestMethod.POST)
    SigmaResponse saveToken(@RequestBody CustomUser customUser);

    @RequestMapping(value = "/account/token/check/{token}", method = RequestMethod.POST)
    SigmaResponse<Boolean> checkToken(@PathVariable("token") String token);

    @RequestMapping(value = "/account/token/{token}", method = RequestMethod.DELETE)
    SigmaResponse deleteToken(@PathVariable("token") String token);
}
```

3.测试

首先登陆localhost:8040/login，获取当前token，再去访问account微服务。![](/assets/login.jpg)![](/assets/auth.jpg)

