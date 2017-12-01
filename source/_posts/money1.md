title: Java Money 和 Currency API
date: 2017/12/01 08:46:00
tags: 
    - Money
    - JSR-354
categories:
    - java
---

## 概述

JSR 354 - “金钱和货币” 解决了Java中货币和货币金额的标准化问题。

它的目标是为Java生态系统添加一个灵活的可扩展的API，并使货币量更简单，更安全。

JSR没有进入JDK 9，而是未来JDK版本的候选人。

## 安装

首先，让我们定义依赖到我们的pom.xml文件中

```xml

<dependency>
    <groupId>org.javamoney</groupId>
    <artifactId>moneta</artifactId>
    <version>1.1</version>
</dependency>

```

最新版本的依赖关系可以在这里[检查](https://mvnrepository.com/search?q=moneta)。


## JSR-354功能

“货币和金钱”API的目标：

* 提供处理和计算货币金额的API
* 定义货币和货币金额的类别，以及货币四舍五入
* 处理汇率
* 处理货币和货币金额的格式化和解析

## 模型

JSR-354规范的主要类如下图所示：

![monetary](/img/context/javax-monetary3-1.png)

该模型包含两个主要接口CurrencyUnit 和MonetaryAmount， 在下面的章节中进行了解释。

### 1. CurrencyUnit

CurrencyUnit模拟货币的最小属性。它的实例可以使用Monetary.getCurrency方法获得
我们使用货币的字符串表示形式创建CurrencyUnit ，这可能会导致我们尝试使用不存在的代码创建货币的情况。使用不存在的代码创建货币会引发UnknownCurrency异常。

### 2. MonetaryAmount

MonetaryAmount是货币金额的数字表示。它始终与CurrencyUnit 关联，并定义货币的货币表示形式。

金额可以用不同的方式来实现，重点放在由每个具体用例所定义的货币表示要求的行为上。例如。Money和FastMoney是MonetaryAmount接口的实现。

FastMoney实现MONETARYAMOUNT使用长为数字表示，并且比更快的BigDecimal在精度的成本; 它可以在我们需要性能时使用，精度不是问题。

通用实例可以使用默认工厂创建。

最后

你可以通过 本文 [Github](https://github.com/tong12580/money) 上的代码片段查询实例的使用方法。