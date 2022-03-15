## 基本配置

1. 在根目录下导入**log4j.properties**在运行时控制台内可以获取日志信息，查询语句等
2. 如果使用映射配置而不是注解的话，映射路径应该和类路径保持一致





## 主配置文件的配置

SqlMapConfig.xml文件配置在resources根目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 主配置文件-->
    <configuration>
        <!--配置别名-->
        <typeAliases>
            <!-- typeAlias用于配置别名。type属性指定的是实体类全限定类名。alias属性指定别名，当指定了别名就再区分大小写-->
            <!--<typeAlias type="com.itheima.domain.User" alias="user"></typeAlias>-->
            <!--用于指定要配置别名的包，当指定之后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写-->
            <package name="com.itheima.domain"/>
        </typeAliases>

        <!-- 配置环境-->
        <environments default="mysql"><!--default:选择的默认值，选择其中一个environment-->
            <!-- 配置mysql的环境-->
            <environment id="mysql">
                <!-- 配置事务类型-->
                <transactionManager type="JDBC"></transactionManager>

                <!-- 配置连接池（连接池）
                    type:
                        POOLED:从池中获取一个连接来用
                        UNPOOLED:每次创建一个新的连接用
                -->
                <dataSource type="POOLED">
                    <!--配置连接数据库的基本信息-->
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/eesy"/>
                    <property name="username" value="root"/>
                    <property name="password" value="root"/>
                </dataSource>
            </environment>
        </environments>

        <!-- 配置映射文件的位置-->
        <mappers>
            <!-- package标签是用于指定dao接口所在的包,当指定了之后就不需要在写mapper以及resource或者class了，写package必须保证映射的路径和名称对应-->
            <package name="com.itheima.dao"/>
            <!--<mapper resource="com/itheima/dao/UserDao.xml"/>-->
        </mappers>
    </configuration>
```





## pom.xml：以及需要导入的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>day03_02dynamicSQL</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging><!--默认值为jar-->

    <dependencies>
        <!--mybatis依赖-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>
        <!--数据库连接依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
        <!--日志部分-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <!--junit单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>
    </dependencies>

</project>
```



## 使用xml映射的话

### 了解内容

1. 抽取重复的Sql语句

   ```xml
   <!-- 了解内容：抽取重复的SQL语句-->
   <!-- 抽取-->
   <sql id="defaultSQL">
       select * from user
   </sql>
   <!-- 使用-->
   <include refid="defaultSQL"></include>
   ```

### resultMap部分

#### 一对一的关系映射

例：（以用户表和账户表为例）

```xml
<!-- 前提：在Account类中包含了一个主表实体User类的对象引用-->
<!-- 定义封装account和user的resultMap-->
<resultMap id="accountUserMap" type="Account">
    <!-- 配置account的相关信息-->
    <id column="aid" property="id"></id>
    <result column="uid" property="uid"></result>
    <result column="money" property="money"></result>

    <!-- 一对一的关系映射:配置封装user的内容
            property：封装到类中的变量名
            uid：根据那个字段来获取的用户信息
            javaType：封装到哪个对象
    -->
    <association property="user" column="uid" javaType="user">
        <id column="id" property="id"></id>
        <result column="username" property="username"></result>
        <result column="birthday" property="birthday"></result>
        <result column="sex" property="sex"></result>
        <result column="address" property="address"></result>
    </association>
</resultMap>
```

#### 一对多的关系映射

例：（以用户表和账户表为例）一个用户对多个账户

```xml
<!-- 前提：主表实体应该包含从表实体的集合引用-->
<!-- 定义User的resultMap-->
<resultMap id="userAccountMap" type="user">
    <!-- 配置User对象自身的属性-->
    <id column="id" property="id"></id>
    <result column="username" property="username"></result>
    <result column="birthday" property="birthday"></result>
    <result column="sex" property="sex"></result>
    <result column="address" property="address"></result>
    <!-- 配置User对象accounts集合的映射
            property：User对象中集合的名字
            ofType：集合的自身类型
    -->
    <collection property="accounts" ofType="account">
        <id column="aid" property="id"></id>
        <result column="MONEY" property="money"></result>
        <result column="uid" property="uid"></result>
    </collection>
</resultMap>
```



## 延迟加载

1. 在主配置文件SqlMapConfig.xml下的configuration下配置参数

```xml
<configuration>
    <!--配置参数-->
    <settings>
        <!--开启Mybatis支持延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"></setting>
    </settings>
</configuration>
```

2. 配置对应的resultMap部分，与select查询部分	例如：账户表延迟加载用户表举例

   ```xml
   <resultMap id="accountUserMap" type="Account">
       <!-- 配置account的相关信息-->
       <id column="aid" property="id"></id>
       <result column="uid" property="uid"></result>
       <result column="money" property="money"></result>
   
       <!-- 延迟加载，与一对一关系映射不同
               一对一的关系映射:配置封装user的内容
                   property：封装到类中的变量名
                   uid：根据哪个字段来获取的用户信息
                   javaType：封装到哪个对象
               延迟加载：
                   select属性指定的内容:查询用户的唯一标识:
                   column属性指定的内容:用户根据id查询时，所需要的参数的值
                   会带着column参数的值去找UserDao.xml的标签select下id为findById的方法，通过column参数的uid值获取到对应的数据库user信息
       -->
   
       <association property="user" column="uid" javaType="user" select="com.itheima.dao.UserDao.findById"></association>
   </resultMap>
   
   <!-- 延迟加载配置查询所有改为select * from Account;要不所有信息都会被提前加载-->
   <select id="findAll" resultMap="accountUserMap">
       select * from Account;
       <!-- SELECT u.*,a.id as aid,a.MONEY,a.UID FROM account a,USER u WHERE u.id=a.UID; -->
   </select>
   ```

一对多情况的延迟加载user表为例

```xml
<!-- 定义User的resultMap-->
<resultMap id="userAccountMap" type="user">
    <!-- 配置User对象自身的属性-->
    .......省略
    
    <!-- 配置User对象accounts集合的映射
            property：User对象中集合的名字
            ofType：集合的自身类型
        延迟加载：不需要collection方法体内容
            select属性指定的内容:查询账户表的唯一标识
            column携带的参数，根据用户id查询account表的信息
		   根据select地址携带column参数获取账户account表中的对应集合信息

    -->
    <collection property="accounts" ofType="account" select="com.itheima.dao.AccountDao.findAccountByUid" column="id">
        <!--<id column="aid" property="id"></id>
        <result column="MONEY" property="money"></result>
        <result column="uid" property="uid"></result>-->
    </collection>
</resultMap>
```









## 缓存




### 大体了解Mybatis缓存

**缓存简要概述：**
		存在于内存中的临时数据。减少和数据库的交互次数，提高执行效率。
**适用于缓存：**
		经常查询并且不经常改变的。
**不适用于缓存：**
		经常改变的数据




### Mybatis中的一级缓存和二级缓存

**一级缓存**：
			它指的是Mybatis中SqlSession对象的缓存。
			当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供一块区域中。
			该区域的结构是一个Map。当我们再次查询同样的数据，mybatis会先去sqlsession中
			查询是否有，有的话直接拿出来用。
			当SqlSession对象消失时，mybatis的一级缓存也就消失了。

```java
//开启关闭SqlSession对象缓存消失，并且查询了两次select * from user where id=?
session.close();//关闭SqlSession对象
SqlSession session = factory.openSession();//再次获取SqlSession对象，缓存消失
UserDao userDao = session.getMapper(UserDao.class);

//此方法也能清除缓存
session.clearCache();
```

**这些操作也会清空一级缓存**

当调用SqlSession的修改，添加，删除，commit()，close()等方法时也会清除一级缓存。



**二级缓存**:
			它指的是Mybatis中SqlSessionFactory对象的缓存。由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。
			**二级缓存的使用步骤：**
				第一步：让Mybatis框架支持二级缓存（在SqlMapConfig.xml中配置）

```xml
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/><!--全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。不配也行默认值为true-->
    </settings>
</configuration>
```

​				第二步：让当前的映射文件支持二级缓存（在IUserDao.xml中配置）
​				第三步：让当前的操作支持二级缓存（在select标签中配置）

```xml
<mapper namespace="com.itheima.dao.UserDao"> <!--UserDao接口的全限定类名-->
    <!--第二步-->
    <!--让当前的映射文件支持二级缓存（在UserDao.xml中配置）-->
    <cache/>
    <!--第三步-->
    <!--让当前的操作支持二级缓存（在select标签中配置useCache属性为true）-->
    <select id="findById" parameterType="int" resultType="user" useCache="true">
        select * from user where id=#{id};
    </select>
<mapper/>
```

​	



​			

## 使用注解相关

### 简介

在mybatis中针对CRUD一共有四个注解： @Select  @Insert  @Update  @Delete，使用注解就不得配置对应的xml映射

### mybatis注解建立实体类属性和数据库表中列的对应关系

例如：数据库列明与属性名不一致，拿查询所有和根据id查询为例

```java
/**
 * 查询所有用户
 */
@Results(id = "userResults",value = {//此处id是为Results取名字(唯一标识),在value中配置列明与实体类中属性对应的值
        @Result(id = true,column = "id",property = "userId"),//此处id代表主键 column为列名 property为实体类中的属性名
        @Result(id = true,column = "username",property = "userName"),
        @Result(id = true,column = "address",property = "userAddress"),
        @Result(id = true,column = "sex",property = "userSex"),
        @Result(id = true,column = "birthday",property = "userBirthday"),
})
@Select("select * from user")
List<User> findAll();

/**
 * 根据id查询用户
 */
@ResultMap(value = {"userResults"})//ResultMap注解指定对应id为"userResults"的Results注解
@Select("select * from user  where id=#{id} ")
User findById(Integer userId);
```



### 一对一的查询配置

例子：以账户表里面包含一个用户为例

```java
/**
 * 查询所有用户，并且获取每个账户所属的用户信息
 * @return
 */
//为了能够查询用户信息需要创建映射
@Results(id = "accountMap",value = {
        //创建Account自己的映射
        @Result(id = true,column = "id",property = "id"),
        @Result(column = "uid",property = "uid"),
        @Result(column = "money",property = "money"),
       /*
       * property：对Account类的user属性赋值
       * @One 查询一个结果
       * select：找到com.itheima.dao.IUserDao.findById提供column的属性uid，并把查询结果赋值给user用户
       * fetchType = FetchType.LAZY使用的是懒查询
       */
        @Result(property = "user",column = "uid",one = @One(select = "com.itheima.dao.IUserDao.findById",fetchType = FetchType.LAZY))
})
@Select("select * from account")
List<Account> findAll();
```

### 一对多查询配置

例子：以用户表和账户表为例，查询所以用户的同时获取用户所对应的账户信息

```java
/**
 * 查询所有用户
 * @return
 */
@Results(id = "userResults",value = {//此处id是为Results取名字(唯一标识),在value中配置列明与实体类中属性对应的值
        //配置用户自己的信息
        @Result(id = true,column = "id",property = "userId"),//此处id代表主键 column为列名 property为实体类中的属性名
        @Result(id = true,column = "username",property = "userName"),
        @Result(id = true,column = "address",property = "userAddress"),
        @Result(id = true,column = "sex",property = "userSex"),
        @Result(id = true,column = "birthday",property = "userBirthday"),
        /*
            property：赋值到User类的属性名
            @Many：一对多的查询
            select：select：指定你要执行操作的全限定方法名
            通过com.itheima.dao.AccountDao.findByUid传入column属性的值把查询的结果返回给property属性
            fetchType = FetchType.LAZY：懒查询
         */
        @Result(property = "accounts",column = "id",many = @Many(select = "com.itheima.dao.AccountDao.findByUid",fetchType = FetchType.LAZY))
})
@Select("select * from user")
List<User> findAll();
```



### 注解开启二级缓存

在方法对应的接口上定义此注解

```
@CacheNamespace(blocking = true)
```





# 第二遍

1. 获取自增值

![image-20220226213352199](D:\学习ing\Java\编程笔记图\image-20220226213352199.png)

2. #{value} 更安全，并相当于 'value' 