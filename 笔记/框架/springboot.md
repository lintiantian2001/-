# 基础入门

> 详细学习资料笔记可看https://www.yuque.com/atguigu/springboot/lcfeme

## 1、系统要求

- [Java 8](https://www.java.com/) & 兼容java14 .
- Maven 3.3+

- idea 2019.1.2

## 2、maven配置   

> C:\apache-maven-3.6.3\conf\settings.xml中配置

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>

<profiles>
    <profile>
        <id>jdk-1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profile>
</profiles>
```

## 3、引入依赖

> 在pom.xml文件中导入依赖

```xml
 <!--1.导入父工程:做依赖管理，不需要操心常用的依赖版本-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.5</version>
</parent>

<dependencies>
    <!--2.添加web场景依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

## 4、创建主程序类运行及开始使用

```java
package com.ltt.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 主程序类
 * @SpringBootApplication：这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```



# 配置



## 配置文件

> 在resources资源根下创建application.properties及可在里面修改springBoot配置
>
> - 配置文件只要在一处就可配置所有文件

```properties
# 修改服务器端口号
server.port=8888 

#开启对restful风格put delete请求的支持
spring.mvc.hiddenmethod.filter.enabled= true 

#开启带参数的内容协商
spring.mvc.contentnegotiation.favor-parameter= true
```



## 配置类(可选配置)

>优先以配置类位置，然后在是springboot的默认配置

```java
package com.ltt.boot.config;

import org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

/**
 * 配置web层的配置类
 */
@Configuration(proxyBeanMethods = false)//方法间不共享
public class WebConfig {

    /**
     * 自己配置HiddenHttpMethodFilter做restful风格而不使用springBoot提供的
     * @return
     */
    @Bean
    HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter filter = new HiddenHttpMethodFilter();
        filter.setMethodParam("_lttMethod");//把原本的_method自定义
        return filter;
    }

    
    @Bean
    WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            /**
             * 开启对矩阵变量的支持
             * @return
             */
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                urlPathHelper.setRemoveSemicolonContent(false);//不移除路径分号后边的内容
                configurer.setUrlPathHelper(urlPathHelper);
            }
            
            /**
             * 自定义类型转换器 把路径中的 pet=阿猫,3参数 用自定义的规则封装到Pet对象中
             * 例如路径为 /aaa?pet=阿猫,3  通过自定义类型转换器封装到Pet类中
             * @param registry
             */
            @Override
            public void addFormatters(FormatterRegistry registry) {
                registry.addConverter(new Converter<String, Pet>() {//把String转为Pet类型
                    @Override
                    public Pet convert(String source) {
                        if(!StringUtils.isEmpty(source)){//如果字符串不为空
                            Pet pet = new Pet();
                            String[] split = source.split(",");
                            pet.setName(split[0]);
                            pet.setAge(Integer.valueOf(split[1]));
                            return pet;
                        }
                        return null;
                    }
                });
            }
            
        };
    }
}


```

# 知识点

## @Value的用法

```java
@Value("${person.name:李四}")//直接拿到配置文件中的值，如果配置文件中没有值则值为李四
private String name;

@Value("${JAVA_HOME}")//也能拿到计算机中环境变量的值
private String msg;
```

## 矩阵变量

1.在配置类开启对矩阵变量的支持

```java
	@Bean
    WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                urlPathHelper.setRemoveSemicolonContent(false);//不移除路径分号后边的内容
                configurer.setUrlPathHelper(urlPathHelper);
            }
        };
    }
```

2.语法    请求路径：/cars/sell;low=34;brand=byd,audi,yd （分隔号左边是路径封号右边是矩阵变量，多个变量以分号区分）

```html
<h1>矩阵变量</h1>
例如： /boss/1;age=30/2;age=30;sex=male  （分隔号左边是路径封号右边是矩阵变量，多个变量以分号区分）<br>
<a href="cars/sell;low=34;brand=byd,audi,yd">测试矩阵变量</a><br>
<a href="cars/boss;age=50/emp;age=30">测试矩阵变量有相同的属性值</a>
```

3.用@MatrixVariable接收路径中的值

```java
@GetMapping("/cars/{path}")//矩阵变量是绑定在路径变量中
public Map carsSell(@MatrixVariable("low") Integer low,//获取矩阵变量的值
                    @MatrixVariable("brand") List<String> brand,
                    @PathVariable("path") String path) {
    Map<String, Object> map = new HashMap<>();
    map.put("low", low);
    map.put("brand", brand);
    map.put("path", path);
    return map;
}
// /boss/1;age=20/2;age=10
@GetMapping("/cars/{boss}/{emp}")
public Map boss(@MatrixVariable(value = "age", pathVar = "boss") Integer bossAge, //如果有相同名字可以通过pathVar指定路径获取
                @MatrixVariable(value = "age", pathVar = "emp") Integer empAge) {
    Map<String, Object> map = new HashMap<>();
    map.put("bossAge", bossAge);
    map.put("empAge", empAge);
    return map;

}
```





## 带参数内容协商 

> 指定发送到浏览器的类型

1.在配置文件properties或yaml开启带参数内容协商

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true  #开启带参数的内容协商
```

2.导包

```xml
<!-- 支持带参数内容协商可以向浏览器传xml类型的值-->
<dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

3.发请求，在参数位置写format指定发出去的请求类型（大多类型基本都能被浏览器接收）  如：

```
http://localhost:8080/aaa?format=json
http://localhost:8080/aaa?format=xml
```

## 消息转换器与内容协商器

> 根据浏览器accept请求头的规则把Person对象写成一个自定义类型的数据
>
> 也可以直接在路径后跟format=xxx的参数返回给浏览器对应类型的数据
>
> - 例如：localhost:8080/aaa?format=json  返回application/json类型的数据
>
>   localhost:8080/aaa?format=ltt 返回自定义类型application/x-ltt类型的数据

1.创建一个消息转换器

```java
package com.ltt.boot.converter;

import com.ltt.boot.bean.Person;
import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.http.converter.HttpMessageNotWritableException;

import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

/**
 * 自定义消息转换器
 * 不支持读application/x-ltt的数据，只支持写
 *
 * 如果浏览器accept头想要application/x-ltt，并且返回值类型是Person就能以write()方法的规则写出到浏览器
 */
public class LttMessageConverter implements HttpMessageConverter<Person>  {//支持转换Person类型的数据
    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return clazz.isAssignableFrom(Person.class);//只要是Person类型就能写
    }

    /**
     * 服务器要统计所有MessageConverter都能写那些内容类型
     * application/x-ltt
     * @return
     */
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-ltt");
    }

    @Override
    public Person read(Class<? extends Person> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    /**
     * 自定义协议数据的写出
     */
    @Override
    public void write(Person person, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        //自定义协议数据的写出
        String p=person.getUserName()+"=>"+person.getAge()+"=>"+person.getBirth()+"=>"+person.getPet();
        //吧数据写出去
        OutputStream outputStream = outputMessage.getBody();
        outputStream.write(p.getBytes());
    }

}
```

2.在配置类添加消息转换器或内容协商策略

```java
WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        
        	/**
             * 添加一个消息转换器
             * 根据浏览器accept请求头的规则把Person对象写成一个自定义类型的数据
             * （例如添加了LttMessageConverter类，根据此类如果浏览器Accept头想要application/x-ltt，并且返回值类型是Person，就能以LttMessageConverter类中write()方法的规则写出到浏览器）
             */
            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                converters.add(new LttMessageConverter()); //添加了一个自定义消息转换器
            }

            /**
             * 自定义内容协商策略 
             * 为了可以支持在路径参数位置写format=ltt返回 自定义类型application/x-ltt的数据
             * @param configurer
             */
            @Override
            public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
                //1.根据请求参数内容协商
                //自定义请求参数  例如：自带参数format=xml  返回application/xml的数据
                Map<String, MediaType> mediaTypes = new HashMap<>();
                mediaTypes.put("ltt",MediaType.parseMediaType("application/x-ltt"));//如果携带参数为format=ltt 则返回application/ltt类型的数据
                //指定支持解析哪些参数对应的哪些媒体类型
                ParameterContentNegotiationStrategy parameterStrategy = new ParameterContentNegotiationStrategy(mediaTypes);
                //parameterStrategy.setParameterName("ff"); //把请求参数名字format修改成其它值

                //2.根据请求头内容协商
                HeaderContentNegotiationStrategy headerStrategy = new HeaderContentNegotiationStrategy();

                configurer.strategies(Arrays.asList(parameterStrategy,headerStrategy));
            }
        
    }
}
```





## thymeleaf

### 基础配置

**1.导包**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

**2.springboot自动配置好了thymeleaf**

- 1、所有thymeleaf的配置值都在 ThymeleafProperties

- 2、配置好了 SpringTemplateEngine 

- 3、配好了 ThymeleafViewResolver 

- 4、我们只需要直接开发页面

  ```java
  public static final String DEFAULT_PREFIX = "classpath:/templates/";
  public static final String DEFAULT_SUFFIX = ".html";  //xxx.html
  ```

**3.页面开发**    注意要在html中写  xmlns:th="http://www.thymeleaf.org"  才会生效

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">哈哈</h1>
<h2>
    <a href="www.atguigu.com" th:href="${link}">去百度</a>  <br/>
    <a href="www.atguigu.com" th:href="@{link}">去百度2</a>
</h2>
</body>
</html>
```



### thymeleaf小知识点

#### 基本写法

```html
<!--行内写法-->
[[${session.user.userName}]]  <!--thymeleaf的行内写法获取session域中的User对象的name值-->
[[${#httpSession.getAttribute('user').getUserName()}]] <!--第二种写法-->

<!-- 路径写法-->
th:href="@{/aaa}"  <!-- 一般指的是项目下的路径比如localhost:8080/aaa -->

<!-- 获取session域中的msg对象-->
th:text="${session.msg}"

redirect:/  转发
forward:/	重定向
```

#### 代码片段

```html
<!-- 代码片段-->
<!-- 1.添加代码片段 th:fragment="路径名"：将此标签添加为代码片段-->
footer.html
<div th:fragment="copy" id="aaa">
    <h1>
        aaa
    </h1>
</div>
<!-- 2.使用代码片段 用thymeleaf模板解析器解析到的footer :: th:fragment的值，可以不写~{}-->
<div th:insert="~{footer :: copy}"></div>
也可以使用以下方式直接选择id或在footer页面下的其他标签属性
<div th:insert="~{footer :: #aaa}"></div>
th:insert 是最简单的：它会简单地插入指定的片段作为其宿主标签的主体。
th:replace实际上用指定的片段替换其主机标记。
th:include与 类似th:insert，但它不插入片段，而是仅插入此片段的内容。
```

#### 遍历  

>例如用th:each遍历User集合

```html
<thead>
    <tr>
        <td>#</td>
        <td>用户名</td>
        <td>密码</td>
    </tr>
</thead>
<tbody>
    <!--
     ${users}：获取到的集合
     user：集合中的每一个元素
     stats：当前的状态=>可以点出以下值{
        当前迭代索引，从 0 开始。这是index属性。
        当前迭代索引，从 1 开始。这是count属性。
        迭代变量中的元素总数。这是size物业。
        每次迭代的iter 变量。这是current物业。
        当前迭代是偶数还是奇数。这些是even/odd布尔属性。
        当前迭代是否是第一个。这是first布尔属性。
        当前迭代是否是最后一次。这是last布尔属性。
     }
    -->
    <tr class="gradeX" th:each="user,stats:${users}">
        <td th:text="${stats.index}"></td>
        <td th:text="${user.userName}">默认名</td>
        <td>[[${user.passWord}]]</td>
    </tr>
</tbody>
```

#### 循环与路径传变量参数

```html
<!--
    循环遍历li三个
    ${#numbers.sequence(1,3)}：生成一个从1到3的序列的数组
    num：代表第几页
    给路径带参数
    th:href="@{/dynamic_table(pn=${num})}" ：跳转到dynamic_table下，并带一个pn=当前页的参数，如有多个参数用逗号分格
-->
<li th:each="num:${#numbers.sequence(1,3)}" >
    <a th:href="@{/dynamic_table(pn=${num})}">[[${num}]]</a>
</li>



<!-- 上方代码最终效果为-->
<li>
    <a href="/dynamic_table?pn=1">1</a>
</li>
<li>
    <a href="/dynamic_table?pn=2">2</a>
</li>
<li>
    <a href="/dynamic_table?pn=3">3</a>
</li>
```









##  拦截器

1. 编写自己定义的拦截器

   ```java
   
   
   /**
    * 用来做登录检查的拦截器
    */
   public class LoginInterceptor implements HandlerInterceptor {
   
       /**
        * 目标方法执行之前执行
        */
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           HttpSession session = request.getSession();
           User user = (User) session.getAttribute("user");
           if(user!=null){//代表已登录
               return true;//放行
           }else {//未登录
               //携带信息跳转登录页
               session.setAttribute("msg","清先登录");
               request.getRequestDispatcher("/").forward(request,response);
               return false;
           }
       }
   
       /**
        * 目标方法执行完成以后
        */
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
       }
   
       /**
        * 页面渲染以后
        */
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
       }
   }
   
   ```

2. 在webMvcConfigurer里添加自己定义的拦截器

   ```java
   /*
   springboot的配置类	
   */
   @Configuration
   public class SpringbootConfig {
   
       @Bean
       WebMvcConfigurer webMvcConfigurer() {
           return new WebMvcConfigurer(){//创建web有关配置
               @Override
               public void addInterceptors(InterceptorRegistry registry) {//添加拦截器
                   registry.addInterceptor(new LoginInterceptor())//链式调用方法
                           .addPathPatterns("/**")//添加需要拦截的路径 /**静态资源也会被拦截
                           .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**");//添加不被拦截的路径
               }
           };
       }
   }
   ```

   也能使用此方法配置

```java
@Configuration
public class SpringbootConfig implements WebMvcConfigurer{
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**");
    }

}
```



## 文件上传

1. 在html用post表单 并且数据格式为enctype="multipart/form-data"进行文件上传

```html
<form role="form" th:action="@{/upload}" method="post" enctype="multipart/form-data"><!-- 文件上传方式必须为post 并且 enctype="multipart/form-data"-->
    <div class="form-group">
        <label for="exampleInputEmail1">邮箱</label>
        <input type="email" name="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">姓名</label>
        <input type="text" name="username" class="form-control" id="exampleInputPassword1" placeholder="Enter username">
    </div>
    <div class="form-group">
        <label for="exampleInputFile">头像</label>
        <input type="file" name="headerImg" id="exampleInputFile">
    </div>
    <div class="form-group">
        <label for="exampleInputFile">生活照</label>
        <input type="file" name="photos" multiple><!-- multiple：多文件上传-->
    </div>
    <button type="submit" class="btn btn-primary">提交</button>
</form>
```

2. controller处理上传的文件

```java

/**
 * 表单控制器
 * 主要测试文件上传表单
 */
@Controller
@Slf4j//用来做日志打印
public class FormController {
    /**
     * 点击上传文件后
     * MultipartFile 自动封装上传过来的文件
     */
    @PostMapping("/upload")
    public String upload(
            @RequestParam("email")String email,//@RequestParam("email")：从请求参数中获取email的值
            @RequestParam("username")String username,
            @RequestPart("headerImg") MultipartFile headerImg,//@RequestPart("headerImg")获取表单中的文件项，用MultipartFile封装
            @RequestPart("photos") MultipartFile[] photos) throws IOException {

        //用Slf4j打印日志
        log.info("上传的信息：email={},username={},headerImg的大小={},photos的数量={}",email,username,headerImg.getSize(),photos.length);
        //保存文件
        if(!headerImg.isEmpty()){//上传的文件不为空
            headerImg.transferTo(new File("D:/testUpload/"+headerImg.getOriginalFilename()));//把文件保存到d盘testUpload文件夹
        }
        //保存第二个文件(用MultipartFile[]封装多个文件)
        if(photos.length>0){
            for (int i = 0; i < photos.length; i++) {
                if(!photos[i].isEmpty()){
                    photos[i].transferTo(new File("D:/testUpload/"+photos[i].getOriginalFilename()));
                }
            }
        }
        return "main";
    }
}
```

3.springboot自动配置文件上传限制了文件上传大小，可在配置文件修改

```properties
# 文件上传单个文件最大上传10MB
spring.servlet.multipart.max-file-size=10MB
# 最大请求量100MB
spring.servlet.multipart.max-request-size=100MB
```







## 错误处理

### springboot默认错误处理机制

1. 只要报错则会跳转到/error路径 下的指定页面

> - 例如：报错404则先会找**/error/404.html**页面，如果没有则会找**/error/4xx.html**页面，再没有则会默认白页处理
> - （4xx等错误页面也需要引入xmlns:th="http://www.thymeleaf.org"）



2. 可以在页面取出错误信息值

```html
<h1 th:text="${status}">错误状态码</h1>
<h3 th:text="${message}">错误信息</h3>
<p th:text="${trace}">错误信息详细</p>
```

3. 也能在配置文件中修改默认地址

```properties
# 出现异常默认跳转异常页面的地址 /error为默认地址
server.error.path=/error 
```

![image-20211221235240059](C:\Users\67061\AppData\Roaming\Typora\typora-user-images\image-20211221235240059.png)



### 写类处理全局异常

```java
/**
 * 处理整个项目的异常
 */
@Slf4j//打印日志信息
@ControllerAdvice//相当于把类放入容器中(增强处理)
public class GlobalExceptionHandler {
    //处理器处理异常:处理数学运算异常和空指针异常
    @ExceptionHandler({ArithmeticException.class,NullPointerException.class}) 
    public String handleException(Exception e){//出现异常会把异常封装到Exception中
        //出现异常后的操作
        log.error("异常是：{}",e);
        return "login"; //login为视图地址,出现上面指定的异常则返回登录页面
    }
}
```



### 自定义异常

1. 自定义异常类

   ```java
   /**
    * @ResponseStatus： 返回状态吗信息
    * value=HttpStatus.FORBIDDEN：返回403状态码，reason：异常的原因
    * 存储信息，抛出异常后会被springboot默认错误处理机制处理
    */
   @ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "用户数量太多")
   public class UserTooManyException extends RuntimeException{
       public UserTooManyException(){
       }
       public UserTooManyException(String message){
           super(message);
       }
   }
   ```

2. 在想抛异常的位置抛出异常：抛出异常后会被springboot默认错误处理机制处理

   

### 自定义异常解析器

```java
/**
 * 自定义异常解析器
 * 如果优先级高出现所有异常都会往这里跳
 * 抛出异常后会被springboot默认错误处理机制处理
 */
@Order(-666)//设置优先级，数字越小优先级越高
@Component
public class MyFavoriteException implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,Object handler,Exception ex) {
        try {
            response.sendError(566,"我喜欢的错误");//异常状态码与异常信息
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new ModelAndView();
    }
}
```

### 总结

> 除了写类处理全局异常，其他异常基本都能被springboot默认错误处理机制处理

## 原生组件注入

servlet filter listener

https://www.bilibili.com/video/BV19K4y1L7MT?p=56    =====>详细看视频



## 单元测试Junit5

> springboot2.4版本后用junit5做单元测试，junit5不能使用junit4的功能 @Test

如果想要继续兼容junit4需要自行引入vintage

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



### 使用

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <scope>test</scope>
   </dependency>
   ```

2. 直接使用在测试类上写@SpringBootTest注解

   ```java
   @SpringBootTest
   public class TestDemo {
       
       @Autowired
       UserDao userDao;
       
       @Test
       public void test01(){
           User user = userDao.selectById(2);
       }
   }
   ```

### 常用注解与方法

1. **@Transactional** 标注测试方法，测试完成后自动回滚

```java
package com.ltt.springboot.junit5;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import java.util.concurrent.TimeUnit;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Junit5功能测试")
public class Junit5Test {

    @Autowired
    JdbcTemplate jdbcTemplate;
	
    //在所有单元测试之前执行
    @BeforeAll
    static void testBeforeAll() {
        System.out.println("所有测试就要开始了...");
    }
	
    //在所有单元测试之后执行
    @AfterAll
    static void testAfterAll() {
        System.out.println("所有测试已经结束了...");

    }

    /**
     * @DisplayName :为测试类或者测试方法设置展示名称
     */
    @DisplayName("testDisplayName方法")
    @Test
    void testDisplayName() {
        System.out.println(1);
        System.out.println(jdbcTemplate);
    }

    /**
     * 测试前置条件
     */
    @DisplayName("测试前置条件")
    @Test
    void testassumptions() {
        Assumptions.assumeTrue(false, "结果不是true");
        System.out.println("111111");
    }

    /**
     * 断言：前面断言失败，后面的代码都不会执行
     */
    @DisplayName("测试简单断言")
    @Test
    void testSimpleAssertions() {
        int cal = cal(3, 2);
        //测试值是否相等
        assertEquals(5, cal, "业务逻辑计算失败");//期望值5真实的值为cal，如果不相等则报错:业务逻辑计算失败
        Object obj1 = new Object();
        Object obj2 = new Object();
        //测试两个对象是否相等
        assertSame(obj1, obj2, "两个对象不一样");

    }

    @Test
    @DisplayName("array assertion")
    void array() {
        assertArrayEquals(new int[]{2, 2}, new int[]{1, 2}, "数组内容不相等");
    }

    @Test
    @DisplayName("组合断言")
    void all() {
        /**
         * 所有断言全部需要成功
         */
        assertAll("test",
                () -> assertTrue(true && true, "结果不为true"),
                () -> assertEquals(1, 2, "结果不是1"));

        System.out.println("=====");
    }

    @DisplayName("异常断言")
    @Test
    void testException() {

        //断定业务逻辑一定出现异常
        //如果抛出了ArithmeticException则断言成功，不会报错
        assertThrows(ArithmeticException.class, () -> {
            int i = 10 / 0;
        }, "业务逻辑居然正常运行？");
    }

    @DisplayName("快速失败")
    @Test
    void testFail() {
        //xxxxx
        if (1 == 1) {
            fail("测试失败");//直接报错测试失败
        }

    }

    int cal(int i, int j) {
        return i + j;
    }

    /**
     * 此测试方法不会被执行
     */
    @Disabled
    @DisplayName("测试方法2")
    @Test
    void test2() {
        System.out.println(2);
    }

    /**
     * @RepeatedTest(5) :循环测试此方法5次
     */
    @RepeatedTest(5)
    @Test
    void test3() {
        System.out.println(5);
    }

    /**
     * 规定方法超时时间。超出时间测试出异常
     *
     * @throws InterruptedException
     */
    @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)//TimeUnit.MILLISECONDS：设置时间为毫秒
    @Test
    void testTimeout() throws InterruptedException {
        Thread.sleep(400); 
    }

    //表示在每个单元测试之前执行
    @BeforeEach
    void testBeforeEach() {
        System.out.println("测试就要开始了...");
    }
	//表示在每个单元测试之后执行
    @AfterEach
    void testAfterEach() {
        System.out.println("测试结束了...");
    }
}
```



### 嵌套测试

> 在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。
> 内层可以驱动外层，外层不能驱动内层

```java
package com.ltt.springboot.junit5;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;
import org.junit.jupiter.params.provider.ValueSource;

import java.util.EmptyStackException;
import java.util.Stack;
import java.util.stream.Stream;

import static org.junit.jupiter.api.Assertions.*;

/**
 * 在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。
 * 内层可以驱动外层，外层不能驱动内层
 */
@DisplayName("嵌套测试")
public class TestingAStackDemo {

    Stack<Object> stack;

    static Stream<String> stringProvider() {
        return Stream.of("apple", "banana", "atguigu");
    }

    /**
     * 参数化测试
     * @param i
     */
    @ParameterizedTest
    @DisplayName("参数化测试")
    @ValueSource(ints = {1, 2, 3, 4, 5})
    void testParameterized(int i) {
        System.out.println(i);//循环输出1到5
    }

    @ParameterizedTest
    @DisplayName("参数化测试")
    @MethodSource("stringProvider")//也可以传入一个静态的方法返回值必须为Stream<String>
    void testParameterized2(String i) {
        System.out.println(i);
    }

    @Test
    @DisplayName("new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
        //嵌套测试情况下，外层的Test不能驱动内层的Before(After)Each/All之类的方法提前/之后运行
        assertNull(stack);
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested//代表此类为嵌套类
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            /**
             * 内层的Test可以驱动外层的Before(After)Each/All之类的方法提前/之后运行
             */
            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

# 数据访问

1. pom.xml导入jdbc场景和对应的驱动

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-jdbc</artifactId>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.49</version>
   </dependency>
   ```

2. 在配置文件配置数据源

   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/myschool
       username: root
       password: root
       driver-class-name: com.mysql.jdbc.Driver
     jdbc:
       template:
         query-timeout: 3  #设置超时时间3秒
   #    type: com.zaxxer.hikari.HikariDataSource
   ```

3. 使用

   ```java
   @SpringBootTest
   @Slf4j
   class Springboot04ThymeleafApplicationTests {
   
       @Autowired
       JdbcTemplate jdbcTemplate;
       //测试数据访问
       @Test
       void contextLoads() {
           Integer count = jdbcTemplate.queryForObject("select count(*) from student", Integer.class);
           log.info("学生总人数为{}",count);
       }
   
   }
   ```













## 切换druid数据源

1.导依赖

```xml
<!-- 使用druid数据源-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>
```

2.在配置类中配置druid数据源

```java
@Configuration
public class MyDatasourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource")//绑定配置文件中的spring.datasource项，  赋值给此bean的属性
    DataSource dataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        //已在配置文件中配置此处不要配置
        /*dataSource.setUrl();
        dataSource.setUsername();
        ...*/
        return dataSource;
    }
}
```

可以将基本属性写在配置类中 在bean上面用@ConfigurationProperties("spring.datasource")//绑定配置文件中的spring.datasource项，  赋值给此bean的属性

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myschool
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
```



## druid监控页

> 访问监控页的路径如 localhost:8080/druid 可以访问监控页

### 配置方法1

> 需要导入druid依赖 **不推荐**

1. 在配置类中配置servlet

   ```java
   /**
    * 配置druid监控页功能
    * 用原生的方式注册servlet路径。从而配置监控页
    * localhost:8080/druid 可以访问监控页
    */
   @Bean
   public ServletRegistrationBean statViewServlet(){
       StatViewServlet statViewServlet = new StatViewServlet();
       ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(statViewServlet, "/druid/*");
       return registrationBean;
   }
   ```

2. 可以在配置类加入sql监控功能

   ```java
   @Bean
   DataSource dataSource() throws SQLException {
       DruidDataSource dataSource = new DruidDataSource();
       //加入监控功能
       dataSource.setFilters("stat");
       return dataSource;
   }
   ```



### 配置方法2 

> 使用springboot自带的场景启动器 **推荐**

1. 导入场景启动器依赖

   ```xml
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.1.17</version>
   </dependency>
   ```

2.在配置文件中配置示例

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myschool
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
    jdbc:
      template:
        query-timeout: 3  #设置连接数据库超时时间3秒
    # 设置使用druid连接池
    type: com.alibaba.druid.pool.DruidDataSource

    # druid 数据源专有配置
    druid:
      aop-patterns: com.ltt.springboot  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      # 监控页面账号密码
      stat-view-servlet:
        enabled: true #只开启此功能也能进行监控
        login-username: ltt123
        login-password: ltt123
        # 没有重置按钮
        # resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        # 设置过滤的路径与不被过滤的路径
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false


```

2.1 直接在配置文件中配置以下也能开启监控功能

```yaml
druid:
  stat-view-servlet:
    enabled: true
```





## 整合mybaties

> 需要注入datasource数据源

### 基础入门

1. 导入依赖

   ```xml
   <!-- 整合mybaties-->
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.1.4</version>
   </dependency>
   ```

2. 在springboot的配置文件下配置

   ```yaml
   # 整合mybaties
   mybatis:
     #config-location: classpath:mybaties/mybaties_config.xml #指定全局配置文件的位置(不推荐已被mybatis.configuration代替)
     mapper-locations: classpath:mybaties/mapper/*.xml #指定映射配置文件的位置
     configuration: #mybatis.configuration此行代替了全局配置文件，写了此行就不能指定全局配置文件的位置
       map-underscore-to-camel-case: true #开启匹配驼峰命名
   ```

3. 使用

   ```java
   @Mapper
   public interface AccountDao {
       Account getAccount(int id);
   }
   ```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ltt.springboot.dao.AccountDao">
    <select id="getAccount" resultType="com.ltt.springboot.bean.Account">
        select * from  account where id=#{id}
    </select>
</mapper>
```



3.1 使用注解（也能配置文件加注解混合使用）

```java
@Mapper
public interface CityDao {
    @Select("select  * from city where id=#{id}")
    City getCityById(int id);

    /**
     * 添加城市
     * @param city
     */
    //@Insert("insert into city values (null,#{name},#{state},#{country})")
    //@Options(useGeneratedKeys = true,keyProperty = "id")/*useGeneratedKeys="true",keyProperty="id"：获取插入成功的主键值，赋值给实体类id属性*/
    void insert(City city);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ltt.springboot.dao.CityDao">
    <insert id="insert" useGeneratedKeys="true" keyProperty="id"><!--useGeneratedKeys="true",keyProperty="id"：获取插入成功的主键值，赋值给实体类id属性-->
        insert into city values (null,#{name},#{state},#{country})
    </insert>
</mapper>
```

### 小知识点

可以在springboot配置类上使用@MapperScan("com.ltt.springboot.dao")：在此包下的接口就不需要写@mapper注解



## mybatis-plus

1. 导包

   ```xml
   <!-- 引入mybatis-plus-boot-starter就不需要引入mybatis-spring-boot-starter包 -->
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.4.1</version>
   </dependency>
   ```

> **自动配置好了SqlSessionFactory**：底层是容器中默认配置的数据源
>
> **自动配置好了mapperLocations** 。有默认值。classpath\*:/mapper/\**/\*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件，放在 mapper下
>
> **容器中也自动配置好了** **SqlSessionTemplate**
>
> **@Mapper 标注的接口也会被自动扫描**：建议在配置类上写@MapperScan("com.ltt.springboot.dao")批量扫描

2. 优点
   - 只需要dao(mapper)接口继承BaseMapper就可以拥有crud的能力

### 具体注解与使用

1. bean（entity）：以User实体类为例

```java
//@TableName("表名") //如果实体类与数据库的表名不同可以指定表名
public class User {
    /**
     * 所有的属性都应该在数据库中有,如果没有则需要标注注解-@TableField(exist = false)
     */
    @TableField(exist = false)//代表此属性在数据库表中不存在
    private String userName;
    @TableField(exist = false)
    private String passWord;

    //以下是数据库的字段
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

2. 如果用到mybaties分页则要在配置类中配置

   ```java
   @Configuration
   public class MybatiesConfig {
       /**
        * mybaties分页插件配置
        * @return
        */
       @Bean
       public MybatisPlusInterceptor mybatisPlusInterceptor() {
           MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
           PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.H2);
           paginationInnerInterceptor.setOverflow(true);//点击最后一页跳回到首页
           paginationInnerInterceptor.setMaxLimit(500L);//每页最多显示500条
           interceptor.addInnerInterceptor(paginationInnerInterceptor);
           return interceptor;
       }
   
   }
   ```

3. dao接口继承

   ```java
   /**
    * 继承BaseMapper就可以拥有crud的能力,并告诉数据库想要查询的是user表
    */
   
   public interface UserDao extends BaseMapper<User> {
   }
   
   
   //如果继承则加载完springboot容器即可使用，例如在测试类直接注入并使用dao接口
   @Slf4j
   @SpringBootTest
   public class TestMybaties {
       @Autowired
       AccountDao accountDao;
       /**
        * 测试Account表
        */
       @Test
       public void test01(){
           Account account = accountDao.getAccount(2);
           log.info("根据id查出的account是{}",account);
       }
   }
   ```

4. service继承

   ```java
   public interface UserService extends IService<User> {
   }
   ```

5. service实现类

   ```java
   @Service
   public class UserServiceImpl extends ServiceImpl<UserDao, User> implements UserService {
   
   }
   ```

6. 在controller中使用mybaties-plus的查数据并做分页查询

   ```java
   @GetMapping("/dynamic_table")
   public String dynamic_table(Model model,@RequestParam(value = "pn",defaultValue = "1")int pn){//获取当前页参数
       //mybaties_plus查所有
       List<User> list = userService.list();
       
       //mybaties_plus分页查询数据
       Page<User> userPage = new Page<>(pn, 3);//当前页码为pn每页显示3条
       Page<User> page = userService.page(userPage, null);//查询条件为空
       model.addAttribute("page",page);
       return "table/dynamic_table";
   }
   ```





# 指标监控

> 监控项目指标

## 基础使用

1. 引入依赖

   ```xml
   <!-- 引入actuator指标监控-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 在配置文件中配置

   ```yaml
   management:
     endpoints: #配置所有端点的默认行为
       enabled-by-default: true #暴露所有端点信息
       web:
         exposure:
           include: '*'  #以web方式暴露
     endpoint: #也可以配置单个端点
       health:
         show-details: always #总是显示详细健康信息
   ```

3. 测试与使用

   http://localhost:8080/actuator/beans

   http://localhost:8080/actuator/configprops

   http://localhost:8080/actuator/metrics

   http://localhost:8080/actuator/metrics/jvm.gc.pause

   [http://localhost:8080/actuator/](http://localhost:8080/actuator/metrics)endpointName/detailPath
   ............

> **最常用的Endpoint**
>
> ------
>
> - health：监控状况
> - metrics：运行时指标
> - loggers：日志记录



## 自定义监控与可视化监控

详细看https://www.bilibili.com/video/BV19K4y1L7MT?p=79   =====>79,80集



# 环境切换

## 切换环境

1. 在配置文件中切换环境

> 切换配置文件中的配置

```properties
# 激活application-test.yaml环境,两个配置文件都会生效。如果有相同的配置，则以激活的为准
# spring.profiles.active=test

#指定自己分的组
spring.profiles.active=lttnb

#可以分组使用
#激活application-ltt.yaml和application-prod.yaml环境,并存入lttnb组中。
spring.profiles.group.lttnb[0]=ltt 
spring.profiles.group.lttnb[1]=prod
```



2. 也能在cmd中切换环境

C:\Users\67061\Desktop\新建文件夹>java -jar demo-0.0.1-SNAPSHOT. jar --**spring. profiles.active=prod**



## 使用环境的优先级

5的优先级最高1的最低

> (1) classpath 根路径
>
> (2) classpath 根路径下config目录
>
> (3) jar包当前目录
>
> (4) jar包当前目录的config目录
>
> (5) /config子目录的直接子目录