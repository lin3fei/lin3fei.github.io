---
layout:     post
title:      ELK导入MongoDB慢日志
subtitle:   监控MongoDB慢日志
date:       2020-04-01
author:     Lin3Fei
header-img: img/post-bg-elasticsearch.jpg
catalog: true
tags:
    - Linux
    - Elasticsearch
    - Logstash
    - Kibana
    - Filebeat
    - MongoDB
---

>系统版本: centos8
>Elasticsearch: 6.5
>Kibana: 6.5
>Logstash: 6.5
>Filebeat: 6.5

先看图，在Kibana中查询的结果:

![](https://raw.githubusercontent.com/lin3fei/lin3fei.github.io/master/img/post-bg-elasticsearch-kibana.png)

图中分别有index、mongo操作类型、mongo操作数据库及集合、耗时、消息以及读取时间

再看看整体的时序图:

```sequence
User->>Kibana: 查询mongodb慢日志（请求）
Kibana->>Elasticsearch: Search Api
Logstash->>Elasticsearch: 存入mongo慢日志
Filebeat->>Logstash: 解析日志并写入Logstash
Note right of Filebeat: 循环读取并筛选mongo日志
Elasticsearch-->>Kibana: 返回mongodb慢日志
Kibana-->>User: 展现搜索结果
```

下面分步骤详情说明各组件安装及配置:

#### Elasticsearch

下载及安装elasticsearch

```
# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.0.rpm
# yum localinstall elasticsearch-6.5.0.rpm
```

打开elasticsearch配置文件

```
# vi /etc/elasticsearch/elasticsearch.yml
```

修改配置

```
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
```

启动及查看状态

```
# service elasticsearch start
# service elasticsearch status
```

查看日志

```
# tail -f /var/log/elasticsearch/elasticsearch.log
```

打开浏览器，输入 [地址](http://192.168.104.225:9200)，显示为下图片则启动成功

![](https://raw.githubusercontent.com/lin3fei/lin3fei.github.io/master/img/post-bg-elasticsearch-9200.png)

#### Kibana

下载及安装

```
# wget https://artifacts.elastic.co/downloads/kibana/kibana-6.5.0-x86_64.rpm
# yum localinstall kibana-6.5.0-x86_64.rpm
```

修改配置文件

```
# vi /etc/kibana/kibana.yml
```

修改配置

```
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.104.225:9200"
```

启动及查看状态

```
# service kibana start
# service kibana status
```

查看日志

```
# tail -f /var/log/kibana/kibana.stdout
```

打开浏览器，输入 [地址](http://192.168.104.225:5601/)，显示为下图则启动成功

![](https://raw.githubusercontent.com/lin3fei/lin3fei.github.io/master/img/post-bg-elasticsearch-5601.png)

#### Logstash

下载及安装

```
# wget https://artifacts.elastic.co/downloads/logstash/logstash-6.5.0.rpm
# yum localinstall logstash-6.5.0.rpm
```

在对应的目录下添加配置文件

```
# cd /etc/logstash/conf.d
# vi my.conf
```

修改配置

```
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][type] == "mongo-slowlog" {
    grok {
      match => ["message","%{TIMESTAMP_ISO8601:timestamp}\s+I %{WORD:component}\s+\[%{WORD:connection}\]\s+%{GREEDYDATA:body}"]
      remove_field => [ "message" ]
    }
    if [component] =~ "WRITE" {
      if "appName" in [body] {
        grok {
          match => ["body","%{WORD:command_type}\s+%{DATA:database}\s+%{WORD:appname_key}\:\s+\"%{DATA:appname_value}\"\s+\w+\:\s+%{GREEDYDATA:message}\s+%{INT:duration:int}ms$"]
          remove_field => [ "body", "appname_key", "appname_value" ]
        }
      } else {
        grok {
          match => ["body","%{WORD:command_type}\s+%{DATA:database}\s+\w+\:\s+%{GREEDYDATA:message}\s+%{INT:duration:int}ms$"]
          remove_field => [ "body" ]
        }
      }
    } else {
      if "appName" in [body] {
        grok {
          match => ["body","%{WORD:command}\s+%{DATA:database}\s+%{WORD:appname_key}\:\s+\"%{DATA:appname_value}\"\s+\w+\:\s+%{WORD:command_type}\s+%{GREEDYDATA:message}\s+%{INT:duration:int}ms"]
          remove_field => [ "body", "command", "appname_key", "appname_value" ]
        }
      } else {
        grok {
          match => ["body","%{WORD:command}\s+%{DATA:database}\s+\w+\:\s+%{WORD:command_type}\s+%{GREEDYDATA:message}\s+%{INT:duration:int}ms"]
          remove_field => [ "body", "command" ]
        }
      }
    }
    date {
      match => ["timestamp", "MMMM dd yyyy, HH:mm:ss.SSS", "ISO8601"]
      target => "@timestamp"
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://192.168.104.225:9200"]
    index => "mongo-slowlog-%{+YYYY-MM-dd}"
  }
}
```

启动及查看状态

```
# service logstash start
# service logstash status
```

查看日志

```
# tail -f /var/log/logstash/logstash-plain.log
```

附 Logstash grok 在线解析debug， [grokdebug](http://grokdebug.herokuapp.com/)

#### Filebeat

下载及安装

```
# wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.5.0-x86_64.rpm
# yum localinstall filebeat-6.5.0-x86_64.rpm
```

修改配置文件

```
# vi /etc/filebeat/filebeat.yml
```

修改配置

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    # log path
    - /var/log/mongodb/mongod.log
  include_lines: ['ms$']
    # customer fields
  fields:
    type: mongo-slowlog
    slowlog_host: vbl-mongo1

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 3

# logstash host port
output.logstash:
  hosts: ["192.168.104.225:5044"]

processors:
  - drop_fields:
      fields:
        - "beat.name"
        - "beat.hostname"
        - "beat.version"
        - "offset"
        - "prospector.type"
        - "offset"
```

启动及查看状态

```
# service filebeat start
# service filebeat status
```

查看日志

```
# tail -f /var/log/filebeat/filebeat
```
