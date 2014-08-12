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
