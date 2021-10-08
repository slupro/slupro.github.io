## 分布式系统设计的几大难点

1. 数据一致性分发 data consistency
2. 数据聚合 data aggregation(join)
3. 分布式事务 distributed transaction
4. 单体系统解耦拆分 split a monolithic system

### 数据一致性分发

在分布式系统中，一个场景是某服务的数据在写入DB时，还需要提供给其它多个服务使用，通常采用 push 的方式。由于给多个service提供数据，所以需要考虑到数据一致性。

常见的解决方案有：

1. 事务性发件箱 Transactional outbox
2.  捕获变更数据 Change Data Capture(CDC)

#### Transactional outbox

在把数据写入DB时，同时往DB中的某个专门数据库/表(OUTBOX table)写入需发送的数据，由专门的Message Relay读取并发送。

![](/assets/img/distributed-system-design/2021-10-03-23-11-14.png)

参考：https://github.com/killbill/killbill-commons/tree/master/queue

#### Change Data Capture(CDC)

监控DB的transaction log，再进行发送。

![](/assets/img/distributed-system-design/2021-10-03-23-24-58.png)

参考：

1. 阿里Canal. https://github.com/alibaba/canal
2. Redhat Debezium. https://debezium.io
3. Zendesk Maxwell. https://github.com/zendesk/maxwell

|  | Transactional outbox | CDC |
| :-----| ----: | :----: |
| 复杂性 | 相对简单 | 复杂(高可用/监控) |
| pulling延迟和开销 | 近实时，有一定性能开销 | 教实时，性能开销小 |
| 应用侵入性 | 有 | 无 |
| 使用场合 | 早期/中小规模 | 中大规模，有独立团队治理维护 |

### 数据聚合join的问题

分布式系统数据聚合的问题，实际上是Aggregator或者Backend for Frontend(BFF)的问题。

#### Command Query Responsibility Segregation(CQRS)

CQRS是服务层的读写分离。Command改变数据/状态，Query不会改变数据/状态。具体实现手段主要依赖消息队列、事务性发件箱或者变更数据捕获(CDC)。

![](/assets/img/distributed-system-design/2021-10-04-17-11-56.png)

> CQRS会存在数据延迟，只满足最终一致性。

由于有延迟，所以UI可以使用一些更新策略：

1. 乐观更新 Optimistic update：UI在用户操作后不用等后台返回，即更新UI。
2. 拉模式 Pull：UI请求时带一个版本号，直到后台返回的数据版本号和请求的版本号一致。
3. publish-subscribe：通过websocket等通知UI更新页面。

### 分布式事务

#### 事务的要求 ACID

1. Atomicity: 要么全部发生，要么全部不发生。All or Nothing.
2. Consistency: 数据始终处于合法状态。
3. Isolation: 并发操作的可见性。
4. Durability: 变更持久化。

#### 分布式中事务的处理原则

需要考虑下来事项：

1. 假定网络或服务不可靠
2. 将全局事务建模成一组本地ACID事务
3. 引入事务补偿机制处理失败场景
4. 事务始终处于一种明确的状态（成功或失败）
5. 最终一致性
6. 考虑隔离性
7. 考虑幂等性 idempotence
8. 异步响应，避免直接同步调用

#### 2PC/XA

两阶段提交，The open group定义了XA规范。但存在性能低、实现复杂、deadlock、扩展性等问题。NoSQL、kafka等很多不支持2PC协议，所以应尽量避免使用。

1. Preparation phase
2. Commit or rollback phase

![](/assets/img/distributed-system-design/2021-10-05-11-24-41.png)

#### Try, Commit and Cancel(TCC)

TCC是2PC的一种变体，可以认为是服务层的2PC。流程：

1. 业务应用启动事务。
2. 业务应用调用Try接口，Try通常会执行资源预留的动作，比如机票、酒店预留。
3. 根据情况，调用Commit或者Cancel。

![](/assets/img/distributed-system-design/2021-10-05-11-30-29.png)

#### Saga

Saga模式，也是使用一种补偿机制，使用状态机来表示各个状态。

![](/assets/img/distributed-system-design/2021-10-05-12-35-11.png)

#### 业界实践（推荐Cadence）

1. 阿里的 Seata 通用分布式事务框架：Seata 是2PC/XA的一个改进版本，可以支持AT Mode(推荐), TCC, Saga, Even XA, Hybrid AT&TCC。
2. Uber的微服务编排引擎Cadence：Golang实现，提供了java和golang的接口。