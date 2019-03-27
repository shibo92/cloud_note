### 责任链模式
+ 责任链模式和组件化很像，将各个功能块单独抽离出来，即插即用

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

### redis两种持久化方式
+ RDB 和 AOF
  1. RDB: 快照形式是直接把内存中的数据保存到一个dump文件中，定时保存，一次性写入
  2. 把所有的对redis的服务器进行修改的命令都存到一个文件里，命令的集合
 
### Lock 和 Synchronized区别
+ Synchronized是虚拟机关键字，LocK是个类
+ Synchronized自动释放锁，Lock需要在final中手动释放
+ Synchronized不能判断是否获取到锁，Lock可以判断是否获取到锁
+ synchronized的锁可重入、不可中断、非公平，而Lock锁可重入、可判断、可公平（两者皆可）
+ Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题

### String类为什么是final的
+ 只有当字符串是不可变的，字符串池才有可能实现。字符串池的实现可以在运行时节约很多heap空间，因为不同的字符串变量都指向池中的同一个字符串。但如果字符串是可变的，那么String interning将不能实现，因为这样的话，如果变量改变了它的值，那么其它指向这个值的变量的值也会一起改变。
+ 因为字符串是不可变的，所以是多线程安全的，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。
+ 因为字符串是不可变的，所以在它创建的时候HashCode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

### 对象类型数据和实例数据
+ 对象类型数据就是被虚拟机加载的`类信息`（即Class信息，见2.2.5方法区）
+ 对象实例数据就是被new出来的对象信息。

### java永久代
+ jdk1.7之前永久代其实就是方法区，存放`常量`、`静态变量`、`字符串常量池`、`类信息`等。
+ jdk1.8之后永久代被替换成元空间，元空间使用的是本地内存，不在虚拟机中

### redis和db一致性解决方案
+ 类似mysql的主从复制原理，使用binlog解决。具体实现有阿里的canal
