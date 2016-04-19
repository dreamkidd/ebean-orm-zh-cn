**映射**的主要目的是将程序代码与数据库模型隔离开来。

这意味着即使数据库出现了一些改变，也不需要改变程序代码。写代码的时候也不需要参考特定的表名，视图名，及列名，这意味着程序可以接受一些不可预见的改变。

# JPA 映射
Ebean 遵循 JAP 的映射规范，你可以学习并使用相同的映射注解。

# DDL 生成
Ebean 的 DDL 生成有助于敏捷开发及测试。同时也有助于理解映射。

对于简单的数据库 DDL 生成足够了，但是对于大型数据库，远远达不到_生成质量_，对于大型的数据库，你可以将 _DDL 生成_作为一个起点，DBA 更多的是在表和索引的物理层面控制大型数据库（指定表空间，跨磁盘 IO，分割大型表，视图控制等）

# 命名惯例
Ebean 有一个命名约定 API 以映射列名与属性名。同时也将实体（entity）与 数据库表名映射起来，如果需要的话，也可以顾及数据库 scheme 和目录。

参见`com.avaje.ebean.config.NamingConvention`

默认的`UnderscoreNamingConvention` 会将数据库中的`_`命名映射为一般的 java 的驼峰命名。（如:"first_name"映射为"firstName"）

也可以使用`MatchingNamingConvention`或者自己实现命名惯例。

`matchingnamingconvention`命名列名称匹配的属性名和表名匹配实体名称。ebean遵循JPA规范指出，在没有注释的类的名称将作为表名和属性名的情况将作为该列的名称。

# 映射注解
## 基础注解
- `@Entity`- 仅标注为实体 Bean。Ebean 遵循 JPA 的规范，每个实体 Bean 下有一个默认的构造，并且提供属性的 getter/setter
- `@Table`- 指定实体Bean 使用的表名。
- `@Id`和`@EmbeddedId`- 使用其中之一标注该属性是 ID 属性。在 ID 属性只是简单属性（如 Integer，String 等）的时候使用`@Id`,当ID 属性是复合类型的时候（一个嵌套 Bean），使用`@EmbeddedId`。
- `@Column`- 如果时与数据库列命名不相符的属性名的时候或者在需要使用引号的时候使用该注解，否则不需要使用该注解。
- `@Lob`- 将属性映射为 Clob/Blob/Longvarchar或 Longvarbinary 类型时需要使用
- `@Transient`- 这标志该属性不是永久的
- `@CreateTimestamp`- 当当前实体在 created/inserted 时设置 timestamp 属性。另一种替代该注解的方式是，是设置`@Column(insertable = false, updateable = false)`然后在数据插入时获取当前系统时间（设置在DB列的默认值是SYSTIME等方式）。替代方法的缺点是，在插入后，如果想获取插入的值，需要重新取查询该值才能取回。
- `@UpdateTimestamp`- 设置一个 timestamp 属性设置为datetime当实体的最后更新时间

# 关系
## 数据库设计规范化
如果你熟悉数据库设计，那么关系将变得明快。如果不熟悉，则建议你尽快去维基百科这写主题，这将使你更了解 ORM 映射。

## 数据库外键与 ORM 关系
假设你的数据库有精心设计的外键，那么 ORM 映射也是自然而然的。如果你的数据库设计有一些更"有趣"的设计，那么ORM 则会带来更多伤脑筋的地方伴随着更多的妥协。

## 一对多关系
这可能是最常见的关系，我们也从一对多关系开始。`@OneToMany`和`@ManyToOne`代表关系的两端，这种关系通常是直接映射到数据库外键约束.

## 数据库外键约束
一个经典的数据库设计是通过外键约束来实现充分的"一对多/多对一"关系，一个外键约束有"引入","导出"两个方面：
- "一个客户有多个订单"
- "一个订单属于一个客户"

*
- 客户表导出其主键对应订单表
- 订单表"导入/引用"顾客表的主键

> 一个外键约束可以从导出面（客户表）或导入面（订单表）查看。

> `@OneToMany`和`ManyToOne`可以直接映射到外键约束。

客户实体bean

```

...

@Entity

@Table(name="or_customer")

public class Customer {

    ...

    @OneToMany

    List<Order> orders;

}
```

订单实体 bean

```

﻿...

@Entity

@Table(name="or_order")

public class Order {

    ...

    @ManyToOne

    Customer customer;

}
```

由于`@OntToMany`和`@ManyToOne`都映射的是一个双向的关系，你可以在任意对象上申明。

## 单向关系
如果要将双向关系转化为单向关系，你需要删除`@OneToMany`(Customer.orders 属性)或者`ManyToOne`(Order.customer  属性)之一。

### 删除`OneToMany`- 没问题
例如：从 Custom 删除订单列表 通常你删除`OneToMany`这一方是没有任何问题的。问题是你不能浏览这个方向的对象图。 为什么要删除一对多?有时候使用一对多对应用没有任何帮助甚至是很危险的时候。 例如，产品有一个列表，这个列表是没有用的，或者如果使用它做导航是十分危险的话（并且对于所有订单细节，产品都会采用懒加载模式），这时就应该删除一对多。

### 删除`ManyToOne`- 注意这写插入问题
例如：从订单中删除客户。 如果你删除一个多对一，需要注意如何保存 bean（尤其是插入）。究其原因是因为它是持有外键列的多对一的（导入）一边（例如 or_order 表包含 customer_id 列）。 Q:如果从 Order 对象中删除 Customer 属性，那么当用户创建了一个新订单是你要如何才能创建一个新订单？从数据库层面讲，在插入一个订单的时候 customer_id 是如何填写进去的？ A:你必须使用级联来保存 Customer 中的 customer.orders,这听起来很痛苦，我们一起来看看在现实情况下删除一个多对一。 例如：从订单中删除订单详情 当你从 OrderDetail bean 中删除 Order 属性。现在需要你写一些代码，给订单添加一个订单详情(插入)，你该如何去做？

在`OneToMany`这边开展级联操作

```
@Entity
@Table(name="or_order")
public class Order {
  ...
  // must cascade the save
  @OneToMany(cascade=CascadeType.ALL)
  List<OrderDetail> details;
```

并保存订单，省略级联细节

```
// create or fetch the order
Order order = ...
List<OrderDetail> details = new ArrayList<OrderDetail>();
OrderDetail orderDetail = ...
details.add(orderDetail);

// set the new details...
order.setDetails(details);

// save the order... which cascade saves
// the order details...
Ebean.save(order);
```

因此，当订单保存的时候，因为`@OneToMany`的关系具有cascade.ALL保存级联到所有订单的详细信息。 **注意**你可以单独跟新 OrderDetail(不依赖于级联保存),但是插入一个新的 OrderDetail 时候需要依赖于级联保存。

> 删除多对一通常体现出强烈的"所有权"的关系。该命令"拥有"订单明细，它们通过级联保持一致。

### 管理关系 = `@OneToMany` + 级联保存
如果级联保存是一个`@OneToMany`，当保存从"主干"分解到"细节"的时候，Ebean 将管理"关系"。 举个例子，通过 Order - OrderDetail 的关系，当你保存 Order 的时候，Ebean 会得到 Order id ，并且会确保它被正确设置到 OrderDetail 中去。无论是单向还是双向关系，Ebean 都会这样去做。 这意味着，如果你的 OrderDetail 有一个 Order 属性（双向的）,当您使用级联保存时，你不需要对每一个的OrderDetail设置顺序。 当你保存订单并且落实到级联的时候，Ebean 会为每一个 detail 自动设置"主" 订单。

### `@OneToMany`注释
一般当你为`@OneToMany`指定`mappedBy`属性的时候，意味着这是一个双向关系，并且"join"信息是从关系的另一侧读取(这意味着你不需要在这一端指定任何`@JoinColumn`)。 如果你没有`mappedBy`属性(在其他相关 bean 上没有匹配属性)，那么这是一个单向关系，在这种情况下，你可以指定一个

```
@JoinColumn //if you wish to override the join column information from the default).
@Entity
@Table(name="s_user")
public class User implements Serializable {

  // 单向关系
  // … can explicitly specify the join column if needed
  @OneToMany
  @JoinColumn(name="pref_id")
  List<Preference> preferences;

  // 双向关系
  // … join information always read from the other side
  @OneToMany(mappedBy="userLogged")
  List<Bug> loggedBugs;
```

### `@OneToOne`关系
一个`@OneToOne`关系与`@OneToMany`基本上是一致的，除了 many 那侧被限制为一个。 这意味着在`@OneToOne`一侧的操作就像`@ManyToOne`("入口"端的外键列)，另一侧`OneToOne`的操作就像`@OneToMany`("出口"端)。 所以你可以像`@OneToMany`一样将`mappedBy`放在"出口"端。 从数据库角度来说，一对一的关系实现外键约束（如一对多），并加入外键列的唯一约束来实现。这具有限制"多"侧为最大值1的效果（必须是唯一的）。

### 多对多关系
你可能知道，在数据库物理设计是没有多对多关系的。这些都是与中间表和两个一对多关系来实现。 我们看下面的例子：
- 一个用户可以有多个角色
- 一个角色可以分配给多个用户
- 用户与角色之间是多对多关系

![](http://ebean-orm.github.io/images/docs/A_Many_to_Many_between_user_and_role.png)

在数据库图表上有一个叫做s_user_role一个中间表。这表示用户与角色之间合乎逻辑的多对多关系。 Q：什么时候多对多关系最好表示为两个一对多关系？ A：如果在中间表中有其他列，你需要考虑将多对多关系转为两个一对多关系。 这种方式就是每个`@ManyToMany`操作就像它是一个`OneToMany`。必须管理这种关系，意味着 Ebean 必须注意像中间表插入、删除数据。 其工作原理是，会关注到任何可增加或可删除的 `list/set/map`，并把这些从中间表插入/删除。

```
@Entity
@Table(name="s_user")
public class User implements Serializable {
  ...
  @ManyToMany(cascade=CascadeType.ALL)
  List<Role> roles;

@Entity
@Table(name="s_role")
public class Role {
  ...
  @ManyToMany(cascade=CascadeType.ALL)
  List<User> users;
```

中间表表名和外键列可以是默认的，也可以通过`@JoinTable`等指定。 下面的代码展示了为用户增加一个新角色。

```
User user = Ebean.find(User.class, 1);
List<Role> roles = user.getRoles();
Role role = Ebean.find(Role.class, 27);

// adding a role to the list...this is remembered and will
// result in an insert into the intersection table
// when save cascades...
roles.add(role);

// save cascades to roles... and in this case
// results in an insert into the intersection table
Ebean.save(user);
```

**注意**如果一个角色被从列表中删除，这将导致一个相应的从中间表中删除。

## ID 生成
* DB 标识/自增长
* DB 序列
* UUID
* 自定义 ID 生成
有4种方法可以为实体自动生成 ID。这种情况会在插入一个实体并且该实体ID 没有值的时候发生。
强烈建议使用前3种方式，有以下两个原因：
1. 他们是标准方法，也就是说，如果你选择一个自定义ID生成那么这可以使它更难以用其他程序/工具插入到数据库。
2. 它们支持并发好 - 你真的可以做的更好？大多数数据库支持序列或标识/自动增量。 DB2和H2支持。

### UUID 生成
要使用 UUID 与 Ebean 所有你需要做的是使用你的ID属性的UUID类型。
Ebean会自动分配一个合适的UUID ID生成。
```
@Entity
public class MyEntity {
@Id
UUID id;
...
```

### DB 序列/DB 自增长
参考：`com.avaje.ebean.config.dbplatform.DatabasePlatform`和`com.avaje.ebean.config.dbplatform.DbIdentity`
```
@Entity
public class MyEntity {
@Id
Integer id;
...
```
