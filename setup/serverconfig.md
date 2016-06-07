## 概览

EbeanService 实例的配置是由`ServerConfig` bean 定义的。它提供了所有可配置选项的 getter 和 setter 方法。

在代码方面，创建一个新的`ServerConfig`实例并且可以通过调用 setter 方法，从 ebean.properties 文件读取配置，或者通过外部属性来进行配置。

最终，`EbeanServerFactory` 会取得 `ServerConfig` 并创建一个 `EbeanServer` 实例。

## ebean.properties

`ServerConfig.loadFromProperties()` 会从`ebean.properties`文件读取配置。意味着同时可以通过代码设置一些属性(在 ServerConfig 中调用 setter 方法),和配置文件设置属性。

```java
ServerConfig config = new ServerConfig();
config.setName("pg");
...
// load configuration from ebean.properties
// using "pg" as the server name
config.loadFromProperties();
...

EbeanServer server = EbeanServerFactory.create(config);
```

## 外部属性

你可以通过读取 `Properties` 并通过 `ServerConfig.loadFromProperties(Properties)` 设置 Properties

```java
// load properties externally
Properties externalProps = ...;

ServerConfig config = new ServerConfig();
config.setName("pg");
...
// load configuration from external properties
// using "pg" as the server name
config.loadFromProperties(externalProps);
...

EbeanServer server = EbeanServerFactory.create(config);
```
