一、项目搭建

1、https://start.spring.io/或者使用IDEA自带的Spring Assistant创建工程，依赖管理使用Gradle

2、build.gradle加入阿里云镜像仓库或者gradle配置全局的阿里云镜像仓库

3、Modules添加Spring，使配置文件的属性可以自动提示

4、Mybatis配置

5、mybatis-plus代码生成器

6、yaml插件YAML，Yaml/Ansible support，配置项自动提示



二、配置Druid连接池

1、引入依赖

2、数据库连接池信息、数据源配置、监控配置

3、属性的大小写

4、springboot中使用log4j2需要排除springboot自带的log组件，在build.gradle脚本中配置configurations

5、springcloud-gateway基于webflux和netty，否则会报错



使用[hikari](https://github.com/brettwooldridge/HikariCP)替代druid，没有基于Java代码配置的DataSource，mybatis-plus的MybatisConfigurer配置会出现`Invalid bound statement (not found)`的异常，因为HikariDataSource 为null。





三、请求结果封装

正常请求返回的http status为200，异常请求返回的http status不能为200，要求根据具体场景给出对应的http status和对应的状态码

可以使用((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse()暴露response，注意response可能为null

使用spring自带的ResponseEntity，不过要求返回值类型必须是ResponseEntity



四、自定义消息转换器

控制返回值能否为null



五、全局异常处理

@ExceptionHandler或者重写WebMvcConfigurationSupport的configureHandlerExceptionResolvers方法

SpringMVC官方示例是使用**WebMvcConfigurer** 实现MVC配置的，[spring官网MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-interceptors)

```java
DispatcherServlet代理一个异常处理器(HandlerExceptionResolver)链,configureHandlerExceptionResolvers是在@ExceptionHandler前面
```

[异常处理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers)

[@ExceptionHandler](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-exceptionhandler)详情



六、整合在线文档

swagger和knife4j

注意使用@profile注解在生产环境关闭

```properties
knife4j.production=true
```

在生产环境屏蔽接口文档

swagger依赖的spring-plugin-core版本比spring-boot-starter的要低，getPluginFor方法签名发生变化，发生异常



七、PageHelper分页

PageInfo的分页参数

//当前页 private int pageNum; 
//每页的数量 private int pageSize;
//当前页的数量 private int size; 
//当前页面第一个元素在数据库中的行号 private int startRow; 
//当前页面最后一个元素在数据库中的行号 private int endRow;
//总记录数 private long total; 
//总页数 private int pages; 
//结果集 private List<T> list; 
//第一页 private int firstPage; 
//前一页 private int prePage; 
//是否为第一页 private boolean isFirstPage; 
//是否为最后一页 private boolean isLastPage; 
//是否有前一页 private boolean hasPreviousPage; 
//是否有下一页 private boolean hasNextPage; 
//导航页码数 private int navigatePages; 
//所有导航页号 private int[] navigatepageNums; 

官方文档

https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md



八、通用Mapper

通用Mapper，可以用tk.mybatis和mybatis-plus实现，快速实现单表的增删改查功能



九、代码生成器



十二、配置redis

spring官方推荐的是lettuce或者jedis



十三、防XSS攻击

使用jsoup框架



十四、安全相关

集成shiro或者spring-security，安全问题不仅仅是认证、授权，还需要做CSRF等各种攻击防护



十八、加载资源

CommandLineRunner ApplicationRunner



十九、添加拦截器



二十二、图片压缩

Nginx也有gzip等各种格式的压缩



二十三、前后端分离的跨域、重定向问题

@Configuration使配置生效

CorsFilter处理Options请求，返回满足需要的headers，跨域问题详情参考[阮一峰博客](http://www.ruanyifeng.com/blog/2016/04/cors.html)