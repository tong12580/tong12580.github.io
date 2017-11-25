title: java关于时区的获取的几种方式
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/java9.png
date: 2017/11/06 10:19:13
tags: 
    - 时区
    - 时间
categories:
    - java
---

1.java8中基本只能通过当前位置所在城市名来获取时区
例如：

``` java
//<1> 查看当前的时区
ZoneId defaultZone = ZoneId.systemDefault();
System.out.println(defaultZone); //此处打印为时区所在城市Asia/Shanghai
//<2>查看美国纽约当前的时间
ZoneId america = ZoneId.of("America/New_York");
LocalDateTime shanghaiTime = LocalDateTime.now(america);

```

####  如果是传入时间，又该如何计算时区呢？

2.使用SimpleDateFormat 来获取Date时区

```java
        DateFormat dateFormat = new SimpleDateFormat("Z");
        System.out.println(dateFormat.format(new Date()));//‘z’小写CST；'Z'大写+0800


```

3.使用lang3中的org.apache.commons.lang3.time函数获取

```java

System.out.println(DateFormatUtils.format(new Date(), "z"));//‘z’小写CST；'Z'大写 +0800

System.out.println(DateFormatUtils.format(new Date(), "ZZ"));//'zz'小写一样 "ZZ"大写+08:00

```

4.使用日历类来计算出传入时间所在时区

```java
Calendar cal = Calendar.getInstance();
        int offset = cal.get(Calendar.ZONE_OFFSET);
        cal.add(Calendar.MILLISECOND, -offset);
        Long timeStampUTC = cal.getTimeInMillis();
        Long timeStamp = date.getTime();
        Long timeZone = (timeStamp - timeStampUTC) / (1000 * 3600);
        System.out.println(timeZone.intValue());//8

```

暂时总结出这几种。

