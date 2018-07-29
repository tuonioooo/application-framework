# Mybatis 使用Ehcache缓存机制 

参考文档：

官方cache教程网址：[http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html\#cache   ](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache) 

## Ehcache简介

EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。

## 基本介绍

Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存,Java EE和轻量级容器。它具有内存和磁盘存储，缓存加载器,缓存扩展,缓存异常处理程序,一个gzip缓存servlet过滤器,支持REST和SOAP api等特点。

Ehcache最初是由Greg Luck于2003年开始开发。2009年,该项目被Terracotta购买。软件仍然是开源,但一些新的主要功能\(例如，快速可重启性之间的一致性的\)只能在商业产品中使用，例如Enterprise EHCache and BigMemory。维基媒体Foundationannounced目前使用的就是Ehcache技术。

下图是 Ehcache 在应用程序中的位置：









