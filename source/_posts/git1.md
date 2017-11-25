title: 一般新升级OS就会出现的坑 - git无法打开
date: 2016/11/09 22:46:00
tags: 
    - git
categories:
    - git
---

 -> Git
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
解决办法 安装CommandLineTools)：
 

```xml

xcode-select --install

```
一般安装需要1分钟，然后就ok了。