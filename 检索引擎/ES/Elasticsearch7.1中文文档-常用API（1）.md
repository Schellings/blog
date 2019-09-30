---
title: Elasticsearch7.1中文文档-常用API（1）
date: 2018-10-05 16:56:24
tags: Elasticsearch
---


## 浏览集群

### 使用REST API

现在我们已经启动并运行了节点（和集群），下一步就是了解如何与之通信。幸运的是，ElasticSearch提供了一个非常全面和强大的RESTAPI，您可以使用它与集群进行交互。

可以使用API执行的少数操作如下：

- 检查集群、节点和索引的运行状况、状态和统计信息。
- 管理集群、节点和索引数据和元数据。
- 对索引执行CRUD（创建、读取、更新和删除）和搜索操作。
- 执行高级搜索操作，如分页、排序、筛选、脚本编写、聚合和许多其他操作。

### 集群健康

让我们从一个基本的健康检查开始，我们可以使用它来查看集群的运行情况。我们将使用curl来实现这一点，但您可以使用任何允许您进行HTTP/REST调用的工具。

假设我们仍然在启动ElasticSearch并打开另一个命令shell窗口的同一个节点上。为了检查集群的运行状况，我们将使用_catAPI。


```yaml
curl -X GET "localhost:9200/_cat/health?v"
```

响应结果：

```yaml
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%​
```

我们可以看到名为“elasticsearch”的集群处于绿色状态。
每当我们请求集群健康时，我们要么得到绿色、黄色，要么得到红色。

- 绿色-一切正常（集群功能齐全）
- 黄色-所有数据都可用，但某些副本尚未分配（群集完全正常工作）
- 红色-由于任何原因，某些数据不可用（群集部分正常工作）

>注意：当集群为红色时，它将继续提供来自可用分片的搜索请求，但您可能需要尽快修复它，因为存在未分配的分片。

从上面的响应中，我们可以看到总共1个节点，并且我们有0个碎片，因为我们在其中还没有数据。

请注意，由于我们使用的是默认群集名称（ElasticSearch），并且由于ElasticSearch默认情况下发现在同一台计算机上查找其他节点，因此您可能会意外启动计算机上的多个节点，并使它们都加入单个群集。

在这个场景中，您可能会在上面的响应中看到多个节点。

### 查看集群中的节点列表
```yaml
curl -X GET "localhost:9200/_cat/nodes?v"
```

响应结果：
```yaml
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

我们可以看到一个名为“pb2sgzy”的节点，它是当前集群中的单个节点。

### 查看所有索引

现在让我们来看看我们的索引：

```yaml
curl -X GET "localhost:9200/_cat/indices?v"
```

响应结果：

```yaml
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这就意味着我们在集群中还没有索引。

### 创建索引

现在，让我们创建一个名为“customer”的索引，然后再次列出所有索引：

```yaml
curl -X PUT "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
```
第一个命令使用put动词创建名为“customer”的索引。我们只需在调用的末尾附加pretty命令它漂亮地打印JSON响应（如果有的话）。

```yaml
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.sizeyellow open   customer 95SQ4TSUT7mWBT7VNHH67A   1   1          0            0       260b           260b
```

第二个命令的结果告诉我们，我们现在有一个名为customer的索引，它有一个主碎片和一个副本（默认值），其中包含零个文档。

您可能还会注意到客户索引中有一个黄色的健康标签。回想我们之前的讨论，黄色意味着一些副本尚未分配。

此索引发生这种情况的原因是: 默认情况下，ElasticSearch为此索引创建了一个副本。因为目前只有一个节点在运行，所以在另一个节点加入集群之前，还不能分配一个副本（为了高可用性）。

一旦该副本分配到第二个节点上，该索引的运行状况将变为绿色。

### 查询文档

现在我们把一些东西放到客户索引中。我们将在客户索引中索引一个简单的客户文档，其ID为1，如下所示：

```yaml
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'{  "name": "John Doe"}'​
```
响应结果：

```yaml
{  "_index" : "customer",  "_type" : "_doc",  "_id" : "1",  "_version" : 1,  "result" : "created",  "_shards" : {    "total" : 2,    "successful" : 1,    "failed" : 0  },  "_seq_no" : 0,  "_primary_term" : 1}
```

从上面，我们可以看到在客户索引中成功地创建了一个新的客户文档。

文档还有一个内部ID 1，我们在索引时指定了它。需要注意的是，ElasticSearch不要求您在索引文档之前先显式创建索引。在上一个示例中，如果客户索引之前不存在，那么ElasticSearch将自动创建该索引。

现在让我们检索刚才索引的文档：

```yaml
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```
响应结果：
```yaml
{  "_index" : "customer",  "_type" : "_doc",  "_id" : "1",  "_version" : 1,  "_seq_no" : 25,  "_primary_term" : 1,  "found" : true,  "_source" : { "name": "John Doe" }}
```

除了一个字段之外found，这里没有发现任何异常的地方，说明我们找到了一个具有请求的ID 1的文档和另一个字段_source，它返回了我们从上一步索引的完整JSON文档。

### 删除索引

现在，让我们删除刚刚创建的索引，然后再次列出所有索引：

```yaml
curl -X DELETE "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
```

响应结果：

```yaml
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

### 小结
在我们继续之前，让我们再仔细看看我们迄今为止学到的一些API命令：


- 创建索引 curl -X PUT "localhost:9200/customer"
- 创建文档（添加数据）curl -X PUT "localhost:9200/customer/_doc/1" -H 'Content-Type: application/json' -d'{  "name": "John Doe"}'
- 查询文档（查询数据）curl -X GET "localhost:9200/customer/_doc/1"
- 删除文档（删除数据）curl -X DELETE "localhost:9200/customer"
 
 



