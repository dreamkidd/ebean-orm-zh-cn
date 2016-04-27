# 事务
通过事务，可以控制 JDBC 批处理（batch size，flushing），控制事务日志开关，打开、关闭储存与删除的级联操作等工作

## 隐式
### 隐式创建事务
如果当前没有申明事务，任然可以创建，保存或删除一个 bena。在这种情况下，Ebean 会隐式的创建一个事务并提交（或者在失败时回滚）。

```java
// execute a query without an existing transaction...
// ... a "implicit" transaction will be created
// ... and be commited (or rolled back) for you
List<User> users =
            Ebean.find(User.class)
                .join("customer")
                .where().eq("state", UserState.ACTIVE)
                .findList();

// execute a save without an existing transaction...
// ... will create an implicit transaction
// ... and be commited (or rolled back) for you
Ebean.save(user);
```

下面的 SL4J 日志显示隐式事务的查询创建任务。

```java
16:38:26.362 [main] DEBUG c.a.e.server.lib.thread.ThreadPool - ThreadPool grow created [Ebean-h2.0] size[0]
16:38:26.369 [main] TRACE org.avaje.ebean.TXN - txn[1002] Begin
16:38:26.548 [main] INFO  org.avaje.ebean.TXN - txn[1002] Commit
16:38:26.551 [main] TRACE org.avaje.ebean.TXN - txn[1003] Begin
16:38:26.554 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from or_order_ship
16:38:26.555 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[or_order_ship] rows[0] bind[null]
16:38:26.556 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_order_detail
16:38:26.557 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[o_order_detail] rows[0] bind[null]
16:38:26.557 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_order
16:38:26.557 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[o_order] rows[0] bind[null]
16:38:26.557 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from contact
16:38:26.557 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[contact] rows[0] bind[null]
16:38:26.557 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_customer
16:38:26.557 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[o_customer] rows[0] bind[null]
16:38:26.557 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_address
16:38:26.558 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Delete table[o_address] rows[0] bind[null]
16:38:26.563 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_country
16:38:26.563 [main] DEBUG org.avaje.ebean.SUM - txn[1003] DeleteSql table[o_country] rows[0] bind[null]
16:38:26.563 [main] TRACE org.avaje.ebean.SQL - txn[1003] delete from o_product
16:38:26.564 [main] DEBUG org.avaje.ebean.SUM - txn[1003] DeleteSql table[o_product] rows[0] bind[null]
16:38:26.567 [main] TRACE org.avaje.ebean.SQL - txn[1003] insert into o_country (code, name) values (?,?); --bind(NZ,New Zealand,)
16:38:26.568 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Inserted [Country] [NZ]
16:38:26.568 [main] TRACE org.avaje.ebean.SQL - txn[1003] insert into o_country (code, name) values (?,?); --bind(AU,Australia,)
16:38:26.568 [main] DEBUG org.avaje.ebean.SUM - txn[1003] Inserted [Country] [AU]
16:38:26.570 [main] TRACE org.avaje.ebean.SQL - txn[1003] insert into o_product (id, sku, name, cretime, updtime) values (?,?,?,?,?); --bind
```

## 编程
### 在 try finally 块中使用 begin，commit等
一个声明事务的传统方式，通过 try finally 块

```java
ebeanServer.beginTransaction();
try {
 // fetch some stuff...
 Customer customer = ebeanServer.find(Customer.class, id);
 ...

 // save or delete stuff...
 ebeanServer.save(customer);
 ...

 // commit the transaction
 ebeanServer.commitTransaction();

 finally {
 // rollback the transaction if it was not committed
 ebeanServer.endTransaction();
}
```

## 在事务中控制行为
你可以使用事务明确控制行为.
```java
Transaction transaction = ebeanServer.beginTransaction();
try {
  // turn of cascade persist
  transaction.setCascadePersist(false);

  // control the jdbc batch mode and size
  // transaction.setBatchMode(true); // equivalent to transaction.setBatch(PersistBatch.ALL);
  // transaction.setBatchMode(false); // equivalent to transaction.setBatch(PersistBatch.NONE);
  transaction.setBatch(PersistBatch.ALL); // PersistBatch: NONE, INSERT, ALL
  transaction.setCascadeBatch(PersistBatch.INSERT); // PersistBatch: NONE, INSERT, ALL
  transaction.setBatchSize(30);


  // for a large batch insert if you want to skip
  // getting the generated keys
  transaction.setBatchGetGeneratedKeys(false);

  // for batch processing via raw SQL you can inform
  // Ebean what tables were modified so that it can
  // clear the appropriate L2 caches
  String tableName = "o_customer";
  boolean inserts = true;
  boolean upates = true;
  boolean deletes = false;
  transaction.addModification(tableName, inserts, updates, deletes);

  ...
} finally {
  transaction.end();
}
```

## @Transactional
必须使用 `ENHANCEMENT` 才能使用 `@Transactional`生效，也就是说你必须通过 IDE，Ant 任务或者javaagent 增强 class。更多细节参考usergide 的enhancement。
