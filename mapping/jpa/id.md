# `@Id`

## Enhancement

增强方面`@Id`属性会被区别对待，它们从来不会调用延迟加载。

## 生成值

当对一个数字类型或者`UUID`类型的属性加入`@Id`注解，Ebean会自动分配一个`Id generator`属性。 意味着Ebean会冗余的添加`@GeneratedValue`注解

```java
@Id
Long id;

// ... is effectively the same as

@Id @GeneratedValue
Long id;
```

## UUID

如果`@Id`类型为`UUID`，则Ebean会自动分配一个合适的ID生成到该属性。

## 数据库平台

所有支持`Identity`或`Sequence`(或同时支持)的数据库,Ebean都会基于数据库选择合适的Id生成策略

数据库         | 策略
----------- | ---------------------------------------------
H2          | Sequences
Postgres    | Identity (as serial, also supports sequences)
MySql       | Identity
Oracle      | Sequences
DB2         | Identity (also supports sequences)
SQL Server  | Identity
SQLite      | Identity
SqlAnywhere | Identity

## 自定义ID生成

Ebean支持注册和使用自定义ID

**1\. 实现`com.avaje.ebean.config.IdGenerator`**

```java
public class ModUuidGenerator implements IdGenerator {

  @Override
  public Object nextValue() {
    return ModUUID.newShortId();
  }

  @Override
  public String getName() {
    return "shortUid";
  }
}
```

**2\. 通过`ServerConfig`注册** 通过`ServerConfig`调用`addClass()`或者`add(IdGenerator idGenerator)`或者`setIdGenerators(List idGenerators)`注册`IdGenerator`

**注意：如果是通过classpath 扫描发现实体Bean，也会自动寻找`IdGenerator`的实现，并注册，在这种情况下，你不需要通过`ServerConfig`手动注册**

**3\. `@GeneratedValue`** 之后我们可以通过`@GeneratedValue`注解告诉eban使用自定义的`IdGenerator`

```java
@Id @GeneratedValue(generator = "shortUid")
String id;
```
