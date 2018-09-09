# Hibernate中，对象有三种状态：

在[hibernate](http://lib.csdn.net/base/javaee)中，对象有三种状态：

临时状态\(Transient\)、持久状态\(Persistent\)和游离状态\(Detached\)。

处于持久态的对象也称为 PO\(PersistenceObject\),临时对象和游离对象也称为VO\(ValueObject\).

简单的来讲这三种状态定义是：

* 临时状态：刚new出来一个对象，还没有被保存到[数据库](http://lib.csdn.net/base/mysql)中
* 持久化状态：已经被保存到[数据库](http://lib.csdn.net/base/mysql)中
* 离线状态：数据库中有，但是session中不存在该对象

了解[hibernate](http://lib.csdn.net/base/javaee)的这三种状态的含义后，那我们就通过一张图来开始我们的深入hibernate的三种状态之旅吧。

![](http://images.cnitblog.com/blog/432441/201310/21101906-b68babbf49d34e1aac6599fd2a9d97ae.jpg)

下面我们来具体的讲这三种状态定义及代码解释：

## 临时状态

由 new命令开辟内存空间的java对象,例如：User user=new User\(\);

临时对象在内存孤立存在,它是携带信息的载体,不和数据库的数据有任何关联关系.

\(a\) 如果没有变量对该对象进行引用,它将被gc回收；

\(b\) 在Hibernate中,可通过 session的save\(\)或saveOrUpdate\(\)方法将临时对象与数据库相关联,并将数据对应的插入数据库中,此时该临时对象转变成持久化对象.

**1.TestTransient**

```
session = HibernateUtil.openSession();
session.beginTransaction();
User user = new User();
user.setUsername("aaa");
user.setPassword("aaa");
user.setBorn(new Date());
/*
* 以上user就是一个Transient(瞬时状态)，此时user并没有被session进行托管，即在session的缓存中
* 还不存在user这个对象，当执行完save方法后，此时user被session托管，并且数据库中存在了该对象，
* user就变成了一个Persistent(持久化对象)
*/
session.save(user);
session.getTransaction().commit();
```

此时我们知道hibernate会发出一条insert的语句，执行完save方法后，该user对象就变成了持久化的对象了

`Hibernate: insert into t_user (born, password, username) values (?, ?, ?)`

## 持久状态

处于该状态的对象在数据库中具有对应的记录,并拥有一个持久化标识.通过session的get\(\)、load\(\) 等方法获得的对象都是持久对象。

持久化对象被修改变更后，不会马上同步到数据库，直到数据库事务提交。在同步之前，持久化对象是脏的（Dirty）。

\(a\) 如果是用hibernate的delete\(\)方法,对应的持久对象就变成临时对象,因数据库中的对应数据已被删除,该对象不再与数据库的记录关联.

\(b\) 当一个session执行close\(\)或 clear\(\)、evict\(\)之后,持久对象变成游离对象,此时该对象虽然具有数据库识别值,但它已不在HIbernate持久层的管理之下.

持久对象具有如下特点:

 \(1\)和session实例关联;

\(2\)在数据库中有与之关联的记录,并拥有持久化标识.

Session的不同操作对对象状态的影响：

Session的save\(\)方法:

save\(\)方法将一个临时对象转变为持久对象。

Session的update\(\)方法:

update\(\)方法 将一个游离对象转变为持久对象。

Session的lock\(\)方法:

调用lock\(\)方法将对象同Session相关联而不强制更新。

Session的merge\(\)方法:

拷贝指定对象的状态到具有相同对象标识符的持久对象。

Session的saveOrUpdate\(\)方法:

saveOrUpdate\(\) 方法对于临时对象，执行save\(\)方法，对于游离对象，执行update\(\)方法。

Session的load\(\)和get\(\)方法:

load\(\)方法和get\(\)方法都可以根据对象的标识符加载对象，这两个方法加载的对象都位于Session的缓存中，属于持久对象。

Session的 delete\(\)方法:

delete\(\)方法用于从数据库中删除与持久化对象对应的记录。如果传入的是一个持久化对象，Session就执行一条 delete语句。如果传入的参数是游离对象，先使分离对象与Session关联，使它变为持久化对象，然后才计划执行一个delete语句。

Session 的evict\(\)方法:

evict\(\)方法从Session的缓存中删除一个持久对象。

2.TestPersistent01

session =

HibernateUtil.openSession\(\);

```
 session.beginTransaction\(\);

 User user
```

=

new

User\(\);

```
 user.setUsername\(
```

"

aaa

"

\);

user.setPassword\(

"

aaa

"

\);

user.setBorn\(

new

Date\(\)\);

//

以上user就是Transient（瞬时状态），表示没有被session管理并且数据库中没有

//

执行save之后，被session所管理，而且数据库中已经存在，此时就是Persistent状态

session.save\(user\);

//

此时user是持久化状态，已经被session所管理，当在提交时，会把session中的对象和目前的对象进行比较

//

如果两个对象中的值不一致就会继续发出相应的sql语句

user.setPassword\(

"

bbb

"

\);

//

此时会发出2条sql，一条用户做插入，一条用来做更新

session.getTransaction\(\).commit\(\);

在调用了save方法后，此时user已经是持久化对象了，被保存在了session缓存当中，这时user又重新修改了属性值，那么在提交事务时，此时hibernate对象就会拿当前这个user对象和保存在session缓存中的user对象进行比较，如果两个对象相同，则不会发送update语句，否则，如果两个对象不同，则会发出update语句

。

Hibernate: insert into t\_user \(born, password, username\) values \(?, ?, ?

\)

Hibernate: update t\_user set born

=?, password=?, username=? where id=?

3.TestPersistent02

SimpleDateFormat sdf =

new

SimpleDateFormat\(

"

yyyy-MM-dd

"

\);

session

=

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

```
 user.setBorn\(
```

new

Date\(\)\);

user.setUsername\(

"

zhangsan

"

\);

user.setPassword\(

"

zhangsan

"

\);

session.save\(user\);

user.setPassword\(

"

222

"

\);

//

该条语句没有意义

session.save\(user\);

user.setPassword\(

"

zhangsan111

"

\);

//

没有意义

session.update\(user\);

user.Born\(sdf.parse\(

"

1988-12-22

"

\)\);

//

没有意义

session.update\(user\);

session.getTransaction\(\).commit\(\);

这个时候会发出多少sql语句呢？还是同样的道理，在调用save方法后，user此时已经是持久化对象了，记住一点：

如果一个对象已经是持久化状态了，那么此时对该对象进行各种修改，或者调用多次update、save方法时，hibernate都不会发送sql语句，只有当事物提交的时候，此时hibernate才会拿当前这个对象与之前保存在session中的持久化对象进行比较，如果不相同就发送一条update的sql语句，否则就不会发送update语句

Hibernate: insert into t\_user \(born, password, username\) values \(?, ?, ?

\)

Hibernate: update t\_user set born

=?, password=?, username=? where id=?

```
4.TestPersistent03
```

SimpleDateFormat sdf =

new

SimpleDateFormat\(

"

yyyy-MM-dd

"

\);

session

=

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

user.setBorn\(sdf.parse\(

"

1976-2-3

"

\)\);

user.setUsername\(

"

zhangsan2

"

\);

user.setPassword\(

"

zhangsan2

"

\);

session.save\(user\);

// 以下三条语句没有任何意义

session.save\(user\);

session.update\(user\);

session.update\(user\);

user.setUsername\(

"

zhangsan3

"

\);

session.getTransaction\(\).commit\(\);

相信这个

[测试](http://lib.csdn.net/base/softwaretest)

用例，大家应该都知道结果了，没错，此时hibernate也会发出两条sql语句，原理一样的

Hibernate: insert into t\_user \(born, password, username\) values \(?, ?, ?

\)

Hibernate: update t\_user set born

=?, password=?, username=? where id=?

5.TestPersistent04

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

//

此时user是Persistent

User user = \(User\)session.load\(User.

class

, 4

\);

//

由于user这个对象和session中的对象不一致，所以会发出sql完成更新

user.setUsername\(

"

bbb

"

\);

session.getTransaction\(\).commit\(\);

我们来看看此时会发出多少sql语句呢？同样记住一点：

当session调用load、get方法时，此时如果数据库中有该对象，则该对象也变成了一个持久化对象，被session所托管

。因此，这个时候如果对对象进行操作，在提交事务时同样会去与session中的持久化对象进行比较，因此这里会发送两条sql语句

Hibernate: select user0\_.id as id0\_0\_, user0\_.born as born0\_0\_, user0\_.password as password0\_0\_, user0\_.username as username0\_0\_ from t\_user user0\_ where user0\_.id=?

Hibernate: update t\_user set born

=?, password=?, username=? where id=?

6.TestPersistent05

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

//

此时user是Persistent

User user = \(User\)session.load\(User.

class

, 4

\);

user.setUsername\(

"

123

"

\);

//

清空session

session.clear\(\);

session.getTransaction\(\).commit\(\);

再看这个例子，当我们load出user对象时，此时user是持久化的对象，在session缓存中存在该对象，此时我们在对user进行修改后，

然后调用session.clear\(\)方法，这个时候就会将session的缓存对象清空，那么session中就没有了user这个对象，这个时候在提交事务的时候，发现已经session中已经没有该对象了，所以就不会进行任何操作

，因此这里只会发送一条select语句

Hibernate: select user0\_.id as id0\_0\_, user0\_.born as born0\_0\_, user0\_.password as password0\_0\_, user0\_.username as username0\_0\_ from t\_user user0\_ where user0\_.id=?

#### 游离状态

当与某持久对象关联的session被关闭后,该持久对象转变为游离对象.当游离对象被重新关联到session上 时,又再次转变成持久对象（在Detached其间的改动将被持久化到数据库中）。 游离对象拥有数据库的识别值,但已不在持久化管理范围之内。

\(a\) 通过update\(\)、saveOrUpdate\(\)等方法,游离对象可转变成持久对象.

\(b\) 如果是用hibernate的delete\(\)方法,对应的游离对象就变成临时对象,因数据库中的对应数据已被删除,该对象不再与数据库的记录关联.

\(c\) 在没有任何变量引用它时,它将被gc在适当的时候回收；

游离对象具有如下特点:

```
 \(1\)本质上与瞬时对象相同,在没有任何变量引用它时,JVM会在适当的时候将它回收;

 \(2\)比瞬时对象多了一个数据库记录标识值.
```

7.TestDetached01

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

//

此时user是一个离线对象，没有被session托管

User user =

new

User\(\);

user.setId\(

4

\);

user.setPassword\(

"

hahahaha

"

\);

//

当执行save的时候总是会添加一条数据，此时id就会根据Hibernate所定义的规则来生成

session.save\(user\);

session.getTransaction\(\).commit\(\);

我们看到，当调用了user.setId\(4\)时，此时user是一个离线的对象，因为数据库中存在id=4的这个对象，但是该对象又没有被session所托管，所以这个对象就是离线的对象，要使离线对象变成一个持久化的对象，应该调用什么方法呢？我们知道调用save方法，可以将一个对象变成一个持久化对象，但是，当save一执行的时候，此时hibernate会根据id的生成策略往数据库中再插入一条数据，所以如果调用save方法，此时数据库会发送一条插入的语句：

Hibernate: insert into t\_user \(born, password, username\) values \(?, ?, ?\)

所以对于离线对象，如果要使其变成持久化对象的话，我们不能使用save方法，而应该使用update方法

8.TestDetached02

SimpleDateFormat sdf =

new

SimpleDateFormat\(

"

yyyy-MM-dd

"

\);

session

=

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

user.setId\(

5

\);

//

完成update之后也会变成持久化状态

session.update\(user\);

user.setBorn\(sdf.parse\(

"

1998-12-22

"

\)\);

user.setPassword\(

"

world

"

\);

user.setUsername\(

"

world

"

\);

//

会发出一条sql

session.update\(user\);

session.getTransaction\(\).commit\(\);

此时我们看到，当调用了update方法以后，此时user已经变成了一个持久化的对象，那么如果此时对user对象进行修改操作后，在事务提交的时候，则会拿该对象和session中刚保存的持久化对象进行比较，如果不同就发一条sql语句

Hibernate: update t\_user set born=?, password=?, username=? where id=?

9.TestDetached03

SimpleDateFormat sdf =

new

SimpleDateFormat\(

"

yyyy-MM-dd

"

\);

session

=

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

user.setId\(

5

\);

//

完成update之后也会变成持久化状态

session.update\(user\);

user.setBorn\(sdf.parse\(

"

1998-12-22

"

\)\);

user.setPassword\(

"

lisi

"

\);

user.setUsername\(

"

lisi

"

\);

//

会抛出异常

user.setId\(333

\);

session.getTransaction\(\).commit\(\);

```
我们看这个例子，前面的操作一样，调用update方法后，user变成了一个持久化对象，在对user进行一些修改后，此时又通过 user.setId\(333\)方法设置了user的ID，那么这个时候，hibernate会报错，因为我们的user当前已经是一个持久化对象，
```

如果试图修改一个持久化对象的ID的值的话，就会抛出异常

，这点要特别注意

org.hibernate.HibernateException: identifier of an instance of com.xiaoluo.bean.User was altered from 5 to 333

```
10.TestDetached04
```

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

user.setId\(

5

\);

//

现在user就是transient对象

session.delete\(user\);

//

此时user已经是瞬时对象，不会被session和数据库所管理

user.setPassword\(

"

wangwu

"

\);

session.getTransaction\(\).commit\(\);

这里在调用了session.delete\(\)方法以后，此时后user就会变成一个瞬时对象，因为此时数据库中已经不存在该对象了，既然user已经是一个瞬时对象了，那么对user再进行各种修改操作的话，hibernate也不会发送任何的修改语句，因此这里只会有一条 delete的语句发生：

Hibernate: delete from t\_user where id=?

11.TestDetached05

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

User user

=

new

User\(\);

user.setId\(

4

\);

user.setPassword\(

"

zhaoliu

"

\);

//

如果user是离线状态就执行update操作，如果是瞬时状态就执行Save操作

//

但是注意：该方法并不常用

session.saveOrUpdate\(user\);

session.getTransaction\(\).commit\(\);

这里我们来看看 saveOrUpdate这个方法，这个方法其实是一个

"

偷懒

"

的方法，如果对象是一个离线对象，那么在执行这个方法后，其实是调用了update方法，如果对象是一个瞬时对象，则会调用save方法，记住：

如果对象设置了ID值，例如user.setId\(4\)，那么该对象会被假设当作一个离线对象，此时就会执行update操作

。

Hibernate: update t\_user set born=?, password=?, username=? where id=?

如果此时我将user.setId\(4\)这句话注释掉，那么此时user就是一个瞬时的对象，那么此时就会执行save操作，就会发送一条insert语句

Hibernate: insert into t\_user \(born, password, username\) values \(?, ?, ?\)

12.TestDetached06

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

//

u1已经是持久化状态

User u1 = \(User\)session.load\(User.

class

, 3

\);

System.out.println\(u1.getUsername\(\)\);

//

u2是离线状态

User u2 =

new

User\(\);

u2.setId\(

3

\);

u2.setPassword\(

"

123456789

"

\);

//

此时u2将会变成持久化状态，在session的缓存中就存在了两份同样的对象,在session中不能存在两份拷贝，否则会抛出异常

session.saveOrUpdate\(u2\);

我们再来看一下这个例子，此时我们的u1已经是持久化的对象了，保存在session缓存中，u2通过调用saveOrUpdate方法后也变成了一个持久化的对象，此时也会保存在session缓存中，这个时候session缓存中就存在了一个持久化对象有两个引用拷贝了，这个时候hibernate就会报错

org.hibernate.NonUniqueObjectException:

a different object with the same identifier value was already associated with the session

: \[com.xiaoluo.bean.User\#3\]

一个session中不能存在对一个持久化对象的双重copy的，要解决这个方法，

我们这里又要介绍session的另一个方法  merge方法，这个方法的作用就是解决一个持久化对象两分拷贝的问题，这个方法会将两个对象合并在一起成为一个对象

。

session =

HibernateUtil.openSession\(\);

session.beginTransaction\(\);

//

user1已经是持久化状态

User user1 = \(User\)session.load\(User.

class

, 3

\);

System.out.println\(user1.getUsername\(\)\);

//

user2是离线状态

User user2 =

new

User\(\);

user2.setId\(

3

\);

user2.setPassword\(

"

123456789

"

\);

//

此时user2将变成持久化状态，在session的缓存中就存在了两份同样的对象,在session中不能存在两份拷贝，否则会抛出异常

session.saveOrUpdate\(user2\);

//

merge方法会判断session中是否已经存在同一个对象，如果存在就将两个对象合并

session.merge\(user2\);

//

最佳实践：merge一般不用

session.getTransaction\(\).commit\(\);

我们看到通过调用了merge方法以后，此时会将session中的两个持久化对象合并为一个对象，但是merge方法不建议被使用

Hibernate: select user0\_.id as id0\_0\_, user0\_.born as born0\_0\_, user0\_.password as password0\_0\_, user0\_.username as username0\_0\_ from t\_user user0\_ where user0\_.id=?

zhangsan

Hibernate: update t\_user set born

=?, password=?, username=? where id=?

本篇随笔可能概念性的内容比较少，基本都是通过测试用例来分析hibernate的三种状态可能会出现的各种情况。

#### 最后总结一下：

①.对于刚创建的一个对象，如果session中和数据库中都不存在该对象，那么该对象就是瞬时对象\(Transient\)

②.瞬时对象调用save方法，或者离线对象调用update方法可以使该对象变成持久化对象，如果对象是持久化对象时，那么对该对象的任何修改，都会在提交事务时才会与之进行比较，如果不同，则发送一条update语句，否则就不会发送语句

③.离线对象就是，数据库存在该对象，但是该对象又没有被session所托管

