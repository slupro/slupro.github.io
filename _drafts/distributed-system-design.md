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
