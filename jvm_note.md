
<!-- vim-markdown-toc GFM -->

* [java内存区域](#java内存区域)
* [eden和survivor回收过程](#eden和survivor回收过程)
	* [一个对象的这一辈子](#一个对象的这一辈子)
* [类的初始化步骤](#类的初始化步骤)
* [jstat -gcutil 命令使用](#jstat--gcutil-命令使用)
* [jvm常用命令](#jvm常用命令)
* [导出dump文件并分析](#导出dump文件并分析)

<!-- vim-markdown-toc -->
### java内存区域
1. 程序计数器
2. 虚拟机栈、本地方法栈
3. 堆内存
4. 方法区
 + 运行时常量池，1.8之前在方法区，移除方法区之后，在元数据区域
                （区别于字符串常量池：String常量池位置：
				jdk1.6: 永久代（方法区）
				jdk1.7: 堆内存
				jdk1.8: 元空间）
 
5. 元数据区(1.8)
 + 元数据区取代了1.7版本及以前的永久代。元数据区和永久代本质上都是方法区的实现。方法区存放虚拟机加载的类信息，静态变量，常量等数据。

### eden和survivor回收过程
  在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

#### 一个对象的这一辈子
我是一个普通的Java对象，我出生在Eden区，在Eden区我还看到和我长的很像的小兄弟，我们在Eden区中玩了挺长时间。有一天Eden区中的人实在是太多了，我就被迫去了Survivor区的“From”区，自从去了Survivor区，我就开始漂了，有时候在Survivor的“From”区，有时候在Survivor的“To”区，居无定所。直到我18岁的时候，爸爸说我成人了，该去社会上闯闯了。于是我就去了年老代那边，年老代里，人很多，并且年龄都挺大的，我在这里也认识了很多人。在年老代里，我生活了20年(每次GC加一岁)，然后被回收。

### 类的初始化步骤
加载 --> 验证 --> 准备 --> 解析 --> 初始化 --> 使用 --> 卸载

### jstat -gcutil 命令使用
  + 命令格式 jstat -gcutil pid interval(ms)
  + eg: jstat -gcutil  16361 1000
   print:	
   ```
      $ jstat -gcutil 12691 1000
      S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT 
      81.51   0.00   2.57  66.64  94.58  91.30   5730   60.270     4    1.343   61.612
   ```  
  + S0: 新生代中Survivor space 0区已使用空间的百分比
  + S1: 新生代中Survivor space 1区已使用空间的百分比
  + E: 新生代已使用空间的百分比
  + O: 老年代已使用空间的百分比
  + P: 永久带已使用空间的百分比
  + YGC: 从应用程序启动到当前，发生Yang GC 的次数
  + YGCT: 从应用程序启动到当前，Yang GC所用的时间【单位秒】
  + FGC: 从应用程序启动到当前，发生Full GC的次数
  + FGCT: 从应用程序启动到当前，Full GC所用的时间
  + GCT: 从应用程序启动到当前，用于垃圾回收的总时间【单位秒】

### jvm常用命令
  + jps命令(Java Virtual Machine Process Status Tool)
  + jstack命令(Java Stack Trace) - 打印堆栈信息
  + jstat命令(Java Virtual Machine Statistics Monitoring Tool) - 监控jvm
  + jmap命令(Java Memory Map) - 打印java进程所有‘对象’情况,产生了哪些些对象，及其数量(jmap -heap)

### 导出dump文件并分析
  1. jmap -dump:format=b,file=xx.dump pid
  2. scp下载到本地
  3. 使用jvisualvm分析dump文件
  4. chmod a+r xx.dump
