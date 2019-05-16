[详解tomcat的连接数与线程池](http://www.cnblogs.com/kismetv/p/7806063.html)

[详解Tomcat 配置文件server.xml](https://www.cnblogs.com/kismetv/p/7228274.html#title3-3)

server.xml文件中，使用`Executor`标签配置Tomcat的线程池。

`Executor`标签可以由其他组件共享使用；要使用该线程池，组件需要通过`executor`属性指定该线程池。

`Executor`是`Service`元素的内嵌元素。

一般来说，使用线程池的是`Connector`组件；为了使`Connector`能使用线程池，`Executor`元素应该放在`Connector`前面。`Executor`与`Connector`的配置举例如下：

```xml
<Executor name="tomcatThreadPool" namePrefix ="catalina-exec-" maxThreads="150" minSpareThreads="4" />
<Connector executor="tomcatThreadPool" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" acceptCount="1000" />
```

Executor的主要属性包括：
|属性|含义|
---|---|---
|name | 该线程池的标记
|maxThreads | 线程池中最大活跃线程数，默认值200（Tomcat7和8都是）
|minSpareThreads | 线程池中保持的最小线程数，最小值是25
|maxIdleTime | 线程空闲的最大时间，当空闲超过该值时关闭线程（除非线程数小于minSpareThreads），单位是ms，默认值60000（1分钟）
|daemon | 是否后台线程，默认值true
|threadPriority | 线程优先级，默认值5
|namePrefix | 线程名字的前缀，线程池中线程名字为: namePrefix+线程编号
