1. rest协议
Representational State Transfer (REST) is an architectural style that defines a set of constraints and properties based on HTTP
2. TiDB
    CAP全满足？怎么看
    a. 这个表的数据会分散在不同的机器上」这个前提
    b. 存储
        表的数据在 TiDB 内部会被底层存储 TiKV 切分成很多 64M 的 Region（对应 Spanner 的 Splits 的概念），每个 Region 里面存储的都是连续的行，Region 是 TiDB 进行数据调度的单位，随着一个 Region 的数据量越来越大和时间的推移，Region 会分裂/合并，或者移动到集群中不同的物理机上，使得整个集群能够水平扩展。
    c. 时钟漂移
2. TiDB优化
    a. 建议：
    尽可能批量写入，但是一次写入总大小不要超过 Region 的分裂阈值（64M），另外 TiDB 也对单个事务有大小的限制。
    存储超宽表是比较不合适的，特别是一行的列非常多，同时不是太稀疏，一个经验是最好单行的总数据大小不要超过 64K，越小越好。大的数据最好拆到多张表中。
    对于高并发且访问频繁的数据，尽可能一次访问只命中一个 Region，这个也很好理解，比如一个模糊查询或者一个没有索引的表扫描操作，可能会发生在多个物理节点上，一来会有更大的网络开销，二来访问的 Region 越多，遇到 stale region 然后重试的概率也越大（可以理解为 TiDB 会经常做 Region 的移动，客户端的路由信息可能更新不那么及时），这些可能会影响 .99 延迟；另一方面，小事务（在一个 Region 的范围内）的写入的延迟会更低，TiDB 针对同一个 Region 内的跨行事务是有优化的。另外 TiDB 对通过主键精准的点查询（结果集只有一条）效率更高。
3. 可培养的可能性？
