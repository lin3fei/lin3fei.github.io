---
layout:     post
title:      Elasticsearch常用操作
subtitle:   Elasticsearch
date:       2020-04-08
author:     Lin3Fei
header-img: img/post-bg-elasticsearch.jpg
catalog: true
tags:
    - Linux
    - Elasticsearch
---

>Elasticsearch骚操作

```seq
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

```
seq
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

```sequence
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

```
sequence
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

```sequenceDiagram
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

```
sequenceDiagram
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```
