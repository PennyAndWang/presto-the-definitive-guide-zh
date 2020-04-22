# 2. Presto 安装配置

> 本章节实操译者均已验证

在第一章，我们已经简单介绍过 Presto 及使用场景。在本章中，你将会学会如何安装部署 Presto，配置数据源，并且进行数据查询。

## 使用 Docker 安装 Presto

Presto 提供了 Docker 容器，让用户很容易构建一个初始的运行环境。

要在 Docker 中运行 Presto，你需要在自己的机器上安装 Docker。你可以从 Docker 的官网（[https://www.docker.com/](https://www.docker.com/)）上下载 Docker，或者使用你自己的服务器进行打包。

使用 Docker 下载容器镜像（原书中没有这一步）：

```text
$ docker pull prestosql/presto:latest
```

启动容器，命名为 `presto-trail`，并在后台运行：

```text
$ docker run -d --name presto-trial prestosql/presto
```

现在可以连接到容器，并且使用 Presto 的命令行工具：Presto。它可以连接到容器中运行的 Presto。在命令行中，你可以执行对 `tpch` 数据的查询：

```text
$ docker exec -it presto-trial presto
presto> select count(*) from tpch.sf1.nation;
 _col0
-------
    25
(1 row)

Query 20181105_001601_00002_e6r6y, FINISHED, 1 node
Splits: 21 total, 21 done (100.00%)
0:06 [25 rows, 0B] [4 rows/s, 0B/s]
```

{% hint style="info" %}
说明：如果在运行时产生该错误：`Query 20181217_115041_00000_i6juj failed: Presto server is still initializing，`请稍等一会儿再尝试执行。
{% endhint %}

你可以继续使用 SQL 浏览数据了，同时使用 help 命令可以学习如何使用 Presto 命令行工具。

当查询结束时，执行 `quit` 退出。

当需要停止并移除容器时，执行以下操作：

```text
$ docker stop presto-trial
presto-trial
$ docker rm presto-trial
presto-trial
```

当你需要使用时，可以再次运行该容器。如果不再需要这个 Docker 镜像，请执行删除操作：

```text
$ docker rmi prestosql/presto
Untagged: prestosql/presto:latest
...
Deleted: sha256:877b494a9f...
```

## 使用安装包

使用完 Docker 以后，你同样可以在自己的工作机上安装 Presto。

Presto 可以运行在各种 Linux 发行版和 macOS 上。运行时需要安装 Java 和 Python 环境。

### JVM

Presto 由 Java 语言开发，运行时需要 JVM 的支持。Presto 需要 LTS 版本的 Java 11，并且不支持更早的版本。对于新发行的版本，可能不会很好的支持。

确认 `java` 已经被安装并可用：

```text
$ java --version
openjdk 11.0.4 2019-07-16
OpenJDK Runtime Environment (build 11.0.4+11)
OpenJDK 64-Bit Server VM (build 11.0.4+11, mixed mode, sharing)
```

如果 Java 没有正确安装，Presto 无法启动。

### Python

需要 Python 2.6 及以上的环境来运行 Presto。

确认 `python` 已经被安装并可用：

```text
$ python --version
Python 2.7.15+
```

### 安装

Presto 的发行版可以在 Maven 中央仓库找到。服务端安装包后缀为 _tar.gz_。

此处有所有的可用版本列表：

{% embed url="https://repo.maven.apache.org/maven2/io/prestosql/presto-server" %}

版本号越大，版本越新。比如下载 330 版本的 Presto：

```text
$ wget https://repo.maven.apache.org/maven2/\
  io/prestosql/presto-server/330/presto-server-330.tar.gz
```

解压：

```text
$ tar xvzf presto-server-*.tar.gz
```

目录结构如下：

_**lib**_

包含组成 Presto 服务的Java（JAR）和所有必需的依赖。

_**plugins**_ 

在每个插件的单独目录中包含 Presto 插件及其依赖项。 Presto 默认情况下包括许多插件，并且也可以添加第三方插件。Presto 允许可插拔组件与 Presto 集成，例如连接器，功能和安全访问控件。

_**bin**_ 

包含 Presto 的启动脚本。这些脚本用于启动，停止，重新启动，终止并获取正在运行的Presto 进程的状态。

_**etc**_ 

这是配置目录。它由用户创建，并提供 Presto 所需的必要配置文件。

_**var**_ 

这是一个数据目录，即日志的存储位置。它是在首次启动 Presto 服务时创建的。默认情况下，它位于安装目录中。我们建议在安装目录之外对其进行配置，以允许在升级期间保留数据。

### 配置

启动 Presto 前，可以配置一系列的文件：

* 日志配置
* 节点配置
* JVM 配置

默认情况下这些配置文件都在 _etc_ 目录中。

除 JVM 配置外，所有配置均遵循 Java 属性标准。 作为 Java 属性的一般说明，每个配置参数都以一对字符串的形式存储，格式为 key=value 的多行内容。

在上一节“安装“完成后，你需要生成以下配置文件：

_**etc/config.properties**_

```text
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8080
query.max-memory=5GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://localhost:8080
```

_**etc/node.properties**_

```text
node.environment=demo
```

_**etc/jvm.config**_

```text
-server
-Xmx4G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
-Djdk.nio.maxCachedBufferSize=2000000
-Djdk.attach.allowAttachSelf=true
```

完成配置后 Presto 就已经可以启动了。

## 添加数据源

尽管 Presto 安装已经准备就绪，但是你还没有开始运行它。毕竟，你希望能够在 Presto中查询某种外部数据。这要求添加配置为目录的数据源。

Presto  catalog 定义了可供用户使用的数据源。数据访问由 catalog 中配置的 Presto 连接器执行，该连接器具有 **connector.name** 属性。catalog 将数据源内的所有 schema 和 table 公开给 Presto。

例如，Hive 连接器将每个 Hive 数据库映射到一个 schema。如果 Hive 数据库 **web** 包含一个名为 **click** 的表，并且 catalog 名为 **sitehive**，则 Hive 连接器将可以使用该表。必须在 catalog 文件中指定 Hive 连接器。可以使用标准名称语法 **catalog.schema.table** 访问 catalog。因此在此示例中，访问表的方式为：**sitehive.web.clicks**。

通过在 etc/catalog 目录中创建 catalog 属性文件来注册 catalog。文件名设置了 catalog 的名称。例如，假设创建目录属性文件 **etc/cdh-hadoop.properties**，**etc/sales.properties**，**etc/web-traffic.properties** 和 **etc/mysql-dev.properties**。然后在Presto 中公开的 catalog 是 **cdh-hadoop**，**sales**，**web-traffic** 和 **mysql-dev**。

可以使用 TPC-H 连接器进行 Presto 示例的首次探索。 TPC-H 连接器内置在 Presto 中，并提供一组架构来支持 TPC Benchmark H（TPC-H）。

要使用 TPC-H 连接器，请创建一个文件：etc/catalog/tpch.properties，在其中配置一个 **tpch** 连接器：

```text
connector.name=tpch
```

每一个 catalog 文件都需要配置 `connector.name` 这个属性。其他的属性由连接器的种类决定。这些在 Presto 的使用文档中都有提到。

## 运行 Presto

当准备好以后，就可以启动 Presto 了。安装目录中配置好了启动脚本，可以这样启动：

```text
$ bin/launcher run
```

**run** 命令是将 Presto 在前台启动。日志和其他输出通过 **stdout** 和 **stderr** 的形式生成。当如下内容打印出后，说明 Presto 已经启动成功：

```text
INFO        main io.prestosql.server.PrestoServer ======== SERVER STARTED
```

在前台启动 Presto 可以快速验证配置的正确性。使用 Ctrl-C 退出。

## 总结

你现在已经知道如何安装、配置和启动 Presto 了。

