# 增强
## 概览
术语"Enhancement"涵盖了用于修改实体 Bean 的所有方法（javaagent,ant,maven,ide 插件等）。 类似"Weaving","Transformation"以及"byte code manipulation"等术语用于描述增强。

可以通过`javaagent`在类加载是操作类使得类得以增强，而一般通过`maven`,`ant`等，则是在构建期间增强字节码

在原生层面，一个类是一个 `byte[]`，增强的含义是通过其类加载器加载类然后操作其字节。

## 使用多个增强
Ebean Enchancement 知道增强是何时发生的。一般情况下，是使用多种形式的增强，如 IDE，Maven 插件等来同时执行增强。

## 建议
对于 IDEA 及 Eclipse 用户建议使用 IDE 插件来增强，在构建过程中，使用 maven 插件或者代理来执行增强。 也就是说,IDE 插件适用于在 IDE 编译 classes 时增强 beans。在开发，单元测试过程中可以减少一些麻烦。 对于生成环境，则可以选择使用 maven 插件在构建时增强，或者使用 javaagent 在运行时增强。

## IDEA 插件
安装 IDEA 插件介绍 [YOUTOBE](https://youtu.be/o4kmglM48Vc)

## Eclipse 插件
安装 Eclipse 插件,地址`http://ebean-orm.github.io/eclipse/update` [YOUTOBE](https://youtu.be/_DWxNj-_orA)

## Maven 增强
一个Maven插件进行增强作为Maven的编译过程的一部分，这一过程成为构建时增强。

### Maven tile
Ebean 提供了一个 maven tile ,同时提供了对于实体 bean以及`query bean`的增强。 这比传统的方式更加简洁，方便，是最推荐的方式

**ebean-enhancement tile**

```xml
<plugin>
  <groupId>io.repaint.maven</groupId>
  <artifactId>tiles-maven-plugin</artifactId>
  <version>2.8</version>
  <extensions>true</extensions>
  <configuration>
    <tiles>
       <!--
           Leave out java-compile tile if you already
           have the compile plugin defined
        -->
      <!-- <tile>org.avaje.tile:java-compile:1.1</tile> -->
      <tile>org.avaje.tile:ebean-enhancement:1.1</tile>
    </tiles>
  </configuration>
</plugin>
```

使用`mvn help:effective-pom`来查看该 tile 提供的插件.

### Maven 插件
如果想使用传统的 maven 插件方式而不是使用 maven tile 的方式，需要注意一下方法并不提供`query Bean Enhancement`

```xml
<plugin>
  <groupId>org.avaje.ebeanorm</groupId>
  <artifactId>avaje-ebeanorm-mavenenhancer</artifactId>
  <version>4.10.1</version>
  <executions>
    <execution>
      <id>main</id>
      <phase>process-classes</phase>
      <configuration>
        <transformArgs>debug=1</transformArgs>
      </configuration>
      <goals>
        <goal>enhance</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

## 代理
可以在命令使用如下所示的命令在命令行中通过`javaagent`参数来使用增强，这被称为`运行时增强`或`加载增强`。

注意:从avaje-ebeanorm-agent的4.7.1版本开始 可以不指定包名。意味着 agent 会自动排除其不感兴趣的类(jdk, groovy, scala, kotlin, jdbc drivers 以及一些常见的 apache, google 公共类库，及测试等类).

```bash
java -javaagent:avaje-ebeanorm-agent-4.10.1.jar MyApplication
java -javaagent:avaje-ebeanorm-agent-4.10.1.jar=debug=3 MyApplication
java -javaagent:avaje-ebeanorm-agent-4.10.1.jar=packages=org.example.model.** MyApplication
java -javaagent:avaje-ebeanorm-agent-4.10.1.jar=debug=1;packages=org.example.model.**,org.example.model2.**
MyApplication
```

avaje-ebeanorm-agent jar 可以从 maven 中下载:
```xml
<dependency>
  <groupId>org.avaje.ebeanorm</groupId>
  <artifactId>avaje-ebeanorm-agent</artifactId>
  <version>4.10.1</version>
</dependency>
```

## Agent Loader
The agent loader can be used to programmatically load the javaagent onto a running JVM. However, this very occasionally does not enhance a bean as an entity bean class somehow gets loaded prior to the agent being loaded on the JVM and for this reason this is not a recommended approach.

Note that this approach is used in the testing code for Ebean itself and if you use this approach you need to make sure the agent loading occurs before any class loading of entity beans or transactional beans occurs.

```xml
<dependency>
  <groupId>org.avaje</groupId>
  <artifactId>avaje-agentloader</artifactId>
  <version>2.1.2</version>
</dependency>
```

The code below loads the enhancer agent programmatically onto the running JVM.
```java
import org.avaje.agentloader;
...
public void someApplicationBootupMethod() {
  // Load the agent into the running JVM process
  if (!AgentLoader.loadAgentFromClasspath("avaje-ebeanorm-agent","debug=1;packages=org.example.model.**")) {
    logger.info("avaje-ebeanorm-agent not found in classpath - not dynamically loaded");
  }
}
```

## Ant 增强

修改您的`build.xml`:

1. 定义一个AntEnhanceTask；
2. 创建使用AntEnhanceTask增强实体类的目标。

```xml
<taskdef
    name="ebeanEnhance"
    classname="com.avaje.ebean.enhance.ant.AntEnhanceTask"
    classpath="your_path_to/ebean-x.x.x.jar"/>

<target name="ormEnhance" depends="clean,compile">
  <!--
  classSource: This is the directory that contains your class files. That is, the directory where your IDE will compile your java class files to, or the directory where a previous ant task will compile your java class files to.

  packages: a comma delimited list of packages that contain entity classes. All the classes in these packages are searched for entity classes to be enhanced. transformArgs: This contains a debug level (0 - 10) .
  -->
  <ebeanEnhance
      classSource="your_classes_directory"
      packages="app.domain.*"
      transformArgs="debug=1"/>
</target>
```
