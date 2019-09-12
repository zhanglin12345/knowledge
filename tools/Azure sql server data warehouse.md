	1. 不支持高并发，可以短时间内处理大量数据。不适合 高并发查询和插入，不适合一次大量读取，不适合高并发小数据量写入
	2. smallRC，largeRC就是并发槽的大小。large支持的并发数会多一些
	3. 收费高，所以只在需要的时候启动。平时只需要暂停
	4. Control node - compute node - storage node
	5. 不推荐：最好不要update
	6. 不推荐：用合适的key来分布数据，这样会把数据均匀的分布。如果不均匀分布会性能不好，为什么？
	7. 不推荐：查询用的key跟数据分布的key不一样
	8. 支持row store和column store 在看下官网的介绍。感觉report更加适合column store，因为数据量大，那么column store适合聚合么？
	9. DW并不会自动记录统计数据。可以自己手动去加：create statistics。。。
	10. Join column做为distribution key，filter column做为partition key
	11. 事实表更适合，纬度表并不好
	12. 不适合做direct query，适合做计算之后的query


	1. Column Store是否适合我们的场景？表记录了历史数据会不断增长，聚合查询，和明细查询
	2. 历史数据，column store更适合？对数据做filter，多个表join，做聚合查询或者明细查询？

坑：
	1. 
	
	
