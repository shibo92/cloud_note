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

### Lock 和 Synchronized区别
+ Synchronized是虚拟机关键字，LocK是个类
+ Synchronized自动释放锁，Lock需要在final中手动释放
+ Synchronized不能判断是否获取到锁，Lock可以判断是否获取到锁
+ synchronized的锁可重入、不可中断、非公平，而Lock锁可重入、可判断、可公平（两者皆可）
+ Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题

### String类为什么是final的
+ 只有当字符串是不可变的，字符串池才有可能实现。字符串池的实现可以在运行时节约很多heap空间，因为不同的字符串变量都指向池中的同一个字符串。但如果字符串是可变的，那么String intern将不能实现，因为这样的话，如果变量改变了它的值，那么其它指向这个值的变量的值也会一起改变。
+ 因为字符串是不可变的，所以是多线程安全的，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。
+ 因为字符串是不可变的，所以在它创建的时候HashCode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

### 对象类型数据和实例数据
+ 对象类型数据就是被虚拟机加载的`类信息`（即Class信息，见2.2.5方法区）
+ 对象实例数据就是被new出来的对象信息。

### java永久代
+ jdk1.7之前永久代其实就是方法区的一种实现方式，存放`常量`、`静态变量`、`字符串常量池`、`类信息`等。
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

### 对象进入老年代的条件
1. 大对象直接进入老年代
2. 到达年龄之后进入老年代
3. 相同年龄的对象大小之和大于survivor(幸存区)大小的的一半，所有相同年龄的对象都进入老年代
3. 老年代空间担保策略失败

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
  + 原因是mysq自动把插件改成了"auth_socket"；
  + 解决方法：
	1. sudo 进入 mysql， 执行语句
	```
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

### mybatis原理
  + 初始化阶段
    1. 创建SqlSessionFactory实例;
    2. 实例化过程中，加载配置文件创建configuration对象(mybatis-config.xml);
    3. 通过factory创建SqlSession对象,把configuaration传入SqlSession;
    4. 通过SqlSession获取并初始化mapper接口动态代理; // UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    5. spring ioc会接管getMapper生成的代理对象并注入
  + 执行阶段
    5. 通过代理对象调用sqlsession的查询方法; // User user = mapper.findById(1);
    6. sqlsession将查询方法转发给executor;
    7. executor基于JDBC访问数据库获取数据;
    8. executor通过反射将数据转换成POJO并返回给sqlsession;
    9. 数据返回给调用者

### 高并发秒杀解决方案
  + 转自知乎：https://www.zhihu.com/question/54895548

### solr参数说明
  + q : 查询字段，使用方式field:query_content，filed相当于sql中where后边的column,query_content相当于条件值
  + dq: 过滤条件，在q的结果集中加一层过滤，也相当于where
  + fl: 需要显示的field，相当于column
  + start: 分页
  + rows: 每页记录数
  + sort: 排序
  + wt: (writer type)指定输出格式，可以有 xml, json, php, phps。 
  + fl: 表示索引显示那些field( *表示所有field,如果想查询指定字段用逗号或空格隔开（如：Name,SKU,ShortDescription或Name SKU ShortDescription【注：字段是严格区分大小写的】）) 
  + q.op 表示q 中 查询语句的 各条件的逻辑操作 AND(与) OR(或) 
  + hl 是否高亮 ,如hl=true
  + hl.fl 高亮field ,hl.fl=Name,SKU
  + hl.snippets :默认是1,这里设置为3个片段
  + hl.simple.pre 高亮前面的格式 
  + hl.simple.post 高亮后面的格式 
  + facet 是否启动统计 

### springmvc执行流程
  1. `DispatcherServlet`继承自`HttpServletBean`，首先要执行其中的init方法来初始化`web.xml`中的内容
  2. 执行`doDispatch`方法（主要）
  3. 通过request去一个`HandlerMappings`的List中获取对应的`mappedHandler`, 同时，`mappedHandler`会将请求封装成一个`HandlerMethod`，这个是最后要执行的类
  4. 检查所有注册的`HandlerAdapter`，通过`mappedHandler`获取对应的`HandlerAdapter`
     - 执行HandlerAdapter之前会去执行相应的`Interceptor`，拦截器返回正常之后，才去执行下边的内容
  5. 通过执行`HandlerAdapter`的`handle`方法执行刚才拿到的`HandlerMethod`，返回一个ModelAndView
  6. 调用viewResolver将mv的内容`out.write到客户端`

### tomcat启动慢
  + 添加参数-Djava.security.egd=file:/dev/./urandom，加快随机数产生过程

### spring的Beanfactory和ApplicationContext区别
  + beanfactory加载的bean只有用到的时候才会实例化（懒加载）
  + ApplicationContext立即加载（finishBeanFactoryInitialization方法）

### mybatis有则更新，无则插入的关键字
   + ON DUPLICATE KEY UPDATE 
  + link: https://segmentfault.com/p/1210000020019645

### 线程池任务队列(workQueue)
4.任务缓存队列及排队策略 
  + workQueue的类型为BlockingQueue，通常可以取下面三种类型：
    1. ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小； 
    2. LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；
    3. synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
  + 根据《阿里编码规约》，Executors创建的线程池都是Linked方式或Synchronous方式，所以建议使用`ThreadPoolExecutor`手动创建线程池,并设置workQueue为`ArrayBlockingQueue`

### 线程池拒绝策略 RejectedExecutionHandler
  1. AbortPolicy: 丢弃任务并抛出RejectedExecutionException异。(默认)
  2. DiscardPolicy: 直接丢弃任务，不抛出异常。
  3. DiscardOldestPolicy：丢弃队列最前面的任务，执行新加入的任务
  4. CallerRunsPolicy：由调用线程处理该任务 

### ubuntu 修改wine分辨率
 + WINEPREFIX=~/.deepinwine/Deepin-WeChat deepin-wine winecfg

### scp上传下载
 + scp [参数] [原路径] [目标路径]
  - 上传
   1. 命令格式：scp local_file remote_username@remote_ip:remote_folder 
   2. 例如： scp  -P 40022 /Users/shibo/local/huawei_unsigned_signed.apk root@ip:/home/qiban/bzit_app/
  - 下载
   1. 例如： scp root@ip:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/
   2. 参数：-v 查看进度 
        -P 端口
        -r 传输目录

### 内存模型(JMM)
  + 了保证并发编程中可以满足原子性、可见性及有序性。有一个重要的概念，那就是——内存模型
  + 关键字：volatile、synchronized、final、concurren

### mysql索引最左匹配原则的理解
  + 因为b+tree在b-tree的基础上增加了顺序访问指针，且只有叶子节点存储数据，所以在访问数据时，需要从左到右遍历索引，最左不匹配的话则停止访问

### URI和URL
  + URL是URI的子集
  + URI只要可以标记一个资源就可以，不管是以什么方式，比如urn:isbn:0-486-27557-4，这个是一本书的isbn
  + URL就是使用服务器路径来定位一个资源

### activemq发布订阅保证消息可靠
  + 这个情况比较多，理论来说使用topic 加持久化可保证服务器消息可靠，客户端使用Producer.DeliveryMode = MsgDeliveryMode.Persistent，来保证客户端消息持久化

### activemq保证顺序
  + 通过高级特性consumer独有消费者（exclusive consumer）
  + 利用Activemq的高级特性：messageGroups

### ubuntu自定义桌面应用路径
  + /usr/share/application

### dubbo加载配置文件标签
  1. 通过DubboNamespaceHandler初始化各种标签对应的处理类（Parser）
  2. 由各自的处理类去做转换，比如`DubboBeanDefinitionParser`

### zip乱码解决方案
  + unzip -O GBK xxx.zip

### 学习方法论
 + 将自己学习的新内容讲给别人，并且能让别人理解

### eureka缓存机制
 + eureka是没有用数据库去存储数据的，而是使用了双层的concurrentHashMap存储服务信心
 + 第一层的key为`spring.application.name`,即服务名称,只读
   1. `readOnlyCacheMap`本质上是 ConcurrentHashMap，依赖定时从`readWriteCacheMap`同步数据，默认时间为 30 秒;
   2. 主要是为了供客户端获取注册信息时使用，其缓存更新，依赖于定时器的更新，通过和 readWriteCacheMap 的值做对比，如果数据不一致，则以 readWriteCacheMap 的数据为准;
 + 第二层的key为`实例的唯一ID`，一个服务可以拥有多个实例;
   1. readWriteCacheMap 的数据主要同步于存储层。当获取缓存时判断缓存中是否没有数据，如果不存在此数据，则通过 `CacheLoader` 的 load 方法去加载，加载成功之后将数据放入缓存，同时返回数据。
   2. `readWriteCacheMap` 缓存过期时间，默认为 180 秒，当服务下线、过期、注册、状态变更，都会来清除此缓存中的数据;
 + Eureka Client 获取全量或者增量的数据时，会先从一级缓存中获取；如果一级缓存中不存在，再从二级缓存中获取；如果二级缓存也不存在，这时候先将存储层的数据同步到缓存中，再从缓存中获取。
 + 这样做可以提高节点获取数据时的响应效率

### 系统异常监控sentry
### mysql or查询不生效
 1. 使用union代替or
 2. 可能是表数据量太小，mysql有索引优化，详见：https://dev.mysql.com/doc/refman/5.6/en/index-merge-optimization.html

### mysql-Employees Sample table download
 + https://dev.mysql.com/doc/employee/en/employees-installation.html
 + git clone git@github.com:datacharmer/test_db.git

### CAS存在的问题
 1. ABA问题
   + 原值为A，改为B，又改回A，CAS检测时会认为没有变化。
   + 解决方案：修改时加版本号，如：1A->2B->3A
 2. 循环开销时间大
 3. 只能保证一个共享变量的原子操作，无法保证多个变量的原子性
   + jdk1.5提供了AtomicRefrence类，可以吧多个变量放在一个对象里进行cas操作

### 自旋锁
 1. 避免了线程切换的开销
 2. 如果锁的占用时间很短，很适合用自旋锁，反之则不合适
 3. 自旋锁实现原理同样是cas，AtomicInteger中调用unsafe进行自增操作就是自旋锁`unsafe.getAndAddInt`，源码中是一个do-while循环, unsafe-->直接读内存

### 事务隔离级别
 1. 未提交读(read-uncommitted): 事务A未提交，事务B可读取A中已修改的内容
 2. 提交读(read-committed): 事务A提交后，事务B未提交，可看到A中刚修改的内容
 3. 可重复读(repeatable-read): 事务A提交修改，事务B也需要提交才能看到
 4. 串行化(Serializable): 事务A没有提交，事务B不能进行修改
 5. 参考：https://blog.csdn.net/zhouym_/article/details/90381606

### 事务隔离级别（理解）
 1. 未提交读(read-uncommitted): 事务A未提交，事务B可及时读取(脏读、不可重复读、幻读)
 2. 提交读(read-committed): 事务A已提交，事务B会看到最新版本的快照(不可重复读、幻读)
 3. 可重复读(repeatable-read): 事务A提交修改，事务B始终读取第一次快照(幻读)
 4. 串行化(Serializable): 事务A没有提交，事务B不能进行修改

### 聚簇索引和非聚簇索引
 + 聚簇索引(InnoDB): 
   - 非叶子节点存储索引列，叶子节点存储数据，二级索引(辅助索引)为非聚簇索引，叶子节点存储聚簇索引列
   - 优点
     1. 移动行时二级索引无需更新，因为二级索引维护的是对应的id
   - 缺点
     1. 更新聚簇列时，会强制移动被更新的行数据到新的位置，因为索引位置变了，数据必须跟着移动
 + 非聚簇索引
   - 非叶子节点存储索引列，叶子节点存储数据行的物理地址
### 将vim复制的内容粘贴到vim之外
 + 同时按下 shift " + y 四个按键

### 组合和聚合
 + 组合
   - 人和手脚、头部。人消失后其他也消失
   - 组合符号是空心菱形
 + 聚合
   - 人和电脑。人消失后电脑还存在，电脑是通过set方法聚合到人这个类当中的。
   - 聚合符号是实心菱形
### 各类加载器执行的目录
 + BootStrapClassLoader - $JAVA_HOME/jre/lib/rt.jar或者自定义/jre/classes
 + Extension ClassLoader - $JAVA_HOME/jre/lib/ext/classes
 + Application ClassLoader - 用户类所在路径(classpath)
### 提高反射性能
 + 反射后加入到缓存当中，避免forName的耗时
 + setAccessible(true) - 禁用安全检查
### 字段基数和索引效率的关系
 + count(DISTINCT(column))/count(*) 越大，索引效果越好

### 通过跳板机上传 /下载文件
 + install zssh
 + 用类似ssh的方式连接服务器：zssh username@ip
 + 上传文件
   - 先用组合键ctrl+2进入zssh：
   - zssh > ls // 看一下有哪些文件，可以用shell命令切换文件夹等
   - zssh > sz abc.py // 将abc.py传到远程机器当前目录下
 + 下载文件，在服务器上：
   - sz abcde.py // 准备好要下载的文件
   - 然后ctrl+2进入zssh：
   - zssh > ls // 选择下载到的目录
   - zssh > rz // 下载对应的文件

### loadClass和class.forName在虚拟机层面的区别
 + Class.forName(className)装载的class已经被初始化，也就是到了类加载的初始化阶段
 + ClassLoader.loadClass(className)装载的class还没有被link
 + configInfo https://blog.csdn.net/dataiyangu/article/details/86321678

### newInstance效率比new差的原因之一
 + 由于反射所涉及的类型是动态解析的，因此无法执行某些java虚拟机优化，因此反射操作的性能比非反射操作慢，应该在性能敏感的应用程序中频繁调用的代码部分中避免使用反射操作

### new一个对象的步骤
 1. 当虚拟机遇到一条new指令时候，首先去检查这个指令的参数是否能 在常量池中能否定位到一个类的符号引用 （即类的带路径全名），并且检查这个符号引用代表的类是否已被加载、解析和初始化过，即验证是否是第一次使用该类。如果没有（不是第一次使用），那必须先执行相应的类加载过程（class.forname()）。
 2. 在类加载检查通过后，接下来虚拟机将 为新生的对象分配内存 。对象所需的内存的大小在类加载完成后便可以完全确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来，目前常用的有两种方式，根据使用的垃圾收集器的不同使用不同的分配机制：
   - 指针碰撞（Bump the Pointer）：假设Java堆的内存是绝对规整的，所有用过的内存都放一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅把那个指针向空闲空间那边挪动一段与对象大小相等的距离。
   - 空闲列表（Free List）：如果Java堆中的内存并不是规整的，已使用的内存和空间的内存是相互交错的，虚拟机必须维护一个空闲列表，记录上哪些内存块是可用的，在分配时候从列表中找到一块足够大的空间划分给对象使用。
 3. 内存分配完后，虚拟机需要将分配到的内存空间中的数据类型都 初始化为零值（不包括对象头）；
 4. 虚拟机要 对对象头进行必要的设置 ，例如这个对象是哪个类的实例（即所属类）、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息，这些信息都存放在对象的对象头中。
      至此，从虚拟机视角来看，一个新的对象已经产生了。但是在Java程序视角来看，执行new操作后会接着执行如下步骤：
 5. 调用对象的init()方法 ,根据传入的属性值给对象属性赋值。
 6. 在线程 栈中新建对象引用 ，并指向堆中刚刚新建的对象实例。

### 代理模式和适配器模式和装饰者模式区别
 + 整体组件结构一样
 + 不同点是
   1. 代理模式代理类和被代理类需要实现同一个接口，适配器模式只需要适配器去实现接口，realObject不需要实现
   2. 代理模式，作用是不把实现直接暴露给client，而是通过代理这个层，代理能够做一些处理
   3. 适配器模式，旧的接口不能用，为了将旧接口转换成我们使用的新接口
   4. 装饰器模式，原有的不能满足现有的需求，对原有的进行增强

### 红黑树和avl树区别
  1. 红黑树在左右平衡上非严格
  2. 减少了旋转次数

### vim保存readonly文件
  1. :w !sudo tee %

### junit
 + Deencapsulation 可以访问类的私有方法和属性
 + new Expectations() 可以构造预期结果

### zookeeper超时时间
 + 服务器在经过时间t之后收不到客户端的消息，则认为回话过期；
 + 而客户端在经过t/3时间后未收到服务器消息，就会向服务器发送心跳包；在经过2t/3之后，客户端会寻找其他服务器，此时还剩t/3时间去寻找和发包；

### zookeeper服务注册与发现
 1. 服务提供方启动 
   + 所谓注册服务，就是在ZooKeeper的/dubbo/com.foo.BarService/providers节点下创建一个子节点，并写入自己的URL地址，这就代表了com.foo.BarService这个服务的一个提供者。
 2. 服务消费者启动
   + 服务消费者在启动的时候，会向ZooKeeper注册中心订阅自己的服务。其实，就是读取并订阅ZooKeeper上/dubbo/com.foo.BarService/providers节点下的所有子节点，并解析出所有提供者的URL地址来作为该服务地址列表。 
   + 同时，服务消费者还会在ZooKeeper的/dubbo/com.foo.BarService/consumers节点下创建一个临时节点，并写入自己的URL地址，这就代表了com.foo.BarService这个服务的一个消费者。
 3. 消费者远程调用提供者
   + 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一个提供者进行调用，如果调用失败，再选另一个提供者调用。
 4. 增加服务提供者
   + 增加提供者，也就是在providers下面新建子节点。一旦服务提供方有变动，zookeeper就会把最新的服务列表推送给消费者。
 5. 减少服务提供者
   + 同上
 6. ZooKeeper宕机之后
   + 消费者每次调用服务提供方是不经过ZooKeeper的，消费者只是从zookeeper那里获取服务提供方地址列表。所以当zookeeper宕机之后，不会影响消费者调用服务提供者
   + 影响的是zookeeper宕机之后如果提供者有变动，增加或者减少，无法把最新的服务提供者地址列表推送给消费者，所以消费者感知不到

### 同步非阻塞的nio中包含阻塞的selector的理解
 + 同步阻塞io：指的是bio在进行accept、read、write时，是同步阻塞的，此时如果是单线程，且io没有数据处理，会被一直挂着
 + 同步非阻塞io: 单线程selector会处理多个io，当任何一个io有操作时，selector会被唤醒，所以在io1没数据时，io2可能被处理，所以线程并没有被io1阻塞
 + 总结：
   1. java nio说的同步非阻塞指的是对于io操作来说，而并非对于整个java nio 来说都是非阻塞的
   2. 因为java nio 的selector是阻塞的，虽然说selector是阻塞的但是它实现了多路复用，而且相对于传统io，nio还节省了线程，以及线程之间切换带来的系统消耗。
   3. 同步非阻塞的nio中包含阻塞的selector，只是关注的点不同而已，而且严格意义上将java nio应该说是多路复用，而不是同步非阻塞


### http长连接和短连接
  + 之所以说HTTP分为长连接和短连接，其实本质上是说的TCP连接。TCP连接是一个双向的通道，它是可以保持一段时间不关闭的，因此TCP连接才有真正的长连接和短连接这一说。
  + 长连接好处：长连接情况下，多个HTTP请求可以复用同一个TCP连接，这就节省了很多TCP连接建立和断开的消耗。

### 长轮询和短轮询
 + 长轮询和短轮询最大的区别是，短轮询去服务端查询的时候，不管库存量有没有变化，服务器就立即返回结果了。
 + 而长轮询则不是，在长轮询中，服务器如果检测到库存量没有变化的话，将会把当前请求挂起一段时间（这个时间也叫作超时时间，一般是几十秒）。在这个时间里，服务器会去检测库存量有没有变化，检测到变化就立即返回，否则就一直等到超时为止。

### LRU原理
 + LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。
 + 【命中率】
  - 当存在热点数据时，LRU的效率很好，但偶发性的、周期性的批量操作会导致LRU命中率急剧下降，缓存污染情况比较严重。
 + 【复杂度】
  - 实现简单。
 + 【代价】
  - 命中时需要遍历链表，找到命中的数据块索引，然后需要将数据移到头部。

### LRU-K原理
 + link : https://www.jianshu.com/p/d533d8a66795
 + 数据第一次被访问，加入到访问历史列表；
 + 如果数据在访问历史列表里后没有达到K次访问，则按照一定规则（FIFO，LRU）淘汰；
 + 当访问历史队列中的数据访问次数达到K次后，将数据索引从历史队列删除，将数据移到缓存队列中，并缓存此数据，缓存队列重新按照时间排序；
 + 缓存数据队列中被再次访问后，重新排序；
 + 需要淘汰数据时，淘汰缓存队列中排在末尾的数据，即：淘汰“倒数第K次访问离现在最久”的数据。


### mysql全表扫描比索引快的情况
 + 假设一张表含有10万行数据--------100000行
   我们要读取其中20%(2万)行数据----20000行
   表中每行数据大小80字节----------80bytes
   数据库中的数据块大小8K----------8000bytes

   所以有以下结果：
   每个数据块包含100行数据---------100行
   这张表一共有1000个数据块--------1000块

   通过索引读取20000行数据 = 约20000个table access by rowid = 需要处理20000个块来执行这个查询
   但是，请大家注意：整个表只有1000个块！
   所以：如果按照索引读取全部的数据的20%相当于将整张表平均读取了20次！！So，这种情况下直接读取整张表的效率会更高。
 + 通过索引去读数据，在索引中找到一个键值，通过键值对应的rowId去对应的`数据块`中读取记录，查询20000条记录，需要查20000个数据块，表中共1000个块，所以要读20次表。
 + 通过全表扫描，只需要顺序访问1000个数据块
 + 同时要考虑20%的数据的分布的离散程度，如这20%集中在前面的20%的块，也就是集中在前面的200块，那回表20000次也就是索引块+200块，索引块+200如果小于1000块，走索引扫描块

### mysql行锁死锁
 + 报错：Deadlock found when trying to get lock; try restarting transaction
 + CREATE TABLE `user_item` (
    `id` BIGINT(20) NOT NULL,
    `user_id` BIGINT(20) NOT NULL,
    `item_id` BIGINT(20) NOT NULL,
    `status` TINYINT(4) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `idx_1` (`user_id`,`item_id`,`status`)
    ) ENGINE=INNODB DEFAULT CHARSET=utf-8
 + 执行语句 ： update user_item set status=1 where user_id=? and item_id=?
 + 原因：行级锁并不是直接锁记录，而是锁索引，如果一条SQL语句用到了主键索引，mysql会锁住主键索引；如果一条语句操作了非主键索引，mysql会先锁住非主键索引，再锁定主键索引。
  - 这个update语句会执行以下步骤：
   1. 由于用到了非主键索引，首先需要获取idx_1上的行级锁
   2. 紧接着根据主键进行更新，所以需要获取主键上的行级锁；
   3. 更新完毕后，提交，并释放所有锁。
  - 如果在步骤1和2之间突然插入一条语句：update user_item .....where id=? and user_id=?,这条语句会先锁住主键索引，然后锁住idx_1。
 + 解决方案： 
  - 先获取需要更新的记录的主键 ： select id from user_item where user_id=? and item_id=?
  - 再根据主键id做update 

### zookeeper选举机制
 + 按顺序启动时，共n台服务器，选取第n/2台服务器（刚超过半数）
 + paxos协议

### es名词解释
 + index: 库
 + type: 表
 + document: 记录
 + field: 字段

### es查询表达式(https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html)
 + math：模糊匹配，term：精准匹配

### 熔断触发条件
 1. 失败总请求数：默认20
 2. 失败率：默认50%

### zookeeper崩溃选举规则
 1. 初始阶段，都会给自己投票。
 2. 当接收到来自其他服务器的投票时，都需要将别人的投票和自己的投票进行pk，规则如下：
   + 优先检查zxid。zxid比较大的服务器优先作为leader。
   + 如果zxid相同的话，就比较sid，sid比较大的服务器作为leader。

### win10激活
 1. slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
 2. slmgr /skms zh.us.to 
 3. slmgr /ato

### 大文件查找
## 查看文件大小
ls -lh |sort -nr
ls -Slrh

## 大于800M
find . -type f -size +800M

## 文件信息、属性
find . -type f -size +800M  -print0 | xargs -0 ls -l

## 具体大小
find . -type f -size +800M  -print0 | xargs -0 du -h

## 大小排序
find . -type f -size +800M  -print0 | xargs -0 du -h | sort -nr

## 查看磁盘情况
du -h --max-depth=1

##  二级目录
du -h --max-depth=2 | sort -n

## 只显示前12个
du -hm --max-depth=2 | sort -nr | head -12

## 查看未删除的句柄
lsof -n | grep deleted

### 大访问量ip查找
awk '{a[$1] += 1;} END {for (i in a) printf("%d %s\n", a[i], i);}' com.daojia.access.log.2021-01-04  |sort -nr |head -10

### 自定义注解加事务后失效的解决方案
  + 在自定义注解中添加 @Inherited

### xss 注入/防注入
  + case 
   - <details ontoggle="alert`23`"></details>
   - <iframe srcdoc="<script src=http://www.baidu.com/1.js></script>"</iframe>
  + 解决方案
    ` public static String XSSReplace(String htmlStr) {
    Pattern p = null; // 正则表达式
    Matcher m = null; // 操作的字符串
    StringBuffer tmp = null;
    String str = "";
    boolean isHave = false;

    String[] Rstr = { "meta", "script", "object", "embed","iframe" };
    if (htmlStr == null || !(htmlStr.length() > 0)) {
      return "";
    }
    str = htmlStr.toLowerCase();
    for (int i = 0; i < Rstr.length; i++) {
      p = Pattern.compile("<" + Rstr[i] + "(.[^>])*>");
      m = p.matcher(str);
      tmp = new StringBuffer();
      if (m.find()) {
        m.appendReplacement(tmp, "<" + Rstr[i] + ">");
        while (m.find()) {

          m.appendReplacement(tmp, "<" + Rstr[i] + ">");
        }
        isHave = true;
      }

      m.appendTail(tmp);
      str = tmp.toString();

      p = Pattern.compile("</" + Rstr[i] + "(.[^>])*>");
      m = p.matcher(str);
      tmp = new StringBuffer();
      if (m.find()) {
        m.appendReplacement(tmp, "</" + Rstr[i] + ">");
        while (m.find()) {
          m.appendReplacement(tmp, "</" + Rstr[i] + ">");
        }
        isHave = true;
      }
      m.appendTail(tmp);
      str = tmp.toString();

    }

    String[] Rstr1 = { "function", "window\\.", "javascript:", "script",
        "js:", "about:", "file:", "document\\.", "vbs:", "frame",
        "cookie", "onclick", "onfinish", "onmouse", "onexit=",
        "onerror", "onclick", "onkey", "onload", "onfocus", "onblur","ontoggle","srcdoc" };

    for (int i = 0; i < Rstr1.length; i++) {
      p = Pattern.compile("<([^<>])*" + Rstr1[i] + "([^<>])*>([^<>])*</([^<>])*>");

      m = p.matcher(str);
      tmp = new StringBuffer();
      if (m.find()) {
        m.appendReplacement(tmp, "");
        while (m.find()) {
          m.appendReplacement(tmp, "");
        }
        isHave = true;
      }
      m.appendTail(tmp);
      str = tmp.toString();
    }

    if (isHave) {
      htmlStr = str;
    }

    htmlStr = htmlStr.replaceAll("%3C", "<");
    htmlStr = htmlStr.replaceAll("%3E", ">");
    htmlStr = htmlStr.replaceAll("%2F", "");
    htmlStr = htmlStr.replaceAll("&#", "<b>&#</b>");
    return htmlStr;
    }`

### 关键词查找
  + find . -type f |xargs grep "北京五八到家"

### i/o多路复用和线程池对比
  + i/o多路复用，selector注册事件，单线程队列处理io，适用于io时间短的场景
  + io时间长的场景考虑用线程池

### arrayList线程不安全
  + 场景1
       1. 线程1 -> add -> 扩容 -> 停顿
       1. 线程2 -> add ->发现不需要扩容,element[size++] = e
       1. 线程1 -> element[size++] = e，下标越界
  + 场景2
    1. 线程1 -> add -> 扩容 -> size++ 中途停顿
    2. 线程2 -> add -> 扩容 -> element[size++] = e, 赋值成功, element[1] = e
    3. 线程1 -> element[size++] = e, 赋值成功, element[1] = e，覆盖成功

### 线程池什么时候回收非核心线程
  1. 在没有后续任务产生的情况下，空闲线程等待`keepAliveTime`秒后被回收
  2. 如果有后续任务，则空闲线程一直会轮询等待执行任务

### mysql binlog复制类型
  + 复制类型
    1. 基于语句的复制
    在Master上执行的SQL语句，在Slave上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。一旦发现没法精确复制时，会自动选着基于行的复制
    2. 基于行的复制
    把改变的内容复制到Slave，而不是把命令在Slave上执行一遍。从MySQL5.0开始支持
    3. 混合类型的复制
   + 默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制
   + 相应地，binlog的格式也有三种：STATEMENT，ROW，MIXED。

### 定位cpu消耗高的sql
  1. pidstat -t -p <mysqld_pid> 1  5
  2. select * from performance_schema.threads where thread_os_id = xx;
  3. select * from information_schema.`PROCESSLIST` where id=threads.processlist_id;

### synchronized 锁升级过程
  1. https://blog.csdn.net/zzti_erlie/article/details/103997713

### innodb 笔记
  + 页是innodb的最小存储结构
  + varchar长度超8098会将数据页存到blob页中，小于则还是在数据页当中

## redis
  ### redis集群
    1.数组保存槽数据，计算数据所属槽值之后，查看其所属节点，进行MOVE操作
  ### redis两种持久化方式
    1.RDB:快照形式是直接把内存中的数据保存到一个dump文件中，定时保存，一次性写入
    2.AOF:把所有的对redis的服务器进行修改的命令都存到一个文件里，命令的集合
  ### redis i/o多路复用
    + 假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：
      1. 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡主，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。
      2. 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者线程处理连接。
      3. 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。。。 这种就是IO复用模型，Linux下的select、poll和epoll就是干这个的。
      4. 将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。
  ### 解决redis缓存雪崩问题(多个key同时过期，短暂时间内并发访问数据库，造成雪崩)
    + 使用互斥锁
      1. 当第一个线程进来发现redis没有指定的key
      2. 则执行setNx(loke_key,"1")，然后去操作db
      3. 其他线程进入后也发现redis没有指定的key，执行setNx，但是返回false，此时sleep(n)秒，再去get
      4. 实现如下:
      ```
      String value = redis.get(key);  
         if (value  == null) {  
          if (redis.setnx(key_mutex, "1")) {  
              // 3 min timeout to avoid mutex holder crash  
              redis.expire(key_mutex, 3 * 60)  
              value = db.get(key);  
              redis.set(key, value);  
              redis.delete(key_mutex);  
          } else {  
              //其他线程休息50毫秒后重试  
              Thread.sleep(50);  
              get(key);  
          }  
        }  
      }  
      ```
  ### 解决redis缓存穿透(多次请求为null的数据)
    + 简单粗暴 将value为null的key也存到redis
  ### 线程还没执行完,redis锁已经过期了怎么办
    1.假如某线程成功得到了锁，并且设置的超时时间是30秒。当A线程还未执行完，锁已  被释放
    2.B线程创建锁并执行完毕，并将锁删除，此时删除的是线程B加上的锁
    3.如何避免：可以在del释放锁之前做一个判断，验证当前的锁的value是不是自己加的锁。加锁的时候把当前的线程ID当做value，并在删除之前验证key对应的value是不是自己线程的ID。
  ### Redisson 锁相关
   + lock有过期参数，会使用evalLua加锁
   + lock没有过期参数，会使用evalLua加锁一个30s的key,并添加监听线程持续对锁过期时  间续命
  ### 查看redis哨兵master的ip和端口
    1. telnet host port
    2. 执行 info 命令 查看 主机ip ，用主机的 ip 和端口连接
  ### redis过期策略
   + noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用吧  ，实在是太恶心了。
   + allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的   key（这个是最常用的）。
   + allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个   key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。
   + volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。
   + volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。
   + volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。
  ### redis跳表(skipList)
    + zset数据大时，会用字典(dict)+skipList作为存储结构
    + 字典及跳表的作用
      - zsccore，根据key查询score,由dict完成。时间复杂度为O(1)。
      - zrank，查看key的排名，先由dict中由数据查到分数，再拿分数到skiplist中查出排名。时间复杂度为O(log n)。
      - zrange，查看排行榜，由ziplist完成。时间复杂度为O(log(n)+M)，M为查询返回的元素个数。
    + skiplist查找过程：从header最高层开始，如果当前节点的下一个节点包含的值比目标元素值小，则继续向右查找。如果下一个节点的值比目标值大，就转到当前层的下一层去查找。https://www.jianshu.com/p/09c3b0835ba6
  ### redis Sentinel
    + PING/PONG命令
      - 每秒向Master、Slave以及其他Sentinel发送PING命令，确认在线
      - 如果有实例回应时间超过`own-after-milliseconds`  选项所指定的值，则被Sentinel标记为下线
    + INFO命令
      - 每10秒一次的频率向它已知的所有Master，Slave发送 INFO 命令
      - 当Master被Sentinel标记为客观下线时，Sentinel向下线的Master的所有Slave发送 INFO命令的频率会从10秒一次改为每秒一次。
      - 若没有足够数量的Sentinel同意Master已经下线，Master的客观下线状态就会被移除。 若 Master重新向Sentinel   的PING命令返回有效回复，Master的主观下线状态就会被移除。
    + 订阅sentinel:hello频道
    - sentinel节点通过__sentinel__:hello频道进行信息交换(对节点的"看法"和自身的信息)，达成共识。

### mysql redo、undo区别
 1. redo是个缓冲区，记录下修改后的记录，后续刷入到磁盘
 2. undo记录修改前的记录，用于做rollback操作

### bio, nio,select, epoll
 1. bio 阻塞等待数据
 2. nio 等待数据不需要阻塞，可以同步执行其他事情，同时轮询等待数据
 3. select 将数据放入多个fd当中，统一轮询监听，有数据之后，处理指定fe
 4. epoll 将数据放入多个fd，操作系统维护一个fe记录，统一轮询监听，有数据之后，新开线程处理指定fd

### 线程池中的工作线程如何被回收
  1. ThreadPoolExecutor回收线程：runWorker中循环执行getTask()方法，当getTask()获取不到任务，返回null时，调用processWorkerExit方法从Set集合中remove掉线程
 2. getTask()返回null又分为2两种场景：
    + 线程正常执行完任务，并且已经等到超过keepAliveTime时间，大于核心线程数，那么会返回null，结束外层的runWorker中的while循环
    + 当调用线程池shutdown()方法，会将线程池状态置为shutdown，并且需要等待正在执行的任务执行完，阻塞队列中的任务执行完才能返回null

### JDK动态代理和CGLIB动态地理区别

1. JDK 动态代理是面向接口的，需要实现类通过接口定义业务方法。
2. CGLib采用了非常底层的字节码技术，其原理是通过目标类的字节码为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。因此如果被代理类被final关键字所修饰，会失败。
3. 更详细一点说，代理类将目标类作为自己的父类并为其中的每个非final委托方法创建两个方法：
   + 一个是与目标方法签名相同的方法，它在方法中会通过super调用目标方法；
   + 另一个是代理类独有的方法，称之为Callback回调方法，它会判断这个方法是否绑定了拦截器（实现了MethodInterceptor接口的对象），若存在则将调用intercept方法对目标方法进行代理，也就是在前后加上一些增强逻辑。intercept中就会调用上面介绍的签名相同的方法。
4. 原文链接：https://blog.csdn.net/Dustin_CDS/article/details/79685620



### 多线程

 1.  线程状态 https://blog.csdn.net/xingjing1226/article/details/81977129

    + **新建(NEW)**：新创建了一个线程对象。

    + **可运行(RUNNABLE)**：线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行[线程池](https://so.csdn.net/so/search?q=线程池&spm=1001.2101.3001.7020)中，等待被线程调度选中，获取cpu 的使用权 。

    + **运行(RUNNING)**：可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

    +  **阻塞(BLOCKED)**：阻塞状态是指线程因为某种原因放弃了cpu 使用权，也即让出了cpu timeslice，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分三种： 

      > (一). 等待阻塞：运行(running)的线程执行o.wait()方法，JVM会把该线程放入等待队列(waitting queue)中。
      > (二). 同步阻塞：运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。
      > (三). 其他阻塞：运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。

    +  **死亡(DEAD)**：线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。

2. sleep()和wait()方法区别 https://blog.csdn.net/weixin_44440522/article/details/119352307
   + 属于不同的两个类，sleep()方法是线程类（Thread）的静态方法，wait()方法是Object类里的方法。
   + sleep()方法不会释放锁，wait()方法释放对象锁。
   + sleep()方法可以在任何地方使用，wait()方法则只能在同步方法或同步块中使用。
   + sleep()使线程进入阻塞状态（线程睡眠），wait()方法使线程进入等待队列（线程挂起），也就是阻塞类别不同。