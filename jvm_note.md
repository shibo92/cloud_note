
<!-- vim-markdown-toc GFM -->

* [java内存区域](#java内存区域)
* [eden和survivor回收过程](#eden和survivor回收过程)
	* [一个对象的这一辈子](#一个对象的这一辈子)
* [类的初始化步骤](#类的初始化步骤)
* [jstat -gcutil 命令使用](#jstat--gcutil-命令使用)
* [jvm常用命令](#jvm常用命令)
* [导出dump文件并分析](#导出dump文件并分析)
* [jstack定位到线程](#jstack定位到线程)
* [jvm调优常用参数](#jvm调优常用参数)
* [JVM主动使用的场景](#jvm主动使用的场景)
* [常量何时进入常量池](#常量何时进入常量池)

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
  在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。当Eden满了之后，触发MinorGC，Eden区中所有存活的对象都会被复制到“To”区域，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

#### 一个对象的这一辈子
我是一个普通的Java对象，我出生在Eden区，在Eden区我还看到和我长的很像的小兄弟，我们在Eden区中玩了挺长时间。有一天Eden区中的人实在是太多了，我就被迫去了Survivor区的“From”区，自从去了Survivor区，我就开始漂了，有时候在Survivor的“From”区，有时候在Survivor的“To”区，居无定所。直到我18岁的时候，爸爸说我成人了，该去社会上闯闯了。于是我就去了年老代那边，年老代里，人很多，并且年龄都挺大的，我在这里也认识了很多人。在年老代里，我生活了20年(每次GC加一岁)，然后被回收。

### 类的初始化步骤
加载 --> 验证 --> 准备 --> 解析 --> 初始化 --> 使用 --> 卸载

    
### JVM主动使用的场景
  1. 创建类的实例
  2. 访问类或接口的静态变量，或对静态变量赋值
  3. 调用类的静态方法
  4. 反射(Class.forName)
  5. 初始化其子类
  6. 被标明为启动类的类(main、test)
 
### 常量何时进入常量池
  1. 编译时，jvm会将常量放入引用类的常量池中
  2. 编译期不确定的值，比如uuid，是不会进入常量池的




### jstat -gcutil 命令使用 (查看gc频率)
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
  + P/M: 永久带已使用空间的百分比
  + YGC: 从应用程序启动到当前，发生Yang GC 的次数
  + YGCT: 从应用程序启动到当前，Yang GC所用的时间【单位秒】
  + FGC: 从应用程序启动到当前，发生Full GC的次数
  + FGCT: 从应用程序启动到当前，Full GC所用的时间
  + GCT: 从应用程序启动到当前，用于垃圾回收的总时间【单位秒】

### jvm常用命令
  + jps命令(Java Virtual Machine Process Status Tool)
  + jstack命令(Java Stack Trace) - 跟踪线程信息
  + jstat命令(Java Virtual Machine Statistics Monitoring Tool) - 主要查看GC信息和jvm内存信息(jstat -gcutil pid)
  + jmap命令(Java Memory Map) - 打印java进程所有‘对象’情况,产生了哪些些对象，及其数量(jmap -heap)
  + 查看占用cpu高的线程信息
    1. top -H 找到cpu高的进程pid
    2. top -p pid 按下H, 找到cpu高的线程pid
    3. pid转16进制
    4. jstack 进程pid |grep 16进制内容 （查看对应线程信息）
    
### 导出dump文件并分析
  1. jmap -dump:format=b,file=xx.dump pid
  2. scp下载到本地
  3. 使用jvisualvm分析dump文件
  4. chmod a+r xx.dump

### jstack定位到线程
  + top -Hp pid或ps找到线程pid
  + printf "%x\n" 21742 将pid转换成十六进制
  + jstack 进程pid |grep 线程pid 

### jvm调优常用参数
  + jmap
    - jmap -heap pid 查看堆内存信息
    - jmap -dump:format=b,file=heapdump.phrof pid
    - jmap -histo:live pid 查看堆内存对象信息
  + jstat
    - jstat -gc pid 5000 显示gc的信息,查看gc的次数,及时间，每5秒刷新一次
    - jstat -gcutil pid 统计gc信息
  + jatack
    - jstack pid > /home/xxx/dump17 导出线程文件
    - grep java.lang.Thread.State dump17 | awk '{print $2$3$4$5}'| sort -nr | uniq -c | sort -nr 统计线程状态
    - jstack -l pid > jstack.log 导出线程日志
    - cat jstack.log | grep "java.lang.Thread.State" | sort -nr |uniq -c |sort -nr
  + top -Hp [pid] 查对应进程的线程情况
   - printf '%X \n' [pid]  获取到长时间运行的线程[pid] 并转换为16进制
   - jstack -l [pid] | grep [nid] -A 200 依据获得16进制的线程[pid] 打印堆栈信息

### jvm收集器
  + 新生代收集器
    1. Parallel Scavenge  
      - jdk1.7,1.8 默认的收集器
      - 特点是吞吐量大,但是stop the world时间长
      - 复制算法(eden+survival)
    2. ParNew 收集器
      - 默认开启的收集线程数和 CPU 数量一致
      - 与 CMS 收集器搭配使用
      - -XX:+UserConcMarkSweepGC指定cms收集器时，年轻代默认为ParNew
      - 特点是尽可能缩短stw的时间
      - 复制算法
  + 老年代收集器
    1. Parallel Old 
      - jdk1.7,1.8 默认的收集器, 
      - 标记-整理算法
    2. CMS
      - 以最短回收停顿时间为目标的收集器
      - 随着 CPU 数量下降，占用 CPU 资源越多，吞吐量越小
      - 适用场景：重视服务器响应速度，要求系统停顿时间最短
      - 标记-清除算法, 会产生垃圾碎片（为了降低响应时间，所以没有使用标记-整理算法)
  + G1 
    1. 