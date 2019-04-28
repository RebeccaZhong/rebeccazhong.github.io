MyBatis配置多数据源

- [分类](#分类)
	- [准备工作](#准备工作)
	- [多数据源配置](#多数据源配置)
	- [动态数据源配置](#动态数据源配置)
- [参考链接](#参考链接)


# 分类

MyBatis的多数据源配置分2种，

1. 多数据源配置：两个库业务互不相干，a方法使用a库的数据，b方法使用b库的数据；
2. 动态数据源配置：两个库业务有关联，如读写分离库。

第一种直接配置2个单独的数据源，不同模块引入不同的sqlSessionFactory即可；第二种需要配置可动态切换的数据源。

## 准备工作
两种方式都需要在pom.xml文件引入不同数据库需要的jar包，此处以MySQL和Oracle为例：

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.40</version>
</dependency>
<dependency>
  <groupId>com.oracle</groupId>
  <artifactId>ojdbc</artifactId>
  <version>7</version>
</dependency>
```

此处Oracle的jar包因为版权问题，在maven中央仓库中没有，需要手动安装。 [安装教程](https://www.cnblogs.com/leiOOlei/p/3380568.html)

## 多数据源配置

在applicationContext.xml中配置两套数据源即可
```xml
<!-- ===============第一个数据源的配置开始=============== -->
<!--MySQL的数据库配置-->
<bean id="mysqlDtaSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <property name="driverClassName" value="${mysql.driverClassName}"/>
    <property name="url" value="${mysql.url}"/>
    <property name="username" value="${mysql.username}"/>
    <property name="password" value="${mysql.password}"/>
    <!-- 初始化连接大小 -->
    <property name="initialSize" value="${mysql.druid.initialSize}"></property>
    <!-- 连接池最大数量 -->
    <property name="maxActive" value="${mysql.druid.maxActive}"></property>
    <!-- 连接池最大空闲 -->
    <property name="maxIdle" value="${mysql.druid.maxIdle}"></property>
    <!-- 连接池最小空闲 -->
    <property name="minIdle" value="${mysql.druid.minIdle}"></property>
    <!-- 获取连接最大等待时间 -->
    <property name="maxWait" value="${mysql.druid.maxWait}"></property>
</bean>
<!--配置MySQL的SqlSessionFactory-->
<bean id="sqlSessionFactoryMySQL" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="mysqlDtaSource"/>
    <property name="configLocation" value="classpath*:mybatis-config.xml" />
    <!-- 自动扫描mapping.xml文件 -->
    <property name="mapperLocations" value="classpath*:com/rebecca/mybatismutildb/mysql/**/dao/mapper/*Mapper.xml"/>
    <property name="typeAliasesPackage" value="com.rebecca.mybatismutildb.mysql.**"/>
</bean>
<!--告诉框架扫描指定包中的mapper接口,然后为其自动的生成代理对象-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.rebecca.mybatismutildb.mysql.**.dao"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryMySQL"/>
</bean>
<!--配置事务管理器-->
<bean id="mysqlTxManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="mysqlDtaSource"/>
</bean>
<!--配置事务的传播特性-->
<tx:advice id="txAdvice" transaction-manager="mysqlTxManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="list*" read-only="true"/>
        <tx:method name="query*" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<!-- ===============第一个数据源的配置结束=============== -->

<!-- ===============第二个数据源的配置开始=============== -->
<!--Oracle的数据库配置-->
<bean id="oracleDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${oracle.driverClassName}" />
    <property name="url" value="${oracle.url}" />
    <property name="username" value="${oracle.username}" />
    <property name="password" value="${oracle.password}" />
    <!-- 初始化连接大小 -->
    <property name="initialSize" value="${oracle.druid.initialSize}"></property>
    <!-- 连接池最大数量 -->
    <property name="maxActive" value="${oracle.druid.maxActive}"></property>
    <!-- 连接池最大空闲 -->
    <property name="maxIdle" value="${oracle.druid.maxIdle}"></property>
    <!-- 连接池最小空闲 -->
    <property name="minIdle" value="${oracle.druid.minIdle}"></property>
    <!-- 获取连接最大等待时间 -->
    <property name="maxWait" value="${oracle.druid.maxWait}"></property>
</bean>
<!--配置SqlSessionFactory-->
<bean id="sqlSessionFactoryOracle" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="oracleDataSource"/>
    <property name="configLocation" value="classpath:mybatis-config.xml" />
    <!-- 自动扫描mapping.xml文件 -->
    <property name="mapperLocations" value="classpath*:com/rebecca/mybatismutildb/oracle/**/dao/mapper/*Mapper.xml">
    </property>
    <property name="typeAliasesPackage" value="com.rebecca.mybatismutildb.oracle.**"/>
</bean>
<!--告诉框架扫描指定包中的mapper接口,然后为其自动的生成代理对象-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.rebecca.mybatismutildb.oracle.**.dao"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryOracle"/>
</bean>

<!--配置事务管理器-->
<bean id="txManagerOracle" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="oracleDataSource"/>
</bean>

<!--配置事务的传播特性-->
<tx:advice id="txAdviceOracle" transaction-manager="txManagerOracle">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="list*" read-only="true"/>
        <tx:method name="query*" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<!-- ===============第二个数据源的配置结束=============== -->
```

此处第一个数据源指定去解析`com/rebecca/mybatismutildb/mysql/**/dao/mapper`包下的`*Mapper.xml`文件，即指定这些dao的方法去连MySQL的数据源；第二个数据源指定去解析`com/rebecca/mybatismutildb/oracle/**/dao/mapper`包下的`*Mapper.xml`文件，即指定这些dao的方法去连Oracle的数据源。

applicationContext.xml配置好后，直接在指定的包内编码即可。

## 动态数据源配置

1. 在applicationContext.xml中配置动态切换数据源
```xml

<!--MySQL的数据库配置-->
<bean id="mysqlDtaSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <property name="driverClassName" value="${mysql.driverClassName}"/>
    <property name="url" value="${mysql.url}"/>
    <property name="username" value="${mysql.username}"/>
    <property name="password" value="${mysql.password}"/>
    <!-- 初始化连接大小 -->
    <property name="initialSize" value="${mysql.druid.initialSize}"></property>
    <!-- 连接池最大数量 -->
    <property name="maxActive" value="${mysql.druid.maxActive}"></property>
    <!-- 连接池最大空闲 -->
    <property name="maxIdle" value="${mysql.druid.maxIdle}"></property>
    <!-- 连接池最小空闲 -->
    <property name="minIdle" value="${mysql.druid.minIdle}"></property>
    <!-- 获取连接最大等待时间 -->
    <property name="maxWait" value="${mysql.druid.maxWait}"></property>
</bean>
<!--Oracle的数据库配置-->
<bean id="oracleDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${oracle.driverClassName}" />
    <property name="url" value="${oracle.url}" />
    <property name="username" value="${oracle.username}" />
    <property name="password" value="${oracle.password}" />
    <!-- 初始化连接大小 -->
    <property name="initialSize" value="${oracle.druid.initialSize}"></property>
    <!-- 连接池最大数量 -->
    <property name="maxActive" value="${oracle.druid.maxActive}"></property>
    <!-- 连接池最大空闲 -->
    <property name="maxIdle" value="${oracle.druid.maxIdle}"></property>
    <!-- 连接池最小空闲 -->
    <property name="minIdle" value="${oracle.druid.minIdle}"></property>
    <!-- 获取连接最大等待时间 -->
    <property name="maxWait" value="${oracle.druid.maxWait}"></property>
</bean>
<!--告诉框架扫描指定包中的mapper接口,然后为其自动的生成代理对象-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.rebecca.mybatismutildb.mysql.**.dao,com.rebecca.mybatismutildb.oracle.**.dao"/>
</bean>
<!--配置事务管理器-->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!--配置事务的传播特性-->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="list*" read-only="true"/>
        <tx:method name="query*" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>

<!--配置SqlSessionFactory（此处自定义实现SqlSessionFactoryBean，以支持mapperLocations的通配符配置）-->
<bean id="sqlSessionFactory" class="com.rebecca.mybatismutildb.db.PackagesSqlSessionFactoryBean" scope="prototype">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatis-config.xml" />
    <!-- 自动扫描mapping.xml文件 -->
    <property name="mapperLocations">
        <list>
            <value>classpath*:com/rebecca/mybatismutildb/mysql/**/dao/mapper/*Mapper.xml</value>
            <value>classpath*:com/rebecca/mybatismutildb/oracle/**/dao/mapper/*Mapper.xml</value>
        </list>
    </property>
    <property name="typeAliasesPackage" value="com.rebecca.mybatismutildb.**"/>
</bean>
<!--动态配置数据源-->
<bean id="dataSource" class="com.rebecca.mybatismutildb.db.DynamicDataSource" >
    <property name="targetDataSources">
        <map key-type="java.lang.String">
            <!--通过不同的key决定用哪个dataSource-->
            <entry value-ref="mysqlDtaSource" key="mysqlDtaSource"></entry>
            <entry value-ref="oracleDataSource" key="oracleDataSource"></entry>
        </map>
    </property>
    <!--设置默认的dataSource-->
    <property name="defaultTargetDataSource" ref="mysqlDtaSource">
    </property>
</bean>

<!--动态数据源切换 -->
<bean id="dataSourceAspect" class="com.rebecca.mybatismutildb.db.DataSourceAspect" />
<!--动态数据源切换 spring的aop切面-->
<aop:config>
    <!--请设置 @Order(0)。否则可能出现 数据源切换失败问题！ 因为要在事务开启之前就进行判断，并进行切换数据源！-->
    <aop:aspect ref="dataSourceAspect" order="0">
        <!--expression中要配置多个包时，注意用or连接，如：execution(* com.rebecca.mybatismutildb.mysql..dao..*.*(..)) or execution(* com.rebecca.mybatismutildb.oracle..dao..*.*(..)) ; 即拦截com.rebecca.mybatismutildb.mysql和com.rebecca.mybatismutildb.oracle下所有的dao包 -->
        <aop:pointcut id="dataSourcePointcut" expression="execution(* com.rebecca.mybatismutildb..dao..*.*(..)) "/>
        <aop:before pointcut-ref="dataSourcePointcut" method="changeDataSource" />
        <aop:after pointcut-ref="dataSourcePointcut" method="removeDataSource" />
    </aop:aspect>
</aop:config>

```

自定义实现SqlSessionFactoryBean，以支持mapperLocations的通配符配置
PackagesSqlSessionFactoryBean, DynamicDataSource, 切面注解 DataSourceAspect

2. applicationContext.xml中用到的相关类
- DataSource 注解
```java
package com.rebecca.mybatismutildb.db;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * DataSource的注解
 * @Author: rebecca
 * @Date: Created in 2019/4/25 17:27
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface DataSource {
    String value();
}
```
- DataSourceAspect 切面
```java
package com.rebecca.mybatismutildb.db;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Method;

/**
 * 拦截所有带有注解@DataSource的类和方法
 * @Author: rebecca
 * @Date: Created in 2019/4/25 17:28
 */
public class DataSourceAspect {
    private static final org.slf4j.Logger logger = LoggerFactory.getLogger(DataSourceAspect.class);
    /**
     * @Title: changeDataSource
     * @Description: 拦截目标方法，获取由@DataSource指定的数据源标识，设置到线程存储中以便切换数据源
     * @param joinPoint
     * @return: void
     */
    public void changeDataSource(JoinPoint joinPoint){
        Class<?> target = joinPoint.getTarget().getClass();
        MethodSignature methodSignature = (MethodSignature)joinPoint.getSignature();
        for(Class<?> clazz : target.getInterfaces()){
            resolveDataSource(clazz,methodSignature.getMethod());
        }
        resolveDataSource(target,methodSignature.getMethod());
        logger.debug("数据源切换至--->", DataSourceContextHolder.getDbType());
    }

    private void resolveDataSource(Class<?> clazz, Method method){
        try {
            if(clazz.isAnnotationPresent(DataSource.class)){
                DataSource source = clazz.getAnnotation(DataSource.class);
                DataSourceContextHolder.setDbType(source.value());
            }

            Class<?>[] types = method.getParameterTypes();
            Method m = clazz.getDeclaredMethod(method.getName(), types);
            if(null != m && m.isAnnotationPresent(DataSource.class)){
                DataSource source = m.getAnnotation(DataSource.class);
                DataSourceContextHolder.setDbType(source.value());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void removeDataSource() {
        try {
            DataSourceContextHolder.clearDbType();
            logger.debug("数据源已移除！");
        } catch (Exception e) {
            e.printStackTrace();
            logger.debug("数据源移除报错！", e);
        }
    }
}

```

- DataSourceContextHolder 类
```java
package com.rebecca.mybatismutildb.db;

/**
 * @Author: rebecca
 * @Description:
 * @Date: Created in 2019/3/28 11:43
 * @Modified By:
 */
public class DataSourceContextHolder {
    // 注意此处的属性值要与applicationContext.xml中配置的一致
    public static final String DATA_SOURCE_MYSQL = "mysqlDtaSource";
    public static final String DATA_SOURCE_ORACLE = "oracleDataSource";
    //用ThreadLocal来设置当前线程使用哪个dataSource
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>();

    public static void setDbType(String customerType) {
        contextHolder.set(customerType);
    }

    public static String getDbType() {
        return contextHolder.get();
    }

    public static void clearDbType() {
        contextHolder.remove();
    }
}

```

- DynamicDataSource 类
```java
package com.rebecca.mybatismutildb.db;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import java.util.logging.Logger;

/**
 * 动态切换数据源
 * @Author: rebecca
 * @Description:
 * @Date: Created in 2019/3/28 11:42
 * @Modified By:
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDbType();
    }

    @Override
    public Logger getParentLogger() {
        return null;
    }
}
```

- PackagesSqlSessionFactoryBean 类

```java
package com.rebecca.mybatismutildb.db;

import org.apache.commons.lang.StringUtils;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.core.type.classreading.CachingMetadataReaderFactory;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.util.ClassUtils;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * 支持包名的通配符解析
 * @Author: rebecca
 * @Description:
 * @Date: Created in 2019/3/28 15:30
 * @Modified By:
 */
public class PackagesSqlSessionFactoryBean extends SqlSessionFactoryBean {

    static final String DEFAULT_RESOURCE_PATTERN = "**/*.class";

    private Logger logger = LoggerFactory.getLogger(PackagesSqlSessionFactoryBean.class);

    @Override
    public void setTypeAliasesPackage(String typeAliasesPackage) {
        ResourcePatternResolver resolver = (ResourcePatternResolver) new PathMatchingResourcePatternResolver();
        MetadataReaderFactory metadataReaderFactory = new CachingMetadataReaderFactory(resolver);
        typeAliasesPackage = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
                ClassUtils.convertClassNameToResourcePath(typeAliasesPackage) + "/" + DEFAULT_RESOURCE_PATTERN;

        // 将加载多个绝对匹配的所有Resource
        // 将首先通过ClassLoader.getResource("META-INF")加载非模式路径部分
        // 然后进行遍历模式匹配
        try {
            List<String> result = new ArrayList<String>();
            Resource[] resources =  resolver.getResources(typeAliasesPackage);
            if(resources != null && resources.length > 0){
                MetadataReader metadataReader = null;
                for(Resource resource : resources){
                    if(resource.isReadable()){
                        metadataReader =  metadataReaderFactory.getMetadataReader(resource);
                        try {
                            result.add(Class.forName(metadataReader.getClassMetadata().getClassName()).getPackage().getName());
                        } catch (ClassNotFoundException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
            if(result.size() > 0) {
                super.setTypeAliasesPackage(StringUtils.join(result.toArray(), ","));
            }else{
                logger.warn("参数typeAliasesPackage:"+typeAliasesPackage+"，未找到任何包");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

3. 在对应的dao的接口上，增加`@DataSource(string value)`注解，其中value的值是`DataSourceContextHolder.DATA_SOURCE_MYSQL`或`DataSourceContextHolder.DATA_SOURCE_ORACLE`。<br/> 因为在`applicationContext.xml`中配置的默认数据库连接是MySQL，所以在连MySQL库的接口上可以不写`@DataSource(string value)`注解。同样，此注解也可以加在接口的方法上。


# 参考链接

[mybatis配置多数据源](https://www.cnblogs.com/digdeep/p/4512368.html)

[mybatis配置动态数据源](https://blog.csdn.net/u010648555/article/details/85109308)

[Spring aop:pointcut--expression--多个execution连接方法](https://blog.csdn.net/heatdeath/article/details/80243884)
