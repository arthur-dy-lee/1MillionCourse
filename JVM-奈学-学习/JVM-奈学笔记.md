# JVM奈学笔记



jvm、jre和jdk关系

![Snipaste_2020-06-17_20-42-46-jvm、jre和jdk关系](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-17_20-42-46-jvm、jre和jdk关系.png)



类加载过程

[https://docs.oracle.com/javase/specs/jls/se8/html/index.html](https://docs.oracle.com/javase/specs/jls/se8/html/index.html)

![Snipaste_2020-06-17_22-15-33-类加载过程](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-17_22-15-33-类加载过程.png)



tomcat类加载器、spring自定义的类加载器



### 运行时内存

![Snipaste_2020-06-20_20-53-24-运行时](/Users/lidongyue/Desktop/Hive-奈学/Snipaste_2020-06-20_20-53-24-运行时.png)



#### 栈

一个栈帧包手：局部变量表、操作数栈、动态链接、方法返回地址。【操局动返】 



![Snipaste_2020-06-20_20-27-42-栈和栈桢](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-27-42-栈和栈桢.png)

栈是和Main线程程绑定在一起

![Snipaste_2020-06-20_20-30-50-栈调用顺序](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-30-50-栈调用顺序.png)







![Snipaste_2020-06-20_20-54-20-执行引擎](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-54-20-执行引擎.png)

结合字节码指令理解Java虚拟机栈和栈帧

![Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧.png)



![Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧2](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧2.png)

虚拟机栈的栈桢动态链接到hashCode方法

![Snipaste_2020-06-20_21-58-42-动态链接到hashCode方法](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_21-58-42-动态链接到hashCode方法.png)





![Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧3](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_20-58-04-结合字节码指令理解Java虚拟机栈和栈帧3.png)









![Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧4](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧4.png)

上面的方法fun在调用栈中的调用过程

2个数相加，局部变量表中有3个变量:1, 2, 0。因为2个数相加，所以操作数栈有2个。

![Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧5](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧5.png)



![Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧5](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_22-05-51-J结合字节码指令理解Java虚拟机栈和栈帧6.png)



![Snipaste_2020-06-20_22-31-16-Java对象内存部局](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_22-31-16-Java对象内存部局.png)



![Snipaste_2020-06-20_23-06-01-对象头-Markword](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-20_23-06-01-对象头-Markword.png)



一个对象在多线程并发访问时，被某一线程加了偏向锁后，去读hashCode是否会被影响？

hashCode仍可被读取。线程拿到偏向锁后，hashCode被线程模型monitor所管控。



对象头中的信息

![Snipaste_2020-06-21_09-28-41-对象的偏向锁](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-21_09-28-41-对象的偏向锁.png)



[http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d2c2cd90513e/src/share/vm/oops](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d2c2cd90513e/src/share/vm/oops)

[http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d2c2cd90513e/src/share/vm/runtime](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d2c2cd90513e/src/share/vm/runtime)



对象的初始化过程

![Snipaste_2020-06-21_10-48-00-对象的初始化过程](/Users/lidongyue/Desktop/JVM-奈学-学习/blogpic/Snipaste_2020-06-21_10-48-00-对象的初始化过程.png)





[官方垃圾回收器介绍](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)

[https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)

如何选择垃圾回收器

```txt
Selecting a Collector
Unless your application has rather strict pause time requirements, first run your application and allow the VM to select a collector. If necessary, adjust the heap size to improve performance. If the performance still does not meet your goals, then use the following guidelines as a starting point for selecting a collector.

If the application has a small data set (up to approximately 100 MB), then

select the serial collector with the option -XX:+UseSerialGC.

If the application will be run on a single processor and there are no pause time requirements, then let the VM select the collector, or select the serial collector with the option -XX:+UseSerialGC.

If (a) peak application performance is the first priority and (b) there are no pause time requirements or pauses of 1 second or longer are acceptable, then let the VM select the collector, or select the parallel collector with -XX:+UseParallelGC.

If response time is more important than overall throughput and garbage collection pauses must be kept shorter than approximately 1 second, then select the concurrent collector with -XX:+UseConcMarkSweepGC or -XX:+UseG1GC.

These guidelines provide only a starting point for selecting a collector because performance is dependent on the size of the heap, the amount of live data maintained by the application, and the number and speed of available processors. Pause times are particularly sensitive to these factors, so the threshold of 1 second mentioned previously is only approximate: the parallel collector will experience pause times longer than 1 second on many data size and hardware combinations; conversely, the concurrent collector may not be able to keep pauses shorter than 1 second on some combinations.

If the recommended collector does not achieve the desired performance, first attempt to adjust the heap and generation sizes to meet the desired goals. If performance is still inadequate, then try a different collector: use the concurrent collector to reduce pause times and use the parallel collector to increase overall throughput on multiprocessor hardware.
```





GC root是批？



工具

PerfMa线上工具

memory.console.perfma.com



parseInt、Integer.valueOf 方法实现

虚拟动物园，有很多种动物，其中

老虎出现1支，熊猫。狮子都是2支，总动物千万级别，怎么从这里找到只有1支的老虎？

从前到后，使用异或，成对的就会消掉。

如果3个3个动物出现，1个动物只有1个支，又该怎么处理？













