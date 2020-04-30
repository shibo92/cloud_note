### P4
 + 在jvm中，*类型*的加载、连接和初始化过程都是在运行期间（runtime）完成的
 + 加载：将class文件加载到jvm内存
 + 连接：处理类与类之间的关系或是验证
 + 虚拟机结束生命周期的情况
   1. System.exit()方法
   2. 正常执行完毕
   3. 异常或错误退出
   4. 操作系统级别的错误导致退出

### p14
 + 数组类对象不是由类加载器加载的，而是由java虚拟机在运行期间动态创建的（其他对象都是有classLoader加载的）
 + String[] 的getClassLoader是bootstrap类加载器（null）
 + MyTest[] 自定义数组的getClassLoader 是 appClassLoader加载器
 + int[] 基本类型数组，getClassLoader没有类加载器（null）
 + ctrl+q 打开文档
