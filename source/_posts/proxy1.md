title: 设计模式之代理模式
date: 2017/11/26 23:31:00
tags:
    - proxy
    - java
    - 代理模式
    - 设计模式
categories:
    - 设计模式
---

### 代理模式点解

代理模式对于技术人员来说是最常见的设计模式了吧。现实应用中对于代理模式的应用，比如我们熟知的Nginx,
大名鼎鼎的Socket5。用户通过代理来完成自己当前请求，而不直接访问资源，就像律师，用户不用直接参与诉讼，派出律师参与诉讼的整个环节。

综上所述，代理模式的好处在于以下几点：
* 安全性。在代理模式中，用户与资源之间通过代理访问，供求双方并不直接接触，一定程度上提高了安全性；
* 模糊处理细节。类比nginx，使用代理，模糊其中的技术细节，使访问过程变得容易。同时，在这里，代理或许会智能的处理找寻最近的资源反馈给用户；
* 延迟加载。类似在对数据库链接查询初始化的操作中，当服务刚刚启动时，因为并没有真正的用户操作，所以只初始化没有任何属性操作的代理类，而非初始化查询类。加快了启动速率。当用户真正去做查询操作时，再由代理了单独去加载真正的查询类，完成用户的请求。

咱们来举个🌰说明一下

<p style="text-align:center;">
    <img src="/img/context/proxy.png">
    <span>图1-代理初始化与访问查询时序图</span>
</p>

 角色  | 作用
|: ---:|: --- :|
user | 用户, 代理类的使用者，命令发出者
DBProxy| 数据库代理
DBQuery| 数据访问实体

当用户刚启用服务时，因为并没有真实访问数据库，所以只初始化了数据代理服务，并没有建立数据库连接，减少因建立连接而产生的等待时间。这样是非常快捷并节约资源的。当用户需要数据时，此时用户访问代理，由代理去构建数据访问实例，得到结果并返回给用户。在此过程中，由于是代理去寻找数据资源，而用户并不知道真实资源是从谁那里得到的，因此，更加安全。


## 构建一个简单的代理

基于以上设计，我们来构建一个简单的代理; 首先，我们构造一个数据查询接口如下：

```java

public interface IDBQuery {
    String request();
}

```

然后，我们构建一个重量级的实现，来模拟其中的数据资源加载，网络请求等待：

```java

public class DBQuery implements IDBQuery {
    
    public DBQuery() {
        try{
            Thread.sleep(1000);
        } catch (InterruptedException ignore) {}
    }
    
    @Override
    public String request() {
        return "hello!";
    }
}

```

再来，我们构建一个快捷的访问代理，同时实现该数据查询接口：

```java

public class DBProxy implements IDBQuery {
    private DBQuery query = null;
    
    @Override
    public String request() {
        if (null == query) {
            query = new DBProxy();
        }
       return query.request();
    }
}

```

我们用一个main函数来模拟软件初始化，以及用户访问的过程：

```java

public class User {
    
    private static IDBQuery  idbQuery = new DBProxy();//软件初始化，先加载代理。
    
    public static void main(String[] args){
      idbQuery.request();//用户真正访问时，创建真实数据访问对象。
    }
}

```

此实例能较为直观的反应代理的延迟加载作用和机制。当然在java中更多的使用了类似CGLIB，代理工厂等来动态加载代理类，来做到运行时加载，提供系统效率。后面将会慢慢分解。

<style type="text/css">
    p {
        text-indent: 2em
    }
</style>