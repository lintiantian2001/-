# 入门案例



## 必要操作

> 基于springboot的web项目，springSecurity本质上是一个过滤器链

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 直接使用
   - 默认用户名：user
   - 默认密码：在控制台中可以看到



## 如何自定义用户名密码

### 1在配置文件application.yaml中设置

```yaml
spring:
  security:
    user:
      name: 用户名
      password: 密码
```

### 2使用配置类设置用户名密码

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        String encode = new BCryptPasswordEncoder().encode("1234");//对密码进行加密
        auth.inMemoryAuthentication().withUser("lttnb2").password(encode).roles("admin");//设置用户名，密码，角色
    }

    /**
     * 需要把PasswordEncoder放到容器中，否则使用对象加密时会报错
     * @return
     */
    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder();
    }
}
```

### 3自定义配置类

1.  编写配置类

   ```java
   package com.ltt.demo01.config;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
   import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
   import org.springframework.security.core.userdetails.UserDetailsService;
   import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
   import org.springframework.security.crypto.password.PasswordEncoder;
   
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Autowired
       private UserDetailsService userDetailsService;
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService).passwordEncoder(password());
       }
       /**
        * 需要把PasswordEncoder放到容器中，否则使用对象加密时会报错
        * @return
        */
       @Bean
       PasswordEncoder password(){
           return new BCryptPasswordEncoder();
       }
   }
   ```

2. 编写service

   ```java
   package com.ltt.demo01.service;
   import org.springframework.security.core.GrantedAuthority;
   import org.springframework.security.core.authority.AuthorityUtils;
   import org.springframework.security.core.userdetails.User;
   import org.springframework.security.core.userdetails.UserDetails;
   import org.springframework.security.core.userdetails.UserDetailsService;
   import org.springframework.security.core.userdetails.UsernameNotFoundException;
   import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
   import org.springframework.stereotype.Service;
   import java.util.List;
   
   @Service("userDetailsService")//名字要与配置类中自动注入的名字对应
   public class MyUserDetailService implements UserDetailsService {
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");//权限的集合
           /**
            * 用户名，加密后的密码"123",权限
            */
           return new User("mary",new BCryptPasswordEncoder().encode("123"),role);
       }
   }
   ```



## 自定义登录页面与其他配置

1. 在配置类SecurityConfig 中重写configure方法

   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter 

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin() //自定义自己编写的登录页面
    .loginPage("/login.html")//会调转到此登录页面
    .loginProcessingUrl("/user/login")//登录的访问路径，点击登录后提交的controller中，springsecurity以帮我们完成，不需要我们自己编写
    .defaultSuccessUrl("/test/success").permitAll()//登录成功后跳转的路径
    .and().authorizeRequests()
            .antMatchers("/test/hello","/user/login").permitAll()//当访问/test/hello和/user/login不需要做权限认证
    .anyRequest().authenticated() //表示所有请求都可以直接访问
    .and().csrf().disable();//关闭csrf的防护
}
```

2. 编写login.html

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <p>此页面为自定义登录页面</p>
       <form action="/user/login" method="post"><!-- 此action必须和 HttpSecurity.loginProcessingUrl("/user/login")中的参数一致-->
           用户名 <input type="text" name="username"> <br> <!-- 自定义登录页面名字必须为username和password-->
           密码 <input type="text" name="password"> <br>
           <input type="submit" value="提交">
       </form>
   </body>
   </html>
   ```





## 基于角色或权限进行访问控制

> 有权限访问成功,没有权限报错403



### 配置类

1. 在配置类SecurityConfig 中重写configure方法

   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter 

```java
//基于角色或权限进行访问控制
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling().accessDeniedPage("/unauth.html");//如果没有权限则会跳到此页面
        http.formLogin() //自定义自己编写的登录页面
                .and().authorizeRequests()

                //基于角色或权限进行访问控制
                //基于权限
                //.antMatchers("/test/index").hasAuthority("admin")//当前登录用户,只有具有admin权限才可以访问这个路径
                //.antMatchers("/test/index").hasAnyAuthority("admin","manager")//当前登录用户,有admin或者manager权限都可以访问此路径
                //基于角色
                //.antMatchers("/test/index").hasRole("sale")//当前登录用户,只有具有ROLE_sale角色才可以访问这个路径
                .antMatchers("/test/index").hasAnyRole("sale","sale1")//当前登录用户,具有ROLE_sale或者ROLE_sale1角色可以访问这个路径

                ;
    }
```

```java
http.formLogin() //自定义自己编写的登录页面
        .loginPage("/login.html")//会调转到此登录页面
        .loginProcessingUrl("/user/login")//登录的访问路径，点击登录后提交的controller中，springsecurity以帮我们完成，不需要我们自己编写
        .defaultSuccessUrl("/success").permitAll()//登录成功后跳转的路径
        .and().authorizeRequests()
        .antMatchers("/login.html").permitAll()//放行login.html

        //基于角色或权限进行访问控制
        .antMatchers("/add.html").hasRole("ltt") //有此角色即可访问


        .anyRequest().authenticated()//表示所有请求都可以直接访问
        .and().csrf().disable();//关闭csrf的防护,让自己网站更安全
```



2. 在service中给用户设置权限

```java
 List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_sale1");//权限的集合，此处有admin角色和sale1权限
```

 

### 基于注解

1. 在配置类上加上

   ```java
   @EnableGlobalMethodSecurity(securedEnabled=true,prePostEnabled = true)
   ```

2. 在方法上加@Secured注解

   ```java
   @RequestMapping("update")
   //@Secured({"ROLE_sale","ROLE_manager"})//在方法上添加角色，是sale或者manager角色才能访问
   //@PreAuthorize("hasAnyAuthority('admin')")//在方法执行前调用，当用户有admin权限则此方法可以被执行（之前四个方法都可以）
   //@PostAuthorize("hasAnyAuthority('admin1')")//在方法执行之后校验,如果没有权限页面会403，但是方法会执行
   @PostFilter("filterObject.username=='admin1'")//会在之前做过滤只有用户名为admin1才会被return返回，否则会被过滤掉
   List<User> update(){
       System.out.println("update");
       List<User> users = new ArrayList<>();
       users.add(new User(1,"admin1","6666"));
       users.add(new User(2,"admin2","7777"));
       return users;
   }
   ```

3. 在service中给用户设置权限

   ```java
    List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_sale1");//权限的集合，此处有admin角色和sale1权限
   ```

   

## 退出

1. 在配置类中定义退出操作

   ```java
   @Configuration
   public class SecurityConfig_02 extends WebSecurityConfigurerAdapter {
       //自定义登录页面与其他配置
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           //退出权限操作
           http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();//设置退出的路径为/logout，退出成功后跳转到/test/hello
       }
   }
   ```

2. 编写a标签

   ```html
   <a href="/logout">退出</a><!--href必须与http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();中的url对应-->
   ```



## 实现记住我

1. 在配置类SecurityConfig 中重写configure方法

   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter 

```java
//实现记住我功能
@Autowired
private DataSource dataSource;
@Bean
public PersistentTokenRepository persistentTokenRepository(){
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    jdbcTokenRepository.setCreateTableOnStartup(true);//自动生产表
    return jdbcTokenRepository;
}
```

2. 在配置类中配置如下

   ```java
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       /*//退出权限操作
       http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();//设置退出的路径为/logout，退出成功后跳转到/test/hello
   
       http.formLogin() //自定义自己编写的登录页面
       .loginPage("/login.html")//会调转到此登录页面
       .loginProcessingUrl("/user/login")//登录的访问路径，点击登录后提交的controller中，springsecurity以帮我们完成，不需要我们自己编写
       .defaultSuccessUrl("/success.html").permitAll()//登录成功后跳转的路径
   
       .and().authorizeRequests()
               .antMatchers("/test/hello","/user/login").permitAll()//当访问/test/hello和/user/login不需要做权限认证
       .anyRequest().authenticated() //表示所有请求都可以直接访问*/
   
       //实现记住我
       /*.and().*/http.rememberMe().tokenRepository(persistentTokenRepository())
               .tokenValiditySeconds(80)//80秒过期
               .userDetailsService(userDetailsService)//要用到他的底层操作数据库,登录逻辑页面
   
       .and().csrf().disable();//关闭csrf的防护
   
   
   }
   ```

3. 4在登陆页面添加复选框

   ```html
   记住我：<input type="checkbox"name="remember-me"title="记住密码"/><br/>
   ```



## csrf防护

> 在登录页面上写隐藏域开启开启，Spring Security CSRF 会针对 PATCH，POST，PUT 和 DELETE 方法进行防护。

```html
<input type="hidden"th:if="${_csrf}!=null"th:value="${_csrf.token}"name="_csrf"/>
```

