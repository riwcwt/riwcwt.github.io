---
layout: article
title: Tomcat+redis+nginx配置
---

为客户开发的一个绩效系统，采用了java web的开发方式，使用了一些spring mvc, mybatis之类的框架。相比于oracle ebs的二次开发，这种开发更加灵活，虽然和ebs集成的时候遇到一些问题，但是最后也都解决了。

在部署的时候，客户要求要能同事承受一两千人在线，相对于客户公司的总人数（七八万人），应该足够了。ebs的二次都是直接部署在oracle ebs的application server上面，之前也没怎么关注过程序的部署。这次采用tomcat部署，考虑到单个tomcat的最大也就能承受500左右的在线人数，这次采用了一个小的集群部署，使用了5个tomcat，反向代理使用的nginx。

现在程序基本稳定，压力测试也都能没什么大的问题，趁着有时间，把部署和配置都整理一下。

### 准备 ###

apache tomcat 7.0.55

nginx 1.7.2

redis 2.8.9

配置环境使用三个tomcat, 三台tomcat、redis和nginx都在一台机器上，为了方便测试和部署。

大致的整个配置的架构：

![tomcat-nginx-redis](/img/tomcat-redis-nginx.png)

在这个图中，nginx做为反向代理，将客户请求根据权重随机分配给三台tomcat服务器，redis做为三台tomcat的共享session数据服务器。

### 规划 ###

**redis**
	
	localhost:6379

**nginx**

	localhost:80

**tomcat**

	localhost:8081
	localhost:8082
	localhost:8083

### 配置 ###

**tomcat**

修改tomcat文件夹中conf/context.xml文件，在context节点下添加如下配置：

	<Valve  className="com.radiadesign.catalina.session.RedisSessionHandlerValve" />
	<Manager className="com.radiadesign.catalina.session.RedisSessionManager"
         host="localhost" 
         port="6379"
         database="0" 
         maxInactiveInterval="60" />

conf/server.xml文件中的端口根据规划依次修改

**nginx**

修改nginx文件目中的conf/nginx.conf文件为：
	
	#user  nobody;
	worker_processes  1;

	error_log  logs/error.log;

	pid        logs/nginx.pid;

	events {
    	worker_connections  1024;
	}


	http {
    	include       mime.types;
    	default_type  application/octet-stream;

    	log_format  main  '$remote_addr - $remote_user [$time_local] 	"$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    	access_log  logs/access.log  main;

    	sendfile        on;
    	#tcp_nopush     on;

    	#keepalive_timeout  0;
    	keepalive_timeout  65;

    	#gzip  on;

		upstream  localhost   {  
              server   localhost:8081 weight=1;  
              server   localhost:8082 weight=2;  
			  server   localhost:8083 weight=3; 
    	}  
	
    	server {
        	listen       80;
        	server_name  localhost;

        	#charset koi8-r;

        	#access_log  logs/host.access.log  main;

        	location / {
            	root   html;
            	index  index.html index.htm;
				proxy_pass        http://localhost;  
           	 	proxy_set_header  X-Real-IP  $remote_addr;  
            	client_max_body_size  100m;  
        	}

        	#error_page  404              /404.html;

        	# redirect server error pages to the static page /50x.html
        	#
        	error_page   500 502 503 504  /50x.html;
        	location = /50x.html {
            	root   html;
        	}

    	}
 
 	}

redis的配置就直接使用默认配置，因为只是测试用，和tomcat一样没有做参数优化配置。

### 运行 ###

分别启动redis、nginx和三台tomcat。

![redis](/img/redis-cluster.jpg)

![nginx](/img/nginx-cluster.jpg)

![tomcat](/img/tomcat-cluster.jpg)

### 测试 ###

在三个tomcat的webapps/ROOT目录下，分别添加session.jsp

    <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
	<!DOCTYPE html>
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	<title>shared session</title>
	</head>
	<body>
		<br>session id=<%=session.getId()%>
		<br>tomcat 3
	</body>
	</html>

> 注:每个tomcat下的标示不同

![tomcat](/img/tomcat-1.jpg)

![tomcat](/img/tomcat-2.jpg)

![tomcat](/img/tomcat-3.jpg)

