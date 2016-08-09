# 关系对

我们可以认为`@ManyToOne` 是`@OneToMany`/`@ManyToOne`关系对中'一'的一方。

因此他是映射到外键列的一侧。

```java
@Entity
public class Contact ...

  // maps to "customer_id" foreign key column
  @ManyToOne
  Customer customer;
  ...
```

如果外键列的列名不是**属性名＋_id**的组合，那么我们需要定义一个`@JoinColumn`

```java
@Entity
public class Contact ...

  // explicit @JoinColumn of "cust_id" as the foreign key column
  @ManyToOne @JoinColumn("cust_id")
  Customer customer;
  ...
```

## optional=false

如果底层外键有非空约束，那么我们需要声明`optional=false`

```java
@Entity
public class Contact ...

  // not null constraint as optional=false
  @ManyToOne(optional=false)
  Customer customer;
  ...
```

## `@NotNull`

同样可以使用javax validation 的`@NotNull`替代`optional=false`

```java
@Entity
public class Contact ...

  // @NotNull same as optional=false
  @NotNull @ManyToOne
  Customer customer;
...
```
