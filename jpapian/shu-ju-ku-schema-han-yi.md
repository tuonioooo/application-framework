# 数据库schema含义

数据库Schema有两种含义，一种是概念上的Schema，指的是一组DDL语句集，该语句集完整地描述了数据库的结构。还有一种是物理上的 Schema，指的是数据库中的一个名字空间，它包含一组表、视图和存储过程等命名对象。物理Schema可以通过标准SQL语句来创建、更新和修改。例如以下SQL语句创建了两个物理Schema：

```
create schema SCHEMA\_A;

create table SCHEMA\_A.CUSTOMERS\(ID int not null,……\);

create schema SCHEMA\_B;

create table SCHEMA\_B.CUSTOMERS\(ID int not null,……\);
```

简单的说：就是一个数据库用户所拥有的数据库的对象。

比如scott用户建立了表，索引，视图，存储过程等对象，那么这些对象就构成了schema   scott

在一个数据库中可以有多个应用的数据表，这些不同应用的表可以放在不同的schema之中，同时，每一个schema对应一个用户，不同的应用可以以不同的用户连接数据库，这样，一个大数据库就可以根据应用把其表分开来管理。不同的schema之间它们没有直接的关系，不同的shcema之间的表可以同名，也可以互相引用（但必须有权限），在没有操作别的schema的操作根权下，每个用户只能操作它自己的schema下的所有的表。不同的schema下的同名的表，可以存入不同的数据（即schema用户自己的数据\).

---

如果我们想了解数据库中的User和Schema到底什么关系，那么让我们首先来了解一下数据库中User和Schema到底是什么概念。

   在SQL Server2000中，由于架构的原因，User和Schema总有一层隐含的关系，让我们很少意识到其实User和Schema是两种完全不同的概念，不过在SQL Server2005中这种架构被打破了，User和Schema也被分开了。     



   首先我来做一个比喻，什么是Database，什么是Schema，什么是Table，什么是Column，什么是Row，什么是User？我们可以把Database看作是一个大仓库，仓库分了很多很多的房间，Schema就是其中的房间，一个Schema代表一个房间，Table可以看作是每个Schema中的床，Table（床）就被放入每个房间中，不能放置在房间之外，那岂不是晚上睡觉无家可归了J。，然后床上可以放置很多物品，就好比Table上可以放置很多列和行一样，数据库中存储数据的基本单元是Table，现实中每个仓库放置物品的基本单位就是床， User就是每个Schema的主人，【所以Schema包含的是Object，而不是User】，其实User是对应于数据库的（即User是每个对应数据库的主人），既然有操作数据库（仓库）的权利，就肯定有操作数据库中每个Schema（房间）的权利，就是说每个数据库映射的User有每个Schema（房间）的钥匙，换句话说，如果他是某个仓库的主人，那么这个仓库的使用权和仓库中的所有东西都是他的（包括房间），他有完全的操作权，可以扔掉不用的东西从每个房间，也可以放置一些有用的东西到某一个房间，当然也可以拆除一个房间（Remove Schema）。呵呵，和现实也太相似了吧。我（仓库的管理员）还可以给User分配具体的权限，也就是他到某一个房间能做些什么，是只能看（Read-Only），还是可以像主人一样有所有的控制权（R/W），这个就要看这个User所对应的角色Role了，至于分配权限的问题，我留在以后单独的blog中详述。比喻到这里，相信大家都清楚了吧。

OK，我们话题继续！

```
   在SQL Server2000中，假如我们在某一个数据库中创建了用户Bosco，按么此时后台也为我们默认地创建了默认Schema 【Bosco】。Schema的名字和User的名字相同，这也是我们分不清楚用户和Schema的原因。

   在SQL Server2005中，为了向后兼容，当你用sp\_adduser 存储过程创建一个用户的时候，SQL Server2005同时也创建了一个和用户名相同的Schema，然而这个存储过程是为了向后兼容才保留的，我们应该逐渐熟悉用新的DDL语言Create User和Create Schema来操作数据库。在SQL Server2005中，当我们用Create User创建数据库用户时，我们可以为该用户指定一个已经存在的Schema作为默认Schema，如果我们不指定，则该用户所默认的Schema即为dbo Schema，dbo 房间（Schema）好比一个大的公共房间，在当前登录用户没有默认Schema的前提下，如果你在大仓库中进行一些操作，比如Create Tabe，如果没有指定特定的房间（Schema），那么你的物品就只好放进公共的dbo房间（Schema）了。但是如果当前登录用户有默认的Schema，那么所做的一切操作都是在默认Schema上进行（比如当前登录用户为login1，该用户的默认Schema为login1，那么所做的所有操作都是在这个login1默认Schema上进行的。实验已经证明的确如此）。估计此时你会有一点晕，为什么呢？我刚才说dbo是一个Schema，但是你可以在数据库中查看到，dbo同时也是一个user，晕了吧，呵呵。

   在SQL Server2005中创建一个数据库的时候，会有一些默认Schema被创建，被默认创建的Schema有：dbo，INFORMATION\_SCHEMA, guest，sys等等（还有一些角色Schema，不提了，又晕了）。

   我在上文中已经提到了，在SQL Server2005中当用存储过程sp\_adduser创建一个user时，同时SQL Server2005也为我们创建了一个默认的和用户名相同的Schema，这个时候问题出来了，当我们create table A时，如果没有特定的Schema做前缀，这个A表创建在了哪个Schema上，即进入了哪个房间？答案是：
```

如果当前操作数据库的用户（可以用Select current\_user查出来）有默认的Schema（在创建用户的时候指定了），那么表A被创建在了默认的Schema上。

如果当前操作数据库的用户没有默认的Schema（即在创建User的时候默认为空），但是有一个和用户名同名的Schema，那么表A照样被创建在了dbo Schema上，即使有一个和用户名同名的Schema存在，由于它不是该用户默认的Schema，所以创建表的时候是不会考虑的，当作一般的Schema来处理，别看名字相同，可是没有任何关系哦。

如果在创建表A的时候指定了特定的Schema做前缀，则表A被创建在了指定的 Schema上（有权限吗？）

现在问题又出来了，在当前操作数据库的用户（用select current\\_user可以查看到，再次强调）没有默认Schema的前提下，当我们用Create table A语句时，A表会去寻找dbo Schema，并试图创建在dbo Schema上，但是如果创建A表的用户只有对dbo Schema的只读权限，而没有写的权限呢？这个时候A表既不是建立不成功，这个就是我以后会提及到的Login,User, Role和Schema四者之间的关系。在这里，为了避免混淆和提高操作数据库的速度（在少量数据范围内，对我们肉眼来说几乎看不到差异），我们最好每次在操作数据库对象的时候都显式地指定特定的Schema最为前缀。

现在如果登录的用户为Sue，该用户有一个默认Schema也为Sue，那么如果现在有一条查询语句为Select \\* from mytable, 那么搜寻每个房间（Schema）的顺序是怎样的呢？顺序如下：

首先搜寻 sys.mytable   （Sys Schema）

然后搜寻 Sue.mytable   （Default Schema）

最后搜寻 dbo.mytable      （Dbo Schema）

执行的顺序大家既然清楚了，那么以后在查询数据库表中的数据时，最好指定特定的Schema前缀，这样子，数据库就不用去扫描Sys Schema了，当然可以提高查询的速度了。

另外需要提示一下的是，每个数据库在创建后，有4个Schema是必须的（删都删不掉），这4个Schema为：dbo，guest，sys和INFORMATION\\_SCHEMA，其余的Schema都可以删除。

