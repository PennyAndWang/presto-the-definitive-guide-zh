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

## Presto 与 ODBC

## 客户端

## Presto Web 界面

## Presto 与 SQL

## 总结

