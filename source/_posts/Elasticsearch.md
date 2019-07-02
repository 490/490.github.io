---
title: ELK
date: 2019-06-09 14:35:14
tags: Spring
---

Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
<!--more-->

# ElasticSearch特点

（1）可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司；也可以运行在单机上
（2）将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES；
（3）开箱即用的，部署简单
（4）全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理

# ElasticSearch体系结构

| Elasticsearch | 关系型数据库Mysql |
| 索引(index)   | 数据库(databases) |
| 类型(type)     | 表(table) |
| 文档(document) | 行(row) |


# ElasticSearch部署与启动

无需安装，解压安装包后即可使用
在命令提示符下，进入ElasticSearch安装目录下的bin目录,执行命令`elasticsearch`

即可启动。
我们打开浏览器，在地址栏输入http://127.0.0.1:9200/ 即可看到输出结果


# Head插件的安装与使用


如果都是通过rest请求的方式使用Elasticsearch，未免太过麻烦，而且也不够人性化。我
们一般都会使用图形化界面来实现Elasticsearch的日常管理，最常用的就是Head插件
步骤1：
下载head插件：`https://github.com/mobz/elasticsearch-head` 
步骤2：
解压到任意目录，但是要和elasticsearch的安装目录区别开。
步骤3：
安装node js ,安装cnpm 
`npm install ‐g cnpm ‐‐registry=https://registry.npm.taobao.org`
步骤4：
`npm install ‐g grunt‐cli`
将grunt安装为全局命令 。Grunt是基于Node.js的项目构建工具。它可以自动运行你所
设定的任务
步骤5：安装依赖 
`cnpm install` 
步骤6：
进入head目录启动head，在命令提示符下输入命令
`grunt server`
步骤7：
打开浏览器，输入 http://localhost:9100 
步骤8：
点击连接按钮没有任何相应，按F12发现有如下错误 
No 'Access-Control-Allow-Origin' header is present on the requested resource 
这个错误是由于elasticsearch默认不允许跨域调用，而elasticsearch-head是属于前端工程，所以报错。
我们这时需要修改elasticsearch的配置，让其允许跨域访问。
修改elasticsearch配置文件：elasticsearch.yml，增加以下两句命令：
```
http.cors.enabled: true 
http.cors.allow‐origin: "*"
```

此步为允许elasticsearch跨越访问 点击连接即可看到相关信息 

# Elasticsearch与MySQL数据同步


Logstash是一款轻量级的日志搜集处理框架，可以方便的把分散的、多样化的日志搜集起来，并进行自定义的处理，然后传输到指定的位置，比如某个服务器或者文件。

## Logstash安装与测试

解压，进入bin目录

`logstash ‐e 'input { stdin { } } output { stdout {} }'`

stdin，表示输入流，指从键盘输入 stdout，表示输出流，指从显示器输出
命令行参数:-e 执行 --config 或 -f 配置文件，后跟参数类型可以是一个字符串的配置或全路径文件名或全路径路径(如：/etc/logstash.d/，logstash会自动读取/etc/logstash.d/目录下所有*.conf 的文本文件，然后在自己内存里拼接成一个完整的大配置文件再去执行)


## MySQL数据导入Elasticsearch
（1）在logstash-5.6.8安装目录下创建文件夹mysqletc （名称随意）
（2）文件夹下创建mysql.conf （名称随意） ，内容如下：

```
input {
  jdbc {
      # mysql jdbc connection string to our backup databse 后面的test 对应mysql中的test数据库 jdbc_connection_string =>
      "jdbc:mysql://127.0.0.1:3306/tensquare_article?characterEncoding=UTF8"
      # the user we wish to excute our statement as
      jdbc_user => "root"
      jdbc_password => "123456"
      # the path to our downloaded jdbc driver
      jdbc_driver_library => "D:/logstash‐5.6.8/mysqletc/mysql‐
      connector‐java‐5.1.46.jar"
      # the name of the driver class for mysql
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      #以下对应着要执行的sql的绝对路径。 statement => "select id,title,content from tb_article"
      #定时字段 各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为
      每分钟都更新 schedule => "* * * * *"
    }
} 
output {
  elasticsearch {
      #ESIP地址与端口 hosts => "localhost:9200"
      #ES索引名称（自己定义的） index => "tensquare"
      #自增ID编号 document_id => "%{id}"
      document_type => "article"
    }
    stdout {
      #以JSON格式输出 codec => json_lines
  }
}
```


3）将mysql驱动包mysql-connector-java-5.1.46.jar拷贝至D:/logstash-5.6.8/mysqletc/ 下 。D:/logstash-5.6.8是你的安装目录
（4）命令行下执行 
`logstash ‐f ../mysqletc/mysql.conf`

观察控制台输出，每间隔1分钟就执行一次sql查询。再次刷新elasticsearch-head的数据显示，看是否也更新了数据。


# Kibana
Kibana 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。
