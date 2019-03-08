### 责任链模式
1.  责任链模式和组件化很像，将各个功能块单独抽离出来，即插即用

### springcloud 和 dubbo的区别
+ 暴露的接口层不一样
  1. dubbo暴露的是service层的接口，使用的方式为：消费者的controller层调用服务提供者的service.
  2. springcloud暴露的是controller接口，使用的方式为：消费者的service层调用服务者的controller。（类似于http方式调用）
+ 一个是基于rpc，一个是基于http
+ 传输协议不一样
  1. dubbo由于是二进制的传输，占用带宽会更少
  2. springCloud是http协议传输，带宽会比较多，同时使用http协议一般会使用JSON报文，消耗会更大
+ dubbo的开发难度较大，原因是dubbo的jar包依赖问题很多大型工程无法解决
+ dubbo的注册中心可以选择zk,redis等多种，springcloud的注册中心只能用eureka或者自研

### 分布式-微服务-集群的区别
+ 集群:同一套项目跑在不同机器上，实现服务的负载均衡。集群模式需要做好session共享，确保在不同服务器切换的过程中不会因为没有获取到session而中止退出服务。
+ 分布式：多个业务模块（如订单模块、登录注册模块）分别部署到不同的机器上。注：分布式需要做好事务管理
+ 微服务：将业务模块细化（如订单支付、退款，登录、注册）微服务的设计是为了不因为某个模块的升级和BUG影响现有的系统业务。
+ 微服务与分布式的细微差别是，微服务的应用不一定是分散在多个服务器上，他也可以是同一个服务器。

### mysql orderBy的优化
+ 问题 `select * from t where xxx limit xx,xx` 加上`orderBy`之后查询速度下降。
  - 慢的原因主要是排序，尤其是分片的排序，mycat会在所有分片进行排序操作取limit50，然后在mycat内存中再次排序取limit50。
  如果不排序的话，mycat只需要随便取一个分片的50条即可，这个计算量差别是很大的，分片越多越慢按主键排序的话，innodb的索引是带有主键的，所以where加order是可以走索引（覆盖索引），前提是select不能有索引字段以外的列
+ 解决方法：
  1. 加索引
  2. 嵌套一个子查询，只用来查询有索引的字段
    ```mysql
	    SELECT a.id, a.emailto, a.channel, a.AppName, a.AmazonOrderId 
	    FROM eis_email_history a JOIN (
			SELECT id FROM eis_email_history 
	        WHERE AppName = 21 AND custidStatus IN (0, 1, 2) AND channel = '*' ORDER BY id DESC LIMIT 50
		) b ON a.id=b.id;
    ```
	[链接地址](https://explainextended.com/2010/08/24/20-latest-unique-records/)


