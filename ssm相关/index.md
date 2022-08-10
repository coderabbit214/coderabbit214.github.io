# SSM相关


# SSM相关

### 项目结构

- projectname

  - src

    - main

      - java

      - resources

        - mvc.xml

        - applicationContext.xml
        - db.properties
        - mybatis-config.xml
        - log4j.properties
        - shiro.xml
        - shiro-ehcache.xml
        - generatorConfig.xml

      - webapp

        - WEB_INF
          - views
          - web.xml
        - css
        - ...

    - test

## 项目搭建

### pom.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.wolfcode</groupId>
    <artifactId>wolfcar</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <spring.version>5.0.8.RELEASE</spring.version>
        <shiro.version>1.5.2</shiro.version>
    </properties>

    <packaging>war</packaging>
    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
<!--        aop-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.19</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.45</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.1</version>
        </dependency>
        <!--        分页-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.2</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
        </dependency>


        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.22</version>
            <scope>provided</scope>
        </dependency>

        <!-- 将freemarker等第三方库整合进Spring应用上下文的依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--        freemaker-->
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.30</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.6</version>
        </dependency>

        <!--shiro 核心-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <!--shiro 的 Web 模块-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-web</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <!--shiro 和 Spring 集成-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <!--shiro 底层使用的 ehcache 缓存-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <!--shiro 依赖的日志包-->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
        <!--shiro 依赖的工具包-->
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
        <!--Freemarker 的 shiro 标签库-->
        <dependency>
            <groupId>net.mingsoft</groupId>
            <artifactId>shiro-freemarker-tags</artifactId>
            <version>1.0.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.shiro</groupId>
                    <artifactId>shiro-all</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.51</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.1.2</version>
        </dependency>

        <!--fileupload-->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.6.5</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <!-- Tomcat 插件 -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>80</port> <!-- 应用的端口号 -->
                    <path>/</path> <!-- 应用的上下文路径 -->
                    <uriEncoding>UTF-8</uriEncoding> <!-- 针对 GET 方式中文乱码 -->
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!--mybatis的generator插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>false</overwrite>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.45</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

### web.xml配置

```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
         http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <multipart-config>
            <max-file-size>1024000</max-file-size>
            <max-request-size>1024000</max-request-size>
        </multipart-config>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--  shiro 配置  -->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### mvc.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 注解扫描 -->
    <context:component-scan base-package="cn.wolfcode.wolfcar.controller"/>
    <context:component-scan base-package="cn.wolfcode.wolfcar.advice"/>
    <context:component-scan base-package="cn.wolfcode.wolfcar.service"/>

    <!--   处理静态资源  使用默认的Servlet来响应静态文件  -->
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>

    <import resource="classpath:shiro.xml"/>
    <import resource="classpath:applicationContext.xml"/>

<!--  不使用commons-fileupload时  -->
<!--    <bean class="org.springframework.web.multipart.support.StandardServletMultipartResolver">-->
<!--    </bean>-->

    <!--文件上传解析器 id必须是multipartResolver-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--最大上传文件大小 10M-->
        <property name="maxUploadSize" value="#{1024*1024*10}"/>
    </bean>

    <!-- 注册 FreeMarker 配置类 -->
    <bean class="cn.wolfcode.wolfcar.config.MyFreemarkerConfiger">
        <!-- 配置 FreeMarker 的文件编码 -->
        <property name="defaultEncoding" value="UTF-8" />
        <!-- 配置 FreeMarker 寻找模板的路径 -->
        <property name="templateLoaderPath" value="/WEB-INF/views/" />
        <property name="freemarkerSettings">
            <props>
                <!-- 兼容模式 ，配了后不需要另外处理空值问题，时间格式除外 -->
                <prop key="classic_compatible">true</prop>
            </props>
        </property>
    </bean>

    <!-- 注册 FreeMarker 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
        <!-- 是否把session中的attribute复制到模板的属性集中，可以使用FreeMarker的表达式来访问并显示-->
        <property name="exposeSessionAttributes" value="true" />
        <!-- 配置逻辑视图自动添加的后缀名 -->
        <property name="suffix" value=".ftl" />
        <!-- 配置响应头中 Content-Type 的指 -->
        <property name="contentType" value="text/html;charset=UTF-8" />
    </bean>

</beans>
```

#### freemaker配置类

> config.MyFreemarkerConfiger

```java
package cn.wolfcode.wolfcar.config;

import com.jagregory.shiro.freemarker.ShiroTags;
import freemarker.template.Configuration;
import freemarker.template.TemplateException;
import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;

import java.io.IOException;

public class MyFreemarkerConfiger extends FreeMarkerConfigurer {
    @Override
    public void afterPropertiesSet() throws IOException, TemplateException {
        //继承之前的属性配置，这步不能省
        super.afterPropertiesSet();
      
        Configuration cfg = this.getConfiguration();
        cfg.setSharedVariable("shiro", new ShiroTags());//注册shiro 标签
    }
}

```



### applicationContext.xml配置

> 注意修改包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:property-placeholder location="classpath:db.properties"/>
<!--  数据源  -->
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

<!--  数据库链接  -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据源 -->
        <property name="dataSource" ref="druidDataSource"/>
        <!-- 配置mapper中实体类别名 -->
        <property name="typeAliasesPackage" value="cn.wolfcode.wolfcar.domain"/>
        <!-- mybatis配置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 将pageHelper配置到sqlsessionfactory -->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <!--使用下面的方式配置参数，一行配置一个，下面配的是合理化分页 -->
                        <value>
                            reasonable=true
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>

<!--  扫描mapper接口 注入spring  -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.wolfcode.wolfcar.mapper"/>
    </bean>

<!--  开启事务注解配置  -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="druidDataSource"/>
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

#### db.properties

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/wolfcar?characterEncoding=utf8
jdbc.username=root
jdbc.password=123456
```

#### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="cacheEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
		<setting name="lazyLoadTriggerMethods" value="clone"/>
	</settings>
</configuration>
```



### shirt.xml 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="cn.wolfcode.wolfcar.realm"></context:component-scan>

<!--  aop自动代理  -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor" />

<!--  管理shiro生命周期  -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"></bean>

<!--  安全管理器  -->
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>

    <!--指定当前需要使用的凭证匹配器-->
    <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
        <!-- 指定加密算法 -->
        <property name="hashAlgorithmName" value="MD5"/>
    </bean>


    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!--引用指定的安全管理器-->
        <property name="securityManager" ref="securityManager"/>
        <!--shiro默认的登录地址是/login.jsp 现在要指定我们自己的登录页面地址-->
        <property name="loginUrl" value="/login.html"/>
        <property name="successUrl" value="/employee/list"></property>
        <!--路径对应的规则-->
        <property name="filterChainDefinitions">
            <value>
                /index.htm=anon
                /employee/login=anon
                /css/**=anon
                /img/**=anon
                /js/**=anon
                /**=authc
            </value>
        </property>
    </bean>


    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!--注册自定义数据源-->
        <property name="realm" ref="myRealm"></property>
        <!--注册缓存管理器-->
        <property name="cacheManager" ref="cacheManager"/>
    </bean>


    <!-- 缓存管理器 -->
    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <!-- 设置配置文件 -->
        <property name="cacheManagerConfigFile" value="classpath:shiro-ehcache.xml"/>
    </bean>

</beans>
```

#### shiro 配置类（认证 授权）

```java
package cn.wolfcode.wolfcar.realm;

import cn.wolfcode.wolfcar.domain.Employee;
import cn.wolfcode.wolfcar.service.IEmployeeService;
import cn.wolfcode.wolfcar.service.IPermissionService;
import cn.wolfcode.wolfcar.service.IRoleService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authc.credential.CredentialsMatcher;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

@Component
public class MyRealm extends AuthorizingRealm{

    @Resource
    private IEmployeeService employeeService;

    @Resource
    private IRoleService roleService;

    @Resource
    private IPermissionService permissionService;

    @Autowired
    public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
        super.setCredentialsMatcher(credentialsMatcher);
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //获取当前的主体对象
        Employee employee = (Employee) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        if(employee.isAdmin()){
            info.addRole("admin");
            info.addStringPermission("*:*");
            return info;
        }
        //获取当前用户的角色
        List<String> rolesnlist = roleService.getRoleSnByEmployeeId(employee.getId());
        //获取当前用户的权限
        List<String> permissionList = permissionService.getPermissionExpressionByEmployeeId(employee.getId());
        //给当前用户添加角色和权限（授权）

        info.addRoles(rolesnlist);
        info.addStringPermissions(permissionList);

        return info;
    }


    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        Employee employee =  employeeService.selectByUsername(usernamePasswordToken.getUsername());
        if(employee == null){
            //用户不存在
            return null;
        }else{
            //第二个参数的密码，是数据库查询出结果的密码
            //第一个参数传递employee，当认证成功后，走到授权方法时，授权方法将得到employee对象做参数
          //第二个参数 密码 
          //第三个参数 md5加密 可以不加 
            AuthenticationInfo info = new SimpleAuthenticationInfo(employee ,employee.getPassword(),ByteSource.Util.bytes(employee.getName()) ,this.getName());
            return info;
        }

    }
}

```

#### shiro缓存配置 ： shiro-ehcache.xml

```xml
<ehcache>
    <defaultCache
            maxElementsInMemory="1000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>
</ehcache>
```

### log4j.properties

```properties
# Global logging configuration
log4j.rootLogger=ERROR, stdout

log4j.logger.cn.wolfcode.wolfcar.mapper=TRACE
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

```

### generatorConfig.xml mybatis逆向工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
		PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>

	<context id="mysql" defaultModelType="hierarchical"
			 targetRuntime="MyBatis3Simple">

		<!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表； 一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖 -->
		<property name="autoDelimitKeywords" value="false" />
		<!-- 生成的Java文件的编码 -->
		<property name="javaFileEncoding" value="UTF-8" />
		<!-- 格式化java代码 -->
		<property name="javaFormatter"
				  value="org.mybatis.generator.api.dom.DefaultJavaFormatter" />
		<!-- 格式化XML代码 -->
		<property name="xmlFormatter"
				  value="org.mybatis.generator.api.dom.DefaultXmlFormatter" />

		<!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
		<property name="beginningDelimiter" value="`" />
		<property name="endingDelimiter" value="`" />

		<commentGenerator>
			<property name="suppressDate" value="true" />
			<property name="suppressAllComments" value="true" />
		</commentGenerator>

		<!-- 必须要有的，使用这个配置链接数据库 @TODO:是否可以扩展 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
						connectionURL="jdbc:mysql:///wolfcar" userId="root" password="123456">
			<!-- 这里面可以设置property属性，每一个property属性都设置到配置的Driver上 -->
		</jdbcConnection>

		<!-- java类型处理器 用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl； 注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 
			NUMERIC数据类型； -->
		<javaTypeResolver
				type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
			<!-- true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型 false：默认, scale>0;length>18：使用BigDecimal; 
				scale=0;length[10,18]：使用Long； scale=0;length[5,9]：使用Integer； scale=0;length<5：使用Short； -->
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>


		<!-- java模型创建器，是必须要的元素 负责：1，key类（见context的defaultModelType）；2，java类；3，查询类 
			targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制； targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录 -->
		<javaModelGenerator targetPackage="cn.wolfcode.wolfcar.domain"
							targetProject="src/main/java">
			<!-- for MyBatis3/MyBatis3Simple 自动为每一个生成的类创建一个构造方法，构造方法包含了所有的field；而不是使用setter； -->
			<property name="constructorBased" value="false" />

			<!-- for MyBatis3 / MyBatis3Simple 是否创建一个不可变的类，如果为true， 那么MBG会创建一个没有setter方法的类，取而代之的是类似constructorBased的类 -->
			<property name="immutable" value="false" />

			<!-- 设置是否在getter方法中，对String类型字段调用trim()方法
			<property name="trimStrings" value="true" /> -->
		</javaModelGenerator>

		<!-- 生成SQL map的XML文件生成器， 注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
			或者只使用Mapper接口+Annotation，所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
			targetPackage/targetProject:同javaModelGenerator -->
		<sqlMapGenerator targetPackage="cn.wolfcode.wolfcar.mapper"
						 targetProject="src/main/resources">
			<!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
			<property name="enableSubPackages" value="true" />
		</sqlMapGenerator>


		<!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口 targetPackage/targetProject:同javaModelGenerator
			type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）： 1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
			2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中； 3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
			注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER -->
		<javaClientGenerator targetPackage="cn.wolfcode.wolfcar.mapper"
							 type="XMLMAPPER" targetProject="src/main/java">
			<!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
			<property name="enableSubPackages" value="true" />

			<!-- 可以为所有生成的接口添加一个父接口，但是MBG只负责生成，不负责检查 <property name="rootInterface"
				value=""/> -->
		</javaClientGenerator>

		<table tableName="consumption_item">
			<property name="useActualColumnNames" value="true"/>
			<property name="constructorBased" value="false" />
			<generatedKey column="id" sqlStatement="JDBC" />
		</table>
	</context>
</generatorConfiguration>
```





## 分解

### 一.FreeMarker

1. pom

   ```xml
   <!-- 将freemarker等第三方库整合进Spring应用上下文的依赖-->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context-support</artifactId>
       <version>${spring.version}</version> 
   </dependency>
   <!--        freemaker-->
   <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.30</version>
   </dependency>
   ```

2. mvc配置视图解析器

   ```xml
   <!-- 注册 FreeMarker 配置类 -->
   <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer"> 
       <!-- 配置 FreeMarker 的文件编码 -->
       <property name="defaultEncoding" value="UTF-8" />
       <!-- 配置 FreeMarker 寻找模板的路径 -->
       <property name="templateLoaderPath" value="/WEB-INF/views/" />
       <property name="freemarkerSettings">
               <props>
                   <!-- 兼容模式 ，配了后不需要另外处理空值问题，时间格式除外 -->
                   <prop key="classic_compatible">true</prop>
               </props>
           </property>
   </bean>
   
   <!-- 注册 FreeMarker 视图解析器 -->
   <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">  
       <!-- 是否把session中的attribute复制到模板的属性集中，可以使用FreeMarker的表达式来访问并显示-->   
       <property name="exposeSessionAttributes" value="true" /> 
       <!-- 配置逻辑视图自动添加的后缀名 --> 
       <property name="suffix" value=".ftl" />   
       <!-- 配置响应头中 Content-Type 的指 -->  
       <property name="contentType" value="text/html;charset=UTF-8" />
   </bean>
   ```

3. 添加配置 例如配置shiro标签

   - 配置类

     ```java
     package cn.wolfcode.wolfcar.config;
     
     import com.jagregory.shiro.freemarker.ShiroTags;
     import freemarker.template.Configuration;
     import freemarker.template.TemplateException;
     import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;
     
     import java.io.IOException;
     
     public class MyFreemarkerConfiger extends FreeMarkerConfigurer {
         @Override
         public void afterPropertiesSet() throws IOException, TemplateException {
             //继承之前的属性配置，这步不能省
             super.afterPropertiesSet();
           
             Configuration cfg = this.getConfiguration();
             cfg.setSharedVariable("shiro", new ShiroTags());//注册shiro 标签
         }
     }
     
     
     ```

   - mvc视图解析器修改

     ```xml
         <!-- 注册 FreeMarker 配置类 -->
         <bean class="cn.wolfcode.wolfcar.config.MyFreemarkerConfiger">
             <!-- 配置 FreeMarker 的文件编码 -->
             <property name="defaultEncoding" value="UTF-8" />
             <!-- 配置 FreeMarker 寻找模板的路径 -->
             <property name="templateLoaderPath" value="/WEB-INF/views/" />
             <property name="freemarkerSettings">
                 <props>
                     <!-- 兼容模式 ，配了后不需要另外处理空值问题，时间格式除外 -->
                     <prop key="classic_compatible">true</prop>
                 </props>
             </property>
         </bean>
     ```

4. 常用标签

   **判空：全扩起来加！**

   **例如：value="${(qo.startDate?string('yyyy-MM-dd'))!}"**

   > 指令：其实就是指 ftl 的标签，这些标签一般以符号#开头

   **include指令**

   在当前模板文件中引入另一个模板文件

   ```html
   <!--freemarker引入模板文件 之前已经配置了模板路径为/WEB-INF/views/ 这里就不需要写了 -->
   <#include "/common/link.ftl" >
   ```

   **assign指令**

   使用该指令可以在当前模板中创建一个新的变量， 或者替换一个已经存在的变量

   创建变量 currentMenu并赋值：

   ```html
   <#assign currentMenu="department"/>
   ```

   可使用${}获取该变量

   ```html
   ${currentMenu}
   ```

   **list指令**

   用于循环遍历序列  

   ```html
   <#list pageInfo.list as department>
       <tr>
           <td>${department_index+1}</td>
           <td>${department.name}</td>
           <td>${department.sn}</td>
       </tr>
   <#/list>               
   ```

   **注释**：FreeMarker的注释和 HTML 的注释相似，但是它用<#--和-->来分隔的。任何介于这两个分隔符（包含分隔符本身）之间内容会被 FreeMarker  忽略，不会显示到页面，一般用来注释有freemarker指令相关的代码。

   ```html
   <#--  <td>${department.name}</td> -->
   ```

   **if指令**

   用于条件判断

   ```html
   <#if condition>
     ...
   <#elseif condition2>
     ...
   <#elseif condition3>
     ...
   <#else>
     ...
   </#if>
   ```

    condition : 将被计算成布尔值的表达式

   `elseif` 和 `else` 是可选的

### 二.分页插件

##### 前端

```html
<div style="text-align: center;">
    <ul id="pagination" class="pagination"></ul>
</div>
<script>
    //分页
    $(function(){
        $("#pagination").twbsPagination({
            totalPages: ${result.pages} || 1,
                startPage: ${result.pageNum} || 1,
                visiblePages:5,
                first:"首页",
                prev:"上页",
                next:"下页",
                last:"尾页",
                initiateStartPageClick:false,
                onPageClick:function(event,page){
					$("#currentPage").val(page);
					$("#searchForm").submit();
				}
		});
    })
</script>
```

##### 后端

1. pom

   ```xml
           <dependency>
               <groupId>com.github.pagehelper</groupId>
               <artifactId>pagehelper</artifactId>
               <version>5.1.2</version>
           </dependency>
          
   ```

2. xml配置

   ```xml
   <!--  数据库链接  -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <!-- 将pageHelper配置到sqlsessionfactory -->
           <property name="plugins">
               <array>
                   <bean class="com.github.pagehelper.PageInterceptor">
                       <property name="properties">
                           <!--使用下面的方式配置参数，一行配置一个，下面配的是合理化分页 -->
                           <value>
                               reasonable=true
                           </value>
                       </property>
                   </bean>
               </array>
           </property>
       </bean>
   
   ```

3. 实现代码

   ```java
   package cn.wolfcode.wolfcar.query;
   
   import lombok.*;
   
   @Setter
   @Getter
   @NoArgsConstructor
   @AllArgsConstructor
   @ToString
   public class QueryObject {
       private int currentPage = 1;
       private int pageSize = 10;
   }
   
   ```
   
   ```java
   public PageInfo<Department> query(QueryObject qo) {
       //使用分页插件,传入当前页,每页显示数量
       PageHelper.startPage(qo.getCurrentPage(), qo.getPageSize());
       List<Department> departments = departmentMapper.selectForList(qo);
       return new PageInfo(departments);
   }
   ```

### 三.Bootstrap 模态框

1. 代码去https://v5.bootcss.com/

2. ```js
   $('#模态框的id').modal('show'); //官方文档中表示通过该方法即可弹出模态框
   //关闭模态框
   $("#模态框的id").modal("hide");
   ```

### 四.sweetalert2 弹出框插件

1. 引入插件

   ```html
   <link rel="stylesheet" href="/js/plugins/sweetalert2/sweetalert2.min.css">
   <script src="/js/plugins/sweetalert2/sweetalert2.min.js"></script>
   ```

2. 使用插件（如何配置看官方文档）

   ```js
   $(function(){
       Swal.fire({
           title: 'Are you sure?',
           text: "You won't be able to revert this!",
           icon: 'warning',
           showCancelButton: true,
           confirmButtonColor: '#3085d6',
           cancelButtonColor: '#d33',
           confirmButtonText: 'Yes, delete it!'
       }).then((result) => {
           if (result.value) { 
               //点击确认按钮后做的事情
               Swal.fire(
                   'Deleted!',
                   'Yrour file has been deleted.',
                   'success'
               )
           }
       })
   })
   ```

### 五.表单验证插件

1. 引入插件

   ```html
   <!--引入验证插件的样式文件-->
   <link rel="stylesheet" href="/js/plugins/bootstrap-validator/css/bootstrapValidator.min.css"/>
   <!--引入验证插件的js文件-->
   <script type="text/javascript" src="/js/plugins/bootstrap-validator/js/bootstrapValidator.min.js"></script>
   <!--中文语言库-->
   <script type="text/javascript" src="/js/plugins/bootstrap-validator/js/language/zh_CN.js"></script>
   ```

2. 使用插件

   ```js
   $("#editForm").bootstrapValidator({
       feedbackIcons: { //图标
           valid: 'glyphicon glyphicon-ok',
           invalid: 'glyphicon glyphicon-remove',
           validating: 'glyphicon glyphicon-refresh'
       },
       fields:{ //配置要验证的字段
           username:{
               validators:{ //验证的规则
                   notEmpty:{ //不能为空
                       message:"用户名必填" //错误时的提示信息
                   },
                   stringLength: { //字符串的长度范围
                       min: 1,
                       max: 5
                   }
               }
           },
           password:{
               validators:{
                   notEmpty:{ //不能为空
                       message:"密码必填" //错误时的提示信息
                   },
               }
           },
           repassword:{
               validators:{
                   notEmpty:{ //不能为空
                       message:"密码必填" //错误时的提示信息
                   },
                   identical: {//两个字段的值必须相同
                       field: 'password',
                       message: '两次输入的密码必须相同'
                   },
               }
           },
           email: {
               validators: {
                   emailAddress: {} //邮箱格式
               }
           },
           age:{
               validators: {
                   between: { //数字的范围
                       min: 18,
                       max: 60
                   }
               }
           }
       }
   }).on('success.form.bv', function() { //表单所有数据验证通过后执行里面的代码
   	//提交异步表单
       $("#editForm").ajaxSubmit(function() {
         
       })
   });
   ```

### 六.POI操作excel

> poi中关于excel的概念
>
> - Workbook（对应为一个excel）
>
> - Sheet（excel中的表）
>
> - Row（表中的行）
>
> - Column（表中的列）
>
> - Cell（表中的单元格，由行号和列号组成）

1. pom

   ```xml
     <dependency>
          <groupId>org.apache.poi</groupId>
          <artifactId>poi</artifactId>
          <version>4.1.2</version>
      </dependency>
   ```

2. controller

   ```java
   //导出
   @RequestMapping("/exportXls")
   public void exportXls(HttpServletResponse response) throws Exception {
       //文件下载的响应头（让浏览器访问资源的的时候以下载的方式打开）
       response.setHeader("Content-Disposition","attachment;filename=employee.xls");
       //创建excel文件
       Workbook wb = employeeService.exportXls();
       //把excel的数据输出给浏览器
       wb.write(response.getOutputStream());
   }
   //导入
   @RequestMapping("/importXls")
   @ResponseBody
   public JsonResult importXls(MultipartFile file) throws Exception {
       employeeService.importXls(file);
       return new JsonResult();
   }
   ```

3. service

   ```java
   //导出
   public Workbook exportXls() {
       //创建excel文件
       Workbook wb = new HSSFWorkbook();
       //创建sheet
       Sheet sheet = wb.createSheet("员工名单");
       //标题行
       Row row = sheet.createRow(0);
       //设置内容到单元格中
       row.createCell(0).setCellValue("姓名");
       row.createCell(1).setCellValue("邮箱");
       row.createCell(2).setCellValue("年龄");
       //查询员工数据
       List<Employee> employees = employeeMapper.selectAll();
       for (int i = 0; i < employees.size(); i++) {
           Employee employee = employees.get(i);
           //创建行(每个员工就是一行)
           row = sheet.createRow(i+1);
           //设置内容到单元格中
           row.createCell(0).setCellValue(employee.getName());
           row.createCell(1).setCellValue(employee.getEmail());
           row.createCell(2).setCellValue(employee.getAge());
       }
       return wb;
   }
   //导入
   public void importXls(MultipartFile file) throws Exception {
       //把接收到的文件以excel的方式去读取并操作
       Workbook wb = new HSSFWorkbook(file.getInputStream());
       //读取第一页
       Sheet sheet = wb.getSheetAt(0);
       //获取最后一行的索引
       int lastRowNum = sheet.getLastRowNum();
       //从索引为1的行数开始读(忽略标题行)
       for (int i = 1; i <= lastRowNum; i++) {
           //获取行数据
           Row row = sheet.getRow(i);
           String name = row.getCell(0).getStringCellValue();
           //判断如果用户名是空，就不再往下读
           if(!StringUtils.hasLength(name.trim())){
               return;
           }
           Employee employee = new Employee();
           employee.setName(name);
           employee.setEmail(row.getCell(1).getStringCellValue());
           if(cell.getCellType() == CellType.NUMERIC){
               //获取数值类型的单元格内容
               double cellValue = row.getCell(2).getNumericCellValue();
               employee.setAge((int)cellValue);
           }else{
               //获取文本格式的单元格内容
               String age = row.getCell(2).getStringCellValue();
               employee.setAge(Integer.valueOf(age));
           }
           //设置默认密码1
           employee.setPassword("1");
           //调用保存方法
           this.save(employee,null);
       }
   }
   ```



### 七.统一异常处理

> @ControllerAdvice
>
> @ExceptionHandler(Exception.class)
>
> 异常处理的类必须加到ioc容器中

```java
package cn.wolfcode.wolfcar.advice;


import cn.wolfcode.wolfcar.result.JSonResult;
import com.alibaba.fastjson.JSON;
import org.apache.shiro.authc.IncorrectCredentialsException;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authz.AuthorizationException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.method.HandlerMethod;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(AuthorizationException.class)
    public String func(AuthorizationException e, HttpServletResponse response, HandlerMethod handlerMethod) throws IOException {
        e.printStackTrace();;
        ResponseBody methodAnnotation = handlerMethod.getMethodAnnotation(ResponseBody.class);
        if (methodAnnotation == null) {
            return "common/nopermission";
        } else {
            response.setContentType("application/json;charset=utf-8");
            JSonResult jSonResult = JSonResult.error("没有权限");
            response.getWriter().write(JSON.toJSONString(jSonResult));
            return null;
        }
    }

    @ExceptionHandler(UnknownAccountException.class)
    public String unknownAccountExceptionHandler(AuthorizationException e, HttpServletResponse resp) throws IOException {
        e.printStackTrace();
        resp.setContentType("application/json;charset=utf-8");
        JSonResult jsonreslut = JSonResult.error("无此用户");
        resp.getWriter().write(JSON.toJSONString(jsonreslut));
        return null;
    }


    @ExceptionHandler(IncorrectCredentialsException.class)
    public String incorrectCredentialsExceptionHandler(AuthorizationException e, HttpServletResponse resp) throws IOException {
        e.printStackTrace();
        resp.setContentType("application/json;charset=utf-8");
        JSonResult jsonreslut = JSonResult.error("密码错误");
        resp.getWriter().write(JSON.toJSONString(jsonreslut));
        return null;
    }


    @ExceptionHandler(Exception.class)
    public String exceptionHandler(AuthorizationException e, HttpServletResponse resp, HandlerMethod handlerMethod) throws IOException {
        e.printStackTrace();
        ResponseBody ann = handlerMethod.getMethodAnnotation(ResponseBody.class);
        if(ann == null){
            return "common/error";
        }else{
            resp.setContentType("application/json;charset=utf-8");
            JSonResult jsonreslut = JSonResult.error("系统错误，请联系管理员");
            resp.getWriter().write(JSON.toJSONString(jsonreslut));
            return null;
        }
    }
}

```



### 八.shiro

### ![image-20200707152041577](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20200707152041577.png)



- **Subject（用户）**： 访问系统的用户，主体可以是用户、程序等，进行认证的都称为主体； Subject 一词是一个专业术语，其基本意思是“当前的操作用户”。 

  在程序任意位置可使用：Subject currentUser = SecurityUtils.getSubject() 获取到subject主体对象，类似 Employee user = UserContext.getUser() 

- **SecurityManager（安全管理器）**：它是 shiro 功能实现的核心，负责与后边介绍的其他组件(认证器/授权器/缓存控制器)进行交互，实现 subject 委托的各种功能。有点类似于SpringMVC 中的 DispatcherServlet 前端控制器，负责进行分发调度。

- **Realms（数据源）**： Realm 充当了 Shiro 与应用安全数据间的“桥梁”或者“连接器”。；可以把Realm 看成 DataSource，即安全数据源。执行认证（登录）和授权（访问控制）时，Shiro 会从应用配置的 Realm 中查找相关的比对数据。以确认用户是否合法，操作是否合理。

- Authenticator（认证器）： 用于认证，从 Realm 数据源取得数据之后进行执行认证流程处理。

- Authorizer（授权器）：用户访问控制授权，决定用户是否拥有执行指定操作的权限。

- SessionManager （会话管理器）：Shiro 与生俱来就支持会话管理，这在安全类框架中都是独一无二的功能。即便不存在 web 容器环境，shiro 都可以使用自己的会话管理机制，提供相同的会话 API。

- CacheManager （缓存管理器）：用于缓存认证授权信息等。

- Cryptography（加密组件）： Shiro 提供了一个用于加密解密的工具包。

#### 认证-授权

> 在RBAC基础上
>
> 用户（Employee）：角色施加的主体；用户通过拥有某个或多个角色以得到对应的权限。
>
> 角色（Role）：表示一组权限的集合。
>
> 权限（Permission）：一个资源代表一个权限，是否能访问该资源，就是看是否有该权限。
>
> ![role](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/role.png)

1. pom

   ```xml
   <properties>
   	<shiro.version>1.5.2</shiro.version>
   </properties>
   ```

   

   ```xml
   <!--shiro 核心-->
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-core</artifactId>
       <version>${shiro.version}</version>
   </dependency>
   <!--shiro 的 Web 模块-->
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-web</artifactId>
       <version>${shiro.version}</version>
   </dependency>
   <!--shiro 和 Spring 集成-->
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring</artifactId>
       <version>${shiro.version}</version>
   </dependency>
   <!--shiro 底层使用的 ehcache 缓存-->
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-ehcache</artifactId>
       <version>${shiro.version}</version>
   </dependency>
   <!--shiro 依赖的日志包-->
   <dependency>
       <groupId>commons-logging</groupId>
       <artifactId>commons-logging</artifactId>
       <version>1.2</version>
   </dependency>
   <!--shiro 依赖的工具包-->
   <dependency>
       <groupId>commons-collections</groupId>
       <artifactId>commons-collections</artifactId>
       <version>3.2.1</version>
   </dependency>
   <!--Freemarker 的 shiro 标签库-->
   <dependency>
       <groupId>net.mingsoft</groupId>
       <artifactId>shiro-freemarker-tags</artifactId>
       <version>1.0.1</version>
       <exclusions>
           <exclusion>
               <groupId>org.apache.shiro</groupId>
               <artifactId>shiro-all</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

2. 配置 代理过滤器

   > 因为真正的shiroFilter需要注入很多复杂的对象，而web.xml中只能配置字符串或数字的参数，是不能满足的，因此我们会把shiroFilter交给 Spring 进行管理，通过spring的xml文件来配置。 

   ```xml
   <filter>
       <filter-name>shiroFilter</filter-name>
       <filter-class>
           org.springframework.web.filter.DelegatingFilterProxy
       </filter-class>
   </filter>
   <filter-mapping>
       <filter-name>shiroFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

3. 编写单独的shiro配置 shiro.xml 并在 mvc.xml中引入

   **shiro.xml**
   
   >***anon:*** 匿名过滤器，即不需要登录即可访问；一般用于静态资源过滤；
   >
   >示例 /static/**=anon
   >
   >***authc:*** 表示需要认证(登录)才能使用;
   >
   >示例 /**=authc
   >
   >***roles:***角色授权过滤器，验证用户是否拥有资源角色；
   >
   >示例 /admin/\*=roles[admin]
   >
   >***perms:***权限授权过滤器，验证用户是否拥有资源权限；
   >
   >示例 /employee/input=perms["user:update"]
   >
   >***logout:***注销过滤器
   >
   >示例 /logout=logout
   
   ****
   
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   
   
       <context:component-scan base-package="cn.wolfcode.wolfcar.realm"></context:component-scan>
   
   <!--  aop自动代理  -->
       <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
             depends-on="lifecycleBeanPostProcessor" />
   
   <!--  管理shiro生命周期  -->
       <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"></bean>
   
   <!--  安全管理器  -->
       <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
           <property name="securityManager" ref="securityManager"/>
       </bean>
   
   
       <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
           <!--引用指定的安全管理器-->
           <property name="securityManager" ref="securityManager"/>
           <!--shiro默认的登录地址是/login.jsp 现在要指定我们自己的登录页面地址-->
           <property name="loginUrl" value="/login.html"/>
           <property name="successUrl" value="/employee/list"></property>
           <!--路径对应的规则-->
           <property name="filterChainDefinitions">
               <value>
                   /index.htm=anon
                   /employee/login=anon
                   /css/**=anon
                   /img/**=anon
                   /js/**=anon
                   /**=authc
               </value>
           </property>
       </bean>
   
   
       <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
           <!--注册自定义数据源-->
           <property name="realm" ref="myRealm"></property>
       </bean>
   </beans>
   ```
   
   **mvc.xml**
   
   ```xml
   <import resource="classpath:shiro.xml"/>
   ```
   
4. 编写shiro配置类MyRealm

   ```java
   package cn.wolfcode.wolfcar.realm;
   
   import cn.wolfcode.wolfcar.domain.Employee;
   import cn.wolfcode.wolfcar.service.IEmployeeService;
   import cn.wolfcode.wolfcar.service.IPermissionService;
   import cn.wolfcode.wolfcar.service.IRoleService;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.authc.credential.CredentialsMatcher;
   import org.apache.shiro.authz.AuthorizationInfo;
   import org.apache.shiro.authz.SimpleAuthorizationInfo;
   import org.apache.shiro.realm.AuthorizingRealm;
   import org.apache.shiro.subject.PrincipalCollection;
   import org.apache.shiro.util.ByteSource;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;
   
   import javax.annotation.Resource;
   import java.util.List;
   
   @Component
   public class MyRealm extends AuthorizingRealm{
   
       @Resource
       private IEmployeeService employeeService;
   
       @Resource
       private IRoleService roleService;
   
       @Resource
       private IPermissionService permissionService;
   
       @Autowired
       public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
           super.setCredentialsMatcher(credentialsMatcher);
       }
   
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
           //获取当前的主体对象
           Employee employee = (Employee) principals.getPrimaryPrincipal();
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           if(employee.isAdmin()){
               info.addRole("admin");
               info.addStringPermission("*:*");
               return info;
           }
           //获取当前用户的角色
           List<String> rolesnlist = roleService.getRoleSnByEmployeeId(employee.getId());
           //获取当前用户的权限
           List<String> permissionList = permissionService.getPermissionExpressionByEmployeeId(employee.getId());
           //给当前用户添加角色和权限（授权）
   
           info.addRoles(rolesnlist);
           info.addStringPermissions(permissionList);
   
           return info;
       }
   
   
       //认证
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
   
           UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
           Employee employee =  employeeService.selectByUsername(usernamePasswordToken.getUsername());
           if(employee == null){
               //用户不存在
               return null;
           }else{
               //第二个参数的密码，是数据库查询出结果的密码
               //第一个参数传递employee，当认证成功后，走到授权方法时，授权方法将得到employee对象做参数
             //第二个参数 密码 
             //第三个参数 md5加密 可以不加 
               AuthenticationInfo info = new SimpleAuthenticationInfo(employee ,employee.getPassword(),ByteSource.Util.bytes(employee.getName()) ,this.getName());
               return info;
           }
   
       }
   }
   ```

5. 登录功能

   ```java
       @RequestMapping("/login")
       @ResponseBody
       public JSonResult login(String username, String password) {
           UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);
           SecurityUtils.getSubject().login(usernamePasswordToken);
           return JSonResult.success(null);
       }
   ```

#### 登陆流程

> 登陆请求：/employee/login

- 设置登录页面

  ```xml
   <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
         。。。
          <!--shiro默认的登录地址是/login.jsp 现在要指定我们自己的登录页面地址-->
          <property name="loginUrl" value="/login.html"/>
          <property name="successUrl" value="/employee/list"></property>
         。。。
      </bean>
  ```

- 编写登陆请求

  > 这里走的统一异常处理 前端使用ajax
  >
  > UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);
  > SecurityUtils.getSubject().login(usernamePasswordToken);

  ```java
      @RequestMapping("/login")
      @ResponseBody
      public JSonResult login(String username, String password) {
          UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);
          SecurityUtils.getSubject().login(usernamePasswordToken);
          return JSonResult.success(null);
      }
  ```

- 去realm里验证

  ```java
   //认证
      @Override
      protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
  
          UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
          Employee employee =  employeeService.selectByUsername(usernamePasswordToken.getUsername());
          if(employee == null){
              //用户不存在
              return null;
          }else{
              //第二个参数的密码，是数据库查询出结果的密码
              //第一个参数传递employee，当认证成功后，走到授权方法时，授权方法将得到employee对象做参数
            //第二个参数 密码 
            //第三个参数 md5加密 可以不加 
              AuthenticationInfo info = new SimpleAuthenticationInfo(employee ,employee.getPassword(),ByteSource.Util.bytes(employee.getName()) ,this.getName());
              return info;
          }
      }
  }
  ```

#### 扫描controller上所有权限注解 添加到数据库

从 Spring 容器中获取到的 Controller 对象是代理对象，该对象的类继承自原始的Controller，而注解是贴在原始的 Controller 类方法上的，所以此时从代理对象中是获取不到注解的，所以需要获取到代理对象的父类（原始 Controller）的字节码对象，再获取方法。

```java
    @RequestMapping("/reload")
    @ResponseBody
    public JSonResult reload(){
        //获取数据库中所有的permission
        List<Permission> permissions = permissionService.selectAll();
        Map<String, String> permap = new HashMap<>();
        for (Permission permission: permissions) {
            permap.put(permission.getName(), permission.getExpression());
        }
        Map<String, Object> map = ctx.getBeansWithAnnotation(Controller.class);
        Collection<Object> controllers = map.values();
        for (Object controller : controllers) {
            Class<?> clzz = controller.getClass().getSuperclass();
            Method[] methods = clzz.getDeclaredMethods();
            for (Method method : methods) {
                RequiresPermissions annotation = method.getAnnotation(RequiresPermissions.class);
                if(annotation!=null){
                    String name = annotation.value()[1];
                    String expression = annotation.value()[0];
                    //数据库中没有这个权限
                    if(permap.get(name)==null){
                        permissionService.insert(new Permission(null, name, expression));
                    }
                }
            }
        }
        return JSonResult.success(null);
    }
```

#### 授权

> 在RBAC基础上

```java
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //获取当前的主体对象
        Employee employee = (Employee) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        if(employee.isAdmin()){
            info.addRole("admin");
            info.addStringPermission("*:*");
            return info;
        }
        //获取当前用户的角色
        List<String> rolesnlist = roleService.getRoleSnByEmployeeId(employee.getId());
        //获取当前用户的权限
        List<String> permissionList = permissionService.getPermissionExpressionByEmployeeId(employee.getId());
        //给当前用户添加角色和权限（授权）

        info.addRoles(rolesnlist);
        info.addStringPermissions(permissionList);

        return info;
    }
```

#### 加密

1. 添加shiro.xml配置

   ```xml
   <!--指定当前需要使用的凭证匹配器-->
   <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher"> 
       <!-- 指定加密算法 -->
       <property name="hashAlgorithmName" value="MD5"/> 
   </bean>
   ```

2. 加密

   ```java
       @Override
       public int insert(Employee record) {
           if (record.getPassword() != null) {
               Md5Hash md5Hash = new Md5Hash(record.getPassword(), record.getName());
               record.setPassword(md5Hash.toString());
           }
           return employeeMapper.insert(record);
       }
   ```

3. 判断

   > 在配置类 中 认证 

   ```java
    //第二个参数的密码，是数据库查询出结果的密码
               //第一个参数传递employee，当认证成功后，走到授权方法时，授权方法将得到employee对象做参数
             //第二个参数 密码 
             //第三个参数 md5加密 可以不加 
   AuthenticationInfo info = new SimpleAuthenticationInfo(employee ,employee.getPassword(),ByteSource.Util.bytes(employee.getName()) ,this.getName());
   ```

4. 数据库更改

   ```sql
   update employee set password = MD5(concat(name,password));
   ```

#### 缓存

1. 集成EhCache

   > shiro.xml

   ```
   <!--安全管理器-->
   <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
       <!--注册自定义数据源-->
       <property name="realm" ref="crmRealm"/>
       <!--注册缓存管理器-->
       <property name="cacheManager" ref="cacheManager"/>
   </bean>
   
   <!-- 缓存管理器 -->
   <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"> 
       <!-- 设置配置文件 -->
       <property name="cacheManagerConfigFile" value="classpath:shiro-ehcache.xml"/>
   </bean>
   ```

2. 添加缓存配置文件 

   shiro-ehcache.xml

   ```xml
   <ehcache>
       <defaultCache
               maxElementsInMemory="1000"
               eternal="false"
               timeToIdleSeconds="120"
               timeToLiveSeconds="120"
               memoryStoreEvictionPolicy="LRU">
       </defaultCache>
   </ehcache>
   ```


#### 使用

##### 注解

```java
@RequiresPermissions({"employee:list", "员工列表"},logical = Logical.OR)
```

##### 标签

**authenticated 标签**：已认证通过的用户。

```xml
<@shiro.authenticated> </@shiro.authenticated>
```

**notAuthenticated 标签**：未认证通过的用户。与 authenticated 标签相对。

```xml
<@shiro.notAuthenticated></@shiro.notAuthenticated>
```

**principal 标签**：输出当前用户信息，通常为登录帐号信息 

后台是直接将整个员工对象作为身份信息的，所以这里可以直接访问他的 name 属性得到员工的姓名

```xml
<@shiro.principal property="name" />
```

对应realm中返回的SimpleAuthenticationInfo对象的第一个参数

```java
new SimpleAuthenticationInfo(employee,employee.getPassword(),this.getName());
```

**hasRole 标签**：验证当前用户是否拥有该角色 

```xml
<@shiro.hasRole name="admin">Hello admin!</@shiro.hasRole>
```

**hasAnyRoles 标签**：验证当前用户是否拥有这些角色中的任何一个，角色之间逗号分隔 

```xml
<@shiro.hasAnyRoles name="admin,user,operator">Hello admin</@shiro.hasAnyRoles>
```

**hasPermission 标签**：验证当前用户是否拥有该权限 

```xml
<@shiro.hasPermission name="department:delete">删除</@shiro.hasPermission>
```



##### java获取用户信息

```java
//用户实体
Employee employee = (Employee) SecurityUtils.getSubject().getPrincipal();

System.out.println("当前用户是否有admin角色："+ SecurityUtils.getSubject().hasRole("admin"));
System.out.println("当前登录用户是否有employee:delete权限："+SecurityUtils.getSubject().isPermitted("employee:delete"));
```







### 九.文件上传下载commons-fileupload

##### 准备

1. 添加文件上传依赖

   ```xml
   <!--fileupload-->
   <dependency>
       <groupId>commons-fileupload</groupId>
       <artifactId>commons-fileupload</artifactId>
       <version>1.3.1</version>
   </dependency>
   ```

2. 添加文件上传解析器

   ```xml
    <!--文件上传解析器 id必须是multipartResolver-->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!--最大上传文件大小 10M-->
       <property name="maxUploadSize" value="#{1024*1024*10}"/>
   </bean>
   ```


##### 上传

```java
    @RequestMapping("/saveOrUpdate")
    public String saveOrUpdate(MultipartFile file, Business business, HttpServletRequest request) throws IOException, IllegalAccessException {
      //判断有无上传文件  
      if (file.getSize() != 0) {
        //删除旧文件
            if (business.getId() != null) {
                String license_img = business.getLicense_img();
              //获取当前项目路径
                String realPath = request.getServletContext().getRealPath("/");
                File oldFile = new File(realPath+license_img);
                oldFile.delete();
            }
            //处理图片名称
            String originalFilename = file.getOriginalFilename();
            String date = DateUtil.format(new Date(), "yyyyMMddHHmmss");
            String s1 = RandomUtil.randomNumbers(5);
            String[] split = originalFilename.split("\\.");
            String s = split[split.length-1];
            StringBuffer stringBuffer = new StringBuffer();
            stringBuffer.append(date);
            stringBuffer.append(s1);
            stringBuffer.append(".");
            stringBuffer.append(s);
            String filename = stringBuffer.toString();
            //处理地址
            String realPath = request.getServletContext().getRealPath("/img/");
            //建立文件
            File file1 = new File(realPath,filename);
            //文件传输
            FileOutputStream fileOutputStream = new FileOutputStream(file1);
            InputStream inputStream = file.getInputStream();
            byte[] bytes = new byte[100];
            int len = 0;
            while ((len = inputStream.read(bytes)) != -1) {
                fileOutputStream.write(bytes,0,len);
            }
            fileOutputStream.close();
            //存储
            business.setLicense_img("/img/"+filename);
        }
              //处理乱码问题
        CharaUtil.convert(business,Business.class);
        if (business.getId() == null) {
            BusinessService.insert(business);
        } else {
            BusinessService.updateByPrimaryKey(business);
        }
        return "redirect:/business/list";
}
```

##### 下载

```java
    @RequestMapping(value = "/download")
    public void download(HttpServletRequest request, HttpServletResponse response)throws IOException {
        //模拟文件，myfile.txt为需要下载的文件
        String filename = "aaa";
        String path = "D:\\file" + "\\" + filename;
        //获取输入流
        InputStream bis = new BufferedInputStream(new FileInputStream(newFile(path)));
        //转码，免得文件名中文乱码
        filename = URLEncoder.encode(filename, "UTF-8");//设置文件下载头
        response.addHeader("Content-Disposition", "attachment;filename=" + filename);
        //1.设置文件ContentType类型，这样设置，会自动判断下载文件类型
        response.setContentType("multipart/form-data");
        BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
        int len = 0;
        while ((len = bis.read()) != -1) {
            out.write(len);
            out.flush();
        }
        out.close();
    }
```



### 十.统一返回JSON

```java
@Setter
@Getter
public class JSonResult<T> {
    private int code;
    private String msg;
    private boolean success;
    private T data;

    public static <T> JSonResult success(T data) {
        JSonResult jSonResult = new JSonResult();
        jSonResult.setSuccess(true);
        jSonResult.setData(data);
        return jSonResult;
    }

    public static <T> JSonResult error(String msg){
        JSonResult jsonResult = new JSonResult();
        jsonResult.setSuccess(false);
        jsonResult.setMsg(msg);
        return jsonResult;
    }

}

```

### 十一.枚举解决状态问题

枚举类

```java
public enum AppointmentStatusEnum {

    PEND(0,"待确认"),
    PERFORM(1,"履行中"),
    CONSUME(2,"消费中"),
    FINISH(3,"归档"),
    FAILURE(4,"废弃");

    private String name;
    private Integer value;

    AppointmentStatusEnum(Integer value,String name) {
        this.value = value;
        this.name =name;
    }
    public static String getName(Integer value) {
        AppointmentStatusEnum[] valus = values();
        for (AppointmentStatusEnum appointmentStatusEnum : valus) {
            if (appointmentStatusEnum.value.equals(value)) {
                return appointmentStatusEnum.name;
            }
        }
        return null;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getValue() {
        return value;
    }

    public void setValue(Integer value) {
        this.value = value;
    }
}

```

使用

1. 实体类使用

   ```java
       public String getStatusName() {
           String name = AppointmentStatusEnum.getName(this.status);
           return name;
       }
   ```

2. 获取所有值

   ```java
           AppointmentStatusEnum[] values = AppointmentStatusEnum.values();
           model.addAttribute("statuses", values);
   ```



### 十二.json转换问题

#### 工具类(自写)反射

```java
package cn.wolfcode.wolfcar.util;

import cn.hutool.core.date.DateUtil;
import com.alibaba.fastjson.JSON;

import java.lang.reflect.Field;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JsonUtil {
    public static String Convert(Object obj,Class clzz) throws IllegalAccessException {
        Map<String,Object> map = new HashMap<>();
        Field[] fields = clzz.getDeclaredFields();
        for (Field field:fields) {
            field.setAccessible(true);
            Object o = field.get(obj);
            map.put(field.getName(),o);
        }
        return JSON.toJSONString(map);
    }

    public static String Convert(Object obj, Class clzz, String format) throws IllegalAccessException {
        Map<String, Object> map = new HashMap<>();
        Field[] fields = clzz.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            if(field.getType().equals(Date.class)){
                String sdate = DateUtil.format((Date) field.get(obj),  format);
                map.put(field.getName(), sdate);
            }
            else
                map.put(field.getName(), field.get(obj));
        }
        return JSON.toJSONString(map);
    }
}

```

实体类使用

```java
package cn.wolfcode.wolfcar.domain;

import cn.wolfcode.wolfcar.myenum.AppointmentStatusEnum;
import cn.wolfcode.wolfcar.util.JsonUtil;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.format.annotation.DateTimeFormat;

import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Appointment {
    private Long id;

    private String ano;

    private Integer status;

    //    private Long category_id;
    private SystemDictionaryItem category;

    private String info;

    private String contact_tel;

    private String contact_name;

    //    private Long business_id;
    private Business business;

    private Date create_time;

    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
    private Date appointment_time;

    public String getStatusName() {
        String name = AppointmentStatusEnum.getName(this.status);
        return name;
    }
    public String getJson() throws IllegalAccessException {
        return JsonUtil.Convert(this,Appointment.class,"yyyy-MM-dd HH:mm");
    }
}
```

前端使用

```html
<a href="javascript:void(0);" data-json='${tmp.json}' class="btn btn-info btn-xs btn-input btnedit" >
  <span class="glyphicon glyphicon-pencil"></span> 编辑
</a>


    $(".btnedit").click(function (){
        var json = $(this).data("json");
        $("#business").val(json.business.id);
        $("input[name=appointment_time]").val(json.appointment_time);
        $("#category").val(json.category.id);
        $("input[id=contact_name]").val(json.contact_name);
        $("input[id=contact_tel]").val(json.contact_tel);
        $("textarea[name=info]").html(json.info);
        $("input[name=id]").val(json.id);
        $("#editModal").modal("show");
    });
```


