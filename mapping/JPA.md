# JPA 映射

## 当前限制

如下的一些JPA映射暂时不被Eban支持，但以后会支持。下列是一些记录作为增强请求

issues                                               | info
---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------
[107](https://github.com/ebean-orm/ebean/issues/107) | JPA2 - Add support for @PrimaryKeyJoinColumn/@PrimaryKeyJoinColumns.
[115](https://github.com/ebean-orm/ebean/issues/115) | JPA2 - Add support for @ElementCollection. Note that Ebean does have support for @DbArray (Postgres ARRAY) and @DbJson.
[116](https://github.com/ebean-orm/ebean/issues/116) | Only single table inheritance is supported. This enhancement request is to add support for JOINED or TABLE PER CLASS inheritance strategies.
[123](https://github.com/ebean-orm/ebean/issues/123) | Add support for OneToMany JoinTables.
