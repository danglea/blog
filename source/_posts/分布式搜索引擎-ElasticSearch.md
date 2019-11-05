---
title: 分布式搜索引擎-ElasticSearch
date: 2019-10-16 13:18:56
tags: [分布式,索引库]
   
---
***介绍：***Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速 
度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分 
布式多用户能力的全文搜索引擎，基于RESTful web接口。
<!--more-->

### 1.ElasticSearch的本地部署和使用

下载ElasticSearch 5.6.8版本 

 [](https://www.elastic.co/downloads/past-releases/elasticsearch-5-6-8)

进入ElasticSearch的bin目录下，点击elasticsearch.bat即可以启动，默认端口号为9200。

我们打开浏览器，在地址栏输入http://127.0.0.1:9200/ 即可看到输出结果:

```json
{ "name" : "uV2glMR", 
"cluster_name" : "elasticsearch",
"cluster_uuid" : "RdV7UTQZT1‐Jnka9dDPsFg",
"version" : 
{ "number" : "5.6.8",
"build_hash" : "688ecce",
"build_date" : "2018‐02‐16T16:46:30.010Z", 
"build_snapshot" : false,
"lucene_version" : "6.6.1" },
"tagline" : "You Know, for Search" }
```

### 2.head插件的安装

如果都是通过rest请求的方式使用Elasticsearch，未免太过麻烦，而且也不够人性化。我 

们一般都会使用图形化界面来实现Elasticsearch的日常管理，最常用的就是Head插件 

步骤1： 

下载head插件解压到随意目录：[](https://github.com/mobz/elasticsearch-head)

步骤2:

首先要有node.js,head插件时基于grunt插件的。Grunt是基于Node.js的项目构建工具。它可以自动运行你所 设定的任务
将grunt安装为全局：

`npm install ‐g grunt‐cli `

安装依赖：

`npm install`

步骤3：

在head的目录中打开命令窗口执行：

`grunt server`

打开浏览器，[]( http://localhost:9100 )

**问题：**

点击连接按钮没有任何相应，按F12发现有如下错误 

No 'Access-Control-Allow-Origin' header is present on the requested resource 

这个错误是由于elasticsearch默认不允许跨域调用，而elasticsearch-head是属于前端工 

程，所以报错。 

我们这时需要修改elasticsearch的配置，让其允许跨域访问。 

修改elasticsearch配置文件：*conf/elasticsearch.yml*，增加以下两句命令： 
```
http.cors.enabled: true 
http.cors.allow‐origin: "*"
```
此步为允许elasticsearch跨越访问 点击连接即可看到相关信息 

### 3.hend插件的基本操作

(1)新建索引

选择“索引”选项卡，点击“新建索引”按钮 

![](https://danglea.gitee.io/blog/assets/elasticsearch.png)

(2)新建文档相当于表

点击"复合查询"按钮，发送post请求，带上想要添加的数据：

http://localhost:9200/索引/表/                 外加post请求。

(3)复杂查询：

```
http://localhost:9200/索引/表/1      //id查询
http://localhost:9200/索引/表/_search      //查询所有
http://localhost:9200/索引/表/_search?q=name:dang      //按名字查询
http://localhost:9200/索引/表/_search?q=name:*dang*      //模糊查询
......
```

### 4.IK分词库

我们在浏览器地址栏输入http://127.0.0.1:9200/_analyze? 

analyzer=chinese&pretty=true&text=我是程序员，浏览器显示效果将“我是程序员”全部分开。

默认的中文分词是将每个字看成一个词，这显然是不符合要求的，所以我们需要安装中 

文分词器来解决这个问题。 

操作1：

IK分词库下载：[](https://github.com/medcl/elasticsearch-analysis-ik/releases)

(1)将其解压到ElasticSearch/plugs目录下更改名字为IK，重新启动elasticsearch就好。

(2)再次测试：http://127.0.0.1:9200/_analyze?analyzer=ik_max_word&pretty=true&text=我是程序 

员,效果就出来了。

操作2：自定义分词库

（1）进入elasticsearch/plugins/ik/config目录 

（2）新建一个my.dic文件，编辑内容： 

`我不服输`

修改IKAnalyzer.cfg.xml（在ik/config目录下） 

```
<properties>
<comment>IK Analyzer 扩展配置</comment> 
<!‐‐用户可以在这里配置自己的扩展字典 ‐‐> 
<entry key="ext_dict">  my.dic  </entry> 
<!‐‐用户可以在这里配置自己的扩展停止词字典‐‐> 
<entry key="ext_stopwords"></entry> 
</properties>
```

重新启动elasticsearch,通过浏览器测试分词效果 

### 5.与数据库同步

###### ---------logstash

Logstash是一款轻量级的日志搜集处理框架，可以方便的把分散的、多样化的日志搜集 

起来，并进行自定义的处理，然后传输到指定的位置，比如某个服务器或者文件。 

###### mysql数据导入elasticsearch

（1）在logstash-5.6.8安装目录下创建文件夹mysqletc （名称随意） 

（2）文件夹下创建mysql.conf （名称随意） ，内容如下： 

```
input { 
jdbc { 
# mysql jdbc connection string to our backup databse 后面的test 对应mysql中的test数据库 jdbc_connection_string=>
  "jdbc:mysql://127.0.0.1:3306/tensquare_articlecharacterEncoding=UTF8"
# the user we wish to excute our statement as
   jdbc_user => "root" jdbc_password => "123456"
# the path to our downloaded jdbc driver
   jdbc_driver_library => "D:/logstash‐5.6.8/mysqletc/mysql‐ connector‐java‐5.1.46.jar" 
# the name of the driver class for mysql jdbc_driver_class => "com.mysql.jdbc.Driver"       jdbc_paging_enabled => "true" jdbc_page_size => "50000" 
#以下对应着要执行的sql的绝对路径。 
   statement => "select id,title,content from tb_article"
#定时字段 各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为 每分钟都更新 
schedule => "* * * * *" 
} 
}
output { 
   elasticsearch { #ESIP地址与端口 
    hosts => "localhost:9200" 
#ES索引名称（自己定义的） 
   index => "tensquare" 
#自增ID编号 
   document_id => "%{id}" 
   document_type => "article" 
}
  stdout { 
#以JSON格式输出 
codec => json_lines
} 
}
```

（3）将mysql驱动包mysql-connector-java-5.1.46.jar拷贝至D:/logstash- 

5.6.8/mysqletc/ 下 。D:/logstash-5.6.8是你的安装目录 

（4）在bin目录下执行命令行

`logstash  -f  ../mysqletc/mysql.conf`

一分钟后如图：
elasticsearch控制台可能会打印错误在之前但是没有问题的。
![](https://danglea.gitee.io/blog/assets/logstash.png)