title: npm使用过程中遇到的 Cannot find module 'internal/fs' 的问题
date: 2017/06/07 20:32:00
tags: 
    - internal/fs
    - internal
categories:
    - npm
---

如题: 在使用npm中遇到如下问题:

```node
node 7.0 Cannot find module 'internal/fs'

node 6.9 fs: re-evaluating native module sources is not supported. If you are using the graceful-fs module, please update it to a more recent version.
```

  这时候 应该是graceful-fs  这个模块出现了问题!
  
	可能原因有以下几点:
	1. 没装
	2. npm的graceful-fs 出现了多个版本

----

	解决办法:
	首先查看: npm list graceful-fs 
	其次全局安装 graceful-fs  接着 在对应环境再安装一遍

还有什么问题，也可以问博主，能解决问题就点个赞咯