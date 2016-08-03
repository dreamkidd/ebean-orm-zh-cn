## JPA 映射

Ebean 遵循 JAP 的映射规范，你可以学习并使用相同的映射注解。一般而言这是一个很好的规范，并且我期望这部分规范能经得起时间的考验。
一下是一些 JPA的不足

属性 | 特性
--- | ---
命名惯例 | Ebean 遵循命名惯例 API，这意味着通常我们只需要在没有遵循命名惯例的字段上使用`@Column name`注解。同样一般不要求在`@JoinColumn` 或`@JoinTable`中对遵循命名规则的列、表指定name（默认使用下划线命名转换）
`@Id` | Ebean 假定数据库ID生成是首选，并自动映射到任何身份或基于数据库平台序列的支持。由于没有额外的注解，这提供了良好的跨数据库的支持
枚举 | JPA 对于枚举的支持较差。Ebean 提供了两种更好的替代品`@DbEnumValue` 和 `@EnumValue`
FetchType EAGER / LAZY | JPA 映射建议使用`FetchType.EAGER`和`LAZY`,这与Ebean的对每个查询方法进行优化的查询方式相反(并通过分析应用程序提供自动查询调优).当使用Ebean时，使用 EAGER LAZY 并不是十分的有效。

## `@Size` 和 `@NotNull`

Ebean 支持使用 `javax validation`的`@size`和`@NotNull`注解

注解 ｜ 说明
--- | ---
`@Size` | 定义映射列的长度。例如：`@Size(50)`与`@Column(length=50)`一样
`@NotNull` | 定义映射列值不能为空。同`@Column(nullable = false)`或者`@ManyToOne(optional=false)`
