# `OneToMany`

## 关系对

大多数情况下我们可以认为`@OneToMany`和`@ManyToOne`作为代表关系两侧的一对。因此`@OneToMany`有关系的"many"方和`@ManyToOne`有关系的"one"方。

## mappedBy

`mappedBy`属性大多数应该在`@OneToMany`定义。所述的mappedBy属性有效地指向关系的另一侧。

**Example: Customer has many Contacts**

```java
@Entity
public class Customer ...

  // mappedBy referring to the other side of this relationship
  @OneToMany(mappedBy="customer")
  List<Contact> contacts;
  ...
```

在关系对的另一边`Contact`实体定义了的`mappedBy`是被指向`@ManyToOne`属性

```java
@Entity
public class Contact ...

  // the customer property referred to by @OneToMany(mappedBy="customer")
  @ManyToOne(optional=false)
  Customer customer;
  ...
```

## 双向关系

当我们用@OneToMany和@ManyToOne一对的关系的两侧映射我们可以描述这是一个双向的关系。从关系的两端都可以查看，装载，导航到对方。

> 我们有一个双向的关系时，没有限制或约束时。

## 不映射`@OneToMany`

当关系对中`@OneToMany`这侧的基数过大时(好几千)，如果映射的话，会加载过多的对象，我们不应该允许应有去浏览这样的关系对。

举个例子，当一个顾客又上百万的订单的时候，我们定位到顾客的时候，会有上百万的订单示例被加载到内存中，通常我们不想这样去做。一般情况我们会根据一些条件查询相应的订单(例如一周以内的订单)。另外，我们可以从另一侧的对象图加载(从订单到客户)。

实际上省略了`@OneToMany`意味着，我们无法定位或加载从这个方向的关系，而不是应用程序将其强制始终使用另一个方向（建对象图）。

## 不映射`@ManyToOne`

当我们不映射`@ManyToOne`的时候会增加一些限制并且其中暗藏着一个所有权关系。

如下的例子`Order`会又许多`OrderDetail`，当我们在`OrderDetail`中不映射`@ManyToOne`的时候：

### 订单有详情

```java
@Entity
@Table(name = "o_order")
public class Order ...

  // we MUST have cascade persist here for this
  // unidirectional case (no @ManyToOne)
  @OneToMany(cascade = CascadeType.ALL)
  List<OrderDetail> details;
  ...
```

### 订单详情没有对应的`@ManyToOne`

```java
@Entity
public class OrderDetail ...

  // Does not have - @ManyToOne(optional = false) Order order;
```

这会有效的限制在插入一个新的`OrderDetail`的时候我们必须从`Order`使用级连操作。也就是说`@ManyToOne`会限制`OrderDetail`的外键，我们必须使用Order的级连才能插入外键列。

这也意味着，外键值永远无法改变，因此我们可以在概念上认为这是一个'所有权'的关系。在这个例子中每一个的`OrderDetail`是由`Order`拥有的。

> 就我个人而言，我总会映射`@ManyToOne`
