# Mybatis 使用Ehcache缓存机制/自带缓存与Spring结合使用

参考文档：

官方cache教程网址：[http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html\#cache   ](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache)

## Ehcache简介

EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。

## 基本介绍

Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存,Java EE和轻量级容器。它具有内存和磁盘存储，缓存加载器,缓存扩展,缓存异常处理程序,一个gzip缓存servlet过滤器,支持REST和SOAP api等特点。

Ehcache最初是由Greg Luck于2003年开始开发。2009年,该项目被Terracotta购买。软件仍然是开源,但一些新的主要功能\(例如，快速可重启性之间的一致性的\)只能在商业产品中使用，例如Enterprise EHCache and BigMemory。维基媒体Foundationannounced目前使用的就是Ehcache技术。

下图是 Ehcache 在应用程序中的位置：

![](/assets/import-ehcache-01.png)

## 特性

主要的特性有：

1. 快速

2. 简单

3. 多种缓存策略

4. 缓存数据有两级：内存和磁盘，因此无需担心容量问题

5. 缓存数据会在[虚拟机](https://baike.baidu.com/item/虚拟机)重启的过程中写入磁盘

6. 可以通过RMI、可插入API等方式进行分布式缓存

7. 具有缓存和缓存管理器的侦听接口

8. 支持多[缓存](https://baike.baidu.com/item/缓存)管理器实例，以及一个实例的多个缓存区域

9. 提供Hibernate的缓存实现

## MyBatis中使用Ehcache POM依赖

```
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

## ecache配置文件，文件名必须为ehcache.xml {#ecache配置文件文件名必须为ehcachexml}

```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd" updateCheck="false">
   <diskStore path="java.io.tmpdir/Tmp_EhCache" /><!-- 缓存存放目录(此目录为放入系统默认缓存目录),也可以是”D:/cache“ java.io.tmpdir -->

   <defaultCache 
           eternal="false" 
           maxElementsInMemory="1000" 
           overflowToDisk="false" 
           diskPersistent="false"
        timeToIdleSeconds="0" 
        timeToLiveSeconds="1200" 
        memoryStoreEvictionPolicy="LRU" />

   <!-- 自定义缓存 -->
   <cache 
       name="iseCache" 
       eternal="false" 
       maxElementsInMemory="1000000" 
       overflowToDisk="true" 
       diskPersistent="false"
    timeToIdleSeconds="0" 
    timeToLiveSeconds="36000" 
    memoryStoreEvictionPolicy="LRU" />


    <!--
    name：Cache的唯一标识
    maxElementsInMemory：内存中最大缓存对象数
    maxElementsOnDisk：磁盘中最大缓存对象数，若是0表示无穷大
    eternal：Element是否永久有效，一但设置了，timeout将不起作用
    overflowToDisk：配置此属性，当内存中Element数量达到maxElementsInMemory时，Ehcache将会Element写到磁盘中
    timeToIdleSeconds：设置Element在失效前的允许闲置时间。仅当element不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大
    timeToLiveSeconds：设置Element在失效前允许存活时间。最大时间介于创建时间和失效时间之间。仅当element不是永久有效时使用，默认是0.，也就是element存活时间无穷大
    diskPersistent：是否缓存虚拟机重启期数据
    diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒
    diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区
    memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）
-->

</ehcache>
```

## 在mybatis-config.xml中&lt;settings&gt;&lt;/settings&gt;中设置开启缓存配置

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <settings>
        <!-- 全局映射器启用缓存 -->
        <!-- 外部缓存和二级缓存都需要 -->
        <setting name="cacheEnabled" value="true" />
        ...

    </settings>
</configuration>
```

## mybatis的mapper中启用缓存 {#在mybatis的mapper中启用}

_**有两种写法**_

```
<cache type="org.mybatis.caches.ehcache.LoggingEhcache"/>
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="AccountOfMapper">

    <sql id="aa"> account </sql>

    <!-- 自定义缓存(实现缓存接口或者第三方缓存插件，比如：ehcache)——>ehcache-->

    <!--
    ehcache的使用方式：
    以下两个<cache>标签都可以,第一个可以输出日志,第二个不输出日志
     <cache type="org.mybatis.caches.ehcache.LoggingEhcache"/>
     <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
    -->

    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <property name="timeToIdleSeconds" value="3600"/><!--1 hour-->
        <property name="timeToLiveSeconds" value="3600"/><!--1 hour-->
        <property name="maxEntriesLocalHeap" value="10000"/>
        <property name="maxEntriesLocalDisk" value="10000000"/>
    </cache>

    <!--

    可用的收回策略有:

    LRU – 最近最少使用的:移除最长时间不被使用的对象（默认的）。
    FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
    SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
    WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

    缓存默认配置

    <select ... flushCache="false" useCache="true"/>
    <insert ... flushCache="true"/>
    <update ... flushCache="true"/>
    <delete ... flushCache="true"/>


    -->

    <select id="getAccount" resultType="Account">
        select id, name, money from account where id = #{id}
    </select>


</mapper>
```

## 测试Controller

```
package com.master.controller;

import com.master.bean.Account;
import com.master.dao.AccountDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-7-28
 * Time: 22:41
 * info:
 */
@RestController
public class AccountController {

    @Autowired
    private AccountDao accountDao;

    @RequestMapping("/findAccount")
    public String findAccount(){
        return accountDao.findAccount(1).toString();
    }

    @RequestMapping("/getAccount")
    public String getAccount(){
        return accountDao.getAccount(1).toString();
    }

    @RequestMapping("/saveAccount")
    public String saveAccount(){
        Account account = new Account(4,"allen", 2000);
        return accountDao.save(account) + "";
    }
}
```

## 测试日志

```
JDBC Connection [HikariProxyConnection@139347354 wrapping com.mysql.jdbc.JDBC4Connection@14e7e982] will not be managed by Spring
==>  Preparing: select id, name, money from account where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, name, money
<==        Row: 1, aaa, 1000
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3273fac8]
```

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@30170780] was not registered for synchronization because synchronization is not active
Cache Hit Ratio [AccountOfMapper]: 0.5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@30170780]
```



