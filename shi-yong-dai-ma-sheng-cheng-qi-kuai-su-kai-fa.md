# 使用代码生成器快速开发

### 生成器插件及使用说明：

* 官方生成器插件： [https://github.com/tuonioooo/mybatis-maven-plugin](https://github.com/tuonioooo/mybatis-maven-plugin)
* 自主研发生成器插件：[https://github.com/tuonioooo/mybatis-third-generate](https://github.com/tuonioooo/mybatis-third-generate)

### 第三方插件Mybatis-Generator自动生成Dao、Model、Mapping相关文件

**1、相关文件**

关于Mybatis-Generator的下载可以到这个地址： [https://github.com/mybatis/generator/releases](https://github.com/mybatis/generator/releases)

由于我使用的是Mysql数据库，这里需要在准备一个连接mysql数据库的驱动jar包

以下是相关文件截图:

![](/assets/import-myabtis-01.png)

和Hibernate逆向生成一样，这里也需要一个配置文件：

generatorConfig.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--数据库驱动-->
    <classPathEntry    location="mysql-connector-java-5.0.8-bin.jar"/>
    <context id="DB2Tables"    targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost/mymessages" userId="root" password="root">
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!--生成Model类存放位置-->
        <javaModelGenerator targetPackage="lcw.model" targetProject="src">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--生成映射文件存放位置-->
        <sqlMapGenerator targetPackage="lcw.mapping" targetProject="src">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao类存放位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="lcw.dao" targetProject="src">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!--生成对应表及类名-->
        <table tableName="message" domainObjectName="Messgae" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
    </context>
</generatorConfiguration>
```

需要修改文件配置的地方我都已经把注释标注出来了，这里的相关路径（如数据库驱动包，生成对应的相关文件位置可以自定义）不能带有中文。

上面配置文件中的：

```
<table tableName="message" domainObjectName="Messgae" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
```

tableName和domainObjectName为必选项，分别代表数据库表名和生成的实力类名，其余的可以自定义去选择（一般情况下均为false）。

生成语句文件：

java -jar mybatis-generator-core-1.3.2.jar -configfile generatorConfig.xml -overwrite

**2、使用方法**

在该目录按住Shift键，右键鼠标选择"在此处打开命令窗口"，复制粘贴生成语句的文件代码即可。

看下效果图：

![](/assets/import-mybatis-02.png)

![](/assets/import-mybatis-03.png)

生成相关代码：

Message.java

```
package lcw.model;

public class Messgae {
    private Integer id;

    private String title;

    private String describe;

    private String content;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title == null ? null : title.trim();
    }

    public String getDescribe() {
        return describe;
    }

    public void setDescribe(String describe) {
        this.describe = describe == null ? null : describe.trim();
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content == null ? null : content.trim();
    }
}
```

MessgaeMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="lcw.dao.MessgaeMapper" >
  <resultMap id="BaseResultMap" type="lcw.model.Messgae" >
    <id column="id" property="id" jdbcType="INTEGER" />
    <result column="title" property="title" jdbcType="VARCHAR" />
    <result column="describe" property="describe" jdbcType="VARCHAR" />
    <result column="content" property="content" jdbcType="VARCHAR" />
  </resultMap>
  <sql id="Base_Column_List" >
    id, title, describe, content
  </sql>
  <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer" >
    select 
    <include refid="Base_Column_List" />
    from message
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer" >
    delete from message
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="lcw.model.Messgae" >
    insert into message (id, title, describe, 
      content)
    values (#{id,jdbcType=INTEGER}, #{title,jdbcType=VARCHAR}, #{describe,jdbcType=VARCHAR}, 
      #{content,jdbcType=VARCHAR})
  </insert>
  <insert id="insertSelective" parameterType="lcw.model.Messgae" >
    insert into message
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        id,
      </if>
      <if test="title != null" >
        title,
      </if>
      <if test="describe != null" >
        describe,
      </if>
      <if test="content != null" >
        content,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=INTEGER},
      </if>
      <if test="title != null" >
        #{title,jdbcType=VARCHAR},
      </if>
      <if test="describe != null" >
        #{describe,jdbcType=VARCHAR},
      </if>
      <if test="content != null" >
        #{content,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="lcw.model.Messgae" >
    update message
    <set >
      <if test="title != null" >
        title = #{title,jdbcType=VARCHAR},
      </if>
      <if test="describe != null" >
        describe = #{describe,jdbcType=VARCHAR},
      </if>
      <if test="content != null" >
        content = #{content,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="lcw.model.Messgae" >
    update message
    set title = #{title,jdbcType=VARCHAR},
      describe = #{describe,jdbcType=VARCHAR},
      content = #{content,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

MessgaeMapper.java

```
package lcw.dao;

import lcw.model.Messgae;

public interface MessgaeMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(Messgae record);

    int insertSelective(Messgae record);

    Messgae selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(Messgae record);

    int updateByPrimaryKey(Messgae record);
}
```



