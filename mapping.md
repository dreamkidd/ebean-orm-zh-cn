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
