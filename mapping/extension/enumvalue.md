# 使用

标准的JPA对于枚举类型支持有限。使用`EnumType.ORDINAL`值的时候是很危险的，应为其依赖于枚举元素的顺序不会改变。使用`EnumType.STRING`枚举值的名称。

使用`@DbEnumValue`,我们需要在用于枚举值映射到数据库中的一个值的方法上加上注解：

```java
public enum Status {

  NEW("N"),
  ACTIVE("A"),
  INACTIVE("I");

  String dbValue;
  Status(String dbValue) {
    this.dbValue = dbValue;
  }

  // annotate a method that returns the value
  // in the DB that the enum element maps to

  @DbEnumValue
  public String getValue() {
    return dbValue;
  }
}
```

## Storage

使用`storage`属性来指定枚举值与数据库类型的映射，实际上，我们当我们通常使用要映射枚举值到数据库INTEGER类型。

```java
public enum Status {
  NEW("1"),
  ACTIVE("2"),
  INACTIVE("3");

  String value;
  Status(String value) {
    this.value = value;
  }

  // map to DB INTEGER
  @DbEnumValue(storage = DbEnumType.INTEGER)
  public String getValue() {
    return value;
  }
}
```
