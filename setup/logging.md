Ebean 使用 [SL4J](http://www.slf4j.org/) 记录日志

## SQL 和 事务日志
在开发过程中，期望你包含`org.avaje.ebean.SQL`及`org.avaje.ebean.TXN`的日志.这些日志语句执行和事务划分执行的SQL语句，绑定值。

```xml
<!-- LOGBACK configuration -->

<!-- SQL and bind values -->
<logger name="org.avaje.ebean.SQL" level="TRACE"/>

<!-- Transaction Commit and Rollback events -->
<logger name="org.avaje.ebean.TXN" level="TRACE"/>
```

## 概要日志

`org.avaje.ebean.SUM`的概要日志，对于显示延迟加载查询以及返回查询来源很有用。在更复杂的对象图看都建时，在调优查询为N + 1等，这是非常有用的。

```xml
<!-- LOGBACK configuration -->

<!-- Summary level details -->
<logger name="org.avaje.ebean.SUM" level="TRACE"/>

```

## L2 缓存日志
L2高速缓存事件可以使用下面的记录项进行记录。当你开始了利用二级缓存，并寻找与L2缓存的行为，非常有用。

```xml
<!-- LOGBACK configuration -->

 <!-- L2 logging -->
<logger name="org.avaje.ebean.cache.QUERY" level="TRACE"/>
<logger name="org.avaje.ebean.cache.BEAN" level="TRACE"/>
<logger name="org.avaje.ebean.cache.COLL" level="TRACE"/>
<logger name="org.avaje.ebean.cache.NATKEY" level="TRACE"/>
```
