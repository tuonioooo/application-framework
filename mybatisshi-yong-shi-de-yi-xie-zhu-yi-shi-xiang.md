# Mybatis使用时的一些注意事项

1.Mybatis 使用foreach 时，in 里的参数如果多个是字符串的话，代码不需要加单引号，mybatis应该会自动加，否则查询不出

2.There is no getter for property named ‘XXX’ in ‘class java.lang.String’解决办法

示例1：

```
传入参数为‘parentCategoryId’，运行报错为：There is no getter for property named 'parentCategoryId' in 'class java.lang.String

<select id="selectCategoryList" parameterType="java.lang.String" resultType="MstCategoryBean">
SELECT
  category_id AS categoryId,
  category_name AS categoryName,
  view_orderby AS viewOrderby
FROM
  mst_category
WHERE
  del_flg =0
<if test="parentCategoryId!=null and parentCategoryId!=''">
 and
 parent_category_id = #{parentCategoryId}
</if>
</select>


发现不能将参数设为bean里的名称，如果传入类型为String类型，则参数需统一修改为[_parameter],修改后的sql语句如下(不管你的参数是什么，都要改成"_parameter")


<select id="selectCategoryList" parametertype="java.lang.String" resulttype="MstCategoryBean">
SELECT
  category_id AS categoryId,
  category_name AS categoryName,
  view_orderby AS viewOrderby
FROM
  mst_category
WHERE
  del_flg =0
  and
  parent_category_id = #{_parameter}
</select>
```

示例2：

```
用mybatis查询时，传入一个字符串传参数，且进行判断时，会报


There is no getter for property named 'moduleCode' in 'class java.lang.String
 错误写法：


<select id="queryAllParentModule" resultType="jobModule" parameterType="jobModule">
select 
  modulecode,
  modulename,
  modulevalue,
  linkurl,
  rank,
  parentmodule=isnull(parentmodule,1),
  moduledescription
from job_module
<where>
<choose>
<when test="moduleCode!=null and moduleCode!=''">modulecode = #{moduleCode}</when>
<when test="moduleCode==null or moduleCode==''">(parentmodule is null or len(parentmodule)&lt;=0)</when>
</choose>
</where>
lt;/select>
 需要修改成：


<select id="queryModuleByCode" resultType="jobModule" parameterType="string">
select modulecode,
modulename,
modulevalue,
linkurl,
rank,
parentmodule=isnull(parentmodule,1),
moduledescription
from job_module
<where>
<choose>
<when test="_parameter!=null and _parameter!=''">modulecode = #{_parameter}</when>
<when test="_parameter==null or _parameter==''">(parentmodule is null or len(parentmodule)&lt;=0)</when>
</choose>
</where>
</select>
 不管你的参数是什么，都要改成"_parameter"；
```

3.Error parsing Mapper XML  IllegalArgumentException: Mapped Statements collection already contains value for .......

目前知道有三个原因：

* parameterMap改成parameterType...
* id 重复
* parameterType参数有问题
* mapper.xml中没有加入namespace 
* mapper.xml中的方法和接口mapper的方法不对应 
* mapper.xml没有加入到mybatis-config.xml中\(即总的配置文件\)，例外：配置了mapper文件的包路径的除外 

* mapper.xml文件名和所写的mapper名称不相同。



