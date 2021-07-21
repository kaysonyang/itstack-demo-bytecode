# 小傅哥的《字节码编程》- 文章涉及源码

## 前言

初识字节码编程是从使用非入侵的全链路监控开始，在这之前我所了解的如果需要监控系统的运行状况，通常需要硬编码埋点或者AOP的方式采集方法执行信息；耗时、异常、出入参等来监控一个系统的运行健康度。而这样的监控方式在大量的系统中去改造非常耗时且不好维护，更不要说去监控一个业务流程的调用链路。

在2010年的时候，谷歌发布一篇名为《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》的论文，在文中介绍了谷歌生产环境中大规模分布式系统下的跟踪系统`Dapper`的设计和使用经验。

这样的监控系统采用 `Javaagent` 与字节码操作框架结合使用，在应用系统加载时对需要监控的方法进行字节码增强也叫插桩。对方法处理后的结果就和你之前硬编码类似，但这样就可以减轻认为操作，同时可以对多个系统之间定义调用链路ID进行串联业务流程关系。最终，极大减轻了监控成本也提高了线上问题的快速定位和处理。

这里面监控系统核心知识也主要是 `Javaagent`和字节码操作，在字节码操作中目前有三个比较常用的框架；`ASM`、`Javassist`、`Byte Buddy`，这几个框架都能进行字节码操作，其中`ASM` 更偏向于底层，需要了解字节码指令以及操作数栈等知识，最好学习过《Java虚拟机规范》等书籍，另外两个框架是对 `ASM` 的封装，提供更加高级的API去操作字节码。

在本书中`小傅哥`会分别讲解这三种字节码框架的使用，以及最终与`Javagent`结合完成全链路监控的案例。通过这样的学习让你可以从有抓手的案例开始，把枯燥的字节码编程融入场景，深化理解和实操应用。也能让你忙于CRUD开发的同时提升自己的知识栈，拓展技术视野。也许不久以后这项技术也能为你带来一些有价值的收获！

最后，祝你在学习拼搏的过程中都能，所求皆如愿，所行化坦途。

## 作者

作者小傅哥多年从事一线互联网 Java 开发的学习历程技术汇总，旨在为大家提供一个清晰详细的学习教程，侧重点更倾向编写Java核心内容。如果我的文章能为您提供帮助，请给予**支持**(关注、点赞、分享)！



## 目录

 - [`ASM 篇一，初识ASM使用字节码操作类和方法`](https://bugstack.cn/itstack-demo-agent/2020/03/25/ASM%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-%E5%A6%82%E6%9E%9C%E4%BD%A0%E5%8F%AA%E5%86%99CRUD-%E9%82%A3%E8%BF%99%E7%A7%8D%E6%8A%80%E6%9C%AF%E4%BD%A0%E6%B0%B8%E8%BF%9C%E7%A2%B0%E4%B8%8D%E5%88%B0.html)
 - [`ASM 篇二，JavaAgent+ASM字节码插桩采集方法名称以及入参和出参结果并记录方法耗时`](https://bugstack.cn/itstack-demo-agent/2020/04/05/ASM%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-JavaAgent+ASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%8F%92%E6%A1%A9%E9%87%87%E9%9B%86%E6%96%B9%E6%B3%95%E5%90%8D%E7%A7%B0%E4%BB%A5%E5%8F%8A%E5%85%A5%E5%8F%82%E5%92%8C%E5%87%BA%E5%8F%82%E7%BB%93%E6%9E%9C%E5%B9%B6%E8%AE%B0%E5%BD%95%E6%96%B9%E6%B3%95%E8%80%97%E6%97%B6.html)
 - [`ASM 篇三，用字节码增强技术给所有方法加上TryCatch捕获异常并输出`](https://bugstack.cn/itstack-demo-agent/2020/04/16/ASM%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-%E7%94%A8%E5%AD%97%E8%8A%82%E7%A0%81%E5%A2%9E%E5%BC%BA%E6%8A%80%E6%9C%AF%E7%BB%99%E6%89%80%E6%9C%89%E6%96%B9%E6%B3%95%E5%8A%A0%E4%B8%8ATryCatch%E6%8D%95%E8%8E%B7%E5%BC%82%E5%B8%B8%E5%B9%B6%E8%BE%93%E5%87%BA.html)
 - [`Javassist篇一《基于javassist的第一个案例helloworld》`](https://bugstack.cn/itstack-demo-agent/2020/04/19/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Javassist%E7%AF%87%E4%B8%80-%E5%9F%BA%E4%BA%8Ejavassist%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E6%A1%88%E4%BE%8Bhelloworld.html)
 - [`Javassist篇二《定义属性以及创建方法时多种入参和出参类型的使用》`](https://bugstack.cn/itstack-demo-agent/2020/04/20/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Javassist%E7%AF%87%E4%BA%8C-%E5%AE%9A%E4%B9%89%E5%B1%9E%E6%80%A7%E4%BB%A5%E5%8F%8A%E5%88%9B%E5%BB%BA%E6%96%B9%E6%B3%95%E6%97%B6%E5%A4%9A%E7%A7%8D%E5%85%A5%E5%8F%82%E5%92%8C%E5%87%BA%E5%8F%82%E7%B1%BB%E5%9E%8B%E7%9A%84%E4%BD%BF%E7%94%A8.html)
 - [`Javassist篇三《使用Javassist在运行时重新加载类「替换原方法输出不一样的结果」》`](https://bugstack.cn/itstack-demo-agent/2020/04/21/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Javassist%E7%AF%87%E4%B8%89-%E4%BD%BF%E7%94%A8Javassist%E5%9C%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E9%87%8D%E6%96%B0%E5%8A%A0%E8%BD%BD%E7%B1%BB-%E6%9B%BF%E6%8D%A2%E5%8E%9F%E6%96%B9%E6%B3%95%E8%BE%93%E5%87%BA%E4%B8%8D%E4%B8%80%E6%A0%B7%E7%9A%84%E7%BB%93%E6%9E%9C.html)
 - [`Javassist篇四《通过字节码插桩监控方法采集运行时入参出参和异常信息》`](https://bugstack.cn/itstack-demo-agent/2020/04/27/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Javassist%E7%AF%87%E5%9B%9B-%E9%80%9A%E8%BF%87%E5%AD%97%E8%8A%82%E7%A0%81%E6%8F%92%E6%A1%A9%E7%9B%91%E6%8E%A7%E6%96%B9%E6%B3%95%E9%87%87%E9%9B%86%E8%BF%90%E8%A1%8C%E6%97%B6%E5%85%A5%E5%8F%82%E5%87%BA%E5%8F%82%E5%92%8C%E5%BC%82%E5%B8%B8%E4%BF%A1%E6%81%AF.html)
 - [`Javassist篇五《使用Bytecode指令码生成含有自定义注解的类和方法》`](https://bugstack.cn/itstack-demo-agent/2020/04/29/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Javassist%E7%AF%87%E4%BA%94-%E4%BD%BF%E7%94%A8Bytecode%E6%8C%87%E4%BB%A4%E7%A0%81%E7%94%9F%E6%88%90%E5%90%AB%E6%9C%89%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B3%A8%E8%A7%A3%E7%9A%84%E7%B1%BB%E5%92%8C%E6%96%B9%E6%B3%95.html)
 - [`Byte-buddy篇一《基于Byte Buddy语法创建的第一个HelloWorld》`](https://bugstack.cn/itstack-demo-agent/2020/05/08/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Byte-buddy%E7%AF%87%E4%B8%80-%E5%9F%BA%E4%BA%8EByte-Buddy%E8%AF%AD%E6%B3%95%E5%88%9B%E5%BB%BA%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AAHelloWorld.html)
 - [`Byte-buddy篇二《监控方法执行耗时动态获取出入参类型和值》`](https://bugstack.cn/itstack-demo-agent/2020/05/12/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Byte-buddy%E7%AF%87%E4%BA%8C-%E7%9B%91%E6%8E%A7%E6%96%B9%E6%B3%95%E6%89%A7%E8%A1%8C%E8%80%97%E6%97%B6%E5%8A%A8%E6%80%81%E8%8E%B7%E5%8F%96%E5%87%BA%E5%85%A5%E5%8F%82%E7%B1%BB%E5%9E%8B%E5%92%8C%E5%80%BC.html)
 - [`Byte-buddy篇三《使用委托实现抽象类方法并注入自定义注解信息》`](https://bugstack.cn/itstack-demo-agent/2020/05/14/%E5%AD%97%E8%8A%82%E7%A0%81%E7%BC%96%E7%A8%8B-Byte-buddy%E7%AF%87%E4%B8%89-%E4%BD%BF%E7%94%A8%E5%A7%94%E6%89%98%E5%AE%9E%E7%8E%B0%E6%8A%BD%E8%B1%A1%E7%B1%BB%E6%96%B9%E6%B3%95%E5%B9%B6%E6%B3%A8%E5%85%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B3%A8%E8%A7%A3%E4%BF%A1%E6%81%AF.html)
 - [`基于JavaAgent的全链路监控一《嗨！JavaAgent》`](https://bugstack.cn/itstack-demo-agent/2019/07/10/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E4%B8%80-%E5%97%A8-JavaAgent.html)
 - [`基于JavaAgent的全链路监控二《通过字节码增加监控执行耗时》`](https://bugstack.cn/itstack-demo-agent/2019/07/11/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E4%BA%8C-%E9%80%9A%E8%BF%87%E5%AD%97%E8%8A%82%E7%A0%81%E5%A2%9E%E5%8A%A0%E7%9B%91%E6%8E%A7%E6%89%A7%E8%A1%8C%E8%80%97%E6%97%B6.html)
 - [`基于JavaAgent的全链路监控三《ByteBuddy操作监控方法字节码》`](https://bugstack.cn/itstack-demo-agent/2019/07/12/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E4%B8%89-ByteBuddy%E6%93%8D%E4%BD%9C%E7%9B%91%E6%8E%A7%E6%96%B9%E6%B3%95%E5%AD%97%E8%8A%82%E7%A0%81.html)
 - [`基于JavaAgent的全链路监控四《JVM内存与GC信息》`](https://bugstack.cn/itstack-demo-agent/2019/07/13/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E5%9B%9B-JVM%E5%86%85%E5%AD%98%E4%B8%8EGC%E4%BF%A1%E6%81%AF.html)
 - [`基于JavaAgent的全链路监控五《ThreadLocal链路追踪》`](https://bugstack.cn/itstack-demo-agent/2019/07/14/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E4%BA%94-ThreadLocal%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA.html)
 - [`基于JavaAgent的全链路监控六《开发应用级监控》`](https://bugstack.cn/itstack-demo-agent/2019/07/15/%E5%9F%BA%E4%BA%8EJavaAgent%E7%9A%84%E5%85%A8%E9%93%BE%E8%B7%AF%E7%9B%91%E6%8E%A7%E5%85%AD-%E5%BC%80%E5%8F%91%E5%BA%94%E7%94%A8%E7%BA%A7%E7%9B%91%E6%8E%A7.html)

