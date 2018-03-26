# Java虚拟机
Java的内存管理机制，虚拟机执行子系统，程序编译与代码优化，高效并发。

-----
# TOP
[Java内存管理](#java内存管理)<br>



-----

### Java内存管理机制

![java内存](https://github.com/Zhangchao999/Java-1/raw/master/pictures/1.jpg)
<br>
<p align = "center">
图1
</p>

<br>
讲解:<br>
从线程的私有还是共享来说：<br>
线程私有:程序计数器、Java虚拟机栈、本地方法栈。<br>
线程共享:Java堆、方法区。<br>

1、程序计数器:
> 程序计数器是当前线程所执行的字节码的行号指示器。当CPU要执行指令时，需要从程序计数器中获得指令所存储单元的地址，再得到指令。 

> 由于再任意时刻，一个CPU只能执行一条线程中的指令，因此，为了使每个线程在线程执行后回到原来的线程所执行的位置，所以，每个线程都需要单独的程序计数器。

2、Java虚拟栈:
> Java栈中存放的是一个个的栈帧，每个栈帧对应的是一个被调用的方法，在栈帧中包括:局部变量表，操作数栈，运行时常量池，方法的返回地址，附加信息。

3、本地方法栈:
> 与Java方法栈类似，只是Java方法栈为执行Java方法服务，而本地方法栈执行本地方法服务。

4、Java堆：
> 用来存储对象本身或数组的。Java的程序员不需要关心内存的释放问题，由于有垃圾回收机制会自动处理。(C语言中，是通过malloc函数和free函数来申请和释放内存的).


> 堆也是Java垃圾收集器管理的主要区域，堆是被线程共享的，在JVM中只有一个堆。

5、方法区：
> 方法区在JVM中也是非常重要的区域，与堆一样，也是被线程共享的。方法区存储了每个类的信息(类名，方法信息，字段信息)，静态变量，常量，编译后的代码等。

> Class文件中除了类的字段，方法，接口外，还有常量池，用来存储编译期间生成的字面量和符号引用。

> 还有一个重要的部分就是运行的常量池，它是每个类或者接口的常量池的运行时的表现形式，在类和接口被加载到JVM后，对应的运行时的常量池就被创建出来。当然不是在Class文件中的内容才能进入运行时常量池，在运行期间新的常量也能放入运行时常量池中，如String的intern方法。

> intern方法: 返回字符串对象的规范表示。 最初的空的字符串由String类String。当调用intern方法时，如果池中包含与equals(Object)方法确定的相当于此String对象的字符串，则返回来自池中的字符串。否则，将该String对象添加到池中，并返回，String对象的引用。由此可见，对于任何两个字符串s和t ， s.intern() == t.intern()是true当且仅当s.equals(t)是true 。


![java内存](https://github.com/Zhangchao999/Java-1/raw/master/pictures/2.jpg)
<br>
<p align="center">
	图2:jdk1.8之前的内存
</p>

讲解:<br>

1、堆(Heap):
> 所以new出来的对象都分配在堆中，堆分成年轻代(YoungGeneration)和老年代(OldGeneration)，年轻代分为一个Eden区和两个Survivor区(默认比例是 8:1)

>> 1.1 MinorGC

>>> 年轻代是所有新对象的产生地方。当年轻代的内存空间被用完是，会出发垃圾回收。这个垃圾回收叫做MinorGC。

>>> 大多数的新建的对象位于Eden区，当Eden区占满后，会出发MinorGC，存活下来的对象全部转到survivor0区，survivor0区占满后会触发MinorGC，survivor0区存活的对象全部转到survivor1区，这要就会保证一段时间内总有一个survivor区是空的。经过多次的MinorGC任然存货的对象会转到老年代，这个指由设定的年龄阀值所决定。


>> 1.2 MajorGC

>>> 老年代空间长期存活的对象，在被占满时会触发MajorGC，花费时间较长，由于垃圾回收会导致"stoptheworld"事件，及所有的线程都会停下来等待垃圾回收完成，因此，对于相应高的应用要尽量减少MajorGC，如微博后台发生MajorGC会导致前台页面刷新超时。

2、永生代
> PermanentGeneration 是用来保存程序运行时长期存活的对象，如类的元数据。
（元数据是指用来描述数据的数据，更通俗一点，就是描述代码间关系，或者代码与其他资源（例如数据库表）之间内在联系的数据。）

> 在jdk1.8 中用元空间(Metaspace)替换永久代(PermanentGeneration)
>> 依据:1官方文档是 This is part of the JRockit and Hotspot convergence effort. JRockit customers do not need to configure the permanent generation (since JRockit does not have a permanent generation) and are accustomed to not configuring the permanent generation.即：移除永久代是为融合HotSpot JVM与 JRockit VM而做出的努力，因为JRockit没有永久代，不需要配置永久代。

>> 依据:2现实使用中易出问题 由于永久代内存经常不够用或发生内存泄露，爆出异常java.lang.OutOfMemoryError: PermGen

>> 元空间是方法区的在HotSpot jvm 中的实现，方法区主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。理论上取决于32位/64位系统可虚拟的内存大小。可见也不是无限制的，需要配置参数。

3、NativeArea

>> 3.1 PC程序计数器: 即上面所说的程序计数器

>> 3.2 本地方法栈: 即上面所说的本地方法栈






