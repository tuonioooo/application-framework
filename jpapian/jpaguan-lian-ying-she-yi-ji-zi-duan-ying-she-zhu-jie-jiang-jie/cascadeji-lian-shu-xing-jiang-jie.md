# cascade级联属性讲解

CascadeType.REMOVE

  


 Cascade remove operation，级联删除操作。

  


 删除当前实体时，与它有映射关系的实体也会跟着被删除。

CascadeType.MERGE

  


 Cascade merge operation，级联更新（合并）操作。

  


 当Student中的数据改变，会相应地更新Course中的数据。

CascadeType.DETACH

  


 Cascade detach operation，级联脱管/游离操作。

  


 如果你要删除一个实体，但是它有外键无法删除，你就需要这个级联权限了。它会撤销所有相关的外键关联。

CascadeType.REFRESH

  


 Cascade refresh operation，级联刷新操作。

  


 假设场景 有一个订单,订单里面关联了许多商品,这个订单可以被很多人操作,那么这个时候A对此订单和关联的商品进行了修改,与此同时,B也进行了相同的操作,但是B先一步比A保存了数据,那么当A保存数据的时候,就需要先刷新订单信息及关联的商品信息后,再将订单及商品保存。\(来自

[良心会痛](https://www.jianshu.com/users/a11a21634ee8/timeline)

的评论\)

CascadeType.ALL

  


 Cascade all operations，清晰明确，拥有以上所有级联操作权限。

  


  


作者：三汪

  


链接：https://www.jianshu.com/p/e8caafce5445

  


來源：简书

  


简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

