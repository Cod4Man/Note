# Tomcat

## 1. Tomcat和Weblogic的区别

https://blog.csdn.net/ththcc/article/details/78474476

> 功能上：WebLogic相对Tomcat，主要多支持了EJB

WebLogic应该是J2EE Container(Web Container + EJB Container + XXX规范)！ 

Tomcat只能算Web Container （JSP+Servlet）

> 扩展性：WebLogic的集群高可用和负载均衡

WebLogic Server凭借其出色的群集技术，拥有处理关键Web应用系统问题所需的性能、可扩展性和高可用性。 

WebLogic Server既实现了网页群集，也实现了EJB组件 群集，而且不需要任何专门的硬件或操作系统支持。网页群集可以实现透明的复制、负载平衡以及表示内容容错 。    无论是网页群集，还是组件群集，对于电子商务解决方案所要求的可扩展性和可用性都是至关重要的。共享的客户机/服务器和数据库连接以及数据缓存和EJB都增强了性能表现。这是其它Web应用系统所不具备的    所以，在扩展性方面WebLogic是远远超越了Tomcat。 

> 费用： Weblogic不开源不免费

> EJB 

被称为Java企业bean，服务器端组件，核心应用是部署分布式应用程序。用它部署的系统不限定平台。实际上ejb是一种产品，描述了应用组件要解决的标准  

用通俗话说，EJB就是："把你编写的软件中那些需要执行制定的任务的类，不放到客户端软件上了，而是给他打成包放到一个服务器上了"。是的，没错！EJB 就是将那些"类"放到一个服务器上，用C/S 形式的软件客户端对服务器上的"类"进行调用。 

EJB容器中有三种类也称为组件，分别是会话Bean（Session Bean），实体Bean（Entity Bean）和消息驱动Bean（MessageDriven Bean）  Session bean(逻辑)  EntityBean(数据)  messageDrivenbean（消息） 