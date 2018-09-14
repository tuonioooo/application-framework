# 数据库schema与catalog简介

按照SQL标准的解释，在SQL环境下Catalog和Schema都属于抽象概念，主要用来解决命名冲突问题。

  


从概念上说，一个数据库系统包含多个Catalog，每个Catalog又包含多个Schema，而每个Schema又包含多个数据库对象（表、视图、序列等），反过来讲一个数据库对象必然属于一个Schema，而该Schema又必然属于一个Catalog，这样我们就可以得到该数据库对象的完全限定名称从而解决命名冲突的问题了

  


从实现的角度来看，各种数据库系统对Catalog和Schema的支持和实现方式千差万别，针对具体问题需要参考具体的产品说明书，比较简单而常用的实现方式是使用数据库名作为Catalog名，Oracle使用用户名作为Schema名

  


例如:

  


数据库          Catalog支持     Schema支持

  


Oracle            不支持          用户名\(User Id\)

  


MySQL             不支持         数据库名

