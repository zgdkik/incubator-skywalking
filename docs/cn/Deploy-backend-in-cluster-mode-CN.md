## 所需的第三方软件
- 被监控程序要求JDK6+
- sky-walking server和webui要求JDK8+
- Elasticsearch 5.3
- Zookeeper 3.4.10

## 下载发布版本
- 前向[发布页面](https://github.com/apache/incubator-skywalking/releases)

## 部署Elasticsearch
- 修改`elasticsearch.yml`文件
  - 设置 `cluster.name: CollectorDBCluster`。此名称需要和collector配置文件一致。
  - 设置 `node.name: anyname`, 可以设置为任意名字，如Elasticsearch为集群模式，则每个节点名称需要不同。
  - 增加如下配置

```
# ES监听的ip地址
network.host: 0.0.0.0
thread_pool.bulk.queue_size: 1000
```

- 启动Elasticsearch

### 部署collector
1. 解压安装包`tar -xvf skywalking-collector.tar.gz`，windows用户可以选择zip包
2. 设置Collector集群模式

集群模式主要依赖Zookeeper的注册和应用发现能力。所以，你只需要调整 `config/application.yml`中的host和port配置，使用实际IP和端口，代替默认配置。
其次，将storage的注释取消，并修改为Elasticsearch集群的节点地址信息。


- `config/application.yml`
```
cluster:
# 配置zookeeper集群信息
  zookeeper:
    hostPort: localhost:2181
    sessionTimeout: 100000
naming:
# 配置探针使用的host和port
  jetty:
    host: localhost
    port: 10800
    context_path: /
remote:
  gRPC:
    host: localhost
    port: 11800
agent_gRPC:
  gRPC:
    host: localhost
    port: 11800
agent_jetty:
  jetty:
    host: localhost
    port: 12800
    context_path: /
agent_stream:
  default:
    buffer_file_path: ../buffer/
    buffer_offset_max_file_size: 10M
    buffer_segment_max_file_size: 500M
ui:
  jetty:
    host: localhost
    port: 12800
    context_path: /
# 配置 Elasticsearch 集群连接信息
storage:
  elasticsearch:
    cluster_name: CollectorDBCluster
    cluster_transport_sniffer: true
    cluster_nodes: localhost:9300
    index_shards_number: 2
    index_replicas_number: 0
    ttl: 7
```


3. 运行`bin/startup.sh`启动。windows用户为.bat文件。

- **注意：startup.sh将会启动collector和UI两个进程，UI通过127.0.0.1:10800访问本地collector，无需额外配置。
如需保证UI负载均衡，推荐使用类nginx的HTTP代理服务。**