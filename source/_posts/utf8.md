title: windows环境下强制JVM和Sass使用utf-8编译的方法
date: 2016/06/11 20:32:00
tags:
    - value
    - 中文乱码
categories:
    - 中文乱码
---

### windows环境下强制JVM和Sass使用utf-8编译的方法

`java` `utf-8` `JVM`  `sass`

****

在windows环境进行开发时，会出现了很多环境变量字符编码方面的问题。因为windows默认的中文标准字符集为GBK，而我公司使用的项目都需要进行UTF-8进行启动，特别是在spring boot 中，由于公司的工程需要代码校验本地的环境，非utf-8将不能启动，为此，我寻找到了一个比较好的解决办法--修改本机的系统环境变量 

----
***
_____

- 解决spring boot初始化时jvm选取默认字符集：在环境变量中增加一条环境变量配置---`变量名为: JAVA_TOOL_OPTIONS 变量值为：-Dfile.encoding=UTF-8` 再次启动时jvm就会默认先改变初始化字符集。

- 如果使用IDEA 则在如图所示地方，配置JVM系统环境启动项
![这里写图片描述](http://img.blog.csdn.net/20171114093435528?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjNva2Vy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 图片上是设置FreeMarker的启动方式，可以参照方式设置 `-Dfile.encoding=UTF-8` 
-  解决sass 编译时，中文字体不能正常编译的问题：找到engine.rb文件，目录格式：x:\..\Ruby...\lib\ruby\gems\...\gems\sass-3.xx\lib\sass在末尾添加一行代码：

```
require 'sass/media'
require 'sass/supports'

module Sass
  Encoding.default_external = Encoding.find('utf-8')
```
由于最近只遇到这俩个编码问题，所以只解决了这两个，以后在遇到还会补充，或者大家有遇到其他的，也可以来提问。