# @OneToMany、@ManyToOne级联配置

## 注解讲解

**@OneToMany\(fetch=FetchType,cascade=CascadeType\)**

可选

@OneToMany描述一个一对多的关联,该属性应该为集体类型,在数据库中并没有实际字段.

fetch:表示抓取策略,默认为FetchType.LAZY,因为关联的多个对象通常不必从数据库预先读取到内存

cascade:表示级联操作策略,对于OneToMany类型的关联非常重要,通常该实体更新或删除时,其关联的实体也应当被更新或删除

例如:实体User和Order是OneToMany的关系,则实体User被删除时,其关联的实体Order也应该被全部删除

示例：

```
@OneTyMany(cascade=ALL)
public List getOrders() {
        return orders;
}
```

**@ManyToOne\(fetch=FetchType,cascade=CascadeType\)**

可选

@ManyToOne表示一个多对一的映射,该注解标注的属性通常是数据库表的外键

optional:是否允许该字段为null,该属性应该根据数据库表的外键约束来确定,默认为true

fetch:表示抓取策略,默认为FetchType.EAGER

cascade:表示默认的级联操作策略,可以指定为ALL、PERSIST、MERGE、REFRESH和REMOVE中的若干组合,默认为无级联操作

targetEntity:表示该属性关联的实体类型.该属性通常不必指定,ORM框架根据属性类型自动判断targetEntity.

示例：

```
//订单Order和用户User是一个ManyToOne的关系
//在Order类中定义
@ManyToOne
@JoinColumn(name=”USER”)
public User getUser() {
   return user;
}
```

**@JoinColumn**

可选

@JoinColumn和@Column类似,介量描述的不是一个简单字段,而一一个关联字段,例如.描述一个@ManyToOne的字段.

name:该字段的名称.由于@JoinColumn描述的是一个关联字段,如ManyToOne,则默认的名称由其关联的实体决定.

例如,实体Order有一个user属性来关联实体User,则Order的user属性为一个外键,

其默认的名称为实体User的名称+下划线+实体User的主键名称

示例：见@ManyToOne

## @OneToMany和@ManyToOne单向关联配置

**@ManyToOne单向关联配置**

Order实体类

```
package com.master.bean;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-9-17
 * Time: 9:48
 * info:
 */
@Table(name="t_order")
@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {

    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long orderId;

    private String orderName;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "user_id") //单向关联必须要指定关联字段
    private User user;

}

```

User实体类

```
package com.master.bean;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;

/**
 * @author tony
 */
@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

}

```

dao

```
package com.master.dao;


import com.master.bean.Order;
import org.springframework.data.jpa.repository.JpaRepository;


/**
 * @author tony
 */
public interface OrderRepository extends JpaRepository<Order, Long> {


}


//=======================================================================================


package com.master.dao;

import com.master.bean.User;
import org.springframework.data.jpa.repository.JpaRepository;


/**
 * @author tony
 */
public interface UserRepository extends JpaRepository<User, Long> {


}

```

测试

```
package com.master;

import com.master.bean.Order;
import com.master.bean.User;
import com.master.dao.OrderRepository;
import com.master.dao.UserRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Optional;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootJpaCascadeApplicationTests {

	@Autowired
	private OrderRepository orderRepository;

	@Autowired
	private UserRepository userRepository;

	@Test
	public void contextLoads() {
	}

	/**
	 * @ManyToOne 单向关联测试
	 */
	@Test
	public void cascadeSingle(){

		User user = new User();
		user.setId(1l);
		user.setAge(20);
		user.setName("ALLEN");
		userRepository.save(user);

		Order order = new Order(null,"袜子", user);
		Order order2 = new Order(null,"裤子", user);
		//保存
		orderRepository.save(order);
		orderRepository.save(order2);

		//获取

		//getOne仅仅是通过代理获取并且仅仅是延迟加载策略
		//findById两种策略都支持
		Order getOrder = orderRepository.findById(order.getOrderId()).get();
		System.out.println("getOrder.getOrderName() = " + getOrder.getOrderName());
		System.out.println("getOrder.getUser().getName() = " + getOrder.getUser().getName());

	}

}

```

> 说明：
>
> 单向关联的一方，必须配置其JoinColumn，指定关联字段，注意延迟加载和立即抓取策略的使用方式，代码里已经注释，
>
> 注意其实体类名不能与Mysql关键字冲突等问题MySQL server version for the right syntax to use near



