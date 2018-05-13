title: git提交冲突的解决
date: 2018/05/23
tags: 
    -git
categories:
    -git
    
---

## 1.git在提交代码前，没有pull到最新的代码时会产生以下问题

error: You have not concluded your merge (MERGE_HEAD exists).
hint: Please, commit your changes before merging.
fatal: Exiting because of unfinished merge.


**解决的方法如下**

### 方法一：

保留你本地的修改
    
    > git merge --abort
    > git reset --merge
    
  合并后提交这个本地合并 
  > git add .
  > git commit .
  > git push
  > git pull
  
### 方法二：

checkout 线上最新的代码版本，抛弃本地的修改
  > git fetch --all
  > git reset --hard origin/master
  > git fetch
  
## 2.从git远程仓库中pull最新的代码，出现如下错误：
   
    Please commit your changes or stash them before you merge.


解决方法如下：(git stash 可用来暂存当前正在进行的工作， 比如想pull 最新代码， 又不想加新commit， 或者另外一种情况，为了fix 一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。)

> * git stash //暂存代码
> * git pull  分支名//从远程仓库拉取最新代码
> * git stash pop //合并代码到本地仓库  此时代码是将暂存的代码和远程仓库的代码合并;
> * 这时候需要手动修改合并所需的代码即可。
> * git stash clear//需要清空git栈执行该命令


git stash: 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
git stash pop: 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。
git stash list: 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。
git stash clear: 清空Git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了

## 3.git push 报错，如下：
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

解决命令如下：

> git push -u 代码所在的分支 -f  //强制提交，此时远程上的修改已经被覆盖。这种方法一般不建议使用，除非你把远程上修改的代码复制到本地。

## 4.本地回退历史版本，当提交代码发生冲突或者想回退到某一个版本
> git log (获取提交的历史日志)
> 执行命令：git reset  --hard  版本号（就是git log 中的  commit后面的哈希值（上图中的黄色部分 commit 后面的值））
> 想要修改远程上的代码还需要执行如下命令： git push -u 代码所在的分支 -f  //强制提交，此时远程上的修改已经被覆盖。这种方法一般不建议使用，除非你把远程上修改的代码复制到本地。