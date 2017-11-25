title: Git项目拆迁
thumbnail: https://cdn.webxueyuan.com/cdn/files/attachments/0013848605496402772ffdb6ab448deb7eef7baa124171b000/0
date: 2017/07/29 11:24:18
tags:
    - github
    - git
categories:
    - git
---

本文摘自 http://www.bijishequ.com/detail/260740?p=

目的：

将项目在不同的git服务器上进行转移（coding <---> github等）
保留所有代码（包括分支）
保留提交记录
说明：

将项目从coding.net上面拆迁到github上（或者反过来迁到其他的git服务器 上，方法大致相同）
参考文章：
Coding.net使用和从Github转移项目到Coding.net
操作方法：

说明：

我采用的是从coding.net ---> 迁移到 github上面
步骤：

git clone --bare git@git.coding.net*.git（git 要迁移的项目），会生成 *.git项目
cd 进入生成的项目 *.git 下面
git push --mirror git@github.com:*.git（需要迁移到的git服务器上面的一个空的项目）
大功告成！

