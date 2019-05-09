
<!-- vim-markdown-toc GFM -->

* [责任链模式](#责任链模式)
* [springcloud 和 dubbo的区别](#springcloud-和-dubbo的区别)
* [分布式-微服务-集群的区别](#分布式-微服务-集群的区别)
* [mysql orderBy的优化](#mysql-orderby的优化)
* [redis两种持久化方式](#redis两种持久化方式)
* [Lock 和 Synchronized区别](#lock-和-synchronized区别)
* [String类为什么是final的](#string类为什么是final的)
* [对象类型数据和实例数据](#对象类型数据和实例数据)
* [java永久代](#java永久代)
* [redis和db一致性解决方案](#redis和db一致性解决方案)
* [http解析流程](#http解析流程)
* [class.forName()和classLoader区别](#classforname和classloader区别)
* [redis i/o多路复用](#redis-io多路复用)
* [对象进入老年代的条件](#对象进入老年代的条件)
* [mysql主从同步的工作过程](#mysql主从同步的工作过程)
* [过滤器（filter）和拦截器（interceptor）区别](#过滤器filter和拦截器interceptor区别)
* [scp上传下载命令](#scp上传下载命令)
* [mysql orderBy优化](#mysql-orderby优化)
* [mysql突然无法登录（ERROR 1698 (28000): Access denied for user 'root'@'localhost'）](#mysql突然无法登录error-1698-28000-access-denied-for-user-rootlocalhost)
* [nohup只输出错误日志](#nohup只输出错误日志)

<!-- vim-markdown-toc -->

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

### http解析流程
+ http解析
+ 缓存
+ dns域名解析

### class.forName()和classLoader区别
+ class.forName()前者除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块。
+ classLoader只是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。

### redis i/o多路复用
+ 假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：
  1. 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡主，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。
  2. 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者线程处理连接。
  3. 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。。。 这种就是IO复用模型，Linux下的select、poll和epoll就是干这个的。
  4. 将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。

### 对象进入老年代的条件
1. 大对象直接进入老年代
2. 到达年龄之后进入老年代
3. 相同年龄的对象大小之和大于survivor(幸存区)大小的的一半，所有相同年龄的对象都进入老年代

### mysql主从同步的工作过程 
  1. 主库上会开启了二进制bin-log日志记录，同时运行有一个IO线程；
  2. 主库上对于需要同步的数据库或者表所发生的所有DML操作都会被记录到bin-log二进制日志文件中；
  3. 从库上开启relay-log日志，同时运行有一个IO线程和一个SQL线程；
  4. IO线程负责从主库中读取bin-log二进制日志，并写入到本地的relay-log日志中，同时记录从库所读取到的主库的日志文件位置信息，以便下次从这个位置点再次读取；
  5. SQL线程负责从本地的relay-log日志中读取同步到的二进制日志，并解析为数据库可以识别的SQL语句，然后应用到本地数据库，完成同步；
  6. 执行完relay-log中的操作之后，进入睡眠状态，等待主库产生新的更新；

### 过滤器（filter）和拦截器（interceptor）区别 
  1. filter基于filter接口中的doFilter回调函数，interceptor则基于Java本身的反射机制； 
  2. filter是依赖于servlet容器的，没有servlet容器就无法回调doFilter方法，而interceptor与servlet无关； 
  3. filter的过滤范围比interceptor大，filter除了过滤请求外通过通配符可以保护页面、图片、文件等，而interceptor只能过滤请求，只对action起作用，在action之前开始，在action完成后结束（如被拦截，不执行action）； 
  4. filter的过滤一般在加载的时候在init方法声明，而interceptor可以通过在xml声明是guest请求还是user请求来辨别是否过滤； 
  5. interceptor可以访问action上下文、值栈里的对象，而filter不能； 
  6. 在action的生命周期中，拦截器可以被多次调用，而过滤器只能在容器初始化时被调用一次。

### scp上传下载命令
  scp [参数] [原路径] [目标路径]
  1. 上传
  命令格式：scp local_file remote_username@remote_ip:remote_folder 
  例如： scp  -P 40022 /Users/shibo/local/huawei_unsigned_signed.apk root@ip:/home/qiban/bzit_app/ 
  2. 下载
  例如： scp root@ip:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/

### mysql orderBy优化
  问题 select * from t where xxx limit xx,xx 加上orderBy之后查询速度下降。
  
  慢的原因主要是排序，尤其是分片的排序，
  mycat会在所有分片进行排序操作取limit50，然后在mycat内存中再次排序取limit50
  如果不排序的话，mycat只需要随便取一个分片的50条即可，这个计算量差别是很大的，分片越多越慢
  按主键排序的话，innodb的索引是带有主键的，
  所以where加order是可以走索引（覆盖索引），前提是select不能有索引字段以外的列
  
  解决方案：
  1、加索引
  2、嵌套一个子查询，只用来查询有索引的字段
  SELECT 
  a.id,
  a.emailto,
  a.channel,
  a.AppName,
  a.AmazonOrderId 
  from eis_email_history a join 
  (select id FROM eis_email_history WHERE AppName = 21 AND custidStatus IN (0, 1, 2) AND channel = '*' 
  ORDER BY id DESC LIMIT 50
  ) b on a.id=b.id;
  
  链接地址：https://explainextended.com/2010/08/24/20-latest-unique-records/

  1. dubbo暴露的是service层的接口，使用的方式为：消费者的controller层调用服务提供者的service.
  springcloud暴露的是controller接口，使用的方式为：消费者的service层调用服务者的controller。（类似于http方式调用）
  2. 一个是基于rpc，一个是基于http
  3. dubbo由于是二进制的传输，占用带宽会更少
  springCloud是http协议传输，带宽会比较多，同时使用http协议一般会使用JSON报文，消耗会更大
  4. dubbo的开发难度较大，原因是dubbo的jar包依赖问题很多大型工程无法解决
  5. dubbo的注册中心可以选择zk,redis等多种，springcloud的注册中心只能用eureka或者自研

### mysql突然无法登录（ERROR 1698 (28000): Access denied for user 'root'@'localhost'）
  + 原因是mysq自动把插件改成了`auth_socket`；
  + 解决方法： 
	1. sudo 进入 mysql， 执行语句
	```mysql
    update mysql.user set authentication_string=PASSWORD(''), plugin='mysql_native_password' where user='root';
	flush privileges;
	```
	2. 重启服务

### nohup只输出错误日志
  +  nohup ./xxx.sh >/dev/null 2>xx.log &
  + 关于Linux的3中重定向
	- 0:表示标准输入
    - 1:标准输出,在一般使用时，默认的是标准输出
 	- 2:标准错误信息输出
  + 关于/dev/null文件
	- Linux下还有一个特殊的文件/dev/null，它就像一个无底洞，所有重定向到它的信息都会消失得无影无踪。这一点非常有用，当我们不需要回显程序的所有信息时，就可以将输出重定向到/dev/null。

