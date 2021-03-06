[TOC]
## elasticsearch
配置

### 安装jdk 1.7
查看java --version，如果单个用户有，全局没有，添加到/etc/profile最后面。source /etc/profile文件
```
export JAVA_HOME=/usr/lib/jdk1.7.0/
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

tar  xfelasticsearch-2.3.4.tar.gz-C  /usr/local/
groupadd elk
useradd elk -g elk -p elasticsearch
chown -R  elk.elk/usr/local/elasticsearch-2.3.4/
passwd elk
quanshi
quanshi

mkdir  -p /es-data/data
mkdir  -p /var/log/elasticsearch/
chown -R elk:elk /es-data/data 
chown -R elk:elk  /var/log/elasticsearch/
```
### 修改配置文件

#### 节点1：
```
cluster.name: quanshi
node.name: quanshi-1
path.data: /es-data/data
path.logs: /var/log/elasticsearch/
bootstrap.memory_lock: true
network.host: 10.255.240.8
http.port: 9200
discovery.zen.ping.unicast.hosts: ["IB-hadoop-dn6", "IB-hadoop-dn7","IB-hadoop-dn8"]
```
### 节点2：
```
cluster.name: quanshi
node.name: quanshi-2
path.data: /es-data/data
path.logs: /var/log/elasticsearch/
bootstrap.memory_lock: true
network.host: 10.255.240.9
http.port: 9200
discovery.zen.ping.unicast.hosts: ["IB-hadoop-dn6", "IB-hadoop-dn7","IB-hadoop-dn8"]
```
#### 节点3
```
cluster.name: quanshi
node.name: quanshi-3
path.data: /es-data/data
path.logs: /var/log/elasticsearch/
bootstrap.memory_lock: true
network.host: 10.255.240.10
http.port: 9200
discovery.zen.ping.unicast.hosts: ["IB-hadoop-dn6", "IB-hadoop-dn7","IB-hadoop-dn8"]
```



### 安装插件
```
/usr/local/elasticsearch-2.3.4/bin/plugin install mobz/elasticsearch-head
/usr/local/elasticsearch-2.3.4/bin/plugin install  lmenezes/elasticsearch-kopf
/usr/local/elasticsearch-2.3.4/bin/plugin install hlstudio/bigdesk
```
### 启动
```
su  elk -c  "/usr/local/elasticsearch-2.3.4/bin/elasticsearch &"
```
### 访问插件
```
http://10.255.240.10:9200/_plugin/head/
http://10.255.240.10:9200/_plugin/kopf/
http://10.255.240.10:9200/_plugin/bigdesk/
```


## zookeeper集群搭建

### 配置
<font color=red>zookeeper的配置  所有都一样</font>
```
root@IB-hadoop-dn6:~#  grep ^[a-Z] /usr/lib/zookeeper-3.4.5-cdh5.6.0/conf/zoo.cfg   
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data1/dfs/zppkeeper/zookeeperdir/zookeeper-data
dataLogDir= /data1/dfs/zppkeeper/zookeeperdir/logs
clientPort=2181
server.1=IB-hadoop-dn4:2888:3888
server.2=IB-hadoop-dn5:2888:3888
server.3=IB-hadoop-dn6:2888:3888
server.4=IB-hadoop-dn7:2888:3888
server.5=IB-hadoop-dn8:2888:3888
```
### 修改myid  每个不一样
```
cat /data1/dfs/zppkeeper/zookeeperdir/zookeeper-data/myid
2
```
### 启动
```
/usr/lib/zookeeper-3.4.5-cdh5.6.0/bin/zkServer.sh  start   启动
/usr/lib/zookeeper-3.4.5-cdh5.6.0/bin/zkServer.sh  status  状态
/usr/lib/zookeeper-3.4.5-cdh5.6.0/bin/zkServer.sh  restart  重启 
```
到此zookeeper集群搭建完成

## 搭建kafka集群
```
tar xf /usr/local/src/elk-2.3/kafka_2.11-0.10.0.1.tgz -C /usr/local/
cd /usr/local/kafka_2.11-0.10.0.1/
```
###　kafka配置
```
mkdir /usr/local/kafka_2.11-0.10.0.1/data

grep ^[a-Z] /usr/local/kafka_2.11-0.10.0.1/config/server.properties

broker.id=1
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/usr/local/kafka_2.11-0.10.0.1/data
num.partitions=16
num.recovery.threads.per.data.dir=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=10.255.240.6:2181,10.255.240.7:2181,10.255.240.8:2181,10.255.240.9:2181,10.255.240.10:2181
zookeeper.connection.timeout.ms=6000
```
<font color=red>注: 其他两个节点的配置文件也基本相同，只有一个参数需要修改 broker.id 。 它用于唯一标识节点，所以绝对不能相同，不然会节点冲突。</font>

### 启动kafka
分别启动3个节点
```
nohup  /usr/local/kafka_2.11-0.10.0.1/bin/kafka-server-start.sh /usr/local/kafka_2.11-0.10.0.1/config/server.properties  &
停止
/usr/local/kafka_2.11-0.10.0.1/bin/kafka-server-stop.sh  
```
### 测试kafka
- 建立一个主题

```
[root@kafka1 ~]# /usr/local/kafka_2.11-0.10.0.1/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic summer
```
<font color=red>注意：factor大小不能超过broker数</font>
- 查看有哪些主题已经创建
```
[root@kafka1 ~]# /usr/local/kafka_2.11-0.10.0.1/bin/kafka-topics.sh --list --zookeeper 10.255.240.8:2181   #列出集群中所有的topic
summer  #已经创建成功
```
查看summer这个主题的详情
```
[root@kafka1 ~]# /usr/local/kafka_2.11-0.10.0.1/bin/kafka-topics.sh --describe --zookeeper 10.255.240.8:2181 --topic summer
Topic:summer	PartitionCount:1ReplicationFactor:3Configs:
Topic: summer	Partition: 0Leader: 2Replicas: 2,4,3Isr: 2,4,3
#主题名称：summer
#Partition:只有一个，从0开始
#leader ：id为2的broker
#Replicas 副本存在于broker id为2,3,4的上面
#Isr:活跃状态的broker
```
发送消息，这里使用的是生产者角色
```
[root@kafka1 ~]# /bin/bash /usr/local/kafka_2.11-0.10.0.1/bin/kafka-console-producer.sh --broker-list 10.255.240.8:9092 --topic summer
This is a messages
welcome to kafka   
```
接收消息，这里使用的是消费者角色
```
[root@kafka2 ~]# /usr/local/kafka_2.11-0.10.0.1/bin/kafka-console-consumer.sh --zookeeper  10.255.240.8:2181 --topic summer --from-beginning 
This is a messages
welcome to kafka
```
## filebeat安装
```
tar xf filebeat-1.2.3-x86_64.tar.tar -C /usr/local/
cd filebeat-1.2.3-x86_64/
vim filebeat.yml
```
### 配置
- node1
```
root@IB-hadoop-dn6:/usr/local/logstash-2.3.2/conf.d# cat /usr/local/filebeat-1.2.3-x86_64/filebeat.yml
filebeat:
  prospectors:
    -
      paths:
        - /usr/local/nginx/logs/elk/elk.log

      input_type: log
      
      document_type: messages

      multiline: 
          pattern: '^[[:space:]]'
          negate: true
          match: after

  registry_file: /var/lib/filebeat/registry

  
############################# Output ##########################################
  
output:
  logstash: 
    hosts: ["10.255.240.8:5044"]
  

############################# Shipper #########################################
  
shipper: 
  name: "host_6"
  
  
############################# Logging ######################################### 
  logging:  
  to_files: true
  files:
    path: /var/log/mybeat
    rotateeverybytes: 10485760 # = 10MB
    keepfiles: 7
  level: info

```
### 启动
/usr/local/filebeat-1.2.3-x86_64/filebeat start  &


## logstash安装
```
tar xf logstash-2.3.2.tar.gz -C /usr/local/
mkdir -p   /usr/local/logstash-2.3.2/conf/
```
LS_HEAP_SIZE="${LS_HEAP_SIZE:=4g}" #修改内存大小

###　配置 GeoLiteCity ， 用于地图显示IP访问的城市
官网地址: http://dev.maxmind.com/geoip/legacy/geolite/
下载地址: http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
```
cd /usr/local/logstash-2.3.2
wget    http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
gunzip GeoLiteCity.dat.gz  #解压
```
### logstash配置
- node1：
```
root@IB-hadoop-dn6:/usr/local/logstash-2.3.2/conf.d# cat logstash_in_kafka.conf 
input {
    beats {
    port => 5044
    codec => "json"
}
}
filter {
    if [type] == "nginxacclog" {
    geoip {
        source => "clientip" # 与日志中访问地址的key要对应
        target => "geoip"
        database => "/usr/local/logstash-2.3.2/GeoLiteCity.dat"
        add_field => [ "[geoip][coordinates]","%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]","%{[geoip][latitude]}" ]
}
    mutate {
        convert => [ "[geoip][coordinates]","float" ]
}
}
}
output {
  kafka {
    workers => 2
    bootstrap_servers => "10.255.240.8:9092,10.255.240.9:9092,10.255.240.10:9092"
    topic_id => "test1"
}
}
```
- node2
```
root@IB-hadoop-dn7:/usr/local/logstash-2.3.2/conf.d# cat kafka_to_es.conf 
input{
    kafka {
        zk_connect => "10.255.240.6:2181,10.255.240.7:2181,10.255.240.8:2181,10.255.240.9:2181,10.255.240.10:2181"
        group_id => "logstash"
        topic_id => "test1"
        reset_beginning => false
        consumer_threads => 50
        decorate_events => true
}
}
# 删除一些不需要的字段
filter {
  if [type] == "nginxacclog" {

     mutate {
	 remove_field => ["slbip","kafka","domain","serverip","url","@version","offset","input_type","count","source","fields","beat.hostname","host","tags"]
	}
}
}
output {
    if [type] == "nginxacclog" {
       # stdout {codec => rubydebug }
        elasticsearch {
            hosts => ["10.255.240.8:9200","10.255.240.9:9200","10.255.240.10:9200"]
            index => "logstash-nginxacclog-%{+YYYY.MM.dd}"
            manage_template => true
            flush_size => 50000
            idle_flush_time => 10
            workers => 2
}
}
    if [type] == "messages" {
        elasticsearch {
            hosts => ["10.255.240.8:9200","10.255.240.9:9200","10.255.240.10:9200"]
            index => "logstash-messages-%{+YYYY.MM.dd}"
            manage_template => true
            flush_size => 50000
            idle_flush_time => 30
            workers => 1
}
}
    if [type] == "larlog" {
        elasticsearch {
            hosts => ["10.255.240.8:9200","10.255.240.9:9200","10.255.240.10:9200"]
            index => "logstash-larlog-%{+YYYY.MM.dd}"
            manage_template => true
            flush_size => 2000
            idle_flush_time => 10
}
}
    if [type] == "deposit" {
        elasticsearch {
            hosts => ["10.255.240.8:9200","10.255.240.9:9200","10.255.240.10:9200"]
            index => "logstash-deposit-%{+YYYY.MM.dd}"
            manage_template => true
            flush_size => 2000
            idle_flush_time => 10
}
}
    if [type] == "phpslowlog" {
        elasticsearch {
            hosts => ["10.255.240.8:9200","10.255.240.9:9200","10.255.240.10:9200"]
            index => "logstash-phpslowlog-%{+YYYY.MM.dd}"
            manage_template => true
            flush_size => 2000
            idle_flush_time => 10
}
}
}
```
###　启动：
```
/usr/local/logstash-2.3.2/bin/logstash -f /usr/local/logstash-2.3.2/conf/kafka.conf  &
```


## kibana安装
```
tar xf kibana-4.5.3-linux-x64.tar.gz -C /usr/local/
```
### 配置
修改配置  每个写自己的es
- node.1
```
grep ^[a-Z] /usr/local/kibana-4.5.3-linux-x64/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://10.255.240.8:9200"
```
- node.2
```
grep ^[a-Z] /usr/local/kibana-4.5.3-linux-x64/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://10.255.240.9:9200"
```
- node.3
```
grep ^[a-Z] /usr/local/kibana-4.5.3-linux-x64/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://10.255.240.10:9200"
```
### 启动：
```
/usr/local/kibana-4.5.3-linux-x64/bin/kibana &
```
- 添加到开机启动
```
echo '/usr/local/kibana-4.5.3-linux-x64/bin/kibana &'  >> /etc/rc.local 
```

## nginx
编译环境安装
```
apt-get install  build-essential  make gcc g++  perl  libpcre3 libpcre3-dev  openssl libssl-dev
tar xf /usr/local/src/elk-5.0/nginx-1.6.3.tar.gz -C /usr/local/     #解压
useradd nginx -s /sbin/nologin -M
cd /usr/local/nginx-1.6.3
./configure  --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module  --with-http_ssl_module
make
make install
```
### 配置
```
root@IB-hadoop-dn6:/usr/local/filebeat-1.2.3-x86_64# cat /usr/local/nginx/conf/nginx.conf
worker_processes  4;
 
error_log  logs/error.log  notice;
 
#pid        logs/nginx.pid;
 
 
events {
    worker_connections  1024;
}
 
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
log_format json '{"@timestamp":"$time_iso8601",'
                 '"slbip":"$remote_addr",'
                 '"clientip":"$http_x_forwarded_for",'
                 '"serverip":"$server_addr",'
                 '"size":$body_bytes_sent,'
                 '"responsetime":$request_time,'
                 '"domain":"$host",'
                 '"method":"$request_method",'
                 '"requesturi":"$request_uri",'
                 '"url":"$uri",'
                 '"appversion":"$HTTP_APP_VERSION",'
                 '"referer":"$http_referer",'
                 '"agent":"$http_user_agent",'
                 '"status":"$status",'
                 '"devicecode":"$HTTP_HA"}';
 
    client_max_body_size 100m;
    client_header_buffer_size  256k;
    large_client_header_buffers 4 256k;
    sendfile       on;
    tcp_nopush     on;
    tcp_nodelay    on;
    server_tokens off;
    keepalive_timeout  75;
    client_header_timeout 3m;
    client_body_timeout   3m;
    send_timeout          3m;
 
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 5;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
 
 
 include vhosts/*.conf;
}
```
#### proxy.conf
```
    proxy_ignore_client_abort on;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_connect_timeout 600;
    proxy_send_timeout 600;
    proxy_read_timeout 600;
    proxy_buffer_size  4k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 64k;
    proxy_temp_file_write_size 64k;
```

 
