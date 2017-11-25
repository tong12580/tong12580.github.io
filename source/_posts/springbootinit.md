title: 个人关于Spring Boot的理解1 => Spring Boot的产生
date: 2016/09/03 21:52:00
tags:
    - springBoot
categories:
    - spring
---

从SSH框架说起
----
****
> &nbsp;&nbsp; &nbsp;&nbsp;我们谈SSH已经差不多快十年了，所谓的Struts+Spring+Hibernate统治了这个时代相当长的时间，然而IT行业的知识都是有其有限的使用周期,永远都会有新的事务去取代旧的事务一样。根据现有范围内的数据结果统计,SpringMVC占有率已经排在了J2EE框架的第一位,前十都已经看不见struts的身影,因为他太重了就像当年的EJB一样，现如今ES6标准化的js与H5的兴起，Struts去做视图层的操作就显得尤为鸡肋。现如今企业开发，更加主流的框架莫过于Spring+Hibernate/MyBatis+springMVC的SSH架构设计

****
## 为何要用SpringBoot?
 &nbsp;&nbsp;&nbsp;&nbsp; 最近两年Docker热度大增，基于容器的企业微服务架构的概念火热，在Spring3.x之前的大量的xml配置越来越让人诟病，在企业需要进行快速开发时，往往先面临着大量的xml文件的配置，让醉心业务人员，或缺少架构级CTO的小型互联网功能，或是非成熟的项目团队，面临巨大的技术挑战，和繁琐的文件配置工作。而SpringBoot的出现，正好解决了这些问题。
> - 首先，springBoot最大的特点就是当下所倡导的一种理念“习惯优于配置”springBoot它其实并没有用到特别的技术，而是在项目中预先进行了许多习惯的性的配置。当然，这些习惯性的配置也是充分的相应Spring4.x所提倡的内容。
> - 无代码生成并且项目可以没有一个xml配置，全由注解来完成这一切，其本身就提供准企业级开发的配置
> - 提过基于http ssh telnet对项目运行时的监控。
> - 内嵌Servlet容器，开发者开发完成将项目打包后，可以直接输入java -jar xx.jar运行程序,极大的方便了项目的部署 
> - 提供许多容器接口支持，顺应时代潮流

****
## SpringBoot适合什么样的业务场景?
 &nbsp;&nbsp;&nbsp;&nbsp;其实SpringBoot就是SpringMVC简化配置的一个具体体现，其实只要是spring适合的业务场景他都适合，并且我们也不用考虑他作为新技术可能的技术不确定性，因为他就是SpringMVC的另一种展现方式！它特别适合业务并不成熟的新兴互联公司，喜欢追求新技术与优雅编程体验的公司和个人，具有极客精神的公司和个人，小型外包团队，基于云服务的软件提供商。
&nbsp;&nbsp;&nbsp;&nbsp;当然，他并不适应与那些技术沉淀多年的大型开发团队，因为，同样也没有人愿意，将自己写了大把的xml文件删掉吧!

****
<br><br>
####  后记
&nbsp;&nbsp;本文只是讲解SpringBoot的各种优良特性，接下来，我将会有浅显的语言去分享自己所读出来的SpringBoot的各种特性（主要以代码填坑为主）以及相关的Spring Data JPA 的基于个人的一些见地。