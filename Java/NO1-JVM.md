# JVM的理解和认识

## 什么是JVM
Java虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。  

## Java 程序从源代码到运行一般有下面3步：
   

![](http://ww1.sinaimg.cn/large/9b13c8fdly1g0tfhposslj20lb04swej.jpg)

> 我们需要格外注意的是 .class->机器码 这一步。在这一步 jvm 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的，也就是所谓的热点代码，所以后面引进了 JIT 编译器，JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们为什么经常会说 Java 是编译与解释共存的语言。


![](http://ww1.sinaimg.cn/large/9b13c8fdly1g0tfh6hml0j20ku0amjs1.jpg)

1.**程序计数器**  
>程序计数器（Program Counter 
Register）是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。  
由于Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。  
如果线程正在执行的是一个Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie 方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在Java 虚拟机规范中没有规定任何OutOfMemoryError 情况的区域。  
  
2.**Java 虚拟机栈**
>局部变量表存放了编译期可知的各种基本数据类型（**boolean、byte、char、short、int、
float、long、double**）、**对象引用**（reference 类型，它不等同于对象本身，根据不同的虚拟
机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或
者其他与此对象相关的位置）和**returnAddress 类型**（指向了一条字节码指令的地址）。

3.**本地方法栈**
> 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其
区别不过是虚拟机栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则
是为虚拟机使用到的Native 方法服务。  

4.**java堆**
> Java 堆（Java Heap）是Java 虚拟机所管理的内存中最大的
一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的
唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。  
Java 堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC 堆”（Garbage
Collected Heap，幸好国内没翻译成“垃圾堆”）。  
从内存回收的角度看，由于现在
收集器基本都是采用的分代收集算法，所以Java 堆中还可以细分为：新生代和老年代；
再细致一点的有Eden 空间、From Survivor 空间、To Survivor 空间等。

5.**方法区**  
> 方法区（Method Area）与Java 堆一样，是各个线程共享的内存区域，它用于存
储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽
然Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-
Heap（非堆），目的应该是与Java 堆区分开来。  

6.**运行时常量池**  
> 运行时常量池（Runtime Constant Pool）是方法区的一部分。Class 文件中除了有
类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池（Constant Pool
Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放
到方法区的运行时常量池中。

7.**直接内存** 
> 

## Object obj = new Object();

> 假设这句代码出现在方法体中，那“Object obj”这部分的语义将会反映到Java 栈的本地变量表中，作为一个reference 类型数据出现。
而“new Object()”这部分的语义将会反映到Java 堆中，形成一块存储了Object 类型所有实例数据值（Instance Data，对象中各个实例字段的数据）的结构化内存，
根据具体类型以及虚拟机实现的对象内存布局（Object Memory Layout）的不同，
这块内存的长度是不固定的。另外，在Java 堆中还必须包含能查找到此对象类型数据（如对象类型、父类、实现的接口、方法等）的地址信息，这些类型数据则存储在方法区中。
由于reference 类型在Java 虚拟机规范里面只规定了一个指向对象的引用，并没有定义这个引用应该通过哪种方式去定位，以及访问到Java 堆中的对象的具体位置，
因此不同虚拟机实现的对象访问方式会有所不同，主流的访问方式有两种：使用句柄和直接指针。

**使用句柄访问方式**  
![](http://ww1.sinaimg.cn/large/9b13c8fdly1g0u0clw2xjj20ku09l0tk.jpg)

**使用直接指针访问方式**  
![](http://ww1.sinaimg.cn/large/9b13c8fdly1g0u0e76wtqj20ku09fmxs.jpg)  

参考：https://www.cnblogs.com/dingyingsi/p/3760447.html