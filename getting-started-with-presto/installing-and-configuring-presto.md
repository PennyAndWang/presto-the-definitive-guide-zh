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

当你需要使用时，可以再次运行该容器。如果不在需要这个 Docker 镜像，请执行删除操作：

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

## 添加数据源

## 运行 Presto

## 总结

