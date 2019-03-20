---
title: 消息队列 RabbitMQ基础知识
date: 2019-03-09 21:38:58
tags: [消息队列,系统]
---
# 消息队列

使用消息队列主要有两点好处：
1. 通过异步处理提高系统性能（削峰、减少响应所需时间）;
2. 降低系统耦合性。

<!--more-->

**通过异步处理提高系统性能（削峰、减少响应所需时间）**

在不使用消息队列服务器的时候，用户的请求数据直接写入数据库，在高并发的情况下数据库压力剧增，使得响应速度变慢。但是在使用消息队列之后，用户的请求数据发送给消息队列之后立即 返回，再由消息队列的消费者进程从消息队列中获取数据，异步写入数据库。由于消息队列服务器处理速度快于数据库（消息队列也比数据库有更好的伸缩性），因此响应速度得到大幅改善。

因为用户请求数据写入消息队列之后就立即返回给用户了，但是请求数据在后续的业务校验、写数据库等操作中可能失败。因此使用消息队列进行异步处理之后，需要适当修改业务流程进行配合，比如用户在提交订单之后，订单数据写入消息队列，不能立即返回用户订单提交成功，需要在消息队列的订单消费者进程真正处理完该订单之后，甚至出库后，再通过电子邮件或短信通知用户订单成功，以免交易纠纷。这就类似我们平时手机订火车票和电影票。

**降低系统耦合性**

对新增业务，只要对该类消息感兴趣，即可订阅该消息，对原有系统和业务没有任何影响，从而实现网站业务的可扩展性设计。

另外为了避免消息队列服务器宕机造成消息丢失，会将成功发送到消息队列的消息存储在消息生产者服务器上，等消息真正被消费者服务器处理后才删除消息。在消息队列服务器宕机后，生产者服务器会选择分布式消息队列服务器集群中的其他服务器发布消息。

备注： 不要认为消息队列只能利用发布-订阅模式工作，只不过在解耦这个特定业务环境下是使用发布-订阅模式的。除了发布-订阅模式，还有点对点订阅模式（一个消息只有一个消费者），我们比较常用的是发布-订阅模式。 另外，这两种消息模型是 JMS 提供的，AMQP 协议还提供了 5 种消息模型

**使用消息队列带来的一些问题**

- 系统可用性降低： 系统可用性在某种程度上降低，为什么这样说呢？在加入MQ之前，你不用考虑消息丢失或者说MQ挂掉等等的情况，但是，引入MQ之后你就需要去考虑了！
- 系统复杂性提高： 加入MQ之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消- 息传递的顺序性等等问题！
- 一致性问题： 我上面讲了消息队列可以实现异步，消息队列带来的异步确实可以提高系统响应速度。但是，万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!


# RabbitMQ

AMQP，即 Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦和通讯。
AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
RabbitMQ是一个开源的AMQP实现，服务器端用 Erlang 语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，具有很高的易用性和可用性。


**ConnectionFactory、Connection、Channel ** 都是RabbitMQ对外提供的API中最基本的对象。

ConnectionFactory：ConnectionFactory为Connection的制造工厂。

Connection：Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。

Channel(信道)：信道是建立在“真实的”TCP连接上的虚拟连接，在一条TCP链接上创建多少条信道是没有限制的，把他想象成光纤就是可以了。它是我们与RabbitMQ打交道的最重要的一个接口，我们大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。

![image](http://490.github.io/images/20190310_080814.png)
queue队列
![image](http://490.github.io/images/20190310_080835.png)
生产者Send Message “A”被传送到Queue中，消费者发现消息队列Queue中有订阅的消息，就会将这条消息A读取出来进行一些列的业务操作。这里只是一个消费正对应一个队列Queue，也可以多个消费者订阅同一个队列Queue，当然这里就会将Queue里面的消息平分给其他的消费者，但是会存在一个一个问题就是如果每个消息的处理时间不同，就会导致某些消费者一直在忙碌中，而有的消费者处理完了消息后一直处于空闲状态，因为前面已经提及到了Queue会平分这些消息给相应的消费者。这里我们就可以使用prefetchCount来限制每次发送给消费者消息的个数。详情见下图所示：
![image](http://490.github.io/images/20190310_081017.png)
生产者在将消息发送给 Exchange 的时候，一般会指定一个 routing key，来指定这个消息的路由规则。 Exchange 会根据 routing key 和 Exchange Type（交换器类型） 以及 Binding key 的匹配情况来决定把消息路由到哪个 Queue。RabbitMQ为routing key设定的长度限制为255 bytes。

RabbitMQ常用的Exchange Type有 Fanout、 Direct、 Topic、 Headers 这四种。

**Fanout**

这种类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，这时 Routing key 不起作用。
![image](http://490.github.io/images/20190310_081109.png)

**Direct**

这种类型的Exchange路由规则也很简单，它会把消息路由到那些 binding key 与 routing key完全匹配的Queue中。
![image](http://490.github.io/images/20190310_081116.png)
     当生产者（P）发送消息时Rotuing key=booking时，这时候将消息传送给Exchange，Exchange获取到生产者发送过来消息后，会根据自身的规则进行与匹配相应的Queue，这时发现Queue1和Queue2都符合，就会将消息传送给这两个队列，如果我们以Rotuing key=create和Rotuing key=confirm发送消息时，这时消息只会被推送到Queue2队列中，其他Routing Key的消息将会被丢弃。
     
**Topic**

这种类型的Exchange的路由规则支持 binding key 和 routing key 的模糊匹配，会把消息路由到满足条件的Queue。 binding key 中可以存在两种特殊字符 `*`与 `#`，用于做模糊匹配，其中 `*`用于匹配一个单词，`#` 用于匹配0个或多个单词，单词以符号`.`为分隔符。
![image](http://490.github.io/images/20190310_081253.png)

**Headers**

这种类型的Exchange不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配,对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对；如果完全匹配则消息会路由到该Queue，否则不会路由到该Queue。

**Message durability（消息的持久化）**

如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，我们可以将Queue与Message都设置为可持久化的（durable），这样可以保证绝大部分情况下我们的RabbitMQ消息不会丢失。但依然解决不了小概率丢失事件的发生（比如RabbitMQ服务器已经接收到生产者的消息，但还没来得及持久化该消息时RabbitMQ服务器就断电了），如果我们需要对这种小概率事件也要管理起来，那么我们要用到事务。由于这里仅为RabbitMQ的简单介绍，所以这里将不讲解RabbitMQ相关的事务。具体可以参考 [RabbitMQ之消息确认机制（事务+Confirm）](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fu013256816%2Farticle%2Fdetails%2F55515234)

* * *

接口优化===同步下单改为异步下单
收到请求后由redis先预检库存，不足的话直接返回，减少数据库访问
如果有库存，则请求放到消息队列。返回一个排队中
请求出队。客户端轮询是否秒杀成功
