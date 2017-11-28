# CQRS

**Command Query Responsibility Segregation** Pattern

查询和命令分离架构

Command可认为是写操作，改变服务器状态的请求

Query可认为是读操作，查询请求

---

## 什么是CQRS

一个对象Object的方法Method要么是Command，要么是Query，并且Command Query Separation 描述的是这样一套原则，

* Query只返回数据，不改变Object的状态
* Command只改变Object的状态，不返回数据

CQRS就是基于这个原则而发展的一个模式

## 现在是什么模式

通常web开发的模式是多层架构（**N-tier type of architecture**），比如一个典型的三层结构：UI层，业务逻辑层，数据库层，这个结构下查询和命令

都是在业务逻辑层实现的。

这个架构的问题在于，查询和命令都是经过相同路径进行执行的。其实也没啥问题，但是这个架构的缺点

* 性能
* 可扩展性
* 效率

## 如何选择

CQRS不是万能的，并不建议将整个系统CQRS之。应该根据系统的实际情况，选择合适的上下文边界，来CQRS。

什么是上下文边界？

> Throughout this guide, we use the term _bounded context _in preference to the term _business component _to refer to the context within which we are implementing the CQRS pattern.

#### Scalability

> In many enterprise systems, the number of reads vastly exceeds the number of writes, so your scalability requirements will be different for each side. By separating the read side and the write side into separate models within the bounded context, you now have the ability to scale each one of them independently.

但不要因为这个就选择CQRS，分层模式也可以通过增加资源达到效果，并且也还不错

#### Reduced complexity

> In complex areas of your domain, designing and implementing objects that are responsible for both reading and writing data can exacerbate the complexity. In many cases, the complex business logic is only applied when the system is handling updates and transactional operations; in comparison, read logic is often much simpler. When the business logic and read logic are mixed together in the same model, it becomes much harder to deal with difficult issues such as multiple users, shared data, performance, transactions, consistency, and stale data. Separating the read logic and business logic into separate models makes it easier to separate out and address these complex issues. However, in many cases it may require some effort to disentangle and understand the existing model in the domain.

> Another potential benefit of simplifying the bounded context by separating out the read logic and the business logic is that it can make testing easier.

本好处是考虑CQRS的主要因素

#### Flexibility

> The flexibility of a solution that uses the CQRS pattern largely derives from the separation into the read-side and the write-side models. It becomes much easier to make changes on the read side, such as adding a new query to support a new report screen in the UI, when you can be confident that you won't have any impact on the behavior of the business logic. On the write side, having a model that concerns itself solely with the core business logic in the domain means that you have a simpler model to deal with than a model that includes read logic as well.
>
> In the longer term, a good, useful model that accurately describes your core domain business logic will become a valuable asset. It will enable you to be more agile in the face of a changing business environment and competitive pressures on your organization.
>
> This flexibility and agility relates to the concept of continuous integration in DDD

#### Focus on the business

> If you use an approach like CRUD, then the technology tends to shape the solution. Adopting the CQRS pattern helps you to focus on the business and build task-oriented UIs. A consequence of separating the different concerns into the read side and the write side is a solution that is more adaptable in the face of changing business requirements. This results in lower development and maintenance costs in the longer term.

#### Facilitates building task-based UIs

> When you implement the CQRS pattern, you use commands \(often from the UI\), to initiate operations in the domain. These commands are typically closely tied to the domain operations and the ubiquitous language. For example, "book two seats for conference X." You can design your UI to send these commands to the domain instead of initiating CRUD-style operations. This makes it easier to design intuitive, task-based UIs.



## 实现

其实实现这个模式要先转变观念。。。否则会很不顺，虽然不知道是实际该怎么搞



ES \(event souring\)和CQRS常常放在一起来探讨的

https://msdn.microsoft.com/en-us/library/jj591559.aspx

>

_一个常规的预订过程_

![](https://msdn.microsoft.com/en-us/library/jj591559.7cd90e4af9a90ebd67bb8705e7b99d27%28l=en-us%29.png)

_基于ES的预订过程_![](https://msdn.microsoft.com/en-us/library/jj591559.74859689341057840d3a4e0a518e9cfd%28l=en-us%29.png)



通常通过DDD来实现

_**Commands** are imperatives : _they are requests for the system to perform a task or action.

_**Events** are notifications : _they report something that has already happened to other interested parties.

Both commands and events are types of **message** that are used to exchange data between objects. In DDD terms, these messages represent business behaviors and therefore help the system capture the business intent behind the message.



## 一个🌰

_下图展示了一个CQRS模式的示意_![](https://msdn.microsoft.com/en-us/library/jj591573.82fd3f191997792becb22a9283ce26b1%28l=en-us%29.png)

* 很多系统的读写负载并不均衡，读写分离可以使得分别优化成了可能。
* 写命令往往是复杂的，包括很多验证，数据校检。而写模型相对简单。所以分离开来会更加安全稳定。
* 这种分离也可以应用在数据库层面，数据库为读写操作做一定的优化



_通过事件来同步，通知_![](https://msdn.microsoft.com/en-us/library/jj591573.da82141c6f9950d64c1263fa4da825ec%28l=en-us%29.png)







