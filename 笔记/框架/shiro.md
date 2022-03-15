# Springboot集成shiro

## shiro了解

### 1.1、什么是shiro？

- Apache Shiro 是 Java 的一个安全（权限）框架。
- Shiro 可以非常容易的开发出足够好的应用，其不仅可以用在 JavaSE 环境，也可以用在 JavaEE 环境。
- Shiro 可以完成：认证、授权、加密、会话管理、与Web 集成、缓存等。
- 下载地址
  - 官网：http://shiro.apache.org/
  - github：https://github.com/apache/shiro

### 1.2、有哪些功能？

- Authentication:身份认证/登录，验证用户是不是拥有相应的身份
- Authorization:授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能进行什么操作，如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限
- Session Management:会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境，也可以是Web 环境的
- Cryptography:加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储
- Web Support:Web 支持，可以非常容易的集成到Web 环境
- Caching:缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率
- Concurrency:Shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去
- Testing:提供测试支持
- “Run As”:允许一个用户假装为另一个用户（如果他们允许）的身份进行访问
- Remember Me:记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了

### 1.3、shiro架构

![image-20220314214227307](D:\学习ing\Java\编程笔记图\image-20220314214227307.png)

- Subject：任何可以与应用交互的“用户”；
- SecurityManager：相当于SpringMVC中的DispatcherServlet；是Shiro的心脏；所有具体的交互都通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证、授权、会话及缓存的管理。
- Authenticator：负责Subject 认证，是一个扩展点，可以自定义实现；可以使用认证策略（Authentication Strategy），即什么情况下算用户认证通过了；
- Authorizer：授权器、即访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；
- Realm：可以有1 个或多个Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC 实现，也可以是内存实现等等；由用户提供；所以一般在应用中都需要实现自己的Realm；
- SessionManager：管理Session 生命周期的组件；而Shiro并不仅仅可以用在Web 环境，也可以用在如普通的JavaSE环境
  CacheManager：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少改变，放到缓存中后可以提高访问的性能
- Cryptography：密码模块，Shiro提高了一些常见的加密组件用于如密码加密/解密。

## Hello World

创建一个普通的Maven项目

1.依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.4.1</version>
    </dependency>
    <!-- configure logging -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

2.相关日志文件 log4h.properties

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m %n
# General Apache libraries
log4j.logger.org.apache=WARN
# Spring
log4j.logger.org.springframework=WARN
# Default Shiro logging
log4j.logger.org.apache.shiro=TRACE
# Disable verbose logging
log4j.logger.org.apache.shiro.util.ThreadContext=WARN
log4j.logger.org.apache.shiro.cache.ehcache.EhCache=WARN
```

3. shiro.ini

   ```ini
   [users]
   # user 'root' with password 'secret' and the 'admin' role
   root = secret, admin
   # user 'guest' with the password 'guest' and the 'guest' role
   guest = guest, guest
   # user 'presidentskroob' with password '12345' ("That's the same combination on
   # my luggage!!!" ;)), and role 'president'
   presidentskroob = 12345, president
   # user 'darkhelmet' with password 'ludicrousspeed' and roles 'darklord' and 'schwartz'
   darkhelmet = ludicrousspeed, darklord, schwartz
   # user 'lonestarr' with password 'vespa' and roles 'goodguy' and 'schwartz'
   lonestarr = vespa, goodguy, schwartz
   # -----------------------------------------------------------------------------
   # Roles with assigned permissions
   #
   # Each line conforms to the format defined in the
   # org.apache.shiro.realm.text.TextConfigurationRealm#setRoleDefinitions JavaDoc
   # -----------------------------------------------------------------------------
   [roles]
   # 'admin' role has all permissions, indicated by the wildcard '*'
   admin = *
   # The 'schwartz' role can do anything (*) with any lightsaber:
   schwartz = lightsaber:*
   # The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
   # license plate 'eagle5' (instance specific id)
   goodguy = winnebago:drive:eagle5
   ```

4. quikstart启动类

   ```java
   import org.apache.shiro.SecurityUtils;
   /*
    * Licensed to the Apache Software Foundation (ASF) under one
    * or more contributor license agreements.  See the NOTICE file
    * distributed with this work for additional information
    * regarding copyright ownership.  The ASF licenses this file
    * to you under the Apache License, Version 2.0 (the
    * "License"); you may not use this file except in compliance
    * with the License.  You may obtain a copy of the License at
    *
    *     http://www.apache.org/licenses/LICENSE-2.0
    *
    * Unless required by applicable law or agreed to in writing,
    * software distributed under the License is distributed on an
    * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    * KIND, either express or implied.  See the License for the
    * specific language governing permissions and limitations
    * under the License.
    */
   import org.apache.shiro.SecurityUtils;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.config.IniSecurityManagerFactory;
   import org.apache.shiro.mgt.SecurityManager;
   import org.apache.shiro.session.Session;
   import org.apache.shiro.subject.Subject;
   import org.apache.shiro.util.Factory;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   
   
   /**
    * Simple Quickstart application showing how to use Shiro's API.
    *
    * @since 0.9 RC2
    */
   public class Quickstart {
   
       private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);
   
   
       public static void main(String[] args) {
   
           // The easiest way to create a Shiro SecurityManager with configured
           // realms, users, roles and permissions is to use the simple INI config.
           // We'll do that by using a factory that can ingest a .ini file and
           // return a SecurityManager instance:
           // Use the shiro.ini file at the root of the classpath
           // (file: and url: prefixes load from files and urls respectively):
           Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
           SecurityManager securityManager = factory.getInstance();
           // for this simple example quickstart, make the SecurityManager
           // accessible as a JVM singleton.  Most applications wouldn't do this
           // and instead rely on their container configuration or web.xml for
           // webapps.  That is outside the scope of this simple quickstart, so
           // we'll just do the bare minimum so you can continue to get a feel
           // for things.
           SecurityUtils.setSecurityManager(securityManager);
           // Now that a simple Shiro environment is set up, let's see what you can do:
           // get the currently executing user:
           Subject currentUser = SecurityUtils.getSubject();
           // Do some stuff with a Session (no need for a web or EJB container!!!)
           Session session = currentUser.getSession();
           session.setAttribute("someKey", "aValue");
           String value = (String) session.getAttribute("someKey");
           if (value.equals("aValue")) {
               log.info("Retrieved the correct value! [" + value + "]");
           }
           // let's login the current user so we can check against roles and permissions:
           if (!currentUser.isAuthenticated()) {
               UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
               token.setRememberMe(true);
               try {
                   currentUser.login(token);
               } catch (UnknownAccountException uae) {
                   log.info("There is no user with username of " + token.getPrincipal());
               } catch (IncorrectCredentialsException ice) {
                   log.info("Password for account " + token.getPrincipal() + " was incorrect!");
               } catch (LockedAccountException lae) {
                   log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                           "Please contact your administrator to unlock it.");
               }
               // ... catch more exceptions here (maybe custom ones specific to your application?
               catch (AuthenticationException ae) {
                   //unexpected condition?  error?
               }
           }
           //say who they are:
           //print their identifying principal (in this case, a username):
           log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");
           //test a role:
           if (currentUser.hasRole("schwartz")) {
               log.info("May the Schwartz be with you!");
           } else {
               log.info("Hello, mere mortal.");
           }
           //test a typed permission (not instance-level)
           if (currentUser.isPermitted("lightsaber:wield")) {
               log.info("You may use a lightsaber ring.  Use it wisely.");
           } else {
               log.info("Sorry, lightsaber rings are for schwartz masters only.");
           }
           //a (very powerful) Instance Level permission:
           if (currentUser.isPermitted("winnebago:drive:eagle5")) {
               log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                       "Here are the keys - have fun!");
           } else {
               log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
           }
           //all done - log out!
           currentUser.logout();
           System.exit(0);
       }
   }
   ```

中文

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


/**
 * Simple Quickstart application showing how to use Shiro's API.
 *
 * @since 0.9 RC2
 */
public class Quickstart {

    //使用·log输出日志
    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);


    public static void main(String[] args) {

        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);

        // Now that a simple Shiro environment is set up, let's see what you can do:

        //获取当前的用户对象
        Subject currentUser = SecurityUtils.getSubject();
        //通过当前用户拿到session
        Session session = currentUser.getSession();
        //从session中存值取值
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Retrieved the correct value! [" + value + "]");
        }
        // 判断当前用户是否被认证
        if (!currentUser.isAuthenticated()) {
            // Token：令牌
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);//设置记住我
            try {
                currentUser.login(token); //执行登录操作
            } catch (UnknownAccountException uae) {//用户·不存在·
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {//密码错误
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {//一直登陆失败，用户被锁定
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            catch (AuthenticationException ae) {
                //unexpected condition?  error?
            }
        }
        //say who they are:
        //print their identifying principal (in this case, a username):
        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");
        //test a role:
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }
        //粗粒度
        //test a typed permission (not instance-level)
        if (currentUser.isPermitted("lightsaber:wield")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }
        //细粒度
        //a (very powerful) Instance Level permission:
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }
        //注销
        currentUser.logout();
        //结束
        System.exit(0);
    }
}
```

```java
// 获取当前的用户对象
Subject currentUser = SecurityUtils.getSubject();
// 通过当前用户拿到session
Session session = currentUser.getSession();
// 判断当前用户是否被认证
currentUser.isAuthenticated()
// 获得当前用户的认证
currentUser.getPrincipal()
// 获得当前用户是否拥有某些角色
currentUser.hasRole("")
// 获得当前用户权限
currentUser.isPermitted("lightsaber:wield")
// 注销
currentUser.logout();
```

## springboot整合shiro

1.导包

```xml
<dependencies>

    <!--springboot整合shiro-->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.1</version>
    </dependency>


    <!--基础-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    
    <!--thymeleaf模板-->
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
    </dependency>
    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-java8time</artifactId>
    </dependency>
</dependencies>
```

2. 编写配置类与realm

   编写realm

   ```java
   package com.ltt.config;
   
   import org.apache.shiro.authc.AuthenticationException;
   import org.apache.shiro.authc.AuthenticationInfo;
   import org.apache.shiro.authc.AuthenticationToken;
   import org.apache.shiro.authz.AuthorizationInfo;
   import org.apache.shiro.realm.AuthenticatingRealm;
   import org.apache.shiro.realm.AuthorizingRealm;
   import org.apache.shiro.subject.PrincipalCollection;
   
   //自定义的 UserRealm
   public class UserRealm extends AuthorizingRealm {
       //授权
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
           System.out.println("执行了授权=》doGetAuthorizationInfo");
           return null;
       }
       //认证
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
           System.out.println("执行了认证=》doGetAuthenticationInfo");
           return null;
       }
   }
   ```

   编写配置类

   ```java
   package com.ltt.config;
   
   import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
   import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import sun.security.krb5.Realm;
   
   @Configuration
   public class ShiroConfig {
   
       //3.ShiroFilterFactoryBean
       @Bean
       public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           //设置安全管理器
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           return shiroFilterFactoryBean;
       }
   
   
       //2.DefaultWebSecurityManager
       @Bean(name = "securityManager")
       public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm")UserRealm realm){
           DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
           //关联Realm
           defaultWebSecurityManager.setRealm(realm);
           return  defaultWebSecurityManager;
       }
   
       //1.创建Realm对象，需要自定义类
       @Bean
       public UserRealm userRealm(){
           return new UserRealm();
       }
   }
   ```

### 登录验证授权认证逻辑

> 此处用的是thymeleaf模板

1. 编写登录页面login.html

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th=http://www.thymeleaf.org>
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
   <h1>登录</h1>
   <p th:text="${msg}" style="color: red"></p>
   <form th:action="@{/login}">
       <p>用户名：<input type="text" name="username"></p>
       <p>密码：<input type="text" name="password"></p>
       <p><input type="submit" value="登录"></p>
   </form>
   </body>
   </html>
   ```

2. 在配置类中修改**主要看活代码部分**

```java
//3.ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(securityManager);


        //活代码
        //添加shiro的内置过滤器
        /**
         * anon：无需认证可以访问
         * authc： 必须认证才能访问
         * user： 必须拥有记住我功能才能访问
         * perms： 拥有对某个资源的权限才能访问
         * role： 拥有某个角色权限才能访问
         */
        Map<String,String> map=new LinkedHashMap<>();
        map.put("/user/add","perms[user:add]");// 拥有user:add权限才能访问此路径，否则401
        map.put("/user/update","perms[user:update]");
//        map.put("/user/add","authc");// user/add必须认证才能访问
        map.put("/user/*","authc");// 支持通配符

        bean.setFilterChainDefinitionMap(map);

        bean.setLoginUrl("/toLogin");// 设置登录请求，此请求进入登录页面
        bean.setUnauthorizedUrl("/noauth"); // 未授权页面

        return bean;
    }
```

3. 编写controller

```java
@RequestMapping("/toLogin")
public String toLogin(){
    return "login";//thymeleaf跳转到登录页面
}

@RequestMapping("/login")//登录页面的action路径
public String login(String username,String password,Model model){
    //获取当前用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登录数据
    UsernamePasswordToken token = new UsernamePasswordToken(username,password);
    try {
        subject.login(token); //执行登录的方法，如果没有异常就ok了
        return "index";//登陆成功返回首页
    }catch (UnknownAccountException e){//用户名不存在
        model.addAttribute("msg","用户名错误");
        return "login";
    }catch (IncorrectCredentialsException e){//密码错误
        model.addAttribute("msg","密码错误");
        return "login";
    }
}

    @RequestMapping("/noauth")
    @ResponseBody
    public String unauthorized(){
        return "未经授权无法访问此页面";
    }
```

4. 在UserRealm编写授权认证逻辑

   ```java
   package com.ltt.config;
   import com.ltt.pojo.User;
   import com.ltt.service.UserService;
   import org.apache.shiro.SecurityUtils;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.authz.AuthorizationInfo;
   import org.apache.shiro.authz.SimpleAuthorizationInfo;
   import org.apache.shiro.realm.AuthenticatingRealm;
   import org.apache.shiro.realm.AuthorizingRealm;
   import org.apache.shiro.subject.PrincipalCollection;
   import org.apache.shiro.subject.Subject;
   import org.springframework.beans.factory.annotation.Autowired;
   
   //自定义的 UserRealm
   public class UserRealm extends AuthorizingRealm {
       @Autowired
       UserService userService;
   
       //授权
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
           System.out.println("执行了授权=》doGetAuthorizationInfo");
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           //info.addStringPermission(" user:add");//所有的用户都有user:add权限
   
           //拿到当前登录的这个对象
           Subject subject = SecurityUtils.getSubject();
           User currentUser = (User) subject.getPrincipal();//此处是从认证中的SimpleAuthenticationInfo()获取的
           info.addStringPermission(currentUser.getPerms());//设置当前用户的权限
   
   
           return info;
       }
       //认证(逻辑)，点击登录就会走此方法
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
           System.out.println("执行了认证=》doGetAuthenticationInfo");
           UsernamePasswordToken userToken = (UsernamePasswordToken) token;
   
           //用户名，密码~ 数据库中取
           User user = userService.queryUserByName(userToken.getUsername());
           if(user==null){//没有此人
               return null;//抛出异常，用户名不存在
           }
           //密码认证，shiro做
           return new SimpleAuthenticationInfo(user,user.getPwd(),"");//在认证中传入user授权中可以获取
       }
   }
   
   ```



## shiro整合thymeleaf

https://www.bilibili.com/video/BV1NE411i7S8?p=8