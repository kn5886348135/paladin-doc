### log4j2介绍及日志操作



​		常用的日志框架有commons-logging、jul、log4j、log4j2、logback、slf4j等，log4j2是log4j的升级版，提供了重要的升级，并且拥有许多logback的改进特征，修复了logback固有的架构问题。log4j2的主要特征有API独立、性能提升、支持多个API、自动重载配置、避免死锁、高级过滤器、插件架构、java8lambda表达式、自定义日志级别、Log Builder API、免垃圾回收、应用服务器集成、云支持。

[log4j2](http://logging.apache.org/log4j/2.x/javadoc.html)官网

配置文件

<dependencies>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>2.5</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.5</version>
		</dependency>
	</dependencies>

​		log4j在初始化的时候会自动检测配置文件，json、yaml、properties、xml，会按照log4j2-test.properties、log4j2-test.yaml、log4j2-test.json、log4j2-test.xml、log4j2.properties、log4j2.yaml、log4j2.json、log4j2.xml的顺序寻找配置文件。



additivity="false"，默认true会向上传播，root是所有logger的父亲。

monitorInterval最小5秒

配置语法
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" strict="true" name="XMLConfigTest"
               packages="org.apache.logging.log4j.test">
  <Properties>
    <Property name="filename">target/test.log</Property>
  </Properties>
  <Filter type="ThresholdFilter" level="trace"/>

  <Appenders>
    <Appender type="Console" name="STDOUT">
      <Layout type="PatternLayout" pattern="%m MDC%X%n"/>
      <Filters>
        <Filter type="MarkerFilter" marker="FLOW" onMatch="DENY" onMismatch="NEUTRAL"/>
        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="DENY" onMismatch="ACCEPT"/>
      </Filters>
    </Appender>
    <Appender type="Console" name="FLOW">
      <Layout type="PatternLayout" pattern="%C{1}.%M %m %ex%n"/>
<!-- %-d{yyyy-MM-dd HH:mm:ss.SSS} %-6p%c:%L %x - %m%n-->
​      <!-- class and line number -->
​      <Filters>
​        <Filter type="MarkerFilter" marker="FLOW" onMatch="ACCEPT" onMismatch="NEUTRAL"/>
​        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="ACCEPT" onMismatch="DENY"/>
​      </Filters>
​    </Appender>
​    <Appender type="File" name="File" fileName="${filename}">
​      <Layout type="PatternLayout">
​        <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
​      </Layout>
​    </Appender>
  </Appenders>

  <Loggers>
    <Logger name="org.apache.logging.log4j.test1" level="debug" additivity="false">
      <Filter type="ThreadContextMapFilter">
        <KeyValuePair key="test" value="123"/>
      </Filter>
      <AppenderRef ref="STDOUT"/>
    </Logger>
    <Logger name="org.apache.logging.log4j.test2" level="debug" additivity="false">
      <AppenderRef ref="File"/>
    </Logger>
    <Root level="trace">
      <AppenderRef ref="STDOUT"/>
    </Root>
  </Loggers>

</Configuration>

appenders、logger的配置，异步loggers的配置，配合disruptor等

多环境切换



桥接包

占位符输出 logger.debug("{} {}","hello","tom")

MDC（Mapped Diagnostic Context，映射调试上下文）

MDC 可以看成是一个与当前线程绑定的哈希表，可以往其中添加键值对。MDC 中包含的内容可以被同一线程中执行的代码所访问。当前线程的子线程会继承其父线程中的 MDC 的内容。当需要记录日志时，只需要从 MDC 中获取所需的信息即可。MDC 的内容则由程序在适当的时候保存进去。对于一个 Web 应用来说，通常是在请求被处理的最开始保存这些数据。

https://www.freesion.com/article/6391647219/

​	

