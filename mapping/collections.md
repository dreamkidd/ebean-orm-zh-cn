# List VS Set

对于有关`@OneToMany`和`@ManyToOne`属性的集合映射，我们需要选择使用`List`或者`Set`集合。

## Hibernate bag 语义

Hibernate 有可能更偏好使用`Set`,应为在 Hibernate 中`List`和`Set`有不同的语义。使用 bag 语义的`Set`往往是首选。

## `hashCode() / equals()`

使用Set意味着需要实现`hashCode() / equals()`方法。这会又一个问题，在实体Bean被插入之前，`@Id`属性通常都是没有值的并且实体Bean的属性是变异的。

不好实现`hashCode() / equals()`意味着`List`更适合处理`@OneToMany`和`@ManyToMany`集合数据

```java
@Entity
@Table(name="be_customer")
public class Customer extends BaseModel {
  ...

  // List is recommended for collection types

  @OneToMany(mappedBy="customer", cascade=CascadeType.PERSIST)
  List<Contact> contacts;
  ...
```

# Enhancement

当你定义了一个集合的时候，Eban增强将会确保：

- 移除所有`List/Set`初始化
- `List/Set`统一有Ebean初始化(并且不为空)

## 移除所有`List/Set`初始化

```java
// initialisation of the new ArrayList() is removed

@OneToMany(mappedBy="customer")
List<Contact> contacts = new ArrayList<Contact>;

// you can declare an un-initialised List if you wish
// and there is no actual difference to an initialised one
// because enhancement will always initialise it

@OneToMany(mappedBy="customer")
List<Contact> contacts;
```

在 kotlin 中我们一般定义其为非空

```java
// kotlin: contacts type not nullable
@OneToMany(mappedBy = "customer")
var contacts: MutableList<Contact> = ArrayList()
```

## `List/Set`统一有Ebean初始化(并且不为空)

Ebean 会控制`List/Set`初始化以支持一下特性：

- 懒加载
- 支持`@PrivateOwned`需要在`List/Set`需要明确的清除量
- 支持`@ManyToMany`需要的`List/Set`要注意增加和清除

增强替换所有的持久化集合的getfield命令指示，并替换与保证`List/Set`被初始化。

```java
@OneToMany(mappedBy="customer")
List<Contact> contacts;

public void addContact(Contact contact) {
  // this is safe to write as contacts
  // will never be null here
  contacts.add(contact);
}
```
