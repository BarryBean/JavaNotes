# 1. 概述
## 1.1 是什么
答：SpringBoot是Spring组件的一站式解决方案，并不是独立框架，是用来简化Spring配置，提供各种启动器，让开发者能快速上手的。

## 1.2 优点
- 容易上手，开箱即用，提升开发效率；
- 能和Spring生态系统集成；
- 遵循默认配置，提供了一系列通用的非业务性功能；
- 内置http服务器，如tomcat、jetty，开发web应用更加轻松；
- maven配置直接写入，有效避免冲突。

## 1.3 Spring Boot Starters
答：Spring Boot Starters是一系列依赖关系的集合。
- 内部提供了XXXAutoConfiguration的自动化配置类；
- 通过条件注解来决定一个配置是否生效，同时提供一系列的默认配置
- 之后通过类型安全的属性注入将这些配置属性注入容器，新注入的属性会代替掉默认属性。


## 1.4 核心注解
答：@SpringBootApplication可以看作是由三个注解的集合。
- @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能，允许在上下文注册bean。
- @EnableAutoConfiguration：打开自动配置的功能。
- @ComponentScan：扫描@Component(@Server，@Controller)的bean。


## 1.5 jar包区别
答：Spring Boot打包成的jar是可执行的，通过 java -jar xxx.jar 命令运行，不能作为普通的jar包被项目依赖使用。

- Spring Boot的jar包解压后，代码在\BOOT-INF\classes 目录下。
- 可以在pom.xml文件中增加配置，将项目打包成两个jar ，一个可执行，一个可引用。




# 2. 配置
## 2.1 自动配置
答：自动配置的核心就是@EnableAutoConfiguration, @Conditional注解。

1. @EnableAutoConfiguration，通过@Import注解导入自动配置方法类AutoConfigurationImportSelector。该类中将所有自动配置类信息以list返回，同时被容器当作bean管理。(META-INF/spring.factories文件)
2. @Conditional，类似if条件为true就执行后续，如@ConditionalClass指定类必须存在类路径下，@ConditionalOnBean容器中有指定Bean。

总结：EnableAutoConfiguration实现配置信息的导入，Conditional用来指定此类需要的配置信息。

## 2.2 配置加载方式
答：properties文件，yaml文件，系统环境变量，命令行参数。

## 2.3 Yaml
### 2.3.1 概述
答：yaml是一种可读性友好的数据序列化语言，常用来作为配置文件。
### 2.3.2 优点
答：配置有序；支持数组；简洁结构化。但不支持注解导入自定义的yaml配置。



# 3. 安全
## 3.1 Spring Security
答：官方提供的安全管理框架，添加依赖后，项目的所有接口都被保护，必须登陆后才能访问。通过继承WebSecurityConfigurerAdapter接口，实现登录配置、密码加密、拦截访问等等一系列功能。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    //开启登陆配置
    http.authorizeRequests()
            //访问/hello接口，且有admin的角色
            .antMatchers("/hello").hasRole("admin")
            //剩余其他接口，登录后就能访问
            .anyRequest().authenticated()
            .and()
            .formLogin()
            //定义登录页面，未登录时自动跳转
            .loginPage("/login")
            .loginProcessingUrl("/doLogin")
            //登录时用户名和密码的key，可以自定义
            .usernameParameter("uname")
            .passwordParameter("passwd")
            //登录成功的处理器
            .successHandler(new AuthenticationSuccessHandler() {
                @Override
                public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                    httpServletResponse.setContentType("application/json;charset=utf-8");
                    PrintWriter out = httpServletResponse.getWriter();
                    out.write("success");
                    out.flush();
                    out.close();
                }
            })
            //登录失败的处理器
            .failureHandler(new AuthenticationFailureHandler() {
                @Override
                public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                    httpServletResponse.setContentType("application/json;charset=utf-8");
                    PrintWriter out = httpServletResponse.getWriter();
                    Map<String, Object> map = new HashMap<>();
                    //相关异常去AuthenticException中查找子类
                    if (e instanceof LockedException) {
                        map.put("msg", "账户锁定");
                    } else if (e instanceof BadCredentialsException) {
                        map.put("msg", "输入错误");
                    } else {
                        map.put("msg", "登录失败");
                    }
                    out.write(new ObjectMapper().writeValueAsString(map));
                    out.flush();
                    out.close();
                }
            })
            //和表单登录相关接口直接通过
            .permitAll()
            .and()
            //登出注销配置
            .logout()
            .logoutUrl("/logout")
            .logoutSuccessHandler(new LogoutSuccessHandler() {
                @Override
                public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                    httpServletResponse.setContentType("application/json;charset=utf-8");
                    PrintWriter out = httpServletResponse.getWriter();
                    out.write("success");
                    out.flush();
                }
            })
            .permitAll()
            .and()
            .httpBasic()
            .and()
            .csrf()
            .disable();
}

```

## 3.2 跨域问题
### 3.2.1 CSRF
答：CSRF跨站请求伪造，通过伪装成受信任用户进行请求获取数据。CSRF通过冒用cookie实现，利用网站对Client的信任。

### 3.2.2 CORS
答：浏览器基于安全考虑有同源策略，规定域名端口协议一致，CORS跨域资源共享就是为了避开同源策略。通过继承WebMvcConfigurer重写addCorsMappings()实现。

### 3.2.3 其余方案
- Referer字段验证：不适用，Referer能被篡改。HTTP头中用Referer字段记录请求的源地址。
- Token验证：Server发给Client一个Token，Client发出带Token的请求，若Token不合法，则Server拒绝请求。
- 双重Cookie验证：Client将Cookie参数加入请求参数中，Server校验，若无附加的cookie参数则拒绝请求。
- 验证码：在用户进行敏感操作时，要求用户输入验证码。


