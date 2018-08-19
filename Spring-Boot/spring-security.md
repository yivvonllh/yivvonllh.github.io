## Spring Security

### 一.前言

Spring security 是一个强大的和高度可定制的身份验证和访问控制框架。在Spring Security中，认证过程称之为Authentication\(验证\)，指的是建立系统使用者信息\( principal \)的过程。使用者可以是一个用户、设备、或者其他可以在我们的应用中执行某种操作的其他系统。" Authorization "指的是判断某个 principal 在我们的应用是否允许执行某个操作。在 进行授权判断之前，要求其所要使用到的规则必须在验证过程中已经建立好了。

### 二.导入依赖

在pom文件中添加:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 三.开启服务

```
**
 * springboot security权限配置
 */
@Configuration                                                       
@EnableWebSecurity                                                   1
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        /*自定义认证*/
        auth.authenticationProvider(authenticationProvider);
        auth.userDetailsService(userDetailsService)                   2
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()                                                3
                .antMatchers(HttpMethod.GET, "/flight/user/login")
                .antMatchers(HttpMethod.POST, "/flight/user/register")
                .antMatchers(HttpMethod.GET, "/flight/user/forget/email")
                .antMatchers(HttpMethod.POST, "/flight/user/forget/reset")
                .antMatchers("/swagger-ui.html", "/webjars/**", "/v2/**", "/swagger-resources/**")
                .antMatchers("/actuator/**");

    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {     
        http.securityContext().securityContextRepository(tokenSecurityContextRepository());   4
        http.authorizeRequests()
                .antMatchers("/flight/user/manage")
                .authenticated()
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O fsi) {
                        fsi.setAccessDecisionManager(customAccessDecisionManager());          5
                        return fsi;
                    }
                });
        //关闭csrf
        http.csrf().disable();                                                                6

    }
    @Bean
    public AccessDecisionManager customAccessDecisionManager() {
        return new CustomAccessDecisionManager();
    }
    @Bean
    public TokenSecurityContextRepository tokenSecurityContextRepository() {
        return new TokenSecurityContextRepository();
    }
}
```

整体执行顺序:

首先SecurityContextPersistenceFilter会拦截请求,把页面参数封装成Authentication对象.然后UserNamePasswordAuthenticationFilter会调用FilterSecurityInterceptor,FilterSecurityInterceptor会持有AuthenticationManager对象,而AuthenticationManager会持有AuthenticationProvider对象,AuthenticationProvider对象会从之前封装好的Authentication对象获取信息,进行用户验证.

图中数字的解释:

1-开启SpringSecurity服务

2-通过AuthenticationProvider接口进行用户认证,通过UserDetailsService进行用户权限信息的查找\(一般是通过查找数据库\)和保存.AuthenticationProvider会返回一个Authentication对象.两个接口都需要自己实现.

```
@Component
public class MyAuthenticationProvider implements AuthenticationProvider{
    @Autowired
    private MyUserDetailsService userDetailsService;

    @Override
    public Authentication authenticate(Authentication authentication){
        //1.获取用户输入的用户名 密码
        String username = authentication.getName();
        String password = (String) authentication.getCredentials();
        //2.关于MD5加密：
        //因为我们是自定义Authentication，所以必须手动加密加盐而不需要再配置。
        password = new Md5PasswordEncoder().encodePassword(password,username);
        //3.由输入的用户名查找该用户信息，内部抛出异常
        UserDetails user = userDetailsService.loadUserByUsername(username);
        //4.密码校验
        if (!password.equals(user.getPassword())) {
            throw new DisabledException("---->UserName :" + username + " password error!");
        }
        return new UsernamePasswordAuthenticationToken(user, password, user.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return (UsernamePasswordAuthenticationToken.class
                .isAssignableFrom(aClass));
    }

}
```

```
@Component
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserService userService;

    String role_ = "ROLE_";

    @Override
    public UserDetails loadUserByUsername(String username) {
        //1.业务层根据username获取该用户
        cn.zyzpp.security.entity.User user = userService.findUserByUserName(username);
        if (user == null) {
            throw new DisabledException("---->UserName :" + username + " not found!");
        }
        //2.从业务层获取用户权限并转为Authorities
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (Role role : user.getRoleList()) {
            authorities.add(new SimpleGrantedAuthority(role.getName()));//设置权限
            authorities.add(new SimpleGrantedAuthority(role_ + role.getName()));//设置角色
        }
        //3.返回Spring定义的User权限对象
        return new User(username, user.getPassword(), authorities);
    }

}
```

3-设置不被拦截的url地址

4-自定义TokenSecurityContextRepository实现SecurityContextRepository接口,以便验证token是否有效

5-设置需要被验证的url地址,还有一些其他的设置,参考如下:

```
http.authorizeRequests()
                .antMatchers("/", "/signIn").permitAll()//所有人都可以访问
                .antMatchers("/leve/1").hasRole("VIP1") //设置访问角色
                .antMatchers("/leve/2").hasRole("VIP2")
                .antMatchers("/leve/3").hasAuthority("VIP2")//设置访问权限
                .anyRequest().authenticated() //其他所有资源都需要认证，登陆后访问
                .and()
                .formLogin()//开启自动配置的授权功能
                .loginPage("/login")    //自定义登录页（controller层需要声明）
                .usernameParameter("username")  //自定义用户名name值
                .passwordParameter("password")  //自定义密码name值
                .failureUrl("/login?error") //登录失败则重定向到此URl
                .permitAll() //登录页都可以访问
                .and()
                .logout()//开启自动配置的注销功能
                .logoutSuccessUrl("/")//注销成功后返回到页面并清空Session
                .and()
                .rememberMe()
                .rememberMeParameter("remember")//自定义rememberMe的name值，默认remember-Me
                .tokenValiditySeconds(604800);//记住我的时间/秒
```

自定义CustomAccessDecisionManager类实现AccessDecisionManager接口,进行权限的验证:从SecurityContext中取出之前保存的Authentication对象,然后比较访问路径的权限和Authentication对象中的权限大小.无权限访问时,则返回拒绝信息.

```
**
 * 自定义验证权限
 */
@Component
public class CustomAccessDecisionManager implements AccessDecisionManager {

    //decide 方法是判定是否拥有权限的决策方法
    @Override
    public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException, InsufficientAuthenticationException {
        HttpServletRequest request = ((FilterInvocation) object).getHttpRequest();
        String url, method;
        AntPathRequestMatcher matcher;
        for (GrantedAuthority ga : authentication.getAuthorities()) {
            if (ga instanceof CustomGrantedAuthority) {
                CustomGrantedAuthority urlGrantedAuthority = (CustomGrantedAuthority) ga;
                url = urlGrantedAuthority.getUrl();
                method = urlGrantedAuthority.getMethod();
                matcher = new AntPathRequestMatcher(url);
                if (matcher.matches(request)) {
                    //当权限表权限的method为ALL时表示拥有此路径的所有请求方式权利。
                    if (method.equals(request.getMethod()) || "ALL".equals(method)) {
                        return;
                    }
                }
            }
        }
        throw new AccessDeniedException("This user does not have this permissions");
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

6-关闭spring security crsf protection默认的http 403 access denied处理方式\(或者叫防范CSRF跨站请求伪造攻击\)

### 四.问题及解决

##### 1.跨域问题

1.1问题的产生:

通常我们测试的方法是通过Swagger发起http请求,查看返回信息,这样并不能察觉到跨域问题.但实际上由于我们是前后端分离系统,如果通过页面发起ajax请求就会发现问题.

1.2问题的解决:

增加CROS设置.CORS是一个W3C标准,全称是”跨域资源共享”（Cross-origin resource sharing\\).它允许浏览器向跨源服务器,发出       XMLHttpRequest请求,从而克服了AJAX只能同源使用的限制.增加自定义的Filter:

```
@Component
public class CorsControllerFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        HttpServletResponse res = (HttpServletResponse) response;
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE ,PUT");
        res.setHeader("Access-Control-Max-Age", "3600");
        res.setHeader("Access-Control-Allow-Headers", "Origin, No-Cache, X-Requested-With, If-Modified-Since,"
                + " Pragma, Last-Modified, Cache-Control, Expires, Content-Type, "
                + "X-E4M-With,userId,token,Authorization,deviceId,Access-Control-Allow-Origin,Access-Control-Allow-Headers,Access-Control-Allow-Methods");
        res.setHeader("Access-Control-Allow-Credentials", "true");
        chain.doFilter(request, response);
    }
}
```

由于非简单请求会发起一个OPTIONS方法的预检请求,而我们用了Spring Security拦截所有请求,只开放部分请求,所以在WebSecurityConfig 中配置时,需要把自定义的CorsCOntrollerFilter加在Security拦截器之前,避免拦截冲突.

```
 http.addFilterBefore(corsControllerFilter, SecurityContextPersistenceFilter.class);
```

### 五.参考资料

1.整体流程图:

![](/assets/springsecurity.png)2.SpringSecurity内部filter调用顺序

![](/assets/SpringSecurityFilter执行顺序.png)



