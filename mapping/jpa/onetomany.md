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
