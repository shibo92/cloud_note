### 多线程

#### 线程状态

 1. 线程状态 https://blog.csdn.net/xingjing1226/article/details/81977129

    + **新建(NEW)**：新创建了一个线程对象。

    + **可运行(RUNNABLE)**：线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行[线程池](https://so.csdn.net/so/search?q=线程池&spm=1001.2101.3001.7020)中，等待被线程调度选中，获取cpu 的使用权。

      + 运行(RUNNING) 可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

    + **阻塞(BLOCKED)**：阻塞状态是指线程因为某种原因放弃了cpu 使用权，也即让出了cpu timeslice，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分三种： 

      > (一). 等待阻塞(**WAITING**)：运行(running)的线程执行obj.wait(), join()方法，JVM会把该线程放入等待队列(waiting queue)中。
      > (二). 同步阻塞(**BLOCKED**)：运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。
      > (三). 其他阻塞(**TIMED_WAITING**)：运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。

    + **终止(TERMINATED)**：线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。

2. sleep()和wait()方法区别 https://blog.csdn.net/weixin_44440522/article/details/119352307

   + 属于不同的两个类，sleep()方法是线程类（Thread）的静态方法，wait()方法是Object类里的方法。
   + sleep()方法不会释放锁，wait()方法释放对象锁。
   + sleep()方法可以在任何地方使用，wait()方法则只能在同步方法或同步块中使用。
   + sleep()使线程进入阻塞状态（线程睡眠），wait()方法使线程进入等待队列（线程挂起），也就是阻塞类别不同。

#### 内存屏障和cpu缓存（volatile）

 	1. 重排序：cpu执行写指令，发现缓存区被其他cpu占用时，会优先执行后续的读指令（没有依赖关系的指令）
 	2. 内存屏障：
 	 + store 写内存屏障： 在指令后插入写屏障，强制写入主存，且会一直等待，不会优先执行后续读指令。
 	 + load 读内存配置： 在指令前插入读屏障，使缓存失效，强制从主存加载数据。

#### ThreadLocal

1. 每个线程维护一个ThreadLocalMap
2. ThreadLocalMap.Entry 对 ThreadLocal是弱引用，但是Entry对Value是强引用，会有内存泄漏风险，所以用完需要remove

#### 线程池

1. 为什么要有线程池
   + 线程创建、销毁消耗资源
   + 线程过多，消耗内存（单个线程默认栈内存大小1M）
   + 线程过多时，cpu切换耗时

#### 线程过多问题排查

    1. jstack pid| grep java.lang.Thread.State| awk '{print $2$3$4$5}' | sort | uniq -c
    2. https://blog.csdn.net/fengsheng5210/article/details/123610380



### 中间件

#### mq







