# 3. 使用 Presto

恭喜，通过前面的章节你已经学会了如何安装、配置、启动 Presto。接下来开始使用它吧！

## Presto 命令行

Presto 命令行工具也可以在 Maven 中央仓库中找到。它是一个可执行的 JAR 文件。

可以在这里找到相关内容：

{% embed url="https://repo.maven.apache.org/maven2/io/prestosql/presto-cli" %}

命令行工具的版本要和 Presto 服务端的版本一致。从指定版本的目录下载 _\*-executable.jar_ 文件，并重命名为 `presto`；举例，使用 **wget** 命令下载 330 版本：

```text
$ wget -O presto \
https://repo.maven.apache.org/maven2/\
io/prestosql/presto-cli/330/presto-cli-330-executable.jar
```

确保该文件可执行。为了方便，将其配置在 PATH 中：

```text
$ chmod +x presto
$ mv presto ~/bin
$ export PATH=~/bin/:$PATH
```

现在可以使用 Presto 命令行并确认版本：

```text
$ presto --version
Presto CLI 330
```

获取文档和帮助可以使用 help 选项：

```text
$ presto --help
```

在使用命令行工具前，要确认连接到哪个服务。默认是连接到 [http://localhost:8080](http://localhost:8080)：

```text
$ presto
presto>
```

如果要连接到其他服务，需要指定 URL：

```text
$ presto --server https://presto.example.com:8080
presto>
```

`presto>` 这个显示表示我们已经在交互式的访问 Presto 。输入 help 获取可执行的命令：

```text
presto> help
Supported commands:
QUIT
EXPLAIN [ ( option [, ...] ) ] <query>
    options: FORMAT { TEXT | GRAPHVIZ | JSON }
             TYPE { LOGICAL | DISTRIBUTED | VALIDATE | IO }
DESCRIBE <table>
SHOW COLUMNS FROM <table>
SHOW FUNCTIONS
SHOW CATALOGS [LIKE <pattern>]
SHOW SCHEMAS [FROM <catalog>] [LIKE <pattern>]
SHOW TABLES [FROM <schema>] [LIKE <pattern>]
USE [<catalog>.]<schema>
```

大多数命令，尤其是 SQL 命令，需要以分号结尾。

首先，你可以查看有 catalog 配置了哪些数据源。`system` 是默认存在的内部数据源。

在我们这个例子中，你可以看到 tpch：

```text
presto> SHOW CATALOGS;
 Catalog
---------
 system
 tpch
(2 rows)

Query 20191212_185850_00001_etmtk, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]
```

现在可以查询 catalog 中有哪些 schema 和 table 了。每当查询 Presto 的时候，查询处理的统计信息会和结果一起返回，在上面的例子中可以看到，下面的例子中，我们省略了这些信息。

```text
presto> SHOW SCHEMAS FROM tpch;
       Schema
--------------------
 information_schema
 sf1
 sf100
 sf1000
 sf10000
 sf100000
 sf300
 sf3000
 sf30000
 tiny
(10 rows)

presto> SHOW TABLES FROM tpch.sf1;
  Table
----------
 customer
 lineitem
 nation
 orders
 part
 partsupp
 region
 supplier
(8 rows)
```

现在可以查询一些实际数据：

```text
presto> SELECT count(name) FROM tpch.sf1.nation;
 _col0
-------
    25
(1 row)
```

也可以先选择 schema，再执行查询，即可在查询中省略 schema 信息：

```text
presto> USE tpch.sf1;
USE
presto:sf1> SELECT count(name) FROM nation;
```

也可以在使用命令行时，就直接指定 schema：

```text
$ presto --catalog tpch --schema sf1
```

现在就可以用已有的 SQL 知识进行查询了。

要退出命令行，可以输入 **quit** 或 **exit**，或者使用 Ctrl+D。

### 分页

默认情况下，Presto 的输出结果使用 less 查看。可以通过环境变量 PRESTO\_PAGER 进行设置，可以设置为 more，或置为空以让结果全部输出。

### 执行历史

命令行工具可以保留本次会话中的执行历史，通过 上、下 键来切换，或使用 Ctrl-S 和 Ctrl-R 搜索执行记录。如果想再次执行某条命令，按回车即可。

执行历史默认保存在 _~/.presto\_history_。可以通过修改环境变量 PRESTO\_HISTORY\_FILE 来配置。

### 额外诊断方式

可以使用 --debug 参数来获取 debug 信息：

```text
$ presto --debug

presto:sf1> SELECT count(*) FROM foo;
Query 20181103_201856_00022_te3wy failed:
 line 1:22: Table tpch.sf1.foo does not exist
io.prestosql.sql.analyzer.SemanticException:
 line 1:22: Table tpch.sf1.foo does not exist
...
at java.lang.Thread.run(Thread.java:748)
```

## Presto JDBC 驱动

任何的 Java 应用程序都可以通过 JDBC 驱动来访问 Presto。JDBC 是一种标准的 API，提供了关系型数据库的查询、插入、删除和更新等方法。很多运行在 JVM 之上的程序都支持 JDBC。所有的这些程序都可以通过 JDBC 连接 Presto。

_Presto JDBC 驱动可以让我们在程序中直接通过 SQL 访问 Presto。_

{% hint style="info" %}
_提示：如果你熟悉不同的 JDBC 实现，你就可以知道 Presto JDBC 是一种 Type 4 的驱动。这意味着它使用了原生的 Presto 协议。\(参考：_[_https://en.wikipedia.org/wiki/JDBC\_driver_](https://en.wikipedia.org/wiki/JDBC_driver)_\)_
{% endhint %}

有了 JDBC 驱动，我们可以使用各种强力的数据库连接管理工具，比如开源的 DBeaver，SQuirreL SQL等。使用 JDBC 的报表生成、仪表盘和分析工具都可以连接到 Presto。

使用这些工具的步骤是类似的：

1. 下载 JDBC 驱动
2. 将驱动放在应用程序可以访问的路径
3. 配置 JDBC 驱动
4. 配置 Presto 连接
5. 开始连接并使用

举例，DBeaver 的配置就十分简单。安装 DBeaver 以后，进行以下操作：

1. 从 文件 选择 新建
2. 从 DBeaver 的选项中，选择 数据库连接，点击下一步
3. 输入 prestosql，选择并点击下一步
4. 配置连接，结束。注意这里需要填写用户名，如果 Presto 没有开启认证，就提供一个随机的名字。

现在你可以在数据库导航中看到 schema 和 table 了，如图3-1。你也可以直接使用 SQL 编辑器，开始使用 Presto 进行查询。

![&#x56FE; 3-1](../.gitbook/assets/image%20%284%29.png)

SQuirreL SQL Client 和其他工具也是一样的连接过程。让我们看看如何下载 JDBC 驱动并配置吧！

### 下载注册驱动

Presto 的 JDBC 驱动同样位于 Maven 中央仓库，以 JAR 文件的形式提供。

你可以在 [https://repo.maven.apache.org/maven2/io/prestosql/presto-jdbc](https://repo.maven.apache.org/maven2/io/prestosql/presto-jdbc.) 看到所有可用版本。

版本号越大，代表着版本越新，进入文件夹并下载 _.jar_ 文件。也可以通过命令行下载；举例，下载 330 版本：

```text
$ wget https://repo.maven.apache.org/maven2/\
io/prestosql/presto-jdbc/330/presto-jdbc-330.jar
```

要在程序中使用 JDBC 驱动，你需要把它放入 Java 应用的类目录。通常，是一个名为 lib 的文件夹。有些程序提供了对话框来上传驱动，也可以通过手动复制的方式放入。

配置完驱动路径后，应用程序需要重启。

接下来就可以加载驱动了。在 SQuirreL SQL Client 里，你可以使用用户界面左侧的 + 号来添加驱动。

当配置驱动时，要确认以下参数配置正确：

* 类名：io.prestosql.jdbc.PrestoDriver
* 模板 JDBC URL：jdbc:presto://host:port/catalog/schema
* 名称：Presto
* 网址：[https://prestosql.io](https://prestosql.io)

类名、JDBC URL、JAR 驱动路径是必填项。

### 建立连接

当驱动配置好，Presto 已经启动时，就可以通过程序连接了。

在 SQuirreL SQL Client，连接配置叫做 alias。可以使用用户界面的 Alias 面板，点击 + 号，配置以下参数：

_**名称**_

起一个名字用来描述连接。当你连接到多个 Presto 服务时，这个名称就很有用了。

_**驱动**_

选择之前创建好的驱动。

_**URL**_

JDBC URL 的驱动使用这个格式：_jdbc:presto://host:port/catalog/schema，_catalog 和 schema 都是可设置的。

_**用户名**_

用户名是必填的，即使没有开启验证。这让我们方便定位查询来源。

_**密码**_

当开启认证时，密码是必填的。

另外，还可以配置一些其他的可选参数：

_**SSL**_

是否打开 SSL，true 或 false。

_**SSLTrustStorePath**_

ssl truststore 的配置路径

_**SSLTrushStorePassword**_

访问 truststore 的密码

_**user and password**_

和用户名密码一样

_**applicationNamePrefix**_

在 Presto 中被用来辨识应用来源。

图 3-2 展示了配置好 Presto 连接的 SQuirreL SQL Client，正在进行一些查询：

![&#x56FE; 3-2](../.gitbook/assets/image%20%285%29.png)

## Presto 与 ODBC

略

## 客户端

除了 Presto 命令行工具和 JDBC 驱动，Presto 开发者社区也创造了很多的 Presto 连接客户端。

你可以找到支持 Python，C，Go，Node.js，R，Ruby 等语言的客户端。

这些库可以让使用这些语言的应用连接 Presto。

## Presto Web 界面

Presto 提供了一个 Web 界面，以查看查询状态，如图 3-3。

该 Web 界面可以直接通过 Presto 服务的 HTTP 端口访问，默认是 8080，比如：http://presto.example.com:8080。如果是本机部署，可以直接访问 http://localhost:8080。

主界面展示了 Presto 的利用率和查询，其他的信息也可以看到。这些信息对于维护管理 Presto 都很重要。

![&#x56FE; 3-3](../.gitbook/assets/image%20%286%29.png)

## Presto 与 SQL 协议

Presto 支持标准化的 ANSI SQL 协议。

Presto 致力于兼容已有的 SQL 标准。Presto 的一个设计原则是，不会创造新的类 SQL 查询语言，也不会和已有的标准偏离太多。所有新的功能和语言特性都要考虑和标准兼容。

只有当标准 SQL 不能替代时，才会扩展已有的 SQL 特性内容。同时，在扩展时是格外严谨的，需要考虑是否有相似的实现，以及以后如何将它标准化。

{% hint style="info" %}
提示：在极少数情况下，当在标准实现找不到替代时，Presto 做了扩展。一个例子是 Lambda。请参考 “Lambda 表达式“。
{% endhint %}

Presto 没有定义具体是支持哪个版本的 ANSI SQL。实际上，我们认为标准是一直在更新的，最新的特性往往是越重要的。也就是说，Presto 并没有完全实现已有的 SQL 标准。如果某个实现和标准相悖，那不久之后它就会被弃用并且被标准替代。

### 概念

通过 Presto 可以直接查询关系型数据库，K-V 存储，以及对象存储。下面的概念是 Presto 中比较重要的：

Connector

让 Presto 连接到一个数据源。每个 catalog 都是一种 connector 的实现。

Catalog

定义了如何连接到一个数据源，包括配置 connector 种类，schema 名称。

Schema

组织 table 的一种方式。 catalog 和 table 共同定义了可被查询的 table。

Table

无序记录的集合，包含列的定义和数据类型。

### 简单举例

Presto 默认包含名为 system 的  catalog。通过它可以查询一些元信息。

列出可用的  catalog： 

```text
SHOW CATALOGS;
 Catalog
 ---------
 system
 tpch
 (2 rows)
```

查询 tpch 中所有的 schema：

```text
SHOW SCHEMAS FROM tpch;
       Schema
 ------------------
 information_schema
 sf1
 sf100
 sf1000
 sf10000
 sf100000
 sf300
 sf3000
 sf30000
 tiny
(10 rows)
```

查询 sf1 中的表：

```text
SHOW TABLES FROM tpch.sf1;
  Table
 ---------
 customer
 lineitem
 nation
 orders
 part
 partsupp
 region
 supplier
(8 rows)
```

查询 region 表结构：

```text
DESCRIBE tpch.sf1.region;
  Column   |     Type     | Extra | Comment
-----------+--------------+-------+---------
 regionkey | bigint       |       |
 name      | varchar(25)  |       |
 comment   | varchar(152) |       |
(3 rows)
```

其他的语法，比如 USE，SHOW FUNCTIONS，都是可以用的。

查询 region 表中的数据：

```text
SELECT name FROM tpch.sf1.region;
    name
 ------------
 AFRICA
 AMERICA
 ASIA
 EUROPE
 MIDDLE EAST
(5 rows)
```

可以排序：

```text
SELECT name
 FROM tpch.sf1.region
 WHERE name like 'A%'
 ORDER BY name DESC;
  name
 ---------
 ASIA
 AMERICA
 AFRICA
(3 rows)
```

可以关联表：

```text
SELECT nation.name AS nation, region.name AS region
FROM tpch.sf1.region, tpch.sf1.nation
WHERE region.regionkey = nation.regionkey
AND region.name LIKE 'AFRICA';
   nation   | region
------------+--------
 MOZAMBIQUE | AFRICA
 MOROCCO    | AFRICA
 KENYA      | AFRICA
 ETHIOPIA   | AFRICA
 ALGERIA    | AFRICA
(5 rows)
```

可以通过 \| \| 符号连接字符串，也可以使用如 + 、- 的运算符。

混合使用 join 和字符串连接：

```text
SELECT nation.name || ' / ' || region.name AS Location
FROM tpch.sf1.region JOIN tpch.sf1.nation
ON region.regionkey = nation.regionkey
AND region.name LIKE 'AFRICA';
      Location
 -------------------
 MOZAMBIQUE / AFRICA
 MOROCCO / AFRICA
 KENYA / AFRICA
 ETHIOPIA / AFRICA
 ALGERIA / AFRICA
(5 rows)
```

此外，Presto 还支持非常丰富的函数，可以使用 SHOW FUNCTIONS 来查看。如，计算取整订单的平均价格：

```text
SELECT round(avg(totalprice)) AS average_price
FROM tpch.sf1.orders;
 average_price
 --------------
     151220.0
(1 row)
```

## 总结

Presto 现在已经可以正常运行了。接下来的章节，我们将会讨论：如何安装 Presto 集群，理解 Presto 的架构，并深入了解 SQL 的细节。

