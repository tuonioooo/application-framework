# Actuator系统监控

Actuator可能大家非常熟悉，它是springboot提供对应用自身监控，以及对应用系统配置查看等功能。

springboot使用actuator的方式非常简单，只需要在项目中加入依赖spring-boot-starter-actuator，完整pom文件如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dalaoyang</groupId>
    <artifactId>springboot_actuator</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot_actuator</name>
    <description>springboot_actuator</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

其余没有任何修改，没有新建任何类，在配置文件中加入了几条属性，配置文件如下：

```
##端口号
server.port=8888


##项目信息
info.name=dalaoyang
info.server.port=${server.port}
```

然后启动项目可以看到：

![](/assets/import-actuator-01.png)

介绍一下红框内的Actuator暴露的功能：

| HTTP方法 | 路径 | 描述 | 鉴权 |
| :--- | :--- | :--- | :--- |
| GET | /autoconfig | 查看自动配置的使用情况 | true |
| GET | /configprops | 查看配置属性，包括默认配置 | true |
| GET | /beans | 查看bean及其关系列表 | true |
| GET | /dump | 打印线程栈 | true |
| GET | /env | 查看所有环境变量 | true |
| GET | /env/{name} | 查看具体变量值 | true |
| GET | /health | 查看应用健康指标 | false |
| GET | /info | 查看应用信息 | false |
| GET | /mappings | 查看所有url映射 | true |
| GET | /metrics | 查看应用基本指标 | true |
| GET | /metrics/{name} | 查看具体指标 | true |
| POST | /shutdown | 关闭应用 | true |
| GET | /trace | 查看基本追踪信息 | true |

通过上面表格，我们可以在浏览器上访问 [http://localhost:8888/health](https://link.jianshu.com/?t=http%3A%2F%2Flocalhost%3A8888%2Fhealth)  可以看到如下图:

![](/assets/import-actuator-02.png)

访问  [http://localhost:8888/info](https://link.jianshu.com/?t=http%3A%2F%2Flocalhost%3A8888%2Finfo)，可以看到

![](/assets/import-actuator-03.png)





