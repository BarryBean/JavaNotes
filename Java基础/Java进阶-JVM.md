# 0. JVM组成部分
![JVM组成](BD28C1CEBCEF418D81E8E2AE5EA637AF)

JVM包括两个子系统和两个组件。
- Class Loader类装载：根据全限定类名装在class文件到运行时数据区域。
- Execution Engine执行引擎：执行class指令。
- Native Interface本地接口：和本地方法库交互，与其他语言交互的接口。
- Runtime Data Area运行时数据区域：即JVM内存区域。
 
**流程**
1. Class Loader读取.class文件转换成java.lang.Class的一个实例；
2. Runtime Date Area把字节码加载到内存；
3. Execution Engine把字节码翻译成底层系统指令，交给CPU执行；
4. 过程中调用Native Interface实现功能。

# 1. Java内存区域
答：JVM中内存主要划分为5个区域，即方法区，堆内存，虚拟机栈，本地方法栈和程序计数器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213111359335.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

## 1.1 组成
### 1.1.1 方法区
答：方法区是一个线程之间共享的区域，用于存储已被虚拟机加载的类信息。也被称为“永远代”，二者的关系类似接口和类的关系，即标准和实现。通过-XX：MaxPermSize控制上限。
### 元空间替换方法区
答：JDK1.8后，元空间代替了方法区。方法区本身由JVM设定固定的大小上限，元空间直接使用直接内存，受本机可用内存限制，溢出可能性小。

### 1.1.2 堆内存
答：堆内存是GC的主要场所，线程共享的区域，用来存储创建的对象实例即分配内存。

### 1.1.3 虚拟机栈
答：栈内存主要保存实例方法、基本数据类型变量和对象的引用变量，为Java方法服务。内部由栈帧(一个关于方法和运行期数据的数据集)组成，存储局部变量表(单位是slot)、操作数栈、动态链接等。生命周期和线程相同。

栈中，一个对象只对应一个 4byte 的引用。

总结一下：**栈管运行数据(基本数据类型、对象引用)保存，堆管实例分配内存。**

### 1.1.4 程序计数器/PC寄存器
答：程序计数器其实就是一个**指针，指向程序中下一句要执行的指令**。字节码解释器通过改变程序计数器来选取下一条需要执行的字节码指令，多线程时程序计数器用来记录当前线程执行位置，方便多线程切换。其随线程创建而创建，消亡而消亡。

### 1.1.5 本地方法栈
答：为JVM提供使用到的native方法服务。

## 1.2 堆栈区别

||堆|栈|
|--|--|--|
|物理地址|不连续|连续|
|内存分别|运行时确认，大小不定|编译时确定，大小固定|
|存放内容|对象实例|局部变量|
|透明度|整个进程可见|线程私有|

## 1.3深拷贝和浅拷贝
- 深拷贝：增加一个指针并申请一个新内存，让指针指向新的内存地址
- 浅拷贝：增加一个指针，指向已存在的内存地址。


# 2. HotSpot虚拟机
## 2.1 对象创建
答：5个步骤。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213113603105.png)

### 2.1.1 类加载检查
虚拟机遇到一条new指令时，先去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并检查这个类是否被加载解析和初始化，若没有则进行类加载。


### 2.1.2 分配内存
把一块大小确定的内存从Java堆中划分出来.

### 2.1.3 初始化零值
虚拟机将分配到的内存空间都初始化零值，保证对象实例字段不赋初试值就能使用。


### 2.1.4 设置对象头
虚拟机要对对象进行必要设置，将信息放在对象头中。包括哈希码，GC分代年龄，元数据指针等。


### 2.1.5 执行init方法
从JVM角度看，对象已经新建完毕。从程序角度看，还需执行init方法。

## 2.2 内存分配方式
答：指针碰撞和空闲列表。
- **指针碰撞**：堆内存中没有内存碎片，将**用过**的内存**放在一边**，**没用过**的**放**在**另一边**，**中间**有一个**分界指针**，内存分配就是将指针往没用过内存的方向移动对象内存大小。
- **空闲列表**：堆内存中有内存碎片，虚拟机维护一个**列表记录可用内存块**，内存分配就是找到一块足够大的空闲块划分给对象。

## 2.3 内存分配的线程安全
- **CAS+失败重试**：假设没有冲突去完成某个操作，若有冲突而失败则不断重试直到成功。保证操作原子性。
- **TLAB**：为每一个线程都分配一块内存，每次分配内存时，先在TLAB(Thread Local Allocation Buffer)中分配，不够时再用第一种方法。

## 2.4 对象的访问定位
答：栈内存有一个引用去访问定位堆中的具体对象，这个访问方式有两种。
- **句柄访问**：在堆内存中划分一块内存作为句柄池，**引用**中**存储**的是对象的**句柄地址(指向指针的指针)**，每个句柄包含对象实例数据和类型数据的具体地址信息。优点是对象改变只改变句柄，不动引用。
- **直接指针**：**引用**中**存储**的是**对象的直接地址**，通过指针直接访问对象。优点是速度快。

## 2.5 Java内存泄漏

### 2.5.1 概念
答：内存泄漏就是存在一些不能被 GC 回收，但仍占用内存的对象。

### 2.5.2 原因
答：一般就是**长生命周期的对象持有短生命周期对象的强引用**，导致短生命周期对象无法被GC回收。

### 2.5.3 情况举例
1. 静态集合类；
2. 监听器。删除对象时没有删除监听器；
3. 各种连接。数据库连接,socket连接,IO连接没有手动 close()；
4. 单例模式。若持有外部引用，无法被 GC。


### 2.5.4 解决方案
1. 尽量少用 static，减少生命周期长度；
2. 用完就关闭 close()；
3. 不用的对象，手动设置为空。





# 3.内存分配和回收
答：堆内存分为新生代，老生代。新生代又分为Eden(伊甸)，Survivor、To Survive(幸存)。//这里有个延伸问题，见GC算法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213123139747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)



## 3.1 过程
1. 创建的对象优先在Eden分配(年龄为0)，大对象(需要大量连续内存空间的对象)直接进入老年代。
2. 经过一次新生代GC，对象还存活便进入s0(From Survivor)或s1(To Survivor)，且年龄+1。
3. 每次GC，年龄++，默认 > 15岁，对象进入老年代。

>Q：为何是15? A：分配的空间是4位，最大就是15。


## 3.2 垃圾回收
答：垃圾回收主要是完成清理对象，整理内存的工作。根据区域不同分为两种：
- Minor GC(年轻代GC)：优先Eden分配，没有足够空间便发生一次Minor GC。次数频繁。
- Full GC(老年代GC)：老年代没有足够空间发生Full GC。
 


## 3.3 动态年龄判定
答：若Survivor空间中相同年龄的对象大小总和 > Survivor空间的一半，则年龄 >= 该对象年龄的对象自动晋升老年代。



## 3.4 空间分配担保
答：在发生minor gc之前，虚拟机会检测 : 老年代最大可用的连续空间 > 新生代all对象总空间？
1. 满足，minor gc是安全的，可以进行minor gc。
2. 不满足，虚拟机查看HandlePromotionFailure参数：
（1）为true，允许担保失败，会继续检测老年代最大可用的连续空间 > 历次晋升到老年代对象的平均大小? minor gc ：full gc。
（2）为false，则不允许，要进行full gc。

# 4. 如何判断对象是否需要被回收
## 4.1 引用计数法
答：给堆中的对象实例添加一个**引用计数器**，每当有一个地方引用它，计数器+1；引用失效，计数器-1；计数器为0的对象被GC。


缺点是**无法解决循环引用**的问题。eg. A和B相互引用，计数器一直++，不为0。



## 4.2 可达性分析/root根搜索
答：思想是通过被称为root的对象为起点，向下搜索，节点走过的路径为引用链，当**对象到root没有引用链相连则被GC**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213124001152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)


可以作为 GC Roots的对象：
1. 虚拟机栈引用的对象；
2. 方法区中常量引用对象；
3. 方法区中类静态属性引用的对象；
4. 本地方法栈中 JNI 引用对象。



## 4.3 引用类型
答：分为四种。
- 强引用：最常用的，只要强引用存在，GC就**不会回收**被引用对象；
- 软引用：可有可无，每次**内存不够**，GC**就回收**，内存足够就不动；(省心常用)
- 弱引用：可有可无，每次只要GC就会回收弱引用，**不管内存够不够都回收**；
- 虚引用：形同虚设，主要是当对象被回收时有一个系统通知。

## 4.4 废弃常量和无用类
答：各自的判断标准：
- 常量池回收废弃常量的判断标准是，当前没有任何类型对象引用该常量。
- 方法区回收无用类的判断标准是，类的实例被回收、ClassLoader被回收、类对象没有任何引用和访问。




# 5. GC算法
答：虚拟机中用root根搜索方法进行内存回收，常见的回收算法有标记清除、复制和标记整理算法。
## 5.1 标记-清除算法(Mark-Sweep)
- 标记阶段，标记出所有需要被回收的对象；
- 清除阶段，遍历整个堆，清除被标记对象。

缺点：产生内存碎片且需要暂停应用stop the world，效率慢。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213124320763.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

## 5.2 复制算法
- 将内存空间分成相等的两块，每次只用其中一个。
- GC时，把当前使用区域中存活的对象复制到另一个区域中。

优点：不会产生碎片；缺点：两倍内存空间。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213124547481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

## 5.3 标记-整理算法
结合上两种算法。
- 标记阶段，标记出所有需要被回收的对象；
- 整理阶段，让所有存活的对象都向一端移动，按序排放。

优点：不会产生碎片；缺点：需要进行对象移动。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213124611121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

## 5.4 分代收集
- 根据对象生命周期，将内存划分为新生代，老年代和永久代。
- 新生代变动频繁采用复制算法，老年代因对象存活几率大且没有其他空间进行担保采用标记清除或标记整理。

注：这里可以提问JVM为什么要分新生代和老生代？

答：不同对象的生命周期不一样，根据生命周期采用不同的回收策略，提高回收效率。



# 6. 垃圾回收器
答：GC算法是方法论，垃圾回收器就是具体的实现。JVM中主要包括7种。

- 新生代：Serial、ParNew，Parallel Scavenge
- 老年代：Serial Old、Parallel Old、CMS
- 整堆：G1

新生代一般用复制，老年代一般用标记整理(CMS-标记清除)

## 6.1 Serial串行收集器
单线程收集器，GC时必须Stop the world。简单高效。Client模式下的默认新生代收集器。

## 6.2 Serial Old收集器
Serial Old是串行收集器的老年代版本，单线程收集器。作用是Service模式下作为CMS的备案。标记整理算法。

## 6.3 ParNew收集器
ParNew是串行收集器的多线程版本，新生代是并行，老年代是串行。

## 6.4 Parallel Scavenge收集器
Parallel Scavenge是使用复制算法的多线程收集器，更加关注吞吐量(CPU运行用户代码时间/总时间)，高效率利用CPU。

## 6.5 Parallel Old收集器
Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和标记整理算法。

## 6.6 CMS收集器
CMS(Concurrent Mark Sweep)是一种**牺牲吞吐量以获取
最短回收停顿时间**为目标的老年代收集器。标记清除算法，与ParNew一起使用。

整个过程分为四个步骤：
1. 初始标记：Stop the world，**标记**GC root**直接关联**的对象；
2. 并发标记：**同时开启**GC和用户线程。用闭包结构记录可达对象和引用更新；
3. 重新标记：Stop the world，**更新**并发标记阶段因用户程序运行而导致变动的对象**标记**记录；
4. 并发清除：开启用户线程，基于标记清除对象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213124815318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

优点：停顿时间少，并发收集；

缺点：
- 对CPU资源敏感，CPU变小，性能会出问题；
- **无法处理浮动垃圾**。浮动垃圾指在GC完成时产生的垃圾，这些垃圾只能在下一GC周期回收；
- 标记清除方式会产生**内存碎片**。


## 6.7 G1收集器
传统分代垃圾回收方式无法解决 Full GC 的应用暂停。
- G1 吸取增量收集优点，把堆划分为一个一个等大小的区域 region；
- 同时吸取 CMS 特点，将 GC 分为几个阶段；
- 认同分代回收的理念，对于不同生命周期对象，采用不同收集方式；
- 维护一个**垃圾价值优先列表**，根据停顿时间从表中选择价值最大的区域回收。


> 特点：
> - 并行和并发：充分利用CPU和多核环境，缩短Stop the World的时间。
> - 分代收集：不用其他收集器就可管理整个GC，且保留了分代；
> - 空间整合：整体上是标记-整理算法，局部上是复制算法；
> - 可预测的停顿：能让用户明确指定停顿时间长度，来进行GC。



# 7. 类加载机制
答：类加载机制包括：加载，验证，准备，解析，初始化。最终形成能被虚拟机使用的Java类型。
## 7.1 流程
### 7.1.1 加载
加载通过全类名将**类的.class文件转二进制数据加载到内存**，放在方法区内，然后在堆上创建一个java.lang.Class对象，用来封装静态数据结构在方法区中运行时的数据结构。

### 7.1.2 验证
验证的作用是**确保被加载类的正确性**，符合JVM的规范和安全，包括文件格式验证，元数据验证，字节码验证和符号引用验证。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213125216334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

### 7.1.3 准备
准备阶段为类的**静态变量**在方法区中**分配内存**，并**初始化**为默认值。
eg. public static int value=3，初始值是0；public static final int value=3，初始值为3。

### 7.1.4 解析
解析阶段将常量池中的**符号引用转换为直接引用**。符号引用是以一组符号来描述引用的目标，直接引用就是直接指向内存的地址。

### 7.1.5 初始化
初始化阶段就是一个赋值的操作，为类的变量赋予正确的初始值。

## 7.2 类加载器
答：JVM内置了三个类加载器和用户自定义类加载器。

- **启动类加载器**BootstrapClassLoader：最顶层的加载类，负责加载 %JAVA_HOME%/lib 目录下的jar包或被 -Xbootclasspath 参数指定路径中的类；
- **扩展类加载器**ExtensionClassLoader：负责加载 %JRE_HOME%/lib/ext 目录下的jar包和类，或被 java.ext.dirs 系统变量所指定的路径下的jar包；
- **应用类加载器**AppClassLoader：面向用户的加载器，负责加载当前应用的classpath下的jar包和类。
- 自定义类加载器CustomClassLoader：需要继承ClassLoader，重写loadClass()。


### 7.2.1 类和类加载器的关系
答：比较两个类是否相等，得在两个类是由同一个类加载器加载的前提下才有意义。否则即使来自同一个class，只要类加载器不同，则必不相等。


## 7.3 双亲委派模型
答：协同工作时ClassLoader默认使用双亲委派模式。

- 简单来说就是，类加载时，将请求委派给父类的ClassLoader，父类不能处理时，再由子类自己去完成类的加载。
- 所有的请求**最终都会传送给最顶层的BootstrapClassLoader**
- 当父加载器为空，则默认 BootstrapClassLoader 作为父类加载器。

优点：避免类重复加载，保证API不被篡改。(相同类文件被不同类加载器加载产生两个不同类，同时若让类加载器自己加载自己的，容易产生多个不同的类，如Object类)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213125529563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

### 7.3.1 打破双亲委派
1. 自己写一个类加载器，继承 java.lang.ClassLoader；
2. 重写 loadClass()；
3. 重写 findClass()。

# 8. JVM常用内存调优命令
答：命令在JDK安装目录的bin文件夹下。
- jps(JVM Process Status)：查看所有Java进程的pid、启动类、参数等信息；
- jstat(JVM statistics Monitoring Tool)：查看虚拟机的运行数据 ；
- jinfo(Configuration Info for Java)：显示虚拟机配置信息；
- jmap(Memory Map for Java)：查看堆内存的使用情况；
- jhat：分析heapdump文件，建立一个HTTP服务器，在浏览器中查看分析结果；
- jstack(Stack Trace for Java)：查看进程内的线程堆栈信息。

## 8.1 排查线上的服务异常
答：简单介绍：
- 首先查看当前进程的JVM参数，有没有设置问题；
- 查看GC日志，看GC频率和时间有无异常；
- jps查看进程的具体信息；
- jstack pid查看线程状态，是否有死锁；
- jstat -gcutil pid查看进程的GC情况；
- jmap -heap pid查看进程的堆信息；
- jhat查看dump文件，分析异常。
