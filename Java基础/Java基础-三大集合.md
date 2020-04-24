# 1. 概况
## 1.1 List,Set,Map区别
答：Java容器分为Collection和Map两大类。List和Set是Collection的子接口。
- List（解决顺序问题）：存储一组有序可重复的对象，实现类有ArrayList、LinkedList和Stack等。
- Set（元素独一无二）：不存在重复的元素，实现类有HashSet、TreeSet等。
- Map（Key值搜索）：键值对存储，Key值不能重复，但能引用相同的对象，实现类有HashMap、ConcurrentHashMap等。

## 1.2 快速失败
答：快速失败(fail-fast)是Java集合中的**错误检测机制**。遍历集合过程中，若线程对集合进行增删改，会抛出并发修改异常。

原理
- 迭代器遍历过程中，使用一个modCount变量。若集合在遍历期间发生改变，modCount就会变化。hasNext()前都会检测modCount变量是否为exceptedModCount值，不是就抛出异常。
- 无法处理并发。
- 可以在所有涉及改变modCount的地方加synchronized同步锁避免fail-fast。


还可以将其理解为一种设计原则，当有某种条件导致模块无法正常运行，就立即终止运行，避免下游脏数据/便于排查。



## 1.3 安全失败
答：安全失败(fail-safe)的集合(JUC包)是多线程下使用的。遍历前先拷贝原有集合内容，在拷贝的集合上遍历。

- 迭代时是对拷贝集合操作，不会出现并发修改异常；
- 无法保证读取的数据是最新的数据。(迭代器只有开始遍历时才拿到拷贝，之后原数据发生变动是不知道的)

还可以理解为一种设计原则，当模块遇到错误，不终止执行，而是采用降级策略，尽量往下走。适用于主模块的分支流程。

# 2.Map接口
## 2.1 HashMap
答：HashMap是基于Hash算法实现的由数组和链表组合的数据结构，允许使用null值和null键。

总结：用数组存储，将冲突的key对象放入链表中，再发生冲突就在链表中顺序做对比。

### 2.1.1 插入元素
- put元素时，利用key的hashCode重新hash计算出当前对象的元素在数组中的下标；
- 如果出现hash值相同的key，若key相同，则覆盖原始值；若key不同即出现冲突，则将当前的key-value放入链表中
- java8前用头插法，新来的值代替原有值，原有值被往后推。问题是，扩容后容易形成环形链表。
- java8后用尾插法。在扩容同时保证链表元素原来的顺序，避免形成环。

### 2.1.2 读取元素
- get元素时，找hash值对应的下标，在进一步判断key是否相同，从而找到对应值。

### 2.1.3 多线程
答：hashmap线程不安全。put/get方法没有同步锁，容易出现上一秒put值，下一秒get的还是原值。

## 2.2 HashMap和HashTable的区别
1. 线程安全：HashMap线程不安全；HashTable内部使用synchronized关键字，线程安全。
2. Null Key支持：HashMap中null可作为Key；HashTable中null不能作key也不能作value。
3. 效率：HashMap比HashTable效率高一点，而且HashTable基本淘汰了。
4. 初始容量：HashMap是16，HashTable是11。
5. 扩容机制：HashMap是当前容量翻倍，Hashtable是当前容量翻倍+1。

### 2.2.1 为什么HashTable的null不能做key和value
答：两点原因。
- HashTable在put空值时会抛空指针异常。HashMap做了三目运算的处理，null就设0。
- 安全失败机制。让此次读取的数据不一定是最新，同时key为null就无法判断key是不存在还是空。

### 2.2.2 线程不安全
- HashMap线程不安全。因为多线程环境下扩容，导致hash规则变化，可能形成环形链表，死循环。
- HashTable线程安全。因为内部实现put和remove方法时使用synchronized同步，所以对单个方法的使用是线程安全的。但对多个方法复合操作时，无法保证安全性。


## 2.3 HashMap的底层结构
答：JDK1.8之前，用数组+链表用链地址法实现。JDK1.8之后使用数组+链表+红黑树实现，解决链表过长而查询速度变慢。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200206192803334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)



流程：
- hash后算出下标，没有冲突就直接放进node；
- 有冲突，链地址法，用链表链接相同hash值的数据；
- 链表长度>8且数组长度<64，先进行扩容；
- 链表长度>8且数组长度>64，转为红黑树，加速遍历。


## 2.4 扩容
答：HashMap的**初始容量16，加载因子0.75，扩容增量是原容量的1倍**。HashMap中的元素个数超过 初始容量16 * 加载因子0.75 = 12 后进行扩容。扩容分为两步。
- 创建新数组：创建原来HashMap大小两倍的bucket数组。
- ReHash：遍历原数组，将对象重新hash后放入新数组中。

### 2.4.1 为啥要rehash
答：因为长度扩大后，hash规则也改变。key & (length-1)

### 2.4.2 长度为什么是2的幂/初始长度为什么是16
答：有助于减少碰撞次数，提高查询效率。 
- **hash%length == hash&(length-1)的前提是 length 是2^n**。
- 2的幂时，length-1所有位都是1，哈希结果等同hashCode后四位的值，只要hashcode本身均匀，hash结果就均匀。即实现均匀分布。

eg.length为15，则length-1为14，对应二进制为1110，进行与操作后，最后一位为0，则最后一位为1的位置都不能存放元素。

### 2.4.3 加载因子为什么是0.75
从结果来推导，加载因子为0.75时，能保证和任何2的幂乘积结果都是整数，即负载因子*容量的结果是一个整数，


## 2.5 一致性哈希
答：就是哈希环。服务器平均放环上，一个服务器负责自己顺时针的一片hash区域上的对象。通过增加虚拟节点防止hash环偏移。

## 2.6 ConcurrentHashMap
- **底层数据结构**：ConcurrentHashMap是数组+链表+红黑树
- **实现线程安全**：

1. 在JDK1.8之前，ConcurrentHashMap使用分段锁将Hash表分割为16个桶，每一把锁只锁当前操作用上的桶。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200206191659313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

2. 在JDK1.8之后，使用Node数组+链表+红黑树的数据结构，并发控制用synchronized和CAS操作。

**synchronized锁定当前链表/红黑树的头节点，只要hash不碰撞，就不会并发**，效率大幅提升。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200206191727711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

### 2.6.1 put操作流程
- 根据 key 计算出 hashcode 。
- 判断是否需要进行初始化。
- 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树。

### 2.6.2 get操作流程
- 根据算出的hashcode寻址，在桶上就直接返回值。
- 如果是红黑树，就按树的方式获取值。
- 都不满足，就是按链表遍历获取值。


### 2.6.3 为什么用synchronized
答：JAVA8优化了同步锁，最初都是轻量级锁慢慢升级为重量级锁。
- 先用偏向锁，优先同一线程获取锁。
- 失败，就升级为CAS，失败后短暂自旋。
- 都失败，就升级为重量级锁。


## 2.7 LinkedHashMap和LinkedHashSet
答：
- LinkedHashMap能记录元素的插入顺序和访问顺序。

具体实现：内部通过**双向链表**，保证元素的插入顺序。内部使用LRU(最近最少使用)算法。

- LinkedHashSet底层使用LinkedHashMap实现。
- 二者关系类似HashMap和HashSet。

## 2.8 HashSet,HashMap和TreeSet区别
答：总结如下：
- HashMap底层使用Hash表实现，通过元素的hashCode值和equals()方法保证元素唯一性；
- TreeSet底层使用红黑树实现，通过comparable或comparator接口保证元素唯一性；
- HashSet底层是基于HashMap实现的，基本都是调用hashmap的方法。


## 2.9 TreeMap/红黑树
TreeMap存储键值对，底层是红黑树，可以实现元素的自动排序。了解 TreeMap 必须理解红黑树！

红黑树是一种二叉搜索树，也是均衡二叉树，当不满足红黑树规则时，自动调整节点平衡。

规则有：
- 节点分为红色和黑色；
- 根节点必为黑色；
- 叶子节点都为黑色，且都为null；
- 连接红色节点的两个子节点都为黑色，即不出现相邻的红色节点；
- 从任意节点出发，到任意叶子节点的路径中包含相同数量的黑色节点；
- 新加入红黑树的节点都是红色节点。

维持平衡的方式：变色，左旋，右旋。[具体操作](https://www.cnblogs.com/LiaHon/p/11203229.html/)


# 3. List接口
## 3.1 迭代器Iterator
答：Iterator是为了实现**遍历Collection的接口**。相当于把一个Collection容器的所有对象，做成一个线性表（List），而iterator本身是一个指针，开始时位于第一个元素之前。有三个主要方法：hasNext(),next(),remove()。

### 3.1.1 Iterator和ListIterator区别
答：区别如下：
- Iterator能遍历list和set集合；ListIterator只能遍历list集合。
- Iterator只能后向遍历；ListIterator是双向遍历。
- ListIterator也是继承了Iterator，再添加新的功能。

## 3.2 数组和List的转换
答：Arrays.asList和List.toArray方法。注意事项:
1. Arrays.asList()
    - 转换后，不能修改集合，add/remove/clear会报错。因为转化后，本质上仍是一个数组，返回的是Arrays内部的ArrayList类。
    - 传递数组必须是对象数组，而不能是基本类型。eg.用Integer替代int。
    - 正确的方法有：
```java
List list=new ArrayList<>(Arrays.asList("a","b","c"));

//使用 Java8 的Stream(推荐)
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
//基本类型也可以实现转换（依赖boxed的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```
2. List.toArray转换
    - 无参方法默认返回值是Object[];
    - 有参方法传入list.size()/指定的数据类型。

## 3.3 ArrayList
答：ArrayList是数组列表，用来存储数据。

- 优点：数组实现，支持随机访问。
- 缺点：删除和插入元素效率低，线程不安全。

正常使用中，不会频繁增删，不会多线程，所以选ArrayList。频繁增删就用LinkedList。线程安全就用Vector。

### 3.3.1 ArrayList和LinkedList区别
答：总结如下：
- 底层数据结构：ArrayList底层是Object动态数组；LinkedList底层使用双向链表。
- 随机访问和插入删除元素：ArrayList支持随机访问（相当于get(int index)方法）但删除插入元素慢；LinkedList不支持随机访问但支持随机插入删除元素。
- 内存空间占用：ArrayList必须预留一定的空间；LinkedList要存储前驱后继和节点信息，开销大。

注：ArrayList和LinkedList都是非线程安全的，Vector保证线程安全所有方法都是同步，但耗费大量时间，基本被抛弃了。

### 3.3.2 多线程下的ArrayList
答：ArrayList不是线程安全的。需要通过Collection.synchronizedList()方法转换成线程安全容器。

### 3.3.3 ArrayList怎么存放任意数量对象
答：扩容实现。
- 新建一个原始长度*1.5的数组；
- 把原数组的数据复制到新数组中；
- 最后更新地址。


### 3.3.4 新增元素
1. 判断长度，不够就扩容；
2. 新建数组，复制原数组，在index位开始把元素放进index+1位；
3. 最后在index放入新增元素。

### 3.3.5 删除元素
1. 新建数组，复制原数组，跳过index位，把index+1位放在index位；
2. 相当于被覆盖。

# 4. Set和Queue
## 4.1 HashSet原理
答：基于HashMap实现，HashSet值存放在HashMap的key上，value统一为PRESENT。HashSet通过比较hash值和equals()判断key是否重复。底部基本都是调用HashMap的方法。

### 4.1.1 hashCode()和equals()区别
答：总结为：
- 两对象相等，则hashcode一定相同，equals也返回相同；
- hashcode相同，对象可能不同，equals也可能返回不同；
- 重写equals()时，hashcode一定也得重写。

### 4.1.2 ==和equals()
答：提一嘴。==判断内存地址，equals()判断内容值。

## 4.2 Queue方法
### 4.2.1 poll()和remove()
答：都能弹出第一个元素并把对象从队列中删除。区别是没有元素时poll()返回null，remove()抛出异常。

### 4.2.2 BlockingQueue
答：BlockingQueue是用来实现生产者-消费者模式的队列。添加元素时，等待队列有空间；移除元素时，等待队列为非空。


