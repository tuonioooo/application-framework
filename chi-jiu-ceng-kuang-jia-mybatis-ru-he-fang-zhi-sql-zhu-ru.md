# 持久层框架mybatis如何防止sql注入

## 什么是SQL注入

是一种代码注入技术，用于攻击数据驱动的应用，恶意的SQL语句被插入到执行的实体字段中（例如，为了转储数据库内容给攻击者）

**SQL注入**，大家都不陌生，是一种常见的攻击方式。**攻击者**在界面的表单信息或URL上输入一些奇怪的SQL片段（例如“or ‘1’=’1’”这样的语句），有可能入侵**参数检验不足**的应用程序。所以，在我们的应用中需要做一些工作，来防备这样的攻击方式。在一些安全性要求很高的应用中（比如银行软件），经常使用将**SQL语句**全部替换为**存储过程**这样的方式，来防止SQL注入。这当然是**一种很安全的方式**，但我们平时开发中，可能不需要这种死板的方式。

示例如：

常见的SQL注入式攻击过程类如：

⑴ 某个[ASP.NET](https://baike.baidu.com/item/ASP.NET/197912)Web应用有一个登录页面，这个登录页面控制着用户是否有权访问应用，它要求用户输入一个名称和密码。

⑵ 登录页面中输入的内容将直接用来构造动态的SQL命令，或者直接用作[存储过程](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B)的参数。下面是ASP.NET应用构造查询的一个例子：

System.Text.StringBuilder query = new System.Text.StringBuilder\("SELECT \* from Users WHERE login = '"\).

[Append](https://baike.baidu.com/item/Append/8293363)\(txtLogin.Text\).Append\("' AND password='"\)

.Append\(txtPassword.Text\).Append\("'"\);

⑶ 攻击者在用户名字和密码输入框中输入"'或'1'='1"之类的内容，例如a' or '1'='1。

⑷ 用户输入的内容提交给服务器之后，服务器运行上面的[ASP.NET](https://baike.baidu.com/item/ASP.NET/197912)

代码构造出查询用户的SQL命令，但由于攻击者输入的内容非常特殊，所以最后得到的SQL命令变成：

SELECT \* from Users WHERE login = '' or '1'='1' AND password = '' or '1'='1'。

⑸ 服务器执行查询或存储过程，将用户输入的身份信息和服务器中保存的身份信息进行对比。

⑹ 由于SQL命令实际上已被注入式攻击修改，已经不能真正验证用户身份，所以系统会错误地授权给攻击者。

如果攻击者知道应用会将[表单](https://baike.baidu.com/item/%E8%A1%A8%E5%8D%95)中输入的内容直接用于验证身份的查询，他就会尝试输入某些特殊的SQL字符串篡改查询改变其原来的功能，欺骗系统授予[访问](https://baike.baidu.com/item/%E8%AE%BF%E9%97%AE)权限。

