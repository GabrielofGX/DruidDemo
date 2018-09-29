# DruidDemo
记录自己尝试使用Druid进行数据库监控及对数据库密码进行加密

#### 一、简介
Druid是阿里巴巴开源平台上一个开源的数据库连接池实现，它结合了C3P0、DBCP、PROXOOL等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池(据说是目前最好的连接池,不知道速度有没有BoneCP快)。

[Druid官方项目地址及文档](https://github.com/alibaba/druid)
[Druid与SpringBoot集成官方项目](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
### 二、入门示例(IDEA SpringBoot项目)
1、新建一个SpringBoot工程; 

2、pom.xml中添加依赖，如下：
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>1.5.2.RELEASE</version>
        </dependency>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-jdbc</artifactId>
	</dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
        </dependency>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-test</artifactId>
	    <scope>test</scope>
	</dependency>
    </dependencies>
```
parent依赖为spring-boot-starter-parent，  
其中spring-boot-starter-web表示为一个web项目；    
spring-boot-starter-jdbc表示使用jdbc访问数据库；  
druid-spring-boot-starter为Druid与SpringBoot集成的依赖；  
mysql-connector-java是MySQL的java版驱动包；  
spring-boot-starter-test是SpringBoot支持的test包。  

3、DataSource配置及监控配置
新建一个java类，内容如下：
```java
package com.gabriel.druid.config;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import javax.sql.DataSource;

/**
 * @author Gabriel
 * @email 996492571@qq.com
 * @time 2018/6/20/020 18:18
 * @description Druid数据源配置类及监控配置
 **/
@Configuration
@PropertySource(value = "classpath:application.properties")
public class DruidConfig {

    @Bean(destroyMethod = "close", initMethod = "init")
    @ConfigurationProperties(prefix = "spring.datasource.")
    public DataSource druidDataSource(){
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * 注册一个StatViewServlet
     * @return
     */
    @Bean
    public ServletRegistrationBean druidStatViewServlet(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
        servletRegistrationBean.setServlet(new StatViewServlet());
        servletRegistrationBean.addUrlMappings("/druid/*");
        //添加初始化参数：initParams
        //白名单：
        servletRegistrationBean.addInitParameter("allow","127.0.0.1");
        //IP黑名单 (存在共同时，deny优先于allow) : 如果满足deny的话提示:Sorry, you are not permitted to view this page.
        servletRegistrationBean.addInitParameter("deny","192.168.1.73");
        //登录查看信息的账号密码.
        servletRegistrationBean.addInitParameter("loginUsername","admin");
        servletRegistrationBean.addInitParameter("loginPassword","123456");
        //是否能够重置数据.
        servletRegistrationBean.addInitParameter("resetEnable","false");
        return servletRegistrationBean;
    }

    /**
     * 注册一个：filterRegistrationBean
     * @return
     */
    @Bean
    public FilterRegistrationBean druidStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new WebStatFilter());
        //添加过滤规则.
        filterRegistrationBean.addUrlPatterns("/*");
        //添加不需要忽略的格式信息.
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        return filterRegistrationBean;
    }
}
```

4、属性配置文件
在src/main/resources目录下application.properties添加内容如下：
```
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/gabriel?useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=true
spring.datasource.username=root
spring.datasource.password=123456

spring.datasource.druid.initialSize=5
spring.datasource.druid.minIdle=1
spring.datasource.druid.maxActive=50
spring.datasource.druid.max-wait=600000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.druid.minEvictableIdleTimeMillis=300000
spring.datasource.druid.validationQuery=SELECT 1 FROM DUAL
spring.datasource.druid.testWhileIdle=true
spring.datasource.druid.testOnBorrow=false
spring.datasource.druid.testOnReturn=false
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.druid.poolPreparedStatements=false
spring.datasource.druid.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.druid.filters=stat,wall,log4j
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.druid.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
# 合并多个DruidDataSource的监控数据
#spring.datasource.druid.useGlobalDataSourceStat=true
```

5、程序启动  
启动MySQL服务；  
启动SpringBoot程序；  
浏览器访问：http://localhost:8080/druid/ 即可看到Druid的监控页面了  
![image.png](https://upload-images.jianshu.io/upload_images/6880015-e1b4f9058c307ae8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
登录账号和密码为第3步代码配置的loginUsername和loginPassword

### 三、遇到的问题
1、提示错误，参数类型不符  
![image.png](https://upload-images.jianshu.io/upload_images/6880015-b30807b3acfd7c32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

![image.png](https://upload-images.jianshu.io/upload_images/6880015-5accac04faf49df8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
原因：pom.xml中未添加spring-boot-starter-web依赖
查看StatViewServlet继承关系图，如下：    
![image.png](https://upload-images.jianshu.io/upload_images/6880015-7f0e49b3ccc44167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)    
缺少web依赖，就会缺少tomcat-embed，缺少HttpServlet，所以报类型不匹配。  

2、空指针异常    
![image.png](https://upload-images.jianshu.io/upload_images/6880015-84a23633c30760c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)    
如图1~4行，3和4为创建DruidDataSource对象，任选其一均可；1和2行不是必须的，可以不要；  
2的存在与否不影响数据库配置，但是1和3不能共存，否则报空指针异常  
```java
Caused by: java.lang.NullPointerException: null
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:311) ~[na:1.8.0_131]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_131]
	at com.alibaba.druid.util.JdbcUtils.createDriver(JdbcUtils.java:589) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.pool.DruidDataSource.init(DruidDataSource.java:817) ~[druid-1.1.10.jar:1.1.10]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_131]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_131]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_131]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_131]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeCustomInitMethod(AbstractAutowireCapableBeanFactory.java:1758) ~[spring-beans-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1695) ~[spring-beans-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1624) ~[spring-beans-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	... 16 common frames omitted
```
3、数据库密码加密
  ![image.png](https://upload-images.jianshu.io/upload_images/6880015-78813a79f32be501.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
密码加密的步骤如下：
3.1、生成加密后的密码
      cmd进入druid Jar包所在的目录，输入如下图的命令(前提是配置了java的环境变量)即可；将密码和publicKey添加到application.properties配置文件中
![image.png](https://upload-images.jianshu.io/upload_images/6880015-0d6adbdce21dfd3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就是我在尝试Druid配置监控及密码加密的整个过程，下面就是成功查看监控的页面。项目完整代码参见[项目完整代码工程](https://github.com/GabrielofGX/DruidDemo)
![image.png](https://upload-images.jianshu.io/upload_images/6880015-c9e4bc7beeea1eab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


