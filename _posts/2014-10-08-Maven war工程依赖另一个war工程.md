---
layout: article
title: Maven war工程依赖另一个war工程
---

项目的开发中，总是能遇到各种无奈的需求，最近在做的系统中，就遇到一个war工程依赖另一个war工程的需求。

这里假设A工程依赖B工程，A和B均为使用maven开发的web工程。

B工程的pom.xml文件大致如下：

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>com.*****.ehr</groupId>
		<artifactId>ehr-performance</artifactId>
		<packaging>war</packaging>
		<version>1.0-SNAPSHOT</version>
		<name>ehr-performance Maven Webapp</name>
		<url>http://maven.apache.org</url>
	
		<dependencies>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.11</version>
				<scope>test</scope>
			</dependency>
		</dependencies>
	
		<build>
			<finalName>ehr-performance</finalName>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.4</version>
					<configuration>
						<attachClasses>true</attachClasses><!-- 把class打包jar作为附件 -->
					</configuration>
				</plugin>
			</plugins>
		</build>
	
	</project>

A工程的pom.xml文件，依赖B工程，如下：

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>com.*****.ehr</groupId>
		<artifactId>ehr-simple-portal</artifactId>
		<packaging>war</packaging>
		<version>1.0-SNAPSHOT</version>
		<name>ehr-simple-portal Maven Webapp</name>
		<url>http://maven.apache.org</url>
	
		<dependencies>
			<dependency>
				<groupId>com.*****.ehr</groupId>
				<artifactId>ehr-performance</artifactId>
				<version>1.0-SNAPSHOT</version>
				<type>war</type>
			</dependency>
			<dependency>
				<groupId>com.*****.ehr</groupId>
				<artifactId>ehr-performance</artifactId>
				<version>1.0-SNAPSHOT</version>
				<type>jar</type>
				<classifier>classes</classifier>
				<scope>provided</scope>
			</dependency>
		</dependencies>
	
		<build>
			<finalName>ehr-simple-portal</finalName>
		</build>
	
	</project>

上述配置都是根据网上一些文档，根据自己项目中实践过的，这里仅做记录。
