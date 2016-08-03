# 默认构造

如果在实体 Bean 中没有构造函数，那么 Ebean 增强会 **自动添加一个默认构造函数**

## 例子－Customer, No default constructor

在下面的例子中 `Customer`实体 Bean 没有默认的构造函数 但是 Eban 增强会自动的添加构造函数。

```java
/**
 * Customer entity bean with no default constructor.
 */
@Entity
public class Customer {

  public static final CustomerFinder find = new CustomerFinder();

  @NotNull @Size(max = 100)
  String name;


  //如果没有默认的构造函数 ebean 增强会自动添加一个默认的无参构造函数

  public Customer(String name) {
    this.name = name;
  }

   // getters, setters etc
```

## 局部对象

注意，如果你愿意，你可以让字段是`final`的，而且不会限制读取 customer bean 的部分属性(没有name属性)或者创建一个依赖Bean(即只加载自己的ID属性)

```java
// fetch partially populated beans that
// don't have the name property loaded
List<Customer> customers =
  Customer.find.where()
    .name.startsWith("Rob")
    .select("id") // only select id property
    .findList();
```

## 引用 bean

Ebean 可以构造出只加载`@Id`属性的引用bean

```java
// reference bean with only @Id property loaded
Customer refBean = Customer.find.ref(42);

// reference beans don't hit the database unless
// lazy loading is invoked

// invoke lazy loading as name property is not loaded
String name = refBean.getName();
```

## 示例－Country，没有setter

如下所示，Country 实体 Bean 只有两个参数并且通过构造函数set值，在这种情况下，实体bean里没有 setter 函数。

```java
/**
 * Country entity bean with no default constructor and
 * no setters (only getters).
 */
 @Entity
 public class Country extends Model {

   public static final CountryFinder find = new CountryFinder();

   @Id @Size(max = 2)
   final String code;

   @NotNull @Size(max = 60)
   final String name;

   // enhancement will add a default constructor

   public Country(String code, String name) {
     this.code = code;
     this.name = name;
   }

   // getters only for Country bean

   public String getCode() { return code; }

   public String getName() { return name; }

 }
```

Ebean同样支持实体Bean和关联Bean

```java
// insert a country
new Country("NZ", "New Zealand").save();

// reference bean (only has @Id property loaded)
Country nzRefBean = ebeanServer.getReference(Country.class, "NZ");

// reference bean using a "finder"
Country nzRefBean = Country.find.ref("NZ");
```

## Kotlin

### 通过 kotlin 方式构建 country

```kotlin
@Entity
@Table(name = "o_country")
class Country (

    @Id @Size(max = 2)
    var code: String, // non-null type

    @NotNull @Size(max = 60)
    var name: String  // non-null type

) : Model() {

  override fun toString(): String {
    return "code:$code name:$name";
  }

  companion object : CountryFinder() {}
}
```

```kotlin
Country("NZ", "New Zealand").save()

// reference bean
val nzRef = Country.ref("NZ")

// finder & query bean use
val nz = Country.where()
    .code.equalTo("NZ")
    .findUnique()
```

### 使用kotlin写Customer

```kotlin
@Entity
@Table(name = "be_customer")
class Customer(

  @NotNull @Size(max = 100)
  var name: String,  // non-null type

  @SoftDelete
  var deleted: Boolean = false,

  var registered: Date? = null,

  @Size(max = 1000)
  var comments: String? = null,

  @ManyToOne(cascade = arrayOf(CascadeType.ALL))
  var billingAddress: Address? = null,

  @ManyToOne(cascade = arrayOf(CascadeType.ALL))
  var shippingAddress: Address? = null,

  @OneToMany(mappedBy = "customer", cascade = arrayOf(CascadeType.PERSIST))
  var contacts: MutableList<Contact> = ArrayList()

) : BaseModel() {

  companion object find : CustomerFinder() {}

  override fun toString(): String {
    return "customer(id:$id name:$name)";
  }
}
```

```
// new customer (requires name)
val rob = Customer("Rob")
rob.save()

// reference bean
val refBean = Customer.ref(42L)

// finder / query bean use
val customers =
  Customer.find.where()
    .name.istartsWith("rob")
    .findList()
```
