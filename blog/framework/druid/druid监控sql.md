使用durid监控程序连接的数据库相关状态

1. 在web.xml文件中，配置：
```xml
<filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    	<init-param>  
      	<param-name>sessionStatMaxCount</param-name>  
      	<param-value>2000</param-value>  
  	</init-param>
  	<init-param>
        <param-name>sessionStatEnable</param-name>  
        <param-value>true</param-value>  
    </init-param>
    <init-param>  
        <param-name>principalSessionName</param-name>  
        <param-value>session_user_key</param-value>  
    </init-param>  
    <init-param>  
        <param-name>profileEnable</param-name>  
        <param-value>true</param-value>  
    </init-param>
</filter>

<filter-mapping>
     <filter-name>DruidWebStatFilter</filter-name>
     <url-pattern>/webservice/*</url-pattern>
</filter-mapping>

<servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    <init-param>  
        <!-- 允许清空统计数据 -->  
        <param-name>resetEnable</param-name>  
        <param-value>true</param-value>  
    </init-param>  
    <init-param>  
        <!-- 用户名 -->  
        <param-name>loginUsername</param-name>  
        <param-value>admin</param-value>  
    </init-param>  
    <init-param>  
        <!-- 密码 -->  
        <param-name>loginPassword</param-name>	          
        <param-value>admin</param-value>
    </init-param>    
</servlet>   
<servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>
</servlet-mapping>
```

2. 启动项目，浏览器中访问 `localhost:端口号/项目名/druid/index.html` , 输入配置的用户名、密码，即可查看

浏览器中查看结果

![浏览器中查看结果](https://raw.githubusercontent.com/RebeccaZhong/markdown-photos/master/%E4%B8%93%E4%B8%9A%E6%8A%80%E8%83%BD/framework/druid/duild-%E6%B5%8F%E8%A7%88%E5%99%A8%E8%AE%BF%E9%97%AE%E9%A1%B5%E9%9D%A2.png)
