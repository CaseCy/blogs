---
title: " elasticsearch部署和采集\t\t"
tags:
  - es
url: 321.html
id: 321
categories:
  - elasticsearch
date: 2019-01-15 11:23:15
---
部署分为几个部分
1.Elasticsearch
2.Logstash
3.Kibanna
4.Filebeats(可选)
注：本文使用版本为5.3.3，以上内容均为此版本，[官网传送门](https://www.elastic.co/)
## elasticsearch安装
安装之前，先建一个linux账号，因为elasticsearch不允许用root账号启动

创建用户组
```bash
groupadd es
```
创建用户并分配到用户组
```bash
useradd es -g es
```
以下elasticsearch简称es
将elasticsearch的文件夹授权给新建的用户es(需要root权限)
```bash
chown -R es:es elasticsearch-5.5.3
```
启动(-d表示后台运行。第一次启动建议不加-d，没问题之后加上-d)
```bash
./bin/elasticsearch -d
```
可能遇到的问题
<https://www.cnblogs.com/xxoome/p/6663993.html>
<https://www.cnblogs.com/sunsky303/p/9676646.html>
验证启动
方式一：
```bash
//es绑定到本地（默认）的时候使用
curl -XGET 'http://localhost:9200'
```
方式二：
浏览器访问 [服务器ip]:9200
返回如下信息
```json
{
  "name" : "node-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "riaWrZzvSJOzk3no7gJb8w",
  "version" : {
    "number" : "5.5.3",
    "build_hash" : "9305a5e",
    "build_date" : "2017-09-07T15:56:59.599Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```
## kibana安装
编辑 config/kibana.yml
修改server.host为服务器ip
填上能够访问的es的地址elasticsearch.url（默认为本地）
将kibana所在文件夹授权给es(需要root权限)
```bash
chown -R es:es [kibana文件夹]
```
后台启动
```bash
nohup ./bin/kibana &
```
验证安装
访问 [服务器ip]:5601
## logstash安装
解压之后
在config目录下新建log.conf
将logstash所在文件夹授权给es(需要root权限)
```
chown -R es:es [logstash文件夹]
```
### 配置数据库采集
最好是在数据库中新建一个只有要采集数据表的查看权限的用户来读取日志
CREATE USER 'es'@'localhost' IDENTIFIED BY '密码';
GRANT SELECT ON [数据库名].[日志表名称] to 'es'@'localhost';
将mysql的连接jar包放到logstash的根目录下
这里以mysql-connector-java-5.1.13-bin.jar为例
编辑log.conf，加入如下内容
```yml
input {
#filebeat采集入口
  beats {
    port => 5000
  }
 jdbc {
      #clean_run => true #表示是否重新采集
      jdbc_driver_library => "mysql-connector-java-5.1.13-bin.jar"
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_user => "es"
      jdbc_password=> "密码"
      jdbc_connection_string => "jdbc:mysql://localhost:3306/数据库?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull"
      jdbc_validate_connection => true
      schedule => "* * * * *"
      use_column_value => true
      tracking_column => "id"
      statement => "要执行的sql"
      jdbc_paging_enabled => true
      jdbc_page_size => 5000
      type => "es中的type"
    }
}
output{
elasticsearch {
      hosts => "[es的地址]"
      index => "es中的index"
	  #将id作为es中的id
      document_id => "%{id}"
    }
    stdout {
      codec => json_lines
    }
}
```
`:sql_last_value`表示字段上次执行的值，可用tracking_column配置，可以用在要执行的sql中
eg:
```
select * from table_user where id > :sql_last_value
```
后台启动logstash
```
nohup bin/logstash -f config/log.conf &
```
jdbc部分的配置可以查看[input插件](https://www.elastic.co/guide/en/logstash/5.3/input-plugins.html)中的jdbc部分
[output插件](https://www.elastic.co/guide/en/logstash/5.3/output-plugins.html)
[filter插件](https://www.elastic.co/guide/en/logstash/5.3/filter-plugins.html)


### 采集Tomcat日志（例子）
编辑logstash配置文件在input下加入
```
beats {
    port => 5000
}
```
和Input同级加入
```
filter {
  #Only matched data are send to output.
  if [type] == "log"{
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:time}\s\[%{LOGLEVEL:level}\]\[(?<threadName>([\s\S]*))\]\s%{NOTSPACE:loggerName}\s(?<info>([\s\S]*))"}
    }
    date {
      match => [ "time", "yyyy-MM-dd HH:mm:ss.SSS" ]
    }
  }
}
```
Output修改为
```
if [type] == "log"{
    elasticsearch {
      action => "index"          #The operation on ES
      hosts  => "[es的地址]"   #ElasticSearch host, can be array.
      index  => "systemlog"         #The index to write data to.
}
} else {
   elasticsearch {
      hosts => "[es的地址]"
      index => "数据库采集的index"
      document_id => "%{id}"
    }
    stdout {
      codec => json_lines
    }
  }
```
#### filebeat安装
备份原有fielebeat.yml
新建filebeat.yml
写入
```yml
filebeat:
  prospectors:
    -
      paths:
        - [tomcat的catalina.out的绝对路径]
      input_type: log
      encoding: utf-8
      multiline:
       #pattern: '^[[:space:]]+(at|\.{3})\b|^Caused by:'
        pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
        negate: true
        match: after
      processors:
        - drop_fields:
            fields: ["source","beat","host"]
output:
  logstash:
    hosts: ["[logstash地址]:[logstash中配置的filebeat的端口号]"]
setup.kibana:
  host: ["kibanna地址加端口号"]
```
后台启动filebeat
```bash
nohup ./filebeat -c ./filebeat.yml &
```
#### filebeat执行的过程
采集日志（通过multiline区分多行和单行）-》发往logstash-》logstash通过filter对日志信息进行拆解（通过grok表达式）为多个字段-》通过output插入es
#### 附录
grok调试网站
<http://grokdebug.herokuapp.com/>(科学上网)
<http://grok.qiexun.net/>