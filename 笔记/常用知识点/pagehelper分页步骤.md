1.导包

```xml
<!--分页包-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.0</version>
</dependency>
```

2.在mybaties配置文件或在SqlSessionFactoryBean中配置插件

```xml
<!--插件-->
<property name="plugins">
    <list>
        <!--做分页-->
        <bean class="com.github.pagehelper.PageInterceptor">
            <property name="properties">
                <props>
                    <prop key="helperDialect">mysql</prop><!--指定是哪种数据库（指定方言）-->
                    <prop key="reasonable">true</prop><!--pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。-->
                </props>
            </property>
        </bean>
    </list>
</property>
```

3.在controller中做分页

```java
@GetMapping("/student")
@ResponseBody
public PageInfo getStudent(int pageNum,int pageSize) {
    /**
     * 做分页
     * 1.设置初始化参数      PageHelper.startPage(pageNum,pageSize);//当前页，每页显示所少条
     * 2.把数据转换成PageInfo对象返回    PageInfo<Student> pi = new PageInfo<>(all);
     */
    PageHelper.startPage(pageNum,pageSize);//当前页，每页显示所少条
    List<Student> all = studentService.getAll();
    PageInfo<Student> pi = new PageInfo<>(all);//PageInfo基本包含分页的所以信息
    return pi;
}
```

4.根据api使用PageInfo对象