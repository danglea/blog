---
title: mongodb的远程访问和基本使用
date: 2019-10-12 12:46:01
tags: [mongo,数据库]

---
MongoDB（来自于英文单词“Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业、
各个行业以及各类应用程序的开源数据库。作为一个适用于敏捷开发的数据库，
MongoDB的数据模式可以随着应用程序的发展而灵活地更新。
<!-- more-->
## 1.使用docker中mongodb

1. 搜索并下载镜像
  ```Linux命令
   docker search  mongodb
   docker pull mongo        //默认最新版本
   docker images              //查看镜像
   docker run -di --name=tensquare_mongo ‐p 27017:27017 mongo  #创建mongo容器
   docker ps #查看是否启动成功
  ```

   ![如图1](https://danglea.gitee.io/blog/assets/mongo1.png)
  
  如果你在局域网下想远程访问的话访问时失败的，原因就是mongo配置只允许本地的访问所以需要修改配置。
  
2. 修改配置

   ```Linux命令
   docker exec -it [容器id或者名称]  /bin/bash  #切换到mongo容器中
   #更新源
   apt-get update
   # 安装 vim
   apt-get install vim
   # 修改 mongo 配置文件
   vim /etc/mongod.conf.orig
   ```

   将配置文件中的bindid= "127.0.0.1" 改成0.0.0.0注意前面有一个空格。问题解决。

3. 可以使用Robot图形化界面连接。

### 2. Mongo常用命令

   (1)创建数据库，以dangdb为例，数据库为dangdb，集合为dang

   `use  dangdb `

   (2)插入数据   NumberInt() 为整形，默认浮点型,_id可以不写，也可以自己指定。

   `db.dang.insert({_id:"2",name:"dang",age:NumberInt(21)})`
   (3)查询数据
   `db.dang.find({_id:"2"})`
   `db.dang.findOne({_id:"2"})`
   `db.dang.find({_id:2).limit(3)`

   (4)修改数据

   `db.dangle.update(条件：修改后数据)`

   (5)删除数据

   `db.dang.remove({_id:"2"})`

   MongoDB的模糊查询是通过正则表达式的方式实现的。格式为： 

   `/模糊查询字符串/ `

   (6)不等于命令

   `db.集合名称.find({ "field" : { $gt: value }}) // 大于: field > value `

   `db.集合名称.find({ "field" : { $lt: value }}) // 小于: field < value `

   `db.集合名称.find({ "field" : { $gte: value }}) // 大于等于: field >= value `

   `db.集合名称.find({ "field" : { $lte: value }}) // 小于等于: field <= value `

   `db.集合名称.find({ "field" : { $ne: value }}) // 不等于: field != value`

   我们如果需要查询同时满足两个以上条件，需要使用$and操作符将条件进行关联。（相 

   当于SQL的and） 

   格式为： 

   `$and:[ { },{ },{ } ]` 

   示例：查询吐槽集合中visits大于等于1000 并且小于2000的文档 

   如果两个以上条件之间是或者的关系，我们使用 操作符进行关联，与前面and的使用 

   方式相同 

   格式为： 

   `$or:[ { },{ },{ } ] `

### 3. java操作mongodb

6. （1）创建工程 mongoDemo, 引入依赖 

   ```java
   <dependency> 
   
   <groupId>org.mongodb</groupId> 
   
   <artifactId>mongodb‐driver</artifactId> 
   
   <version>3.6.3</version> 
   
   </dependency> 
   
   ```

   （2）创建测试类 

   ```java
   public class MongoDemo { public static void main(String[] args) {
       MongoClient client=new MongoClient("192.168.184.134");
       //创建连接 MongoDatabase            
       spitdb = client.getDatabase("spitdb");
       //打开数据库 MongoCollection
       <Document> spit = spitdb.getCollection("spit");
       // 获取集合 FindIterable<Document>        
       documents = spit.find();
       //查询记录获取文档集 合 
       for(Document document:documents){ 
         
           System.out.println("用户ID:"+document.getString("userid"));                           
		   System.out.println("浏览量："+document.getInteger("visits")); 
       }
       client.close();//关闭连接
   } 
   }
   ```

   