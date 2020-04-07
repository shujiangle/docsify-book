# ELK Log收集与分析

标签（空格分隔）： linux

---
## Elasticsearch
Elasticsearch 是一个`实时的分布式搜索和分析引擎`，他可以用于全文搜索，结构化搜索以及分析。它是一个建立在全文搜索引擎Apache Lucene基础上的搜索引擎，使用java语言编写。目前最新的版本是7.3.0

主要特点

实时分析

 - 分布式实时文件存储，并将每一个字段都编入索引
 - 文档导向，所有的对象全部是文档
 - 高可用性，易拓展，支持集群（Cluster）、分片和复制（Shards和Replicas）
 - 接口友好，支持JSON



## Logstash
Logstash 是一个具有实时渠道能力的数据收集引擎。使用JRuby语言编写。其作者是著名的运维工程师乔丹西塞（JordanSissel)。目前最新的版本是7.3.0

主要特点

 - 几乎可以访问任何数据
 - 可以和多种外部应用结合
 - 支持弹性拓展

它由三个主要部分组成

* Shipper - 发送日志数据
* Broker - 收集数据，缺省内置 Redis
* Indexer - 数据写入


## Kibana
Kibana是一款基于Apache开源协议，使用javaScript语言编写，为Elasticsearch提供`分析和可视化的平台`。它可以在Elasticsearch的索引中查找，交互数据，并生成各种维度的图表。目前最新的版本是7.3

## ELK的官网：[https://www.elastic.co/cn/downloads/](https://www.elastic.co/cn/downloads/)


## **ELK安装**

## 安装java
查看yum库中的java安装包
```yum -y install java* ```

安装最新版本（Elasticsearch要求java版本为1.8以上）
```yum -y install java* ```

安装完成后查看Java版本信息
```java -version```


##配置limit相关参数及虚拟内存交换区域大小
配置limit相关参数
```
 vim /etc/security/limits.conf
 # 添加：
 * soft nproc 65536
 * hard nproc 65536
 * soft nproc 655366
 * hard nofile 65536
```

修改虚拟内存交换区域大小
```bash
 vim /etc/sysctl.conf
 # 添加参数
 vm.max_map_count=262144
 sysctl -p
```


## 安装Elasticsearch
安装Elasticsearch
```bash
# 下载kibana安装包
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.0-linux-x86_64.tar.gz
# 解压kibana安装包
tar -zxvf elasticsearch-7.3.0.tar.gz
# 将elasticsearch移动至/usr/local/
mv elasticsearch-7.3.0 /usr/local/elasticsearch
```

创建elk用户和组用于存放Elasticsearch data的目录
```bash
mkdir -p /data/es-data
groupadd -f elk
useradd -g elk elk
```

修改目录所有者信息
```bash
# chown -R elk:elk /data/es-data
```

修改Elasticsearch日志存放目录的所有这信息
```bash
chown -R elk:elk /var/log/elasticsearch/
chown -R elk:elk /usr/local/elasticsearch/
```

创建elasticsearch日志目录
```bash
mkdir /var/log/elasticsearch/
```


## 编辑Elasticsearch配置文件
```bash
vim /usr/local/elasticsearch/config/elasticsearch.yml
# 设置集群名称
cluster.name: SJL-ELK
# 设置节点名称
node.name: node-1
# 编辑data存放的路径
path.data: /data/es-data
# 编辑log存放的路径
path.logs: /var/log/elasticsearch/
# 监听的地址
network.host 0.0.0.0
# 监听的端口号
http.port: 9200
# cluster.initial_master_nodes
cluster.initial_master_nodes: ["node-1"]
# 为了使Elasticsearch-head插件可以访问es，添加以下参数
http.cors.enabled: true
http.cors.allow-origin: "*"
```

启动elasticsearch
```bash
# 切换到elk用户
su - elk

# 启动elasticsearch
nohup /usr/local/elasticsearch/bin/elasticsearch &

# 查看端口是否存在
netstat -ntlp|grep 9200
netstat -ntlp|grep 9300
```


## 安装Elasticsearch-head插件
首先安装git和Node.js
```bash
yum install -y git
wget -c http://nodejs.org/dist/v0.10.30/node-v0.10.30-linux-X64.tar.gz
tar --strip-components 1 -xzvf node-v* -C /usr/local
node --version
```

通过Github下载插件项目
```bash
git clone git://github.com/mobz/elasticsearch-head.git
```

进入插件目录开始安装
```bash
cd elasticsearch-head
yum install bzip2 -y
npm install
```

启动插件
```bash
npm run start
```

浏览器访问测试
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  [http://host:9100](http://host:9100)


## 安装Logstash
使用yum安装Logstash
```bash
# 安装logstash安装包
yum install -y logstash
```

测试logstash
```bash
/usr/share/logstash/bin/logstash -e 'input { stdin {}} output{stdout {codec => rubydebug}}'

/usr/share/logstash/bin/logstash -e 'input { stdin { }} output {elasticsearch { hosts => ["192.168.153.103:9200"]}stdout { codec => rubydebug }}'
```


## 安装kibana
安装kibana
```bash
# 下载kibana安装包
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.3.0-linux-x86_64.tar.gz
# 解压kibana安装包
tar -zxvf kibana-7.3.0-linux-x86_64.tar.gz
# 将kibana移动至/usr/local/
mv kibana-7.3.0-linux-x86_64 /usr/local/kibana
```
修改kibana配置文件
```bash
vim /usr/local/kibana/config/kibana.yml
#server.port: 5601
server.port: 5601
#server.host: "localhost"
server.host: "0.0.0.0"
#elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.hosts: ["http://192.168.153.103:9200"]
```

后台启动
```bash
# 修改kibana的所有者
chown -R elk /usr/local/kibana/

# 启动kibana
nohup /usr/local/kibana/bin/kibana &

# 查看端口是否存在
netstat -ntlp|grep 5601
```



#  ELk 收集Web Access Log
## 安装准备：
```txt
centos 7 服务器两台:
(192.168.153.103 elasticsearch kibana elasticsearch-head)
(192.168.153.13 logstash)
192.168.153.13 安装apache服务
```

## 编辑Logstash配置文件
```bash
# 编辑Logstash收集Web Access Log的配置文件
vim apache_access_log.conf
input {
  file {
    path => "/var/log/httpd/access_log"
    start_position => "beginning"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { "type" => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch {
    hosts => ["192.168.153.103:9200"]
    index => "apache-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

## ES vs 关系型数据库
| 关系型数据库    | Database | table | row | column |
| :----:  | :----:    | :----:      | :----:  | :----:  |
| ElasticSearch    | Index(索引库) | type(类型) | Document(文档) | Field(字段) |
