
# 1. Java概述
Java是一种面向对象的编程语言，继承了之前语言的优点，还抛弃了指针等晦涩的内容，简单易用，功能强大。

## 1.1 JDK、JRE和JVM
答：基本概念如下：
- JDK(Java Development Kit)开发工具包，java开发环境的核心组件，提供编译调试和运行java程序所有的工具和文件。
- JRE(Java Runtime Environment)java运行时环境，包括JVM和核心类库。
- JVM(Java Virtual Machine)java虚拟机，Java程序运行在JVM上，将字节码转为机器码，提供内存管理、GC等功能。

### 1.1.1 关系
- JDK > JRE > JVM
- 只运行程序不编码，只装JRE；
- JVM具有平台无关性，让java程序一次编译多个系统执行。

### 1.1.2 平台无关性实现
- Java程序基于JVM运行，JVM屏蔽了操作系统和底层硬件的差异；
- Java先编译生成字节码文件，再交给JVM转为机器码；

## 1.2 字节码
答：字节码是Java源代码经过编译后生成的.class文件，只面向虚拟机。
### 1.2.1 好处
答：既保留了解释性语言的可移植性，又有编译型语言的高效率。
### 1.2.2 流程
Java源代码 -> 编译器 -> 字节码 -> JVM的解释器 -> 二进制代码 -> 程序运行
### 1.2.3 java是编译型还是解释型语言
答：从上可以总结，java是先编译后解释执行的语言，不能单纯地归类。

## 1.3 Java和C++区别
答：我也不知道，为什么面经都有这类题...

- 都是面向对象的语言，支持封装继承和多态
- java没有指针访问内存
- java类是单继承，C++多重继承
- java有自动GC机制
- C/C++中的字符串和数组都自动加一个'\0'作为结束符，java中不需要
 

## 1.4 各版本特性
1. Java 8
    - Lambda 表达式，简化写法。
    - Streams流编程。中间操作返回Stream本身，能将多个操作串起，最终操作才返回结果。
    - Data API。Clock 提供访问当前日期和时间；时区设置；DateTimeFormatter 解决日期格式化；本地日期和本地时间。
    - 注解。
2. Java 9
    - 引入Java平台模块系统，将JDK分为若干模块，创建镜像只需要所依赖的JDK模块，减少JRE大小。
    - 设置 G1 为默认垃圾收集器。
    - 集合、Stream、Optional方法扩展。
    - 接口私有方法。
3. Java 10
    - var关键字。只能用于局部变量和for循环。
    - G1 的FullGC变为并行的标记清除。
4. Java 11
    - 新的长期支持版本。
    - 增加一系列字符串处理方法。
    - ZGC 伸缩GC收集器。
    - HTTP Client API标准化，支持异步非阻塞。
5. Java 12
    - 修改switch，变成类似lambda的形式。
6. Java 13
    - switch 加入 yield，用来跳出当前switch块。
    - 文本块。
7. Java 14
    - 将 switch 的增强转正。
    - 移除 CMS 垃圾收集器。


# 2.基础语法
## 2.1 数据类型
![Java数据类型](12803FA6036D4B66911F766AFF9124F0)

### 2.1.1 BigDecimal
浮点数的等值判断，基础类型不能用==，包装类型不能用equals()，会造成精度丢失。推荐使用BigDecimal操作浮点数，从而进行运算操作。
- 大小判断：a.compareTo(b)，1为大于，0为等于，-1为小于；
- 保留小数：setScale()；
- 创建：new BigDecimal("0.1")/BigDecimal.valueOf(0.1)

总结：BigDecimal操作(大)浮点数，BigInteger操作大整数(超过long)。

## 2.2 运算符
### 2.2.1 &/|和&&/||区别
答：总结为：
- &是按位与，|是逻辑或；
- &&是与运算，||是或运算，都具有短
- 路特性。eg.&&前面为false，右边表达式短路不访问。

### 2.2.2 equals()和==的区别
答：区别：
- 基本类型，==判断值相等，无equals()。
- 引用类型，==判断两个变量是否引用同一个变量即比较对象的地址，equals()判断引用对象是否等价。



## 2.3 关键字
### 2.3.1 final
答：final关键字用于修饰类、变量和方法。
- final修饰的类，不能被继承；
- final修饰的方法，不能被重写；
- final修饰的变量，必须初始化，不能被改变。**若修饰的是基础数据类型则其值不能变，若修饰引用类型则其内存地址不能变**。

### 2.3.2 static
答：static的作用有:
- static修饰的类方法，不属于任何实例，但被类的实例对象共享。
- static修饰的部分，只在第一次加载时初始化。



## 2.4 String
答：String是不可变的，创建后在常量池缓存。
### 2.4.1 不可变
- String是final修饰的char数组，不可继承；
- String不可变，但引用可变。eg.字符串拼接理解为开辟新的内存区域给新串。
- 用反射可以修改String。

### 2.4.2 StringBuffer与StringBuilder
1. 可变性：String底层用final char[]，不可变；StringBuffer和StringBuilder用char[]，可变。
2. 线程安全：String不可变，安全；StringBuffer加了同步锁，线程安全；StringBuilder是非线程安全的。
3. 性能：String改变时，生成新的String对象，将指针指向新的String；StringBuffer每次都对本身进行操作；StringBuilder速度会快一点。

所以：
- 操作少量的数据：适用String；
- 单线程操作字符串缓冲区下操作大量数据：适用StringBuilder；
- 多线程操作字符串缓冲区下操作大量数据：适用StringBuffer。

注：记不清的话用safe来记忆，buffer有safe的一部分，所以它是线程安全的。

### 2.4.3 缓存池
答：new Integer(123) 与 Integer.valueOf(123) 的区别在于：
- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用取同一个对象的引用。valueOf() 方法先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。
 
类似的new String("i")是在堆内存创建对象，String str="i"是在常量池中存储。

### 2.4.4 常用方法
```
indexOf()：返回指定字符的索引。
charAt()：返回指定索引处的字符。
replace()：字符串替换。
trim()：去除字符串两端空白。
split()：分割字符串，返回一个分割后的字符串数组。
getBytes()：返回字符串的 byte 类型数组。
length()：返回字符串长度。
toLowerCase()：将字符串转成小写字母。
toUpperCase()：将字符串转成大写字符。
substring()：截取字符串。
equals()：字符串比较。
```


## 2.5 拆箱装箱
答：装箱就是自动将基本数据类型转换为包装类型；拆箱就是将包装类型转换为基本数据类型。

### 2.5.1 分类
答：Java为每个原始基本类型都提供了包装类。
- 原始类型: boolean，char，byte，short，int，long，float，double
- 包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double


### 2.5.2 Integer的坑
答：写题时遇到。
- 对于int，==比较的是值；
- 对于Integer，当-128~127时，==比较的是值；(直接引用常量池中内容)
- 当超过范围，==比较的是内存地址。




# 3. 面向对象
答：面向对象是一种思想，将复杂问题简单化模块化。面向过程时具体化流程化。面向对象就是把过程抽象成类，封装成模块。

## 3.1 三大特性

 - 封装：将事物封装为一个类，隐藏细节，对外提供接口访问。当接口内部改变时，不影响外部调用。
 - 继承：从已知类中派生新类，新类拥有父类的属性和方法，并能增加新功能或重写父功能。
 - 多态：同一接口能有不同的解释，产生不同的结果。实现方式：继承、重载、向上转型。
多态是同一个行为具有多个不同表现形式或形态的能力。 

### 3.1.1 重写和重载
- 重写，即父类有方法A，子类拓展了方法A并加了新功能，就是子类重写了A。
- 重载，即一个类中存在多个同名，但参数个数、顺序和类型不同的方法。

**只有返回值不同，不构成重载！**

**构造器不能被重写，只能被重载**

## 3.2 五大基本原则
- 单一职责原则SRP(Single Responsibility Principle)：类的功能要单一，不能包罗万象。
- 开放封闭原则OCP(Open－Close Principle)：对拓展开放，对修改封闭。
- 里式替换原则LSP(the Liskov Substitution Principle LSP)：子类可以替换父类且毫无察觉。
- 依赖倒置原则DIP(the Dependency Inversion Principle DIP)：高层模块不应依赖于低层模块，他们都应该依赖于抽象。抽象不应依赖于具体实现，具体实现应该依赖于抽象。
- 接口分离原则ISP(the Interface Segregation Principle ISP)：多个特殊接口比一个通用接口更好。

## 3.3 抽象类和接口
**抽象是对类的抽象，接口是对行为的抽象**

相同点:
- 都不能实例化；
- 都包含抽象方法。

不同点:

| |抽象类 | 接口
|---|---|---
|构造方法|有|不存在
|声明|abstract | interface
|实现|extends继承抽象类 | implements实现接口
|构造器|有 | 无
|继承方式|单继承 | 多继承
|字段声明|任意 | 默认static和final
|修饰符|public、protected、default | public

注：抽象类中不能有静态方法(final就不能被继承，矛盾)，JDK1.8后接口能有静态方法。

### 3.3.1 静态与非静态的区别
答：静态是指用static关键字修饰的类、方法、字段等，非静态是指没有用static修饰的。
- 静态只能访问静态的，非静态既能访问静态的也能访问非静态的
- 静态的直接可以用类名调用，非静态的需要先实例化再调用。
- 静态变量存在常量池中，非静态成员变量在堆中
- 静态变量随类的加载而加载，消失而消失，非静态成员变量随对象创建而存在，消失而消失

## 3.4 内部类
答：把一个类定义在另一个类的内部就是内部类，分为成员内部类、局部内部类、匿名内部类和静态内部类。

### 3.4.1 优点
- 对包的其他类不透明，封装性好；
- 内部类能实现多重继承；
- 内部类能访问创建它的外部类对象。

### 3.4.2 匿名内部类
- 必须继承一个抽象类或者实现一个接口。
- 不能定义任何静态成员/方法。
- 匿名内部类使用的外部变量必须声明为 final。

注：final是因为生命周期不一致。final可以确保局部/匿名内部类使用的变量与外层的局部变量区分开。

```java
public class Outer {
    private void test(final int i) {
        new Service() {
            public void method() {
                for (int j = 0; j < i; j++) {
                    System.out.println("匿名内部类" );
                }
            }
        }.method();
    }
 }
 //匿名内部类必须继承或实现一个已有的接口 
 interface Service{
    void method();
}
```
## 3.5 equals()和hashCode()
答：equals()判断两个对象是否相等，hashCode()获取对象的哈希码，根据哈希码确定对象在哈希表中的索引。
### 3.5.1 关系
1. 两个对象相等，hashCode()也一定相同；
2. hashCode值相同，对象不一定相等；
3. equals()重写时，hashcCode()也一定要被重写，为保证两个对象的哈希值也相等，避免hashmap逻辑冲突。


## 3.6 值传递
答：概括为：
- 值传递，传递对象的副本，即修改副本，不影响源对象。
- 引用传递，传递对象的引用，即修改引用对象，会影响源对象。
- **Java只有值传递，而这个值实际上是对象的引用**！理解为共享传递。

## 3.7 拷贝
1. 浅拷贝：对基础数据类型值传递，对引用数据类型引用传递；
2. 深拷贝：对基础数据类型值传递，对引用数据类型，创建新对象，复制其内容。

# 4. IO流
## 4.1 分类
- 按流向，分为输入流和输出流；
- 按操作单元，分为字节流和字符流；
- 按流的角色，分为节点流和处理流。

归总，所有的IO都是从4个基类拓展得来。
- InputStream字节输入流/Reader字符输入流
- OutputStream字节输出流/Writer字符输出流

## 4.2 BIO、NIO和AIO
- BIO同步阻塞IO：传统IO，数据的读取写入必须阻塞在一个线程内等待完成。使用简单，但并发能力低；
- NIO同步非阻塞IO：升级版，Client和Server通过通道Channel通信，监听通道的事件变化，如果有数据变化就通知线程进行读写操作。多路复用。
- AIO异步非阻塞IO：再升级版，异步是指服务端线程接收到客户端管道后就交给底层处理IO通信，自己可以做其他事情。基于事件和回调实现。

# 5. 反射
答：反射机制是指在运行中，任意一个类，都能知道这个类的所有属性和方法；任意一个对象，都能调用它的属性和方法。简而言之，**动态获取和调用对象的属性和方法。** 

优点是灵活度高，缺点是慢慢慢！

## 5.1 相关类
- Class：类，用于获取类的相关信息
- Field：成员变量，用于获取实例变量和静态变量
- Method：方法，用于获取类中的方法参数和类型
- Constructor：构造器，用于获取构造器的相关参数和类型


## 5.2 流程
1. 获取class对象。Class.forName，getClass()，.class获得。
2. newInstance()获取class对象的实例，通过getFields,getConstructors,getMethods获取类成员，set/get访问字段，invoke()调用方法。


## 5.3 为什么慢
1. class.getMethod()，遍历所有的方法，匹配不到就去找遍历父类。最耗时的地方，用备忘录将method记录能降低一个数量级的消耗。
2. method.invoke()，参数为Object数组，需要对基本类型装箱。


## 5.4 举例
```java
public class Demo1 {
    public static void main(String[] args) throws Exception {
        User ming = new Usetr();
        String className = "com.bys.User";
        // 获取Class对象
        Class clazz = ming.getClass();
        Class clazz = Class.forName(className);
        Class clazz = User.class;
        // 创建User对象
        User user = (User)clazz.newInstance();
        // 和普通对象一样，可以设置属性值
        user.setUsername("bys");
        System.out.println(user);
    }
}
 
class User {
    private String username;
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username;
    }
}
```


# 6. 元注解
答：提供了4个元注解，作用是负责注解其他注解。
- @Target，注明注解用于什么地方。
- @Rentention，注明注解的生命周期，分为SOURCE, CLASS, RUNTIME三种策略。
- @Documented，注明是否将注解信息添加在java文档中。
- @Inherited，注明该注释和子类的关系。即一个类被@Inherited的注解标注，其子类也被这个注解标注。

## 6.1 注解的流程
答：作用：代替繁琐的配置文件，简化开发。
### 定义
定义注解类必须使用@interface
### 属性
value就是属性。指定属性后，使用注解时需要给属性赋值。 
```java
public @interface MyAnn { 
    String value(); 
    int value1(); 
}
// 使用注解MyAnn，可以设置属性
@MyAnn(value1=100,value="hello") 
public class MyClass { 
}
```


# 7. Java中的泛型的理解
答：泛型是当创建对象或调用方法时才去确定数据类型的特殊类型。
## 7.1 好处
不用进行强转；程序更加健壮；可读性更好，在创建集合时限定类型方便。
## 7.2 泛型基础
### 7.2.1 泛型类
泛型类就是把泛型定义在类上，用户使用该类的时候，才把类型明确下来。
```java
public class ObjectTool<T> {
    private T obj;

    public T getObj() {
        return obj;
    }

    public void setObj(T obj) {
        this.obj = obj;
    }
}
```
### 7.2.2 泛型方法
外界调用时传进来什么类型，返回值就是什么类型。
```java
    //定义泛型方法..
    public <T> void show(T t) {
        System.out.println(t);
    }
```
### 7.2.3 泛型类的子类
分为子类明确和子类不明确
```java
//子类明确
public class InterImpl implements Inter<String> {
    @Override
    public void show(String s) {
        System.out.println(s);
    }
}
//子类不明确
public class InterImpl<T> implements Inter<T> {
    @Override
    public void show(T t) {
        System.out.println(t);
    }
}
```
## 7.3 类型通配符'?'
泛型提供了类型通配符 ? ，表示匹配任意类型。在用这个时，必须注意：**只能调用和对象无关的方法，不能调用和对象有关的方法，直到外界确定了具体的类型。**
```java
public void test(List<?> list){
    for(int i=0;i<list.size();i++){     
        System.out.println(list.get(i));  
    }
}
```
### 7.3.1 通配符上限
表示List接收的元素只能是Number自身或子类。
```java
List<? extends Number>
```
### 7.3.2 通配符下限
表示传递进来的只能是Type或其父类。
```java
<? super Type>
```

## 7.4 泛型擦除
在已经确定泛型类型的集合赋值给普通集合时，发生泛型擦除，即**保留类型参数的上限**。如List<String> list传递给List list1，保留的是String的上限Object。

# 8. JIT编译器
答：JIT(Just In Time Compile)即时编译器，把经常运行的代码编译成和本地平台相关的机器码并优化，包括逃逸分析、锁消除、空值检测消除等。
## 8.1 逃逸分析
答：逃逸分析的基本行为就是分析对象动态作用域。当一个对象在方法中被定义时，它可能被外部方法引用，例如作为参数传递，这种称为方法逃逸。作用是，经过分析，发现对象不被外界访问，经过JIT优化，把对象拆解成若干标量替换。

# 9. 异常
## 9.1 分类
1. Throwable是所有错误Error和异常Exception的父类；
2. Error一般为程序运行错误，不能被动态处理，只能安全终止；
3. Exception分为运行时异常RuntimeException可以被捕获处理，非运行时异常CheckedException编译阶段强制要求处理(ide中标红)。

## 9.2 处理方式
1. 抛出异常。遇到异常不做处理直接抛出，throws作用于方法上，throw作用于方法内。
2. try/catch/finally，try中的异常被catch捕获处理，finally无论是否发生异常都执行，一般用来释放资源。

