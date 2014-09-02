在最近给客户做的两个系统之间，有一些需要相互调用的地方，因为两个系统的架构基本相同，都是采用的spring mvc + mybatis,所以尝试使用了一下spring httpinvoker。目前系统访问平稳，用户量也不是很大，没有出现什么问题。

现在常见的远程调用，主要有RMI、 spring httpinvoker、 hessian、 burlap、 web services、 protobuf等。各种方式都有自己的优缺点，网上有很多相关的对比，这里就不再赘述。但是对于两个都是java开发的系统而言，spring httpinvoker应该是综合了传输效率和开发效率的比较好的方式。

### spring invoker调用流程图 ###

![spring httpinvoker interaction](/img/spring-httpinvoker-interaction.gif)

### spring httpinvoker示例 ###

使用了maven开发，三个工程：

hello-remoting-interface

hello-remoting-server

hello-remoting-client

#### hello-remoting-interface ####

定义接口和使用到的实体

![hello remotin interface project](/img/hello-remotin-interface-project.jpg)

pom.xml文件

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>com.riwcwt</groupId>
		<artifactId>hello-remoting-interface</artifactId>
		<packaging>jar</packaging>
		<version>1.0-SNAPSHOT</version>
		<name>hello-httpinvoker-interface</name>
		<url>http://maven.apache.org</url>
	
		<properties>
			<spring.version>3.2.10.RELEASE</spring.version>
		</properties>
	
		<dependencies>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-core</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
				<version>1.7.7</version>
			</dependency>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.11</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-test</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.httpcomponents</groupId>
				<artifactId>httpclient</artifactId>
				<version>4.3.5</version>
			</dependency>
		</dependencies>
	</project>

UserService.java

	package com.riwcwt.remoting.service;
	
	import java.util.List;
	
	import com.riwcwt.remoting.entity.Role;
	import com.riwcwt.remoting.entity.User;
	
	public interface UserService
	{
		public List<User> getAllUsers();
	
		public List<Role> getRolesByUser(User user);
	}

#### hello-remoting-server ####

服务提供方

![hello remoting server project](/img/hello-remoting-server-project.jpg)

web.xml

	<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd" >
	
	<web-app>
		<display-name>Archetype Created Web Application</display-name>
	
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring/*-context.xml</param-value>
		</context-param>
	
		<!-- 防止发生java.beans.Introspector内存泄露,应将它配置在ContextLoaderListener的前面 -->
		<listener>
			<listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
		</listener>
		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener </listener-class>
		</listener>
	
		<!-- 设置WEB应用字符集，也是通过过滤器完成的 -->
		<filter>
			<filter-name>CharacterEncodingFilter</filter-name>
			<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
			<init-param>
				<param-name>encoding</param-name>
				<param-value>UTF-8</param-value>
			</init-param>
			<init-param>
				<param-name>forceEncoding</param-name>
				<param-value>true</param-value>
			</init-param>
		</filter>
		<filter-mapping>
			<filter-name>CharacterEncodingFilter</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
	
		<!-- Spring MVC 的Servlet，它将加载WEB-INF/mvc-dispatcher.xml 的 配置文件， 以启动Spring 
			MVC模块 -->
		<servlet>
			<servlet-name>mvc</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>classpath*:spring/mvc-dispatcher.xml</param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>
		<servlet-mapping>
			<servlet-name>mvc</servlet-name>
			<url-pattern>/</url-pattern>
		</servlet-mapping>
	
		<welcome-file-list>
			<welcome-file>/index.jsp</welcome-file>
		</welcome-file-list>
	
	</web-app>


UserServiceImpl.java

	package com.riwcwt.remoting.service;
	
	import java.util.LinkedList;
	import java.util.List;
	
	import org.apache.log4j.Logger;
	import org.springframework.stereotype.Service;
	
	import com.riwcwt.remoting.entity.Role;
	import com.riwcwt.remoting.entity.User;
	
	@Service(value = "userService")
	public class UserServiceImpl implements UserService
	{
	
		private static Logger logger = Logger.getLogger(UserServiceImpl.class);
	
		@Override
		public List<User> getAllUsers()
		{
			logger.info("invoke the first one");
			List<User> users = new LinkedList<User>();
			for (int i = 0; i < 2; i++)
			{
				User user = new User();
				user.setUsername("username " + i);
				user.setDescription("description " + i);
	
				List<Role> roles = new LinkedList<Role>();
	
				Role admin = new Role();
				admin.setRoleName("amin");
				Role manager = new Role();
				manager.setRoleName("manager");
	
				roles.add(admin);
				roles.add(manager);
	
				user.setRoles(roles);
	
				users.add(user);
			}
			return users;
		}
	
		@Override
		public List<Role> getRolesByUser(User user)
		{
			logger.info("invoke the second one");
			List<Role> roles = new LinkedList<Role>();
			Role admin = new Role();
			admin.setRoleName("amin");
			Role manager = new Role();
			manager.setRoleName("manager");
			roles.add(admin);
			roles.add(manager);
			return roles;
		}
	
	}

mvc-dispatcher.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
		xmlns:lang="http://www.springframework.org/schema/lang" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:p="http://www.springframework.org/schema/p"
		xsi:schemaLocation="
			 http://www.springframework.org/schema/beans
	         http://www.springframework.org/schema/beans/spring-beans.xsd
	         http://www.springframework.org/schema/aop
	         http://www.springframework.org/schema/aop/spring-aop.xsd
	         http://www.springframework.org/schema/lang
	         http://www.springframework.org/schema/lang/spring-lang.xsd
	         http://www.springframework.org/schema/context 
	         http://www.springframework.org/schema/context/spring-context.xsd
	         http://www.springframework.org/schema/mvc   
	         http://www.springframework.org/schema/mvc/spring-mvc.xsd">
		
		<!-- 设置使用注解的类所在的jar包 -->
		<context:component-scan base-package="com.riwcwt.remoting.service"></context:component-scan>

		<bean name="/user-service"
			class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
			<property name="service" ref="userService" />
			<property name="serviceInterface" value="com.riwcwt.remoting.service.UserService" />
		</bean>
	
	</beans>

#### hello-remoting-client ####

客户端调用服务

application-context.xml

	<?xml version="1.0" encoding="UTF-8" standalone="no"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:jee="http://www.springframework.org/schema/jee"
		xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jpa="http://www.springframework.org/schema/data/jpa"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop.xsd      
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd    
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd  
		http://www.springframework.org/schema/jee
		http://www.springframework.org/schema/jee/spring-jee.xsd       
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx.xsd   
		http://www.springframework.org/schema/jdbc
		http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
	
		<!-- 导入属性配置文件 -->
		<context:property-placeholder location="classpath*:remoting.properties" />
	
		<bean id="userService"
			class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
			<property name="serviceUrl"
				value="http://${remoting.host}:${remoting.port}/${remoting.context}/user-service">
			</property>
			<property name="serviceInterface" value="com.riwcwt.remoting.service.UserService">
			</property>
			<!-- 修改默认的HTTP请求链接方式，提升连接效率 -->
			<property name="httpInvokerRequestExecutor">
				<bean
					class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor">
				</bean>
			</property>
		</bean>
	
	</beans>

测试用例代码

	package com.riwcwt.remoting.service;
	
	import java.util.List;
	
	import javax.annotation.Resource;
	
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	
	import com.riwcwt.remoting.entity.Role;
	import com.riwcwt.remoting.entity.User;
	
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration("/spring/application-context.xml")
	public class UserServiceTest
	{
		@Resource(name = "userService")
		private UserService userService = null;
	
		@Test
		public void testGetAllUsersByHttpInvoker() throws InterruptedException
		{
			List<User> users = this.userService.getAllUsers();
			for (User user : users)
			{
				System.out.println(user.getUsername() + " " + user.getDescription());
				for (Role role : user.getRoles())
				{
					System.out.println(role.getRoleName());
				}
			}
	
		}
	
		@Test
		public void anotherTest()
		{
			for (int i = 0; i < 5; i++)
			{
				User user = new User();
				user.setUsername("test");
				List<Role> roles = this.userService.getRolesByUser(user);
				for (Role role : roles)
				{
					System.out.println(role.getRoleName());
				}
			}
		}
	}

server和client工程都依赖于interface工程，这样就方便于开发。通过上面的例子，可以看出spring httpinvoker是很方便于做远程调用的，其实spring对于rmi、hessian、burlap也提供了几乎和上面配置一样的调用，只是serviceexpoter和factoorybean不同，这里就不再一一做示例。
