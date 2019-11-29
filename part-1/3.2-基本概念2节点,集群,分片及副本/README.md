# 基本概念（2）：节点，集群，分片及副本
## 课程Demo
- 需要通过Kibana导入Sample Data的电商数据。具体参考“2.2节-Kibana的安装与界面快速浏览”
- Node 角色
     - `d` (data node),
     - `i` (ingest node),
     - `m` (master-eligible node),
     - `l` (machine learning node),
     - `v` (voting-only node),
     - `-` (coordinating node only)

```
GET _cat/nodes?v
GET /_nodes/es7_01,es7_02
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m


GET _cluster/health
GET _cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_ecommerce,kibana_sample_data_flights
GET /_cluster/health/kibana_sample_data_flights?level=shards

#### cluster state
The cluster state API allows access to metadata representing the state of the whole cluster. This includes information such as
GET /_cluster/state

#cluster get settings
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true
GET /_cluster/settings?include_defaults=true&flat_settings=true

GET _cat/shards
GET _cat/shards?v&h=index,shard,prirep,state,unassigned.reason

```
## 相关阅读
- CAT Nodes API https://www.elastic.co/guide/en/elasticsearch/reference/7.4/cat-nodes.html
- Cluster API https://www.elastic.co/guide/en/elasticsearch/reference/7.4/cluster-update-settings.html
- CAT Shards API https://www.elastic.co/guide/en/elasticsearch/reference/7.4/cat-shards.html
- Node Role https://www.elastic.co/guide/en/elasticsearch/reference/7.4/modules-node.html
