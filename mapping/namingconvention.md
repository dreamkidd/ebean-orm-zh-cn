# 命名惯例
Ebean 有一个命名约定 API 以映射列名与属性名。同时也将实体（entity）与 数据库表名映射起来，如果需要的话，也可以顾及数据库 scheme 和目录。

参见`com.avaje.ebean.config.NamingConvention`

默认的`UnderscoreNamingConvention` 会将数据库中的`_`命名映射为一般的 java 的驼峰命名。（如:"first_name"映射为"firstName"）

```java
@Entity
@Table(name="be_contact")
public class Contact extends BaseModel {

  ...
  // the default underscore naming convention maps
  // "firstName" property to column "first_name"
  // ... so this @Column annotation is not required
  @Column(name="first_name")
  String firstName;
```

也可以使用`MatchingNamingConvention`或者自己实现命名惯例。

`matchingnamingconvention`命名列名称匹配的属性名和表名匹配实体名称。ebean遵循JPA规范指出，在没有注释的类的名称将作为表名和属性名的情况将作为该列的名称。
