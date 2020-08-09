---
title: Elasticsearch7.1中文文档-初识
date: 2018-10-03 16:56:24
tags: Elasticsearch
---
<meta name="referrer" content="no-referrer" />

原文地址：https://juejin.im/post/5cf77ef16fb9a07eeb1396df

## 引言

Elasticsearch是一个高度可扩展开源的全文搜索引擎.它搜索几乎是实时的,用ES作为搜索引擎,为复杂搜索功能的需求提供解决方案.

ES的使用场景:
- 网上商场,搜索商品.
- ES配合logstash,kibana,日志分析.

本教程的其他部分,将指导你完成ES的安装,启动,浏览,以及数据的CRUD.如果你完整的完成了本教程,你应该已经对ES有很好的了解了,希望你能从中受到启发.

## 基本概念

ES有几个核心概念，从一开始理解这些概念将对你后面的学习有很大帮助。

### 近实时（NRT）
ES是一个近实时的搜索引擎（平台），代表着从添加数据到能被搜索到只有很少的延迟。（大约是1s）

### 集群

可以将多台ES服务器作为集群使用，可以在任何一台节点上进行搜索。集群有一个默认的名称（可修改），“elasticsearch”，这个集群名称必须是唯一的，因为集群的节点是通过集群名称来加入集群的。确保在相同环境中不要有相同的集群名称，否则有可能节点会加入到非预期的集群中。

### 节点

节点是作为集群的一部分的单个服务器，存储数据，并且参与集群的索引和搜索功能。与集群一样，节点由一个名称标识，默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。如果不希望使用默认值，则可以定义所需的任何节点名称。此名称对于管理目的很重要，因为您希望确定网络中的哪些服务器对应于ElasticSearch集群中的哪些节点。

### 索引

索引是具有某种相似特性的文档集合。例如，您可以拥有客户数据的索引、产品目录的另一个索引以及订单数据的另一个索引。索引由一个名称（必须全部是小写）标识，当对其中的文档执行索引、搜索、更新和删除操作时，该名称用于引用索引。在单个集群中，您可以定义任意多个索引。如果你学习过Mysql ，可以将其暂时理解为 MySql中的 database。

### 类型

一个索引可以有多个类型。例如一个索引下可以有文章类型，也可以有用户类型，也可以有评论类型。在一个索引中不能再创建多个类型，在以后的版本中将删除类型的整个概念。

> 在Elasticsearch 7.0.0或更高版本中创建的索引不再接受 _default_映射。在6.x中创建的索引将继续像以前一样在Elasticsearch 6.x中运行。在7.0中的API中不推荐使用类型，对索引创建，放置映射，获取映射，放置模板，获取模板和获取字段映射API进行重大更改。

### 文档

一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。在一个index/type里面，你可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引之中，文档必须被索引/赋予一个索引的type。

### 分片和副本

索引可能存储大量数据，这些数据可能会超出单个节点的硬件限制。例如，占用1TB磁盘空间的10亿个文档的单个索引可能不适合单个节点的磁盘，或者速度太慢，无法单独满足单个节点的搜索请求。

为了解决这个问题，ElasticSearch提供了将索引细分为多个片段（称为碎片）的能力。创建索引时，只需定义所需的碎片数量。每个分片（shard）本身就是一个完全功能性和独立的“索引”，可以托管在集群中的任何节点上。

#### 为什么要分片?

- 它允许您水平拆分/缩放内容量
- 它允许您跨碎片（可能在多个节点上）分布和并行操作，从而提高性能/吞吐量

#### 如何分配分片
如何分配分片以及如何将其文档聚合回搜索请求的机制完全由ElasticSearch管理，并且对作为用户的您是透明的。在随时可能发生故障的网络/云环境中，非常有用，强烈建议在碎片/节点以某种方式脱机或因任何原因消失时使用故障转移机制。为此，ElasticSearch允许您将索引分片的一个或多个副本复制成所谓的副本分片，简称为副本分片。

#### 为什么要有副本？

- 当分片/节点发生故障时提供高可用性。因此，需要注意的是，副本分片永远不会分配到复制它的原始/主分片所在的节点上。
- 允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索。

总而言之，每个索引可以分割成多个分片。索引也可以零次（意味着没有副本）或多次复制。

复制后，每个索引将具有主分片（从中复制的原始分片）和副本分片（主分片的副本）。

可以在创建索引时为每个索引定义分片和副本的数量。创建索引后，您还可以随时动态更改副本的数量。您可以使用收缩和拆分API更改现有索引的分片数量，建议在创建索引时就考虑好分片和副本的数量。

默认情况下，ElasticSearch中的每个索引都分配**一个主分片和一个副本**，这意味着如果集群中至少有两个节点，则索引将有一个主分片和另一个副本分片（一个完整副本），每个索引总共有两个分片。

>每个ElasticSearch分片都是一个Lucene索引。在一个Lucene索引中，可以有最多数量的文档。从Lucene-5843起，限制为2147483519（=integer.max_value-128）个文档。您可以使用 api监视碎片大小（以后会讲到）。
 
 ## 安装
 二进制文件可从
 www.slastic.co/downloads
 以及过去发布的所有版本中获得。对于每个版本，Windows、Linux和MacOS以及Linux的DEB和RPM软件包以及Windows的MSI安装软件包都提供了与平台相关的存档版本。
 
### Linux
 
 简单起见，我们使用tar包进行安装
 
#### download

```yaml
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz
```

#### unzip

```yaml
tar -xvf elasticsearch-7.1.1-linux-x86_64.tar.gz
```

#### 进入bin目录

```yaml
cd elasticsearch-7.1.1/bin
```


#### 启动节点和单个集群

```yaml
./elasticsearch
```

### Windows

对于Windows用户，我们建议使用msi安装程序包。该包包含一个图形用户界面（GUI），指导您完成安装过程。

[下载](https://link.juejin.im/?target=https%3A%2F%2Fartifacts.elastic.co%2Fdownloads%2Felasticsearch%2Felasticsearch-7.1.1.msi.)

安装步骤：

- 选择安装目录
- 选择不作为服务安装
- 保留配置默认值
- 不安装任何插件
- 开始安装

默认情况下，ElasticSearch将安装在%ProgramFiles%\Elastic\ElasticSearch。进入安装目录，打开命令提示符，输入

```yaml
.\elasticsearch.exe
```

如果安装一切顺利，您将看到下面的一堆消息：
```yaml
[2018-10-13T12:20:01,766][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] using [1] data paths, mounts [[/home (/dev/mapper/fedora-home)]], net usable_space [335.3gb], net total_space [410.3gb], types [ext4][2018-09-13T12:20:01,772][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] heap size [990.7mb], compressed ordinary object pointers [true][2018-09-13T12:20:01,774][INFO ][o.e.n.Node               ] [localhost.localdomain] node name [localhost.localdomain], node ID [B0aEHNagTiWx7SYj-l4NTw][2018-09-13T12:20:01,775][INFO ][o.e.n.Node               ] [localhost.localdomain] version[7.1.1], pid[13030], build[oss/zip/77fc20e/2018-09-13T15:37:57.478402Z], OS[Linux/4.16.11-100.fc26.x86_64/amd64], JVM["Oracle Corporation"/OpenJDK 64-Bit Server VM/10/10+46][2018-09-13T12:20:01,775][INFO ][o.e.n.Node               ] [localhost.localdomain] JVM arguments [-Xms1g, -Xmx1g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Djava.io.tmpdir=/tmp/elasticsearch.LN1ctLCi, -XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m, -Djava.locale.providers=COMPAT, -XX:UseAVX=2, -Dio.netty.allocator.type=unpooled, -Des.path.home=/home/manybubbles/Workspaces/Elastic/master/elasticsearch/qa/unconfigured-node-name/build/cluster/integTestCluster node0/elasticsearch-7.0.0-alpha1-SNAPSHOT, -Des.path.conf=/home/manybubbles/Workspaces/Elastic/master/elasticsearch/qa/unconfigured-node-name/build/cluster/integTestCluster node0/elasticsearch-7.0.0-alpha1-SNAPSHOT/config, -Des.distribution.flavor=oss, -Des.distribution.type=zip][2018-09-13T12:20:02,543][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [aggs-matrix-stats][2018-09-13T12:20:02,543][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [analysis-common][2018-09-13T12:20:02,543][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-common][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-expression][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-mustache][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-painless][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [mapper-extras][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [parent-join][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [percolator][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [rank-eval][2018-09-13T12:20:02,544][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [reindex][2018-09-13T12:20:02,545][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [repository-url][2018-09-13T12:20:02,545][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [transport-netty4][2018-09-13T12:20:02,545][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] no plugins loaded[2018-09-13T12:20:04,657][INFO ][o.e.d.DiscoveryModule    ] [localhost.localdomain] using discovery type [zen][2018-09-13T12:20:05,006][INFO ][o.e.n.Node               ] [localhost.localdomain] initialized[2018-09-13T12:20:05,007][INFO ][o.e.n.Node               ] [localhost.localdomain] starting ...[2018-09-13T12:20:05,202][INFO ][o.e.t.TransportService   ] [localhost.localdomain] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}[2018-09-13T12:20:05,221][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535][2018-09-13T12:20:05,221][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144][2018-09-13T12:20:08,355][INFO ][o.e.c.s.MasterService    ] [localhost.localdomain] elected-as-master ([0] nodes joined)[, ], reason: master node changed {previous [], current [{localhost.localdomain}{B0aEHNagTiWx7SYj-l4NTw}{hzsQz6CVQMCTpMCVLM4IHg}{127.0.0.1}{127.0.0.1:9300}{testattr=test}]}[2018-09-13T12:20:08,360][INFO ][o.e.c.s.ClusterApplierService] [localhost.localdomain] master node changed {previous [], current [{localhost.localdomain}{B0aEHNagTiWx7SYj-l4NTw}{hzsQz6CVQMCTpMCVLM4IHg}{127.0.0.1}{127.0.0.1:9300}{testattr=test}]}, reason: apply cluster state (from master [master {localhost.localdomain}{B0aEHNagTiWx7SYj-l4NTw}{hzsQz6CVQMCTpMCVLM4IHg}{127.0.0.1}{127.0.0.1:9300}{testattr=test} committed version [1] source [elected-as-master ([0] nodes joined)[, ]]])[2018-09-13T12:20:08,384][INFO ][o.e.h.n.Netty4HttpServerTransport] [localhost.localdomain] publish_address {127.0.0.1:9200}, bound_addresses {[::1]:9200}, {127.0.0.1:9200}[2018-09-13T12:20:08,384][INFO ][o.e.n.Node               ] [localhost.localdomain] started
```

我们可以看到名为“6-bjhwl”的节点(在您的示例中是一组不同的字符)已经启动，并将自己选为单个集群中的主机。现在还不用担心master是什么意思。这里最重要的是，我们已经在一个集群中启动了一个节点。

如前所述，我们可以覆盖集群或节点名。当启动ElasticSearch时，可以从命令行执行此操作，如下所示：

```yaml
./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

还请注意标记为http的行，其中包含可从中访问节点的http地址（192.168.8.112）和端口（9200）的信息。默认情况下，ElasticSearch使用端口9200提供对其RESTAPI的访问。如有必要，此端口可配置。

