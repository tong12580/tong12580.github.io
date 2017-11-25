title: spring使用@Value标签读取.properties文件的中文乱码问题的解决
date: 2017/06/28 09:30:00
tags: 
    - value
    - 中文乱码
    - spring
categories:
    - 中文乱码
---

$begin$

### spring使用@Value标签读取*.properties文件的中文乱码问题的解决
`spring` `中文乱码` `@Value` `*.properties`

****
在spring中我们常常使用.properties对一些属性进行一个提前配置, spring 在读取*.properties文件时,默认使用的是asci码, 这时 我们需要对其编码进行转换.  当然 方法有很多种 ,我说以下几种

****
> 1.$在配置spring.xml文件时,声明所需的*.properties文件时直接使用"utf-8"编码$
```xml
<context:property-placeholder location="classpath:conf/*.properties" file-encoding="UTF-8"/>
```
>2.$ 如果在所需类上注入 可使用以下方式来声明编码格式:$

```java
@Component
@PropertySource(value = "classpath:conf/copyWriteUI.properties",encoding = "utf-8")
@Getter
public class CopyWriteUI {
    @Value("${a}")
    private String a;
    @Value("${b}")
    private String b;
    }
```
>3.$不设置编码格式, 编写文件时 将中文转化为unicode编码$
>4. $如果你用的是IntelliJ IDEA 那么 按如下图操作以上步揍都可以省去啦!!! idea 会自动帮我们进行如上的第三步,解放了我们的双手!$
![idea 会自动帮我们进行如上的第三步的](http://img.blog.csdn.net/20161223170843818?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjNva2Vy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 附录  spring `<context:property-placeholder/>` 的属性说明

```xml
<context:property-placeholder   
        location="属性文件，多个之间逗号分隔"  
        file-encoding="文件编码"  
        ignore-resource-not-found="是否忽略找不到的属性文件"  
        ignore-unresolvable="是否忽略解析不到的属性，如果不忽略，找不到将抛出异常"  
        properties-ref="本地Properties配置"  
        local-override="是否本地覆盖模式，即如果true，那么properties-ref的属性将覆盖location加载的属性，否则相反"  
        system-properties-mode="系统属性模式，默认ENVIRONMENT（表示先找ENVIRONMENT，再找properties-ref/location的），NEVER：表示永远不用ENVIRONMENT的，OVERRIDE类似于ENVIRONMENT"  
        order="顺序"  
        /> 
```


$end$