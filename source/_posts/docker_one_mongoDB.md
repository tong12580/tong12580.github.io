title: Docker mongoDB 单机安装办法
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/mongoDB.png
date: 2017/11/15 19:20:00
tags: 
    - docker
    - SQL
    - MongoDB
categories:
    - 数据库
---

#### 简介

![这里写图片描述](https://raw.githubusercontent.com/docker-library/docs/01c12653951b2fe592c1f93a13b4e289ada0e3a1/mongo/logo.png)

MongoDB（来自“humongous”）是一个跨平台的面向文档的数据库。作为一个NoSQL数据库，MongoDB避开了传统的基于表格的关系数据库结构，而采用动态模式的类似JSON的文档（MongoDB称为BSON格式），使得某些类型的应用程序中的数据集成更加方便快捷。MongoDB是GNU Affero通用公共许可证和Apache许可证的组合，是免费的开源软件。使用Docker安装单机版是比较快捷的办法。

#### 安装方法

安装办法如下：

* 首先，可以打开docker hub 检索mongoDB,搜索你喜欢的MongoDB条目。像我就倾向选择 official 版本，便于拓展或后续操作。

![这里写图片描述](http://img.blog.csdn.net/20171115093620364?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjNva2Vy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 使用 `docker pull mongo` 来拉取一个docker镜像
* 使用密码启动服务 
```
docker run --name mongo -p 27017:27017 -v /home/data/mongodb/data(映射为自己的DB数据文件存储位置):/data/db -d mongo --auth

```
* 启动之后 `docker exec -it some-mongo mongo admin` 连接至admin 来添加初始管理员用户，创建数据库，已经赋予权限。

```
> db.createUser({ user: 'jsmith', pwd: 'some-initial-password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });(创建 admin用户) 
> db.createUser({user:'',pwd:'root',roles:[{ role:'root',db: 'admin'}]}) 创建root用户
> db.auth(“用户名”,”密码”)给admin账户授权
> use octblog 数据库
> db.createUser({user: "gevin",pwd: "gevin",
  roles: [ { role: "readWrite", db: "octblog"},{ role: "readWrite", db: "octblog-log" } ] })创建普通读写用户

```

* 这样就可以通过外部访问数据库了

#### 附录 

MongoDB 权限说明

　　a MongoDB内置角色官网文档介绍：http://docs.mongoing.com/manual-zh/reference/built-in-roles.html
　　b 关于MongoDB的内置角色，我们大概可以分为以下几种来简单说一下
　　　　b.1 Database User Roles(数据库用户角色)：read、readWrite
　　　　b.2 Database Administration Roles(数据库管理角色)：dbAdmin、dbOwner、userAdmin
　　　　b.3 Culster Administration Roles(管理员组，针对整个系统进行管理)：clusterAdmin、clusterManager、clusterMonitor、hostManager
　　　　b.4 Backup and Restoration Roles(备份还原角色组)：backup、restore
　　　　b.5 All-Database Roles(所有数据库角色)：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
　　　　b.6 Superuser Roles(超级管理员)：root、(dbOwner、userAdmin、userAdminAnyDatabase这几个角色角色提供了任何数据任何用户的任何权限的能力，拥有这个角色的用户可以在任何数据库上定义它们自己的权限)
　　　　b.7  Internal Role(内部角色，一般情况下不建议设置)：__system
　　c 关于上面每一个角色的意义是什么，请自行去官网或者这篇文章去查看，地址是：http://www.cnblogs.com/SamOk/p/5162767.html