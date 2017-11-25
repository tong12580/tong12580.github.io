title: JDBI -- 简介
thumbnail: http://jdbi.org/images/logo.svg
date: 2017/11/20 19:03:00
tags: 
    - 翻译
    - SQL
    - java
categories:
    - 数据库
---

### 五分钟介绍
JDBI是Java的SQL连接处理的便利工具库。它试图以Java中使用集合，bean等来映射关系来访问关系数据库，同时保持与JDBC相同的粒度。它公开了两种不同风格的API，一种流式风格和一种sql对象风格。

### 流式风格的API
流式风格API 如下所示：

```java

// using in-memory H2 database
DataSource ds = JdbcConnectionPool.create("jdbc:h2:mem:test",
                                          "username",
                                          "password");
DBI dbi = new DBI(ds);
Handle h = dbi.open();
h.execute("create table something (id int primary key, name varchar(100))");

h.execute("insert into something (id, name) values (?, ?)", 1, "Brian");

String name = h.createQuery("select name from something where id = :id")
                    .bind("id", 1)
                    .map(StringMapper.FIRST)
                    .first();
                    
assertThat(name, equalTo("Brian"));

h.close();

```

DBI对象类似于JDBC数据源，并且通常通过传递JDBC数据源来构造。有参构造函数采取JDBC URL和证书，以及其他手段（密码）。从DBI实例获得Handle实例。Handle表示到数据库的单个连接。Handles处理器们依赖于底层的JDBC连接对象。

使用处理器Handles，您可以创建和执行语句，查询，调用，批处理或准备好的批处理。在上面的例子中，我们执行一个语句来创建一个表。执行另一个SQL语句时使用两个占位符参数来插入值。最后，我们构造一个查询，将一个值绑定到查询中的命名参数，将结果映射为String类型，并采取回来的第一个结果。

创建和查询的命名参数设施由JDBI提供，它在实际构建准备好的语句时解析出SQL并使用位置参数。上面的例子使用默认的冒号做分隔器，但也可以使用问号做分割器。其语法类比于使用冒号的数据库，比如PostgreSQL。

### SQL对象风格的API
第二种，SQL对象风格API简化了创建DAO对象的常见方式，在该方法中，单个方法映射到单个语句。SQL对象定义是一个带注释的接口，例如：

```
public interface MyDAO
{
  @SqlUpdate("create table something (id int primary key, name varchar(100))")
  void createSomethingTable();

  @SqlUpdate("insert into something (id, name) values (:id, :name)")
  void insert(@Bind("id") int id, @Bind("name") String name);

  @SqlQuery("select name from something where id = :id")
  String findNameById(@Bind("id") int id);

  /**
   * close with no args is used to close the connection
   */
  void close();
}

```

这个接口定义了两个更新语句，其中创建与api示例中相同的创建表相同，第二个更新执行相同的插入，第三个定义查询。在后面的两个例子中，注意使用绑定限制符来对应相应的参数。

最后的方法close（）是特殊的。当它被调用时，它将关闭底层的JDBC连接。该方法可能被声明为引发异常，例如close（）方法在java.io.Closeable上执行，使其适用于Java 7中的自动资源管理。

为了使用这个sql对象定义，我们使用这样的代码：

```
// using in-memory H2 database via a pooled DataSource
JdbcConnectionPool ds = JdbcConnectionPool.create("jdbc:h2:mem:test2",
                                                  "username",
                                                  "password");
DBI dbi = new DBI(ds);

MyDAO dao = dbi.open(MyDAO.class);

dao.createSomethingTable();

dao.insert(2, "Aaron");

String name = dao.findNameById(2);

assertThat(name, equalTo("Aaron"));

dao.close();
ds.dispose();

```

我们从DBI实例中获得一个sql对象的实例，然后调用它的方法。这里有几种不同的方式创建sql对象实例。这里的一个将对象绑定到一个特定的处理器中，所以我们需要确保在完成对象时关闭对象。

### 获取JDBI

JDBI是根据Apache许可证2.0许可的，没有额外的运行时依赖性。

Maven的
要在基于Apache Maven的项目中使用JDBI ，添加它是一个依赖项：

```xml

<dependency>
  <groupId>org.jdbi</groupId>
  <artifactId>jdbi</artifactId>
  <version>${jdbi.version}</version>
</dependency>

```

您可以在[Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22jdbi%22)中找到最新版本的JDBI 。

#### 下载Jar
JDBI发布jar文件可以从[Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22jdbi%22)获得。

#### 源代码
JDBI的主要存储库位于Github上，网址为http://github.com/brianm/jdbi/

#### 邮件列表
JDBI的邮件列表以https://groups.google.com/group/jdbi上的Google Group托管
