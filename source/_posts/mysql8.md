title: mysql8.0 zip安裝配置
date: 2017/01/19 19:46:33
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/mysql_logo.jpg
tags: 
    - mysql
    - SQL
    - java
categories:
    - 数据库
---

MySQL 8.0版本的配置和以前有所不同，在这里与大家分享一下经验。
MySQL 8.0版本目前只有zip版本,此文說明Win系統下的配置 Linux可以以此類推;
下載mysql-8.0.0-dmr-winx64.zip
1.  将下载到的文件解压缩到自己喜欢的位置 我的是C:\Program Files\mysql-8.0.0
2. 添加环境变量 Path = C:\Program Files\mysql-8.0.0\bin
3. 添加配置文件 在MySQL的安装目录下的my.ini 文件進行修改
```path
basedir=C:\Program Files\mysql-8.0.0
datadir=C:\Program Files\mysql-8.0.0data
```
----
1. 以管理员自身份打开CMD执行以下命令 mysqld --initialize --user=mysql --console 在控制台消息尾部会出现随机生成的初始密码，记下来
2.  在CMD控制台里执行命令啟動 net start mysql
3.  在CMD控制台里执行命令  mysql -u root -p 回车执行后，输入刚才记录的随机密码 执行成功后，控制台显示 mysql>，则表示进入mysql 输入命令set password for root@localhost = password(‘123123TTT‘); 
此时root用户的密码修改为123123TTT

注意事项
1.管理員權限
2.密碼複雜度