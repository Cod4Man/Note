# RocketMQ

## 1. 分布式事务

https://zhuanlan.zhihu.com/p/115924952

- **半消息**

利用半消息机制，Producer把消息发送出去到Broker**，此时消息的状态被标记为不能投递（成为半消息）**。事实上，该状态下的消息会被放在一个叫做 `RMQ_SYS_TRANS_HALF_TOPIC`的主题下。 

当producer对它进行**二次确认commit的时候**，消息才会发送到consumer端，如果二次确认的是rollback，则删除消息，consumer永远拿不到。

- **事务回查**

我们想，可能会因为网络原因、应用问题等，导致`Producer`端一直没有对这个半消息进行确认，那么这时候 **`Broker`服务器会定时扫描这些半消息，主动找`Producer`端查询该消息的状态。** 

当然，**什么时候去扫描，包含扫描几次，我们都可以配置**。



