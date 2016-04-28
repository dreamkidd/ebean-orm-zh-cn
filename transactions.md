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

```
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

Ebean 可以为 pojo 类添加事务支持。你可以将`@Transactional`注解置于一个方法或类上面，Ebean 会加入事务逻辑支持(开始事务，提交，回滚，暂停和恢复交易等)。

```java
...
// any old pojo
public class MyService {

    @Transactional
    public void runFirst() throws IOException {

        // run in a Transaction (REQUIRED is the default)

        // find a customer
        Customer customer = Ebean.find(Customer.class, 1);

        // call another "transactional" method
        runInTrans();
    }

    @Transactional(type=TxType.REQUIRES_NEW)
    public void runInTrans() throws IOException {

        // run in its own NEW transaction
        // ... suspend an existing transaction if required
        // ... and resume it when this method ends

        // find new orders
        List<Order> orders = Ebean.find(Order.class)
                                    .where().eq("status",OrderStatus.NEW)
                                    .findList();

...
```

将`@Transactional`置于方法上，Ebean会增强类添加事务管理。

## 标准事务范围
支持标准的REQUIRED (默认), REQUIRES_NEW, MANDATORY, SUPPORTS, NOT_SUPPORTS, NEVER规则。这些完全匹配 EJB 的 TransactionAttributeTypes。

## 嵌套事务
事务可以被嵌套，就像上述的例子一样，`runFirst()`中调用了`runInTrans()`。

## 隔离级别并支持具体异常
`@Transactional`（比如Spring）支持隔离级别和异常的显式处理(回滚或不特定类型的异常)。请参考用户指南获取更详细的解释。

## 接口
你可以将`@Transactional`置于接口上，那么其实现类将会被 Ebean 添加事务管理。

# `TxRunnable` / `TxCallable`
`TxRunnable` 和 `TxCallable` 几乎等同于`@Transactional`的纲领性。 如果你愿意，你可以将 `TxRunnable` 和 `TxCallable` 与`@Transactional`混合使用，他们将正确地工作。

```java
public void myMethod() {
  ...
  System.out.println(" Some code in myMethod...");

  // run in Transactional scope...
  Ebean.execute(new TxRunnable() {
    public void run() {

        // code running in "REQUIRED" transactional scope
        // ... as "REQUIRED" is the default TxType
        System.out.println(Ebean.currentTransaction());

        // find stuff...
        User user = Ebean.find(User.class, 1);
        ...

        // save and delete stuff...
        Ebean.save(user);
        Ebean.delete(order);
        ...
    }
  });

  System.out.println(" more code in myMethod...");
}
```

一般来说，你会像上面一样使用 `TxRunnable`创建一个匿名内部类。 `run()`方法内部的代码将会有一个 Ebean 通过事务传递的事务作用域(就像`@Transactional`)。

```java
// programmatic control over the scope such as
// ... isolation level
// ... and to rollback or not for specific exceptions

TxScope txScope = TxScope
            .requiresNew()
            .setIsolation(TxIsolation.SERIALIZABLE)
            .setNoRollbackFor(IOException.class);

Ebean.execute(txScope, new TxRunnable() {
    public void run() {
        ...
}
```

# Spring
(under construction)
# JTA
(under construction)
