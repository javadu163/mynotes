# JDK,JVM,JRE区别
- jdk:java开发工具包。
    - jdk是用来开发java项目的工具包。其包含jvm,jre和一些Java工具（javac编译器，jar打包工具等）。**能够创建和编译java程序**
- jre:java运行时环境。
    - 运行java项目必须的环境。主要包括jvm和java系统库类(java自带的库类) 。通常线上部署项目，可以不用安装jdk，安装jre即可。因为**有jre，java项目便可以运行**。
- jvm:java虚拟机
    - jvm是运行Java字节码的虚拟机。jdk将.java文件编译成.class字节码文件之后。需要由jvm进行解释，翻译成对应平台的机器码执行。**jvm是java跨平台的核心。**
    


# jvm 虚拟机内存

 > Java虚拟机在执行Java程序的过程中会将其管理的内存划分为若干个不同的数据区域，这些区域有各自的用途、创建和销毁的时间，有些区域随虚拟机进程的启动而存在，有些区域则是依赖用户线程的启动和结束来建立和销毁。Java虚拟机所管理的内存包括以下几个运行时数据区域，如下图所示：  

![image](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=47cc45529b2bd40756cadbaf1ae0f534/bf096b63f6246b6042db690ee7f81a4c500fa2f7.jpg)

### 各部分简要说明：

#### 2.1 程序计数器
>行号指示器，字节码指令的分支、循环、跳转、异常处理、线程恢复(CPU切换)，每条线程都需要一个独立的计数器，线程私有内存互不影响,该区域不会发生内存溢出异常。

#### 2.2 虚拟机栈 
>虚拟机栈是Java方法执行的内存模型，每个方法被执行时都会创建一个栈帧.栈帧用于存储：局部变量表（存放了编译期可知的各种数据类型。基本 数据类型和引用数据类型的句柄。）、操作数栈、动态链接、方法出口等，每个方法执行中都对应虚拟机栈帧从入栈到处栈的过程。虚拟机栈是线程私有的，声明周期与线程相同。如果内存不够会抛出stackOverFlowError;

```
 /**
     * 栈溢出 stackOverFlowError
     * 递归调用，每次调用方法都会在虚拟机栈中为该方法开辟内存空间。
     * 如果无限递归，则虚拟机栈空间将不足，抛出stackOverFlowError 系统错误。
     * stackOverFlowError 常见出现场景：
     * 1.无限递归
     * 2.执行了大量方法，导致线程栈空间耗尽
     * 3.方法内声明了海量的局部变量。
     *
     * 如何调整虚拟机栈大小：
     * 通过 JVM 启动参数 -Xss 增加线程栈内存空间
     *
     */
    public static void t(){
        t();
    }
    public static void main(String[] args) {
        t();
    }
    
```
https://blog.csdn.net/lilizhou2008/article/details/100589085


#### 2.3 本地方法栈
>与虚拟机栈类似，虚拟机栈为Java程序服务，本地方法栈支持虚拟机的运行服务，具体实现由虚拟机厂商决定。如果内存不够会抛出 stackOverFlowError、OutOfMemory异常。

#### 2.3 方法区
>用于存储已被**虚拟机加载的类信息**、常量、静态变量、即时编译后的代码等数据  

**常量池**
>方法区的一部分，Class的版本、字段、接口、方法等，及编译期生成的各种字面量、符号引用，编译类加载后存放在该区域。会抛出OutOfMemory异常。
#### 2.4  堆
>虚拟机管理内存中最大的一部分，用于存放对象实例(对象、数组)，物理上不连续的内存空间，被所有线程共享。  

>由于GC垃圾回收机制为分代回收，所以划分为：新生代 Eden、From SurVivor空间、To SurVivor空间，allot buffer(分配空间)，可能会划分出多个线程私有的缓冲区，老年代。

## 3、堆内存详解

![image](https://note.youdao.com/yws/public/resource/398a98a857c1debccb9ae24a91ca79eb/xmlnote/WEBRESOURCE78830b39210f31c4adac67d35ea49d18/4884)





>Java堆的内存划分如图所示，分别为（young gen）年轻代、Old Memory（老年代）、Perm（永久代）。其中在Jdk1.8中，永久代被移除，使用MetaSpace代替。  

### 新生代：
>新生代（Young）几乎是所有java对象出生的地方。即java对象申请的内存以及存放都是在这个地方。java中的大部分对象通常不会长久的存活， 具有朝生夕死的特点。  

**特点：**  
- 分为Eden、Survivor From、Survivor To，比例默认为8：1：1。
    - eden 占新生代80%的存储空间。用来存储对象。
    - survivor from 占新生代10%的空间。用来存储对象。
    - survivor to 占用10%的空间，平时空闲，GC时会将eden和survivor from 两个存储区域中存活的对象复制到此区域。复制完成之后eden和survivor from 需要将清空。
- 使用复制清除算法（Copinng算法），原因是年轻代每次GC都要回收大部分对象。新生代里面分成一份较大的Eden空间和两份较小的Survivor空间。每次只使用Eden和其中一块Survivor空间，然后垃圾回收的时候，把存活对象放到未使用的Survivor（划分出from、to）空间中，清空Eden和刚才使用过的Survivor空间。  
- 内存不足时发生Minor GC
### 老年代
> 当对象在 Eden ( 包括一个 Survivor 区域，这里假设是 from 区域 ) 出生后，在经过一次 Minor GC 后，如果对象还存活，并且能够被另外一块 Survivor 区域所容纳(上面已经假设为 from 区域，这里应为 to 区域，即 to 区域有足够的内存空间来存储 Eden 和 from 区域中存活的对象 )，则使用复制算法将这些仍然还存活的对象复制到另外一块 Survivor 区域 ( 即 to 区域 ) 中，然后清理所使用过的 Eden 以及 Survivor 区域 ( 即 from 区域 )，并且将这些对象的年龄设置为1，以后对象在 Survivor 区每熬过一次 Minor GC，就将对象的年龄 + 1，当对象的年龄达到某个值时 ( 默认是 15 岁，可以通过参数 -XX:MaxTenuringThreshold 来设定 )，这些对象就会成为老年代。 
但这也不是一定的，对于一些较大的对象 ( 即需要分配一块较大的连续内存空间 ) 则是直接进入到老年代。


（1）采用标记-整理算法（mark-compact），原因是老年代每次GC只会回收少部分对象。

## 4、永久代 
> Perm：用来存储类的元数据，也就是方法区。

- Perm的废除：在jdk1.8中，Perm被替换成MetaSpace，MetaSpace存放在本地内存中。原因是永久代进场内存不够用，或者发生内存泄漏。
- MetaSpace（元空间）：元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。
- 

## 5、垃圾回收机制(GC)

### 判断对象是否要回收
java是自动垃圾回收的。java 对象不可达时（对象没有被引用）便有可能被垃圾回收。 

对象被判定可被回收，需要经历两个阶段： 
-  第一个阶段是可达性分析，分析该对象是否可达
-  第二个阶段是当对象没有重写finalize()方法或者finalize()方法已经被调用过，虚拟机认为该对象不可以被救活，因此回收该对象。（finalize()方法在垃圾回收中的作用是，给该对象一次救活的机会）
-
### 方法区中的垃圾回收：
- （1） 常量池中一些常量、符号引用没有被引用，则会被清理出常量池
- （2） 无用的类：被判定为无用的类，会被清理出方法区。  
    判定方法如下：
    - A、 该类的所有实例被回收
    - B、 加载该类的ClassLoader被回收
    - C、 该类的Class对象没有被引用  

### finalize()
- GC垃圾回收要回收一个对象的时候，调用该对象的finalize()方法。然后在下一次垃圾回收的时候，才去回收这个对象的内存。
- 可以在该方法里面，指定一些对象在释放前必须执行的操作



### 常见的垃圾回收算法：
#### 1、Mark-Sweep（标记-清除算法）：
> 思想：标记清除算法分为两个阶段，标记阶段和清除阶段。标记阶段任务是标记出所有需要回收的对象，清除阶段就是清除被标记对象的空间。

> 优缺点：实现简单，容易产生内存碎片

#### 2、Copying（复制清除算法）：
> 思想：将可用内存划分为大小相等的两块，每次只使用其中的一块。当进行垃圾回收的时候了，把其中存活对象全部复制到另外一块中，然后把已使用的内存空间一次清空掉。

> 优缺点：不容易产生内存碎片；可用内存空间少；存活对象多的话，效率低下。

#### 3、Mark-Compact（标记-整理算法）：
> 思想：先标记存活对象，然后把存活对象向一边移动，然后清理掉端边界以外的内存。

> 优缺点：不容易产生内存碎片；内存利用率高；存活对象多并且分散的时候，移动次数多，效率低下

#### 4、分代收集算法：（目前大部分JVM的垃圾收集器所采用的算法）：

> 思想：把堆分成新生代和老年代。（永久代指的是方法区）
（1） 因为新生代每次垃圾回收都要回收大部分对象，所以新生代采用Copying算法。新生代里面分成一份较大的Eden空间和两份较小的Survivor空间。每次只使用Eden和其中一块Survivor空间，然后垃圾回收的时候，把存活对象放到未使用的Survivor（划分出from、to）空间中，清空Eden和刚才使用过的Survivor空间。
（2） 由于老年代每次只回收少量的对象，因此采用mark-compact算法。
（3） 在堆区外有一个永久代。对永久代的回收主要是无效的类和常量

### GC使用时对程序的影响？ 
垃圾回收会影响程序的性能，Java虚拟机必须要追踪运行程序中的有用对象，然后释放没用对象，这个过程消耗处理器时间


## 关于内存溢出
https://www.cnblogs.com/cxxjohnson/p/10481428.html


## java程序执行流程图

![image](https://oscimg.oschina.net/oscnet/7aaf784eea14c13e116f0741ea69370529e.jpg)


# 包装类，装箱拆箱与缓存池
## 为什么要有包装类？
java中数据类型大体分为两种，一种是基本数据类型，一种是引用数据类型。基本数据类型只有字面量，不能调用任何的方法。而java是面向对象的语言，这与对象的概念不符，因此java为每个基本数据类型提供了包装类。并为包装类提供了常用的一些方法。我们操作基本数据类型便可以像操作引用数据类型一样进行对象创建和方法调用。

## 装箱和拆箱
- 装箱：将基本数据类型转换成包装类。
- 拆箱：将包装类转换成基本数据类型。
- 装箱和拆箱操作可以自动实现。

```
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

## Integer.valueOf()与Integer.parseInt()区别
Integer.parseInt（）返回值为基本数据类型  
Integer.valueOf（）返回值为包装类型

## new Integer(123) 与 Integer.valueOf(123) 的区别?
包装类中有一个缓存池。在 Java 8 中，Integer 缓存池的大小默认为 -128~127。当调用Integer.valueOf()方法时，其先会去缓存池中找是否有对应的值，如果有则直接返回缓存池中的值。如果不存在，则会new Integer()。
而new Integer(123)则会直接创建一个新的对象。

```
Integer i=Integer.valueOf(100);
Integer i2=Integer.valueOf(100);
System.out.println(i==i2);//true
Integer i3=Integer.valueOf(500);
Integer i4=Integer.valueOf(500);
System.out.println(i3==i4);//false

Integer i5=600;//自动装箱 ，调用Integer.valueof()
Integer i6=600;//
System.out.println(i5==i6);//false
Integer i7=100;
Integer i8=100;
System.out.println(i7==i8);//true

```
基本类型对应的缓冲池如下：
- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F
 


# BigDecimal
### BigDecimal 的用处
《阿里巴巴Java开发手册》中提到：浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断。 具体原理和浮点数的编码方式有关，这里就不多提了，我们下面直接上实例：
```
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999964
System.out.println(a == b);// false
```
具有基本数学知识的我们很清楚的知道输出并不是我们想要的结果（精度丢失），我们如何解决这个问题呢？   
一种很常用的方法是：使用使用 BigDecimal 来定义浮点数的值，再进行浮点数的运算操作。
```
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);// 0.1
BigDecimal y = b.subtract(c);// 0.1
System.out.println(x.equals(y));// true 
```
### BigDecimal 的大小比较
a.compareTo(b) : 返回 -1 表示小于，0 表示 等于， 1表示 大于。
```
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.compareTo(b));// 1
```
### BigDecimal 保留几位小数
通过 setScale方法设置保留几位小数以及保留规则。保留规则有挺多种，不需要记，IDEA会提示。
```
BigDecimal m = new BigDecimal("1.255433");
BigDecimal n = m.setScale(3,BigDecimal.ROUND_HALF_DOWN);
System.out.println(n);// 1.255
```
### BigDecimal 的使用注意事项
>注意：我们在使用BigDecimal时，为了防止精度丢失，推荐使用它的 BigDecimal(String) 构造方法来创建对象。《阿里巴巴Java开发手册》对这部分内容也有提到如下图所示。
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019/7/BigDecimal.png)




# String

String 类是final的，别的类不能继承该类。且String的内容也是不可变的，因为在 Java 8 中，String内部使用一个final的char数组存储数据。
```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## String 为什么设计成不可变的？
#### 1. 可以缓存 hash 值

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

#### 2. String Pool 的需要

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

#### 3. 安全性

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。

#### 4. 线程安全

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。


## 字符串常量池
字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。
当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

```
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
```
> 字符串常量池的位置  
 Java 7 之前，String Pool 被放在运行时常量池中，即在方法区中。  
 而在 Java 7，String Pool 被移到堆中。
 
 
 
 # java中方法参数是值传递还是引用传递？
 java中只存在值传递。对于基本数据类型传递的是字面量，而对于引用数据类型，传递的是地址值。
 ```
  public static void  change(String str){
        str="abc";
    }
    public static void change(int i){
        i=100;
    }
    public static void change(String[] strArray){
        strArray[0]="hi";
    }

    public static void main(String[] args) {
        int i=10;
        String str="ABC";
        String [] strArray={"hello","yao"};
        change(i);
        change(str);
        change(strArray);
        System.out.println(i);//10
        System.out.println(str);//ABC
        System.out.println(Arrays.toString(strArray));//hi yao

    }
 ```
 
 # 继承关系的初始化顺序
 存在继承的情况下，初始化顺序为：  

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）


 # object类
 Object类是java中所有类的基类。其定义了所有类中共有的一些方法。常见的方法有：
 - equals
 - hashcode
 - clone 
 - toString
 - getClass
 - notify
 - notifyAll
 - wait
 
 
 # 深拷贝 vs 浅拷贝
- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/java-deep-and-shallow-copy.jpg)


# 抽象类和接口区别？
语法层面：  
使用层面：
抽象类是is-a的关系，而接口更像是has-a的关系。通常接口只定义行为方法。如果说你只有行为方法，那么通常使用接口。如果说你既有通用的属性，又有一些通用的方法。并且想要描述的是同一类事物，这时候用抽象类。


# 异常

![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-2/Exception.png)

在 Java 中，所有的异常都继承自Throwable类。Throwable下有Error和exception两个子类。
- error通常为jvm系统错误，程序无法处理。常见的有栈内存溢出和堆内存溢出。
- exception 为程序可以处理的异常。exception 类有一个重要的子类 RuntimeException。RuntimeException 异常由Java虚拟机抛出,为运行时异常。通常为程序代码编写错误导致 的。不需要显示的进行处理。而其他直接继承自exception的类则为检查性异常。必须显示的对其进行处理。通常处理方式为try catch捕获和throws 声明向外抛出。
- 

### 如何自定义异常
除了jdk中自带的异常类，我们使用框架过程中看到的很多异常都是框架自己定义的。自己定义异常的好处是可以针对异常进行特定的处理。  
自定义异常很简单，只需要继承Exception类，并重写其中的一些方法即可。

### finally 一定会执行吗？
不一定!在以下情况下finally不会执行：
 
- 在前面的代码中用了System.exit(int)已退出程序。 
- 程序所在的线程死亡。
- 

# 集合
java集合主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。
collection下面有分为List ，Set和Queue三种。  
具体类图如下：
![image](https://note.youdao.com/yws/public/resource/398a98a857c1debccb9ae24a91ca79eb/xmlnote/9A354C72991B4AC397D8F7EF220AE463/7231)

### 常见集合概述：
- List 列表，有序，可重复
- Queue 队列，有序，可重复
- Set 集合，无序，不可重复
- Map key-value映射，无序，key不可重复,value可重复

### 常见List
- ArrayList
    - 基于动态数组实现。内部维护这一个对象数组。
    ```
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access
    ```
    - 支持快速随机访问支持快速随机访问，查询和修改快，新增删除慢（涉及到数组的扩容和移位）
    - 默认初始大小为10。也可通过构造方法指定初始大小。
    ```
     ArrayList list=new ArrayList();//默认大小10
     ArrayList list2=new ArrayList(20);//指定初始大小为20
    ```
    - 扩容机制：每次扩容至旧容量的 1.5 倍。扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。
    - 扩容时机：当现有大小不够时进行扩容。
    - 线程不安全
 
- Voctor
    - 实现与ArrayList一致。只不过其方法增加了synchronized  关键字，为同步方法，线程安全，但是效率也降低了。
    - 扩容机制：默认每次扩容为原容量的2倍。
    - 因为效率较低，不推荐使用。
    - 替代方案：Collections.synchronizedList();或CopyOnWriteArrayList
     ```
     List<String> list = new ArrayList<>(); 
    List<String> synList = Collections.synchronizedList(list);
     ```
- LinkedList
    - 基于双向链表实现。新增删除快，查询慢。   
    
    内部包含一个Node静态内部类，用来存储链表的单个元素。
    ```
    private static class Node<E> {
        E item;// 当前节点元素内容
        Node<E> next;//下一个节点
        Node<E> prev;//上一个节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    ```
    每个链表存储了头部元素指针和尾部元素指针
    ```
     transient Node<E> first;//头部节点
     transient Node<E> last;//尾部节点
    ```
    - 不支持随机访问，只能顺序访问.从头部或者尾部开始遍历。
    - 常见操作图解
        - 在尾部添加
        ![image](https://images2018.cnblogs.com/blog/1285727/201807/1285727-20180717133746276-313211308.gif)
        - 靠近头部添加
        ![image](https://images2018.cnblogs.com/blog/1285727/201807/1285727-20180717134043714-1008528320.gif)
        - 靠近尾部添加
        ![image](https://images2018.cnblogs.com/blog/1285727/201807/1285727-20180717134100309-1487165805.gif)
        - 靠近头部的获取
        ![image](https://images2018.cnblogs.com/blog/1285727/201807/1285727-20180717134201828-1536755274.gif)
        - 靠近尾部的获取
        ![image](https://images2018.cnblogs.com/blog/1285727/201807/1285727-20180717134212062-1192617618.gif)
    

- copyOnWriteArrayList  
   - 读写分离
   >写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。所以也是线程安全的。
   - 适用场景
   > CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。
   - 存在问题：
   >内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
  
   > 数据不一致：读操作不能读取实时性的数据，因为可能部分写操作的数据还未同步到读数组中。
  
- Stack 
    - 栈结构，底层也是数组，继承自Vector, 先进后出FILO。
    - 默认容量为10
    - 扩容机制：与vector一致，2倍扩容。



### 常见Map
- hashMap
    - 基于哈希表实现。即数组+链表/红黑树。元素存储是无序的。
    - 默认大小为16
    - 扩容机制：默认装载因子为0.75f。即适用到实际容量的0.75时即会进行扩容。每次扩容为原来的2倍。
    - key和value都允许为null
- LinkedHashMap
    - 继承自hashMap ,在hashMap的基础上，增加了双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序
- hashTable
    - 与HashMap类似，但它是线程安全的。但因性能问提，基本不用。
    - 替代方案：ConcurrentHashMap ，适用分段锁，效率更高。
    - key和value都不允许为null 
- TreeMap
    -  基于红黑树实现。会根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序
    -  
- ConcurrentHashMap
    - ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。
- WeakHashMap 
    - WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。**WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。** 

### 常见Set
- HashSet
    - 底层通过HashMap实现。无序不可重复。 
- LinkedHashSet
    - 底层通过LinkedHashMap实现。会保留元素的插入顺序。
- TreeSet
    -  基于红黑树实现，会对存入的元素进行排序。底层由TreeMap 来实现。
    

### 常见Queue
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。
- 
https://www.bbsmax.com/A/nAJvb9e8Jr/


## 集合图解
https://www.cnblogs.com/xdecode/p/9321848.html


## hashMap 常见问题
> hashMap 底层实现？  


hashMap是通过hash表实现，其内部维护了一个entry数组，默认大小为16。
而每个entry又是一个单项链表。hashmap 存入数据时，会先根据存入的key的hashcode并进行hash运算，并将对象映射到entry数组对应的桶位。如果该位置发生hash冲突，则将数据添加到链表中。

> hashmap为什么不是安全的？  

 因为hashmap 底层维护了一个entry数组，这个数组**是一个全局的变量**。而对应的hashMap的方法都不是同步的。这样会导致多线程同时对entry数组进行操作时(赋值，扩容等)，出现线程安全问题。

> 怎样才让hashMap线程安全？

- 使用collections中方法synchronizedMap(map) 将hashMap转换成线程安全的map。
- 使用ConcurrentHashMap 替代Hash。

# 反射
### 概念
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

### 获取Class对象的方式
 使用反射首先要获取到的内容为Class对象。获取Class对象的方法有三种
 - Class.forName()
 - 类.class
 - 对象.getClass() 

### 反射的应用
- 动态创建对象
```
Class<? extends User> userClass=Class.forName("com.seecen.User");
//1、调用class的newInstance方法创建对象。会调用无参构造方法创建对象
User user=userClass.newInstance()
// 2、获取到无参构造方法
Constructor constructor = userClass.getConstructor();
User user2= constructor.newInstance();

 //3、获取有参构造方法
Constructor<? extends User> constructor = aClass.getDeclaredConstructor(String.class, int.class);
//通过有参构造方法创建对象
User user2 = constructor.newInstance("yao", 18);
```
- 动态获取对象属性和方法信息

```
 //获取声明的方法
Method[] declaredMethods = aClass.getDeclaredMethods();
//根据方法名和参数列表获取特定方法
Method setAge = aClass.getDeclaredMethod("setAge", int.class);

//获取属性信息
Field[] declaredFields = aClass.getDeclaredFields();
//根据名称获取属性信息
Field name = aClass.getField("name");
//获取属性的值
Object o = name.get(user1);
```
- 动态调用对象方法
```
Method setAge = aClass.getDeclaredMethod("setAge", int.class);

 //调用方法
setAge.invoke(user1, 18);

```
### 如何操作私有属性和方法
当反射操作私有属性和方法时，因为访问权限问题直接操作会报错。需要先设置允许访问即setAccessible(true)。

# IO 流
### Java 中 IO 流分为几种?
- 按照流的流向分，可以分为输入流和输出流；
- 按照操作单元划分，可以划分为字节流和字符流；
- 按照流的角色划分为节点流和处理流。

类图结构如下：
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/IO-%E6%93%8D%E4%BD%9C%E6%96%B9%E5%BC%8F%E5%88%86%E7%B1%BB.png)

![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/IO-%E6%93%8D%E4%BD%9C%E5%AF%B9%E8%B1%A1%E5%88%86%E7%B1%BB.png)

### 简单的几个问题？
- 如何将字节流转换成字符流？
- 字节流和字符流区别？
- 复制文件的步骤？
- File 类过滤文件夹中的文件？
- 序列化与反序列化使用那些流对象？


# 序列化
### 概念
将java对象转换成字节序列的过程叫序列化。同理将序列化后的字节序列化转换成java对象叫做反序列化。  
> **前提**

java 对要实现Serializable 接口。且该类的所有属性也要实现序列化接口。
> **流对象**  

序列化：ObjectOutputStream  
反序列化：ObjectInputStream

> **transient关键字和static**

transient关键字用于类的属性前，标识该属性不被序列化。static关键字也可以达到同样的效果。但通常会使用transient。
# 多线程


## 线程分类
线程可分为**用户线程(user thread)** 和 **守护线程(daemon thread)**。  
守护线程指在后台运行的线程，也称为后台线程，用于提供后台服务。  
Java创建的线程默认是用户线程。  

两者的差别是，当进程中还有用户线程在运行时，进程不终止；
当进程中只有守护线程在运行时，进程终止。  

Thread类与守护线程有关的方法声明如下：
```

//若on为true，则设置为守护线程，必须在启动线程前调用
public final void setDaemon(boolean on) 

//判断是否为守护线程，若是，则返回true;否则返回false
public final boolean isDaemon() 

```
## 线程状态
> 六种状态  

![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

> 状态之间的转化关系

![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%20%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)
## 创建线程方式

- 继承Thread，重写run方法
- 实现Runnable接口，并传入Thread构造方法
- 通过线程池的方式获取(**推荐**)

## 线程池
> **池化技术**相比大家已经屡见不鲜了，**线程池**、**数据库连接池**、**Http 连接池**等等都是对这个思想的应用。池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。

> 线程池优点
- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。


> 常见线程池

Java通过Executors提供四种线程池，分别为：
- newCachedThreadPool 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
```

```

- newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
```
//定长线程池
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
        for (int i = 0; i <10; i++) {
            fixedThreadPool.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName()+"执行时间："+new Date().toLocaleString());
                    try {
                        //睡一秒
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

```
- newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
```
 public static void  test4(){
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
        //延迟执行
        scheduledExecutorService.schedule( new RunnableTask(),5, TimeUnit.SECONDS);
        System.out.println("开始执行1："+new Date().toLocaleString());
        //周期性执行,initialDelay 首次执行延迟时间，period:执行间隔，时间单位
        scheduledExecutorService.scheduleAtFixedRate(new RunnableTask(),5,2,TimeUnit.SECONDS);
//
        System.out.println("开始执行2："+new Date().toLocaleString());
        //周期性执行，等待前一个任务执行完成之后，延迟指定时间执行下一个任务。
        scheduledExecutorService.scheduleWithFixedDelay(new RunnableTask(),5,2,TimeUnit.SECONDS);
    }

```
- newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。


> 线程池的几个方法区别
- runnable和callable区别？
    - 实现Callable接口的任务线程能返回执行结果；而实现Runnable接口的任务线程不能返回结果；
    - Callable接口的call()方法允许抛出异常；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛；

> **注意点**  
Callable接口支持返回执行结果，此时需要调用FutureTask.get()方法实现，此方法会阻塞主线程直到获取‘将来’结果；当不调用此方法时，主线程不会阻塞！

- submit与execute和invokeAll区别？
    -  excutor没有返回值，submit有返回值，并且返回执行结果Future对象
    - excutor不能提交Callable任务，只能提交Runnable任务，submit两者任务都可以提交
    - 在submit中提交Runnable任务，会返回执行结果Future对象，但是Future调用get方法将返回null（Runnable没有返回值）
    - submit和execute只能执行单个任务，而invokeAll可以执行多个任务。


> 如何确定线程池的大小

- CPU 密集型任务(N+1)：   
    - 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。
- I/O 密集型任务(2N)：                     
     - 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。
     
**Runtime.getRuntime().availableProcessors() 获取cpu核心数**


### 线程池满了再往里面加线程会怎样？
如果线程池中有的线程都在处理任务，那么新提交的任务会在阻塞队列中等待。如果阻塞队列也满了，再提交新的任务的话，则会抛出异常RejectedExecutionException。
解决： 
- 根据情况适当增大任务队列的大小。
- 使用LinkedBlockingQueue存储任务队列



### 线程安全
线程安全：当多个线程并发执行某程序与单个线程执行的结果总是一致时，我们成为线程安全，如果不然，则为线程不安全。

> 线程安全实现方式
- synchronized 关键字
    - 用在方法上
    - 用在代码块中
- Lock对象
    - ReentrantLock 
    - lock方法上锁
    - unLock方法释放锁
    

### 什么情况下会发生死锁，程序死锁了如何解决。
死锁：多线程因无法获取锁资源而一直等待，无法继续执行。
死锁可能的原因：
- 锁资源没有释放，导致后续线程无法获取到锁。
- 同步方法之间相互调用，互相等待对方释放锁，导致死锁。
- 锁的嵌套使用

解决方法：
- 尽量避免锁之间的嵌套使用
- 锁的释放放在finally中保证锁资源最终都会被释放
- 尽量缩短加锁的时间。
- 不使用显示的去锁，我们用信号量去控制。


### 线程间怎么通信，你知道几种方式
https://blog.csdn.net/pengzhisen123/article/details/79455742
# jdk1.8新特性

- 接口允许有default方法和static 方法
```
public interface Interface8 {
    default void hello(){
        System.out.println("hello this is interface default method");
    }
    static void hi(){
        System.out.println("hi this is interface static method");
    }
}
```

- Lambda表达式 
    - Lambda 是一个匿名函数，主要用来简化`函数式接口`（仅包含一个public方法的接口）的创建 。
    -  语法：
        - -> 为lambda操作符。具体写法为
        ```
        Lambda 表达式的参数列表 -> Lambda 表达式中所需执行的功能
        ```
        ```
        public static void main(String[] args) {
        Thread thread = new Thread(
                () -> {
             // runnable 接口 run 方法的方法体，使用lambda,则成为lambda体       
            //lambda 中需要执行的代码，如果只有一行可以省略{}
            System.out.println("hello ");
            System.out.println("lambda");
             });
        thread.start();
            }
        }
        ```
条件 | 表达式写法
---|---
无参数无返回值 | () -> System.out.println(“Hello WOrld”)
有一个参数无返回值 |	(x) -> System.out.println(x)
有且只有一个参数无返回值 |	x -> System.out.println(x)
有多个参数，有返回值，有多条lambda体语句 |	(x，y) -> {System.out.println(“xxx”);return xxxx;}；
有多个参数，有返回值，只有一条lambda体语句 |	(x，y) -> xxxx

- 函数式接口  
    - 简单来说就是只定义了一个抽象方法的接口（Object类的public方法除外），就是函数式接口，并且还提供了注解：@FunctionalInterface
    - 函数式接口的提出是为了给Lambda表达式的使用提供更好的支持

- Stream流
    - java.util.Stream 表示能应用在一组元素上一次执行的操作序列。Stream 操作分为中间操作或者最终操作两种，最终操作返回一特定类型的计算结果，而中间操作返回Stream本身，这样你就可以将多个操作依次串起来。Stream 的创建需要指定一个数据源，比如 java.util.Collection 的子类，List 或者 Set， Map 不支持。
    - Stream操作的三个步骤
        - 创建stream
        ```
        // 1，校验通过Collection 系列集合提供的stream()或者paralleStream()
        List<String> list = new ArrayList<>();
        Strean<String> stream1 = list.stream();
        // 2.通过Arrays的静态方法stream()获取数组流
        String[] str = new String[10];
        Stream<String> stream2 = Arrays.stream(str);
        // 3.通过Stream类中的静态方法of
        Stream<String> stream3 = Stream.of("aa","bb","cc");
       
        ```
        - 中间操作（过滤、map）
        ```
                /**
                 * 筛选 过滤  去重
                 */
          emps.stream()
                  .filter(e -> e.getAge() > 10)
                  .limit(4)
                  .skip(4)
                  // 需要流中的元素重写hashCode和equals方法
                  .distinct()
                  .forEach(System.out::println);
        
        
          /**
           *  生成新的流 通过map映射
           */
          emps.stream()
                  .map((e) -> e.getAge())
                  .forEach(System.out::println);
        
        
          /**
           *  自然排序  定制排序
           */
          emps.stream()
                  .sorted((e1 ,e2) -> {
                      if (e1.getAge().equals(e2.getAge())){
                          return e1.getName().compareTo(e2.getName());
                      } else{
                          return e1.getAge().compareTo(e2.getAge());
                      }
                  })
                  .forEach(System.out::println);
        ```
        - 终止操作
        ```
         /**
         *      查找和匹配
         *          allMatch-检查是否匹配所有元素
         *          anyMatch-检查是否至少匹配一个元素
         *          noneMatch-检查是否没有匹配所有元素
         *          findFirst-返回第一个元素
         *          findAny-返回当前流中的任意元素
         *          count-返回流中元素的总个数
         *          max-返回流中最大值
         *          min-返回流中最小值
         */

        /**
         *  检查是否匹配元素
         */
        boolean b1 = emps.stream()
                .allMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b1);

        boolean b2 = emps.stream()
                .anyMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b2);

        boolean b3 = emps.stream()
                .noneMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b3);

        Optional<Employee> opt = emps.stream()
                .findFirst();
        System.out.println(opt.get());

        // 并行流
        Optional<Employee> opt2 = emps.parallelStream()
                .findAny();
        System.out.println(opt2.get());

        long count = emps.stream()
                .count();
        System.out.println(count);

        Optional<Employee> max = emps.stream()
                .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
        System.out.println(max.get());

        Optional<Employee> min = emps.stream()
                .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
        System.out.println(min.get());

        ```
    - 新的日期API
        - 新的日期API都是线程安全的。
        - LocalDate 本地日期
        ```
        public static void localDateTest() {

        //获取当前日期,只含年月日 固定格式 yyyy-MM-dd    2018-05-04
        LocalDate today = LocalDate.now();
        LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
        System.out.println("明天的日期: "+tomorrow);
        LocalDate yesterday = tomorrow.minusDays(2);
        System.out.println("昨天的日期: "+yesterday);
        LocalDate independenceDay = LocalDate.of(2019, Month.MARCH, 12);
        DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
        System.out.println("今天是周几:"+dayOfWeek);//TUESDAY

        // 根据年月日取日期，5月就是5，
        LocalDate oldDate = LocalDate.of(2018, 5, 1);

        // 根据字符串取：默认格式yyyy-MM-dd，02不能写成2
        LocalDate yesteday = LocalDate.parse("2018-05-03");

        // 如果不是闰年 传入29号也会报错
        LocalDate.parse("2018-02-29");
        
        
         }
        ```
        - LocalTime 本地时间:  
        LocalTime 定义了一个没有时区信息的时间
        ```
         // 当前时间
		LocalTime now = LocalTime.now();
		System.out.println(now);
		
		// 22:33
		LocalTime localTime = LocalTime.of(22, 33);
		System.out.println(localTime);
		
		// 一天中的4503秒
		LocalTime ofDay = LocalTime.ofSecondOfDay(4503);
        ```
        - LocalDateTime  
        是LocalDate和LocalTime的组合体，表示的是不带时区的日期及时间
        ```
        // 当前时间
		LocalDateTime now = LocalDateTime.now();
		System.out.println(now);
 
		// 当前时间加上25小时３分钟
		LocalDateTime plusMinutes = now.plusHours(25).plusMinutes(3);
		System.out.println(plusMinutes);
 
		// 转换
		LocalDateTime of = LocalDateTime.of(1993, 2, 6, 11, 23, 30);
		System.out.println(of);
        ```
        
# 常见算法

- 冒泡排序
```
/**
     * 冒泡排序
     * @param array
     * @return
     */
    public static void bubbleSort(int[] array) {

        for (int i = 0; i < array.length-1; i++) {//控制比较的趟数
            for (int j = 0; j < array.length - 1 - i; j++) {//控制每趟比较的次数
                if (array[j + 1] < array[j]) {//交换两个相邻元素的位置
                    int temp = array[j + 1];
                    array[j + 1] = array[j];
                    array[j] = temp;
                }
            }
        }
    }
```
- 二分查找
```
int bsearchWithoutRecursion(int a[], int key) {
    int low = 0;
    int high = a.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] > key)
            high = mid - 1;
        else if (a[mid] < key)
            low = mid + 1;
        else
            return mid;
    }
    return -1;
}
```
# 常见设计模式

- 单例
    - 饿汉模式:构造私有化，静态初始化对象。提供对象方法。
    ```
    public class Singleton {

        private static Singleton uniqueInstance=new Singleton();
    
        private Singleton() {
        }
    
        public static Singleton getUniqueInstance() {
            return uniqueInstance;
        }
    }
    ```
    - 枚举方式:可以避免反射，反序列化创建对象。
    ```
    public enum Singleton {

    INSTANCE;

    private String objName;
    .....
    
    ```
- 简单工厂

简单工厂把实例化的操作单独放到一个类中，这个类就成为简单工厂类，让简单工厂类来决定应该用哪个具体子类来实例化。

这样做能把客户类和具体子类的实现解耦，客户类不再需要知道有哪些子类以及应当实例化哪个子类。客户类往往有多个，如果不使用简单工厂，那么所有的客户类都要知道所有子类的细节。而且一旦子类发生改变，例如增加子类，那么所有的客户类都要进行修改。

```
public interface Product {
}
```
```
public class ConcreteProduct implements Product {
}
```
```
public class ConcreteProduct1 implements Product {
}
```
```
public class ConcreteProduct2 implements Product {
}
```
```
public class SimpleFactory {

    public Product createProduct(int type) {
        if (type == 1) {
            return new ConcreteProduct1();
        } else if (type == 2) {
            return new ConcreteProduct2();
        }
        return new ConcreteProduct();
    }
}
```
```
public class Client {

    public static void main(String[] args) {
        SimpleFactory simpleFactory = new SimpleFactory();
        Product product = simpleFactory.createProduct(1);
        // do something with the product
    }
}
```



# 数据库
## 常用函数
- 单行函数
    - 字符函数
        * concat(X,Y):连接字符串x和y。与||功能类似
        * length(x)：返回x的length
        * lower(x):小写
        * upper(x):大写
        * substr(x,start,length):从start开始截取length个字符
        * replace(x,old,new):将x中的old替换为new
        * trim([str from] x)：去除x两端的str ，不指定默认为空格。
    - 日期函数
        * add_months(date,month) 在date上添加month月
        * months_between(date1,date2) 两个日期间相差的月份
        * last_day(date) date所在月份的最后一天 
        ```
        select to_char(add_months(last_day(sysdate)+1,-1),'yyyy-mm-dd'),last_day(sysdate) from dual;
        
        ```
        * extract(fm from date) 提取日期中的特定部位
        ```
        fmt 为：YEAR、MONTH、DAY、HOUR、MINUTE、SECOND。其中 YEAR、MONTH、DAY可以为 DATE 类型匹配，也可以与 TIMESTAMP 类型匹配；但是 HOUR、MINUTE、SECOND 必须与 TIMESTAMP 类型匹配。
        ```
    - 数字函数
        * MOD(X,Y) ：X除以Y的余数
        eg：MOD(8，3)=2
        * ROUND(X[,Y]) X在第Y位四舍五入
        eg： ROUND(3.456，2)=3.46
        
        * TRUNC(X[,Y]) X在第Y位截断
        eg：TRUNC(3.456，2)=3.45

    - 转换函数
        * to_date(字符串,日期格式) ：将字符串转换成日期类型
        常见的格式有：
        1）yyyy-mm-dd hh24:mi:ss  24小时制
        2）yyyy-mm-dd hh:mi:ss       12小时制
        
        * to_char(日期/数字,字符串格式) 将日期或者数字转换成字符串
        * TO_NUMBER(X,[,fmt])
- 聚合函数
    - sum
    - avg
    - max
    - min
    - count
- 
## 表连接方式
- 内连接
    - inner join ，inner可省略。两张表根据指定的关联条件进行关联。只有符合关联条件的数据会查询出来。
- 外连接
    -  左外连接：`left join`。以左表为主表，左表的数据都会显示，右表的数据符合关联条件的才显示，不符合关联条件的显示为null。
    -  右外连接：`right join `。以右表为主表，右表的数据都会显示，左表的数据符合关联条件的才显示，不符合关联条件的显示为null。
    -  全外连接：`full join`。两张表的数据都会显示。相当于 左连接 union 右连接。
- 交叉连接：结果为笛卡儿积
- 自连接：特殊的一种表连接方式。连接的两张表为同一张表。

## 索引
索引是用来提供数据库数据检索效率的数据库对象。类似于书的目录。建立在表的字段上。当查询的时候使用到索引字段作为筛选条件时，便会走索引。
> 优点  
能够加快查询效率

> 缺点  
- 增加了数据库大小
- 降低新增，修改删除效率。

> 创建语法  

create index 索引名 on 表名(字段名。。。)

> 如何查看sql是否走索引

查看sql的EXPLAIN 执行计划。
## 视图
用以存储一条sql语句（单表或多表）查询结果的虚表。  视图只有逻辑定义。每次使用的时候,只是重新执行SQL。


> 创建语法：
```
create [ or replace ] [ force ] view [schema.]view_name
                      [ (column1,column2,...) ]
                      as 
                      select ...
                      [ with check option ]
                      [ constraint constraint_name ]
                      [ with read only ];

tips:
 1 or replace: 如果存在同名的视图, 则使用新视图"替代"已有的视图
 2 force: "强制"创建视图,不考虑基表是否存在,也不考虑是否具有使用基表的权限
 3 column1,column2,...：视图的列名, 列名的个数必须与select查询中列的个数相同;   
如果select查询包含函数或表达式, 则必须为其定义列名.  
此时, 既可以用column1, column2指定列名, 也可以在select查询中指定列名.
 4 with check option: 指定对视图执行的dml操作必须满足“视图子查询”的条件即,  
对通过视图进行的增删改操作进行"检查",要求增删改操作的数据,  
 必须是select查询所能查询到的数据,否则不允许操作并返回错误提示.   
默认情况下, 在增删改之前"并不会检查"这些行是否能被select查询检索到. 
 5 with read only：创建的视图只能用于查询数据, 而不能用于更改数据.
```

> 优点：
1. 可以限制用户只能通过视图检索数据。这样就可以对最终用户屏蔽建表时底层的基表。
2. 可以将复杂的查询保存为视图。可以对最终用户屏蔽一定的复杂性。
3. 限制某个视图只能访问基表中的部分列或者部分行的特定数据。这样可以实现一定的安全性。
4. 从多张基表中按一定的业务逻辑抽出用户关心的部分，形成一张虚拟表。

> 视图占用空间吗？

普通视图不占用空间，固化视图占用空间。

> 视图可以加快查询吗

不能。

## 存储过程
是一个数据库对象，一组为了完成特定功能的SQL 语句集。  

> 优点：
* 存储在数据库中，经过第一次编译后再次调用不需要再次编译。速度快
* 安全性高，可以防止sql注入，可以设定执行权限。
* 可以将多个复杂的sql封装到存储过程，减少建立连接执行sql的时间，效率更高

> 缺点:
* 调试比较麻烦
* 移植性和可维护性不是很强

> 语法：
```
CREATE [OR REPLACE] PROCEDURE 存储过程名

[ (argment [ { IN | IN OUT }] Type,

      argment [ { IN | OUT | IN OUT } ] Type ]

       [ AUTHID DEFINER | CURRENT_USER ]

{ IS | AS }

<变量声明区>

BEGIN

<执行部分>

EXCEPTION

<可选的异常错误处理程序>

END;
```
> 存储过程调用  

- 在PL/SQL中调用
可以直接调用
```
declare 
  v_name h_user.name%TYPE;
  v_id h_user.id%TYPE :=&userId;
  v_dept h_user.dept%TYPE;
  
  begin
    queryUser(v_id,v_name,v_dept);
    DBMS_OUTPUT.put_line(v_id||':'||v_name||':'||v_dept);
  end;
```
- 在jdbc中调用
```
public void method2(){
		conn = getConn();
		String sql = "call queryUser(?,?,?)";
		try {
			CallableStatement cs = conn.prepareCall(sql);
			cs.setInt(1,1);
                        cs.registerOutParameter(2, Types.VARCHAR);
			cs.registerOutParameter(3,Types.INTEGER);
			cs.execute();
			String name= cs.getString(1);
                        int dept=cs.getInt(2);
			System.out.println(name+":"+dept);
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

```

- 在mybatis中调用
```
statementType里的CALLABLE是标注此sql为存储过程。
parameterType是标注要传的参数，看了一些资料不写parameterType的话默认传map。还是加上比较清晰
<select id="queryUser" parameterType="java.util.map" statementType="CALLABLE">
{call queryUser(#{userid,mode=IN,jdbcType=VARCHAR},#{name,mode=OUT,jdbcType=VARCHAR},#{dept,mode=OUT,jdbcType=INTEGER})}
</select>

```

- 在hibernate中调用
```
1、在对象的映射文件中定义存储过程
<sql-query name="getUser" callable="true">     
       <return alias="user" class="com.test.User">     
      <return-property name="id" column="id" />     
      <return-property name="name" column="name" />     
      <return-property name="age" column="age" />     
      </return>     
      {call proc()}     
    </sql-query>    

2、Hibernate API 对存储过程的调用：
Session ss= HibernateSessionFactory.getSession()     
 List li=ss.getNamedQuery("getUser").list();     
 ss.close();     
```

## 数据库备份与还原
> 备份导出  

exp 用户名/密码@ip/实例 file='导出的文件.dmp' log=日志文件.log
```
例：exp sc1804/123456@localhost/orcl file=etms20170818_70.dmp log=0818_70.log 
```

> 还原  

imp 用户名/密码@ip/实例 file=备份文件 ignore=y full=y

```
例：imp test/123456@192.168.108.70/orcl file=etms20170818_70.dmp ignore=y full=y log=0818_imp.log
```
## sql 优化

sql优化最主要的就是要避免那些不走索引的sql。以提高执行效率。常见的规则有

- SELECT语句务必指明字段名称，不要使用select * 
- 的
- 使用union all 会union 代替 or 。
- 使用not exists 代替 not in，使用exists 代替 in
- 避免在where子句中对字段进行null值判断,(`is null`, `is not null` )
- 不建议使用%前缀模糊查询
- 避免在where子句中对字段进行表达式操作
- 避免隐式类型转换
- 对于联合索引来说，要遵守最左前缀法则
> 举列来说索引含有字段id、name、school，可以直接用id字段，也可以id、name这样的顺序，但是name;school都无法使用这个索引。所以在创建联合索引的时候一定要注意索引字段顺序，常用的查询字段放在最前面。
- 表关联查询时，利用小表去驱动大表
- 

## 数据库查询很慢，什么原因，怎么优化？
查询慢的原因：
- 数据量特别大
    - 分表分库。减少单表的数据量。
- 数据量不大
  - 是否走索引，没有索引则创建索引。
  - 建了索引，但没走索引，对应sql优化
  - 走了索引，但还是很慢，重建索引。
- 数据库层面优化 数据库设置。
## 行转列，列转行
- 行专列 decode group by 
- 列转行  union all 






## mysql数据库存储引擎

mysql数据库是分层模块化开发的。其中存储引擎主要用来设计数据的存储方式和索引方式等。不能的存储引擎有不同的特性，适用于不同的场景。

常见的四种存储引擎
* InnoDB存储引擎：
支持自动增长列，支持外键约束，支持事务，相对访问速度略差，mysql默认使用该引擎。

* MyISAM存储引擎 ：
不支持事务、也不支持外键，优势是访问速度快，对事务完整性没有 要求或者以select，insert为主的应用基本上可以用这个引擎来创建表

* MEMORY存储引擎
memory类型的表访问非常的快，因为它的数据是放在内存中的，并且默认使用HASH索引，但是一旦服务关闭，表中的数据就会丢失掉。 

* Archive存储引擎
仅支持insert和select
支持很好的压缩功能
不支持事务，不能很好的支持索引
适用于：存储日志信息，或其它按时间序列实现的数据采集类的应用，如监控日志。

查看当前存储引擎
```
SHOW ENGINES;显示所有支持的存储引擎
 SHOW VARIABLES LIKE 'storage_engine'; 显示当前引擎
```


# javaee

## jdbc
jdbc：java数据库连接。
是一个由一套java 类和接口组成，用于执行sql，为多种数据库提供统一的访问的javaee 组件。

> jdbc使用步骤

- 导入jar包
- 加载驱动类   Class.forName(“类名”)
- 获取连接    DriverManager.getConnection()   得到 Connection 
- 得到执行sql 的对象  Statement
- 执行sql语句，返回结果    ResultSet
-  处理结果 
- 关闭资源 .close()

> jdbc关键类
* connection  与数据库的连接（会话）
* DriverManager  驱动管理类
* Statement   用于执行SQL 语句并返回它所生成结果的对象
* ResultSet 表示数据库结果集的数据表，通常通过执行查询数据库的语句生成

> sql注入
所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

> 批量插入
批量插入
```
/**
     * 批量插入选项
     */
    public void insertOptionBatch(HQuestion hquestion){
        Connection conn=null;
        PreparedStatement pstm=null;

        try {
            conn=DBUtil.getConnection();
            //设置手动提交事务
            conn.setAutoCommit(false);
            String sql="insert into H_OPTION (ID,QID,ITEM,CONTENT) VALUES" +
                    " (SEQ_H_OPTION.nextval,?,?,?) ";
            pstm=conn.prepareStatement(sql);

            for (HOption hOption : hquestion.getOptions()) {
                pstm.setInt(1,hquestion.getId());
                pstm.setString(2,hOption.getItem());
                pstm.setString(3,hOption.getContent());
                //将参数添加到批量操作
                pstm.addBatch();
            }
            pstm.executeBatch();

        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            DBUtil.close(conn,pstm,null);
        }
    }
```
## servlet
### 什么是servlet?
servlet是java服务端处理响应客户端网络请求的小程序。
### servlet 生命周期
- 实例化：由servlet容器加载并实例化。实例化的时机有二，
    - 配置了load-on-startup 时，如果值为正数，则tomcat启动时加载，数值越小，优先级越高。为负数是不加载。
    -  没有配置load-on-startup时，则第一次访问的时候加载，并实例化。 
- 初始化：调用init方法，实例化时调用一次。
- 处理请求：service 方法
- 销毁：destory方法，容器关闭的时候调用一次。


### get和post区别
- GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，POST方法是把提交的数据放在HTTP包的Body中.
- GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.
- GET方式提交数据，相对不安全，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码.
- get通常用来获取服务器资源，而post通常用来提交数据到服务器。

### session和cookie区别
- session 存储在服务器端，而cookie存储在客户端(浏览器)
- session 比cookie更安全，但是也会增加服务器压力。
- 单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。而session不限制

> 使用建议：重要的数据存session，更安全。不重要的存cookie,服务器压力小。
## jsp

### jsp执行流程
- 浏览器发送一个 HTTP 请求给服务器。
- Web 服务器识别出这是一个对 JSP 网页的请求，并且将该请求传递给 JSP 引擎。通过使用 URL或者 .jsp 文件来完成。
- JSP 引擎从磁盘中载入 JSP 文件，然后将它们转化为 Servlet。这种转化只是简单地将所有模板文本改用 println() 语句，并且将所有的 JSP 元素转化成 Java 代码。生成一个java文件
- JSP 引擎将 Servlet 编译成可执行类，并且将原始请求传递给 Servlet 引擎。
- Web 服务器的某组件将会调用 Servlet 引擎，然后载入并执行 Servlet 类。在执行过程中，Servlet 产生 HTML 格式的输出并将其内嵌于 HTTP response 中上交给 Web 服务器。
- Web 服务器以静态 HTML 网页的形式将 HTTP response 返回到您的浏览器中。
- 最终，Web 浏览器处理 HTTP response 中动态产生的HTML网页，就好像在处理静态网页一样。
![image](https://note.youdao.com/yws/public/resource/398a98a857c1debccb9ae24a91ca79eb/xmlnote/DA87FA5994424D8A8D00DF161FCC6BD1/8177)
### jsp生命周期
- jsp编译
    - 解析JSP文件。
    - 将JSP文件转为servlet。
    - 编译servlet。
- jsp初始化，执行 `jspInit()`方法
- jsp执行， 执行`_jspService()`方法
- jsp销毁，执行`jspDestroy()` 方法
### jsp指令
- <%@ page %>
- <%@ include %>
- <%@ taglib %>
### jsp内置对象
jsp 中默认存在9个对象，不需要定义即可直接使用，我们称为内置对象。
对象名 | 含义
--|--
request  |  httpServletRequest对象，用来封装请求的数据
response  |  httpServletResponse对象，用来封装响应数据
out |  JspWriter类的实例，用来向页面输出内容
session | HttpSession类的实例,用来表示当前会话
application|当前应用（一个项目产生一个application，tomcat关闭才会消失）
config|jsp页面配置信息
pageContext|当前页面上下文内容（页面的所有对象）
page|当前页面，类似于this
exception|Exception类的对象，代表发生错误的JSP页面中对应的异常对象
### 四个域对象
- page：jsp页面被执行，生命周期开始，jsp页面执行完毕，声明周期结束。
- request:用户发送一个请求，开始，服务器返回响应，请求结束，生命周期结束。
- session:用户打开浏览器访问，创建session(开始),session超时或被声明失效，该对象生命周期结束。
- application：web应用加载的时候创建。Web应用被移除或服务器关闭，对象销毁。[结束]。

## tomcat
### tomcat性能调优
https://blog.csdn.net/zs742946530/article/details/82346707

### 如何在一台服务器上部署多个tomcat
如果在一台服务器上同时启动多个tomcat，在没做任何改动的情况下，会因端口冲突而启动失败。所以需要修改tomcat的server.xml配置文件中的端口号，使其不冲突即可。

# 框架

## spring
### 谈谈对spring的理解
Spring是轻量级控制反转(IOC)和AOP面向切面的容器框架。他集合了javaee全功能的应用解决方案。简化了我们的开发。

其功能主要归结为五大块的内容。分别为
- Core Container、
- AOP
- WEB
- Data Access
- Test  

Spring各个模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式。  

现在主要介绍下spring的两个核心思想IOC和AOP
- IOC是控制反转，即将对象的控制权反过来交给spring。由spring负责对象的生命周期的管理。很好的实现了对象与对象之间的依赖解耦。IOC的实现主要是通过DI（依赖注入），反射和工厂。
- AOP是面向切面编程，可以将一些通用业务逻辑或新增业务代码作为切面，在不改变原有代码的情况下，动态的切入到切入点上。完成了组件之间的解耦合，提高了系统的扩展性和可维护性。  
AOP的实现原理主要是通过动态代理。spring AOP支持两种动态代理实现方式，JDK的动态代理和cglib的动态代理。

### Spring中应用的设计模式
- 工厂设计模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
- 代理设计模式 : Spring AOP 功能的实现。
- 单例设计模式 : Spring 中的 Bean 默认都是单例的。
- 模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- 包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
- 适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。
......

### spring 注入方式
- set方法注入
- 构造器注入
- 注解注入

### spring常用注解

> 扫描bean组件的注解 

- @Service 用于Service层
- @Controller 用于Action层
- @Repository 用于Dao层
- @Component 用于其他组件
> 依赖注入的注解：

- Resource 先按名称，后按类型来匹配
- AutoWired  按类型注入，可以用于构造器、字段、方法注入。

>**区别：**  
@Resource 
使用@Resource可以省略name属性。
修饰方法时，省略name属性，则该name值是该setter方法去掉前面的set字符串，首字母小写后得到的子串。
修饰Field时，省略name属性，则该name与该Field同名。
@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按byName自动注入罢了。@Resource有两个属性是比较重要的，分别是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。 


> 其他注解：
- @Scope 等价于<bean scope="">
- @PostConstruct   等价于<bean init-method="">
- @PreDestroy     等价于<bean destroy-method="">




### spring 生命周期
![image](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)
- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 set()方法设置一些属性值。
- 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入Bean的名字。
- 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
- 与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessBeforeInitialization() 方法
- 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessAfterInitialization() 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。


### spring bean 的作用域

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。

### Spring 管理事务的方式
- 编程式事务，在代码中硬编码。(不推荐使用)
- 声明式事务，在配置文件中配置（推荐使用）
  - 基于XML的声明式事务
  - 基于注解的声明式事务(@Transactional)   

### spring 中指定事务隔离级别和传播特性
> 隔离级别
spring中定义了五个表示隔离级别的常量：
- TransactionDefinition.ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别。
    - Mysql 默认采用的 REPEATABLE_READ隔离级别 
    - Oracle 默认采用的 READ_COMMITTED隔离级别.
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
- TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
- TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- TransactionDefinition.ISOLATION_SERIALIZABLE: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。
## sprnigmvc 
### springmvc 执行流程
![image](https://note.youdao.com/yws/public/resource/398a98a857c1debccb9ae24a91ca79eb/xmlnote/5518A47236494F56A37B264D7736D731/8137)
流程说明（重要）：

- 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
- DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
- 解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
- HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。
- 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
- ViewResolver 会根据逻辑 View 查找实际的 View。
- DispaterServlet 把返回的 Model 传给 View（视图渲染）。
- 把 View 返回给请求者（浏览器）

### springmvc 常用注解
- @Controller
- @RestController
- @RequestMapping
- @GetMapping/@PostMapping/@PutMapping/@DeleteMapping
- @ResponseBody
- @RequestBody 常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。
- @RequestParam 获取请求数据，类似于request.getParamter()
- @PathVariable 在url地址中获取值
- @ExceptionHandler
注解到方法上，出现异常时会执行该方法
- @ControllerAdvice
使一个Contoller成为全局的异常处理类，类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常

### springmvc 拦截器实现
- 实现HandlerInterceptor接口，并重写其方法
    - preHandle：在控制层Controller方法执行之前调用。返回true（放行）或false（拦截）。
    - postHandle：在控制层方法调用之后，视图解析器解析之前调用。可以通过该方法对请求域中的数据及返回的页面做修改。
    - afterCompletion：控制层方法返回之后,并且完成视图解析后执行，通常做一些清理工作。
```
/**
 * 描述:
 *  springmvc 拦截器 implements HandlerInterceptor
 * @author bigpeng
 * @create 2019-06-26 11:12
 */
public class LoginInterceptor implements HandlerInterceptor {

    /**
     *  在控制层Controller方法执行之前调用
     *  其返回一个boolean类型的值，标识是否可以继续访问。
     *  如果返回为true,则进行后续操作
     *  如果返回为false，则不会请求到控制层
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取session中的用户信息
        User user = (User) request.getSession()
                                  .getAttribute("user");
        //判断是或否有用户信息，不存在则跳转到登录
        if(user==null){
            response.sendRedirect("/user/toLogin");
            return false;
        }

        return true;
    }

    /**
     * 在控制层方法调用之后，视图解析器解析之前调用
     * 可以通过该方法对请求域中的数据及返回的页面做修改
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器的postHandle方法被调用");
    }

    /**
     * 在控制层方法返回之后,并且完成视图解析后执行，通常做一些清理工作
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        System.out.println("请求完成，调用afterCompletion");
    }
}
```
- 在springmvc的配置文件中配置拦截器    
```
<!--开启默认servlet,用以处理静态资源文件-->
    <mvc:default-servlet-handler/>
    <!--配置springmvc拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
            <!--mapper path 指定要拦截的请求-->
            <mvc:mapping path="/**"/>
            <!--设置放行的请求地址-->
            <mvc:exclude-mapping path="/user/toLogin"/>
            <mvc:exclude-mapping path="/user/login"/>
            <mvc:exclude-mapping path="/user/toRegister"/>
            <mvc:exclude-mapping path="/user/register"/>
            <mvc:exclude-mapping path="/static/**"/>
            <!--配置拦截器类-->
            <bean id="loginInterceptor"
                    class="com.seecen.springmvc.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```
### springmvc 异常处理
 springmvc 异常处理常见的有三种方式：
- 使用Spring MVC提供的简单异常处理器SimpleMappingExceptionResolver；
```
<!--配置springmvc异常处理机制-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <!--异常映射规则-->
        <property name="exceptionMappings">
            <props >
                <!--key指定异常类-->
                <prop key="org.apache.shiro.authz.UnauthorizedException">
                    nopermission.jsp
                </prop>
            </props>
        </property>
    </bean>
```

- 实现Spring的异常处理接口HandlerExceptionResolver 自定义自己的异常处理器；
```
@Component
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    /**
     *  全局异常处理
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o handler类对象
     * @param e 异常
     * @return
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {

        ModelAndView  mv=new ModelAndView();
        mv.addObject("ex",e);
        if(e instanceof NullPointerException){
            System.out.println("空指针");
            mv.setViewName("xxxx");
        }else {
            mv.setViewName("error");
        }
        return mv;
    }
}

```

- 使用@ControllerAdvice+@ExceptionHandler注解实现异常处理；
```
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler()
    @ResponseBody
    String handleException(Exception e){
        return "Exception Deal! " + e.getMessage();
    }
}
```

## mybatis

### 什么是ORM
ORM 为对象关系映射。是一种用以解决java对象与数据库记录对应关系的规范或者说是思想。  
其主要是通过一个映射配置文件，来设定java对象中的属性与数据库表结构中的字段的对应关系。  
达到可以将数据库中的一条记录直接查询封装成一个对象的效果。
其中主要实现原理有：
- 数据库查询（JDBC），
- 映射文件解析（xml文件解析）
- java对象的创建和赋值（java反射）。
### mybatis xml中resultType和resultMap区别
- resultType 指定返回某java类型。当查询的返回字段与java类中属性完全一致时才能封装进去。不一样的情况则需要对列取别名才能封装。
- resultMap 则指定返回的映射结果。当数据库字段与java类不一致时，可以在resultMap中配置对应的映射关系。且resultMap可以配置关联查询（collection和association）


### mybatis 中 ${} 与#{} 取值区别
#{}取值相当于预编译，可以防止sql注入。
而${}则相当于直接字符拼接。
> 使用建议：  
如果传入的是表名，字段名等不需要预编译的内容，则使用${}.否则都使用#{}。
### mybatis 动态sql实现方式
所谓动态sql即可以根据参数的不同情况生成不同的sql语句。mybatis中可以通过where,set if foreach 等标签来实现动态sql。
### mybatis 如何批量插入数据
- oracle 
```
<insert id="insertBatch" parameterType="java.util.List">
    BEGIN
    <foreach collection="list" item="user" index="index" separator="">
      insert into T_USER (ID, NAME, PASSWORD)
      values (SEQ_T_USER.nextval, #{user.name,jdbcType=VARCHAR}, #{user.password,jdbcType=VARCHAR});
    </foreach>
    COMMIT;
    END;
  </insert>
```

- mysql
```
<insert id="batchInsertProductCategory" parameterType="java.util.List">
	 insert into T_USER ( NAME, PASSWORD)
    VALUES
    <foreach collection="list" item="productCategory" index="index"
             separator=",">
        (
       #{user.name,jdbcType=VARCHAR}, #{user.password,jdbcType=VARCHAR}
        )
    </foreach>
</insert>
```
### mybatis 缓存
mybatis 缓存分为一级缓存和二级缓存
- 一级缓存，为session级别的缓存，
- 二级缓存
二级缓存是mapper级别的缓存，关闭了session同样有效
    - 设置开启二级缓存
```
<settings>
		<!-- 开启二级缓存总开关 -->
		<setting name="cacheEnabled" value="true"/>
		<!-- 开启延迟加载功能 -->
		<setting name="lazyLoadingEnabled" value="true"/>
		<!-- 默认ture，代表使用本对象也会加载关联对象 -->
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
```
   - 在需要缓存的mapper中配置缓存
```
<!--flushInterval 为刷新时间，size为最大存储个数， readyOnly 返回对象为只读-->
<cache 
	eviction="LRU"
	flushInterval="3000"
	size="1024"
	readOnly="true"
	/> 
<!--若想禁用当前select语句的二级缓存，添加useCache="false"修改如下-->
<select  useCache="false" id="getCountByName" parameterType="java.util.Map" resultType="INTEGER" statementType="CALLABLE">
```
**配置二级缓存后，对应的实体类需要实现序列化接口**

### mybatis 注解
```
/配置缓存注解
@CacheNamespace(eviction=LruCache.class,size = 512,readWrite = true,flushInterval = 60000)
public interface HStudentMapper {

    @Delete("delete from h_student where id=#{id}")
    int deleteByPrimaryKey(Long id);

    @SelectKey(statement="select seq_h_student.nextval from dual"
            , keyProperty="id", before=true, resultType=long.class)
    @Insert("insert into h_student (id,name,classid) values " +
            "(#{id},#{name},#{classid})")
    int insert(HStudent record);

//    int insertSelective(HStudent record);
    @Results(id = "studentMap"
            , value = {
            @Result(property = "id", column = "id", id = true),
            @Result(property = "name", column = "name"),
            @Result(property = "classid", column = "classid"),
            @Result(property = "hClass",column = "classid",
                    one = @One(select ="com.seecen.mybatis.mapper.HClassMapper.selectByPrimaryKey"))
      }
    )
    @Select("select * from h_student where  id=#{id}")
    HStudent selectByPrimaryKey(Long id);

//    int updateByPrimaryKeySelective(HStudent record);

    @Update("update h_student set name=#{name},classid=#{classid} where id=#{id}")
    int updateByPrimaryKey(HStudent record);
}
```

### MyBatis四大对象？

![image](https://img2018.cnblogs.com/blog/957248/201910/957248-20191006165055084-315138871.png)
### mybatis如何自定义拦截器？
mybatis插件（准确的说应该是around拦截器，因为接口名是interceptor，而且invocation.proceed要自己调用，配置中叫插件）功能非常强大，可以让我们无侵入式的对SQL的执行进行干涉，从SQL语句重写、参数注入、结果集返回等每个主要环节，典型的包括权限控制检查与注入、只读库映射、K/V翻译、动态改写SQL。

MyBatis默认支持对4大对象（Executor，StatementHandler，ParameterHandler，ResultSetHandler）上的方法执行拦截，具体支持的方法为：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)，主要用于sql重写。
- ParameterHandler (getParameterObject, setParameters)，用于参数处理。
- ResultSetHandler (handleResultSets, handleOutputParameters)，用于结果集二次处理。
- StatementHandler (prepare, parameterize, batch, update, query)，用于jdbc层的控制。
大多数情况下仅在Executor做插件比如SQL重写、结果集脱敏，ResultSetHandler和StatementHandler仅在高级场景中使用，而且某些场景中非常有价值

```
@Intercepts({
        // @Signature(type = Executor.class, method = /* org.apache.ibatis.executor.Executor中定义的方法,参数也要对应 */"update", args = { MappedStatement.class, Object.class}),
        @Signature(type = Executor.class, method = "query", args = { MappedStatement.class, Object.class,RowBounds.class, ResultHandler.class }) })
public class SelectPruningColumnPlugin implements Interceptor {
    public static final ThreadLocal<ColumnPruning> enablePruning = new ThreadLocal<ColumnPruning>(){
        @Override
        protected ColumnPruning initialValue()
        {
            return null;
        }
    };
    
    Logger logger = LoggerFactory.getLogger(SelectPruningColumnPlugin.class);
    
    static int MAPPED_STATEMENT_INDEX = 0;// 这是对应上面的args的序号
    static int PARAMETER_INDEX = 1;
    static int ROWBOUNDS_INDEX = 2;
    static int RESULT_HANDLER_INDEX = 3;
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        if (enablePruning.get() != null && enablePruning.get().isEnablePruning()) {
            Object[] queryArgs = invocation.getArgs();
            MappedStatement mappedStatement = (MappedStatement) queryArgs[MAPPED_STATEMENT_INDEX];
            Object parameter = queryArgs[PARAMETER_INDEX];
            BoundSql boundSql = mappedStatement.getBoundSql(parameter);
            String sql = boundSql.getSql();// 获取到SQL ，进行调整
            String name = mappedStatement.getId();
            logger.debug("拦截的方法名是:" + name + ",sql是" + sql + ",参数是" + JsonUtils.toJson(parameter));
            String execSql = pruningColumn(enablePruning.get().getReserveColumns(), sql);
            logger.debug("修改后的sql是:" + execSql);

            // 重新new一个查询语句对像
            BoundSql newBoundSql = new BoundSql(mappedStatement.getConfiguration(), execSql, boundSql.getParameterMappings(), boundSql.getParameterObject());
            // 把新的查询放到statement里
            MappedStatement newMs = copyFromMappedStatement(mappedStatement, new BoundSqlSqlSource(newBoundSql));
            for (ParameterMapping mapping : boundSql.getParameterMappings()) {
                String prop = mapping.getProperty();
                if (boundSql.hasAdditionalParameter(prop)) {
                    newBoundSql.setAdditionalParameter(prop, boundSql.getAdditionalParameter(prop));
                }
            }
            queryArgs[MAPPED_STATEMENT_INDEX] = newMs;
            // 因为涉及分页查询PageHelper插件，所以不能设置为null，需要业务上下文执行完成后设置为null
//            enablePruning.set(null);
        }
        Object result = invocation.proceed();
        return result;
    }
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    @Override
    public void setProperties(Properties properties) {
    }
```

详细参考： https://www.cnblogs.com/zhjh256/p/11516878.html

## shiro 
### Shiro 架构 核心组件
![image](https://upload-images.jianshu.io/upload_images/1212547-ed07536806fe9943.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Authenticator:管理登陆登出
- Autorizer:授权器赋予主体有那些权限
- session Manager：shiro自己实现session管理器
- session DAO：提供了session的增删改插
- Cache Manager：缓冲管理器
- Realms：和数据库交互的桥梁

### shiro 注解

- 1、 @RequiresAuthentication : 表示当前Subject已经通过login进行了身份验证；即 Subject.isAuthenticated() 返回 true
- 2、@RequiresUser : 表示当前Subject 已经身份验证或者通过记住我登录的
- 3、@RequiresGuest : 表示当前Subject没有身份验证或通过记住我登陆过，即是游客身份
- 4、@RequiresRoles(value = { “admin”, “user” }, logical = Logical.AND) : 表示当前 Subject 需要角色 admin和user
- 5、@RequiresPermissions(value = { “user:a”, “user:b” }, logical = Logical.OR) : 表示当前 Subject 需要权限 user:a 或 user:b

### shiro运行原理
![image](https://upload-images.jianshu.io/upload_images/1212547-1c10b5ea3b2aa573.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 1、Application Code:应用程序代码，就是我们自己的编码，如果在程序中需要进 行权限控制，需要调用
Subject 的 API。 
- 2、Subject:主体，代表的了当前用户。所有的 Subject 都绑定到 SecurityManager， 与 Subject 的所有
交互都会委托给 SecurityManager,可以将 Subject 当成一个 门面，而真正执行者是 SecurityManager 。 
- 3、SecurityManage:安全管理器，所有与安全有关的操作都会与 SecurityManager 交互，并且它管理所有
的 Subject 。 
- 4、Realm:域 shiro 是从 Realm 来获取安全数据（用户，角色，权限）。就是说 SecurityManager 
要验证用户身份， 那么它需要从 Realm 获取相应的用户进行比较以确定用户 身份是否合法；也需要从
Realm 得到用户相应的角色/权限进行验证用户是否 能进行操作； 可以把 Realm 看成 DataSource，即安全数
据源 。


### shiro 过滤器
![image](https://upload-images.jianshu.io/upload_images/1212547-3c28ec9165be43d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### shiro 标签
Shiro 提供了 JSTL 标签用于在 JSP 页面进行权限控制，如根据登录用户显示相应的页面按钮。
guest 标签：用户没有身份验证时显示相应信息，即游客访问信息。

user 标签：用户已经经过认证/记住我登录后显示相应的信息。

authenticated 标签：用户已经身份验证通过，即Subject.login登录成功，不是记住我登录的。

notAuthenticated 标签：用户未进行身份验证，即没有调用Subject.login进行登录，包括记住我自动登录的也属于未进行身份验证。

pincipal 标签：显示用户身份信息，默认调用Subject.getPrincipal() 获取，即 Primary Principal。

hasRole 标签：如果当前 Subject 有角色将显示 body 体内容：

hasAnyRoles 标签：如果当前Subject有任意一个角色（或的关系）将显示body体内容。

lacksRole：如果当前 Subject 没有角色将显示 body 体内容。

hasPermission：如果当前 Subject 有权限将显示 body 体内容。

lacksPermission：如果当前Subject没有权限将显示body体内容。
## springboot
### springboot理解
springboot是用来简化Spring应用的初始搭建以及开发过程的框架。其核心思想是约定优于配置，将大量框架的jar包依赖和与spring的整合进行了约定配置。当导入对应的依赖便可以自动装配，简化了原来spring项目的大量xml配置。做到开箱即用。
具体优点如下：
- 嵌入的Tomcat，无需部署WAR文件
- 简化Maven配置
- 自动配置Spring

### springboot 事务处理
springboot中可以通过@transactional注解进行事务配置。
### springboot 常用注解
- @SpringBootApplication：包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。
    - @ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。
    - @Configuration 等同于spring的XML配置文件；使用Java代码可以检查类型安全。
    - @EnableAutoConfiguration 自动配置。
    
    
## springboot文件上传 
- 前端页面编写form表单，并指定method为post，enctype为mutipart/formdata,要不然后台无法获取到文件数据。
- 后台通过MultipartFile对象接收上传的文件对象
- MultipartFile类关键方法 
    - getOriginalFilename 获取文件名
    - transferTo(file) 将上传文件内容写入对应文件。
    - file.getInputStream() 获取文件输入流
    - getBytes() 获取文件内容字节序列
- 配置请求路径与文件存储的磁盘路径的映射关系
```
 //添加资源映射
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // /uploadFile/**
        registry.addResourceHandler(vpath+"**")// 指定请求URL
                //"file:E:/upload/"
                .addResourceLocations("file:"+uploadpath);//指定映射到的资源
    }
```


## 什么是restful,怎么使用？
restful是一种软件设计风格。其核心的思想是一个url对应一个网络资源，而对应的资源的不同操作，通过http的不同的请求方式来体现。比如说一条用户数据的操作。原来url设计风格可能是这样子的
- 查询 getUser?id=xxxx 
- 删除 deleteUser?id=xxx
- 修改  updateUser?id=xxxx

==> user/1
- 查询  get user/1
- 删除  delete user/1
- 修改  update   user/1

并充分利用http状态码进行数据响应，通常响应的数据格式为json。

- 关于怎么使用的问题。
springmvc 中提供了restful风格接口的实现方案。
@RestController

@GetMapping @PostMapping...来指定方法的请求方式。
@pathVariale 来获取URL中的数据内容

通过返回@ResponseEntity对象，可以指定对应的状态码等数据。

同时通常会配合swagger编写接口文档，生成在线的接口文档信息，并提供接口测试功能。（postman） 

- restful 接口调用   restTemplate  httpClient 。。。。

## redis 
### redis 数据类型
**redis主要有string,list,set,sorted set hash 五种数据类型**

##### 1）string 数据类型
string类型是二进制类型，可以将字符串、图片、视屏等等保存起来，也可以将一些静态文件保存起来，如js、css等等。

**string类型value的最大值为512MB**

具体使用语法：
* set key value 设置值
* get key 根据key获取value
* incr key 对key的值自增 （**值需为整形**）
* decr key 对key自减（**值需为整形**）

**应用场景**
* 计数器:访问量，点赞数
* 缓存
* 分布式锁
* 唯一数：自增自减具有原子性
* 。。。

##### 2）list 数据类型

list类型为列表类型。是双向链表结构，类似于java中的LinkedList的一种数据结构。是一个有序的可重复的集合。

**操作命令**  
* lpush keyList value1 value2 value3 .... 在集合的左边依次插入value
例如：lpush ids 1 2 3 4 5  得到的结果内容为 5 4 3 2 1  
* rpush keyList value1 value2 value3 ... 在集合的右边依次插入value 
例如：rpush ids 1 2 3 4 5 得到的结果为 1 2 3 4 5 
* lpop keyList ：取出并移除keyList的第一个元素  
* rpop keyList ：取出并移除keyList的最后一个元素  

**应用场景**

* 取最新n条数据的操作，比如新浪微博的最新的微博。  
在Redis中我们的最新微博ID使用了常驻缓存，这是一直更新的。但是我们做了限制不能超过5000个ID，因此我们的获取ID函数会一直询问Redis。只有在start/count参数超出了这个范围的时候，才需要去访问数据库
* 实现简单的消息队列，具体可以先用rpush把消息放入到队列尾部，再用lpop把消息从队列头部取出
* 栈的实现：lpush+lpop
* 队列的实现：lpush+rpop
* 消息队列：lpush+brpop

 
##### 3）set 数据结构

set类型为集合类型。是一种无序类型，在redis内部是通过 HashTable实现，查找和删除元素十分快速，可以用于记录一些不能重复的数据

**应用场景**

* 投票系统 ，比如一天一个用户只能投一次票，可以用日期为key，保证唯一性，效率很高。
* 微博等的用户关注及粉丝等  
在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

**常用命令**   
* sadd key value1 value2 ... : 向set添加元素 
* srem key value ：从set中移除元素
* smembers key : 取出所有set元素
* SISMEMBER key value: 查看value是否存在set中 返回1表示存在，返回0不存在
**set的交集，并集，差集等操作：**
* SUNION key1 key2 ... keyN：**并集** ，将所有key合并后取出来，相同的值只取一次  
* SDIFF key1 key2 ... keyN：**差集**，获取第一set中不存在于后面几个set里的元素。。
* SINTER key1 key2 ... keyN:取出这些set的**交集**
* SMOVE srckey dstkey member：将元素member从srckey中转移到dstkey中，这个操作是原子的。


##### 4）sorted set 有序set 
sorted set 类似于set ,只不过sorted set 是一种有序集合。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
有序集合的成员是唯一的,但分数(score)却可以重复。

**应用场景**
* 排行榜 score 
* 

**常用命令**
* ZADD key score member：向有序set中添加元素member，其中score为分数，默认升序；  
* ZRANGE key start end [WITHSCORES]:获取按score从低到高索引范围内的元素，索引可以是负数，-1表示最后一个，-2表示倒数第二个，即从后往前。withscores可选，表示获取包括分数。
* ZREVRANGE key start end [WITHSCORES]：同上，但score从高到低排序。
* ZCOUNT key min max：获取score在min和max范围内的元素的个数
* ZINCRBY key increment member:根据元素，score原子增加increment.



##### 5）hash 数据结构   

hash类型是每个key对用一个HashTable,适合于存储对象，例如用户的信息对象，用户id作为key，具体信息作为value  

**使用场景**
* 用于存储部分变更的对象信息，如用户数据。
**常用命令** 
* HSET key field value:key是对象名，field是属性，value是值；
* HMSET key field value [field value ...]:同时设置多个属性
* HGET key field：获取该对象的该属性



### redis 应用场景
- 排行榜应用（微博）
- 显示最新的项目列表（比如评论）
- 计数 （ 微博阅读量，网站访问量等）
- 缓存
- 队列
- Session共享，默认Session是保存在服务器的文件中，即当前服务器，如果是集群服务，同一个用户过来可能落在不同机器上，这就会导致用户频繁登陆；采用Redis保存Session后，无论用户落在那台机器上都能够获取到对应的Session信息。
- 好友关系
### redis 持久化方式
redis提供两种方式进行持久化
- RDB持久化：原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化。性能较好，移植性较好。但是可能存在数据丢失的情况。
- AOF（append only file）持久化：原理是将Reids的操作日志以追加的方式写入文件），还原的时候则重新执行下日志中的操作即可。相比RDB数据相对更安全，但是数据文件会比较大，效率较RDB低。
### redis 分布式锁
在分布式场景下为了保证数据最终一致性。在单进程的系统中，存在多个线程可以同时改变某个变量（可变共享变量）时，就需要对变量或代码块做同步(lock—synchronized)，使其在修改这种变量时能够线性执行消除并发修改变量。但分布式系统是多部署、多进程的，开发语言提供的并发处理API在此场景下就无能为力了。
此时我们便需要使用到分布式锁。分布式锁常见的解决方案有
- zookeeper
- redis ：redisson 为redis的一个框架，其内部提供了redis的redLock的实现。我们只要引入使用即可。

参考： https://zhuanlan.zhihu.com/p/62274137 

### redis 事务 
一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。

事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis 最简单的事务实现方式是使用 MULTI 和 EXEC 命令将事务操作包围起来。  
**redis事务中有命令执行失败后，其他命令不会回滚。仍然会执行**
### redis 内存淘汰机制(MySQL里有2000w数据，Redis中只存20w的数据，如何保证Redis中的数据都是热点数据?)
redis 配置文件 redis.conf 中有相关注释，我这里就不贴了，大家可以自行查阅或者通过这个网址查看： http://download.redis.io/redis-stable/redis.conf

redis 提供 6种数据淘汰策略：
- volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）
- allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
- no-eviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！
- volatile-lfu：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
- allkeys-lfu：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key

### 缓存雪崩


### 缓存穿透
缓存穿透说简单点就是大量请求的key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B-redis.png)

> 解决方式

- **参数校验**，一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等。
- **缓存无效key**：如果缓存和数据库都查不到某个 key 的数据就写一个到 redis 中去并设置过期时间。  
示例代码如下：
```
public Object getObjectInclNullById(Integer id) {
    // 从缓存中获取数据
    Object cacheValue = cache.get(id);
    // 缓存为空
    if (cacheValue == null) {
        // 从数据库中获取
        Object storageValue = storage.get(key);
        // 缓存空对象
        cache.set(key, storageValue);
        // 如果存储数据为空，需要设置一个过期时间(300秒)
        if (storageValue == null) {
            // 必须设置过期时间，否则有被攻击的风险
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    }
    return cacheValue;
}
```
> **缺点**：会缓存很多无效的key。
- **布隆过滤器**：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，我会先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程。redis 本身以插件的形式提供了布隆过滤器的实现。我们只要引入即可。
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8-%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F-redis.png)

### springboot 中redis使用方式
- 使用redisTemplate 进行操作
- 使用缓存注解进行操作
    - @Cacheable 缓存添加，第一次会将方法结果存入缓存，后续将直接从缓存中获取，不执行方法内容，主要用于查询。
    - @CachePut 缓存更新，每次都会运行方法，并将方法返回结果放进缓存。
    - @CacheEvict 清除缓存，每次执行方法后会清除对应缓存。
## rabbitmq
### 什么是MQ？为什么要用？
MQ即消息队列，简单来说就是一个收发消息的容器，类似于邮局，我们给其他人写信，我们只要将对应的信件写好收信人地址投递到邮局，之后便可以去做其他的事情了，这便是MQ的异步，而邮局会根据你写的地址将你的信件投递到接收者手中，收信人收到信件，便完成了消息的消费。且信件的投递是有顺序的，先寄的先收到，这就是队列。  

消息队列是分布式系统中重要的组件，使用消息队列主要有两个目的
- 异步处理，提高系统性能
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/Asynchronous-message-queue.png)
如上图，在不使用消息队列服务器的时候，用户的请求数据直接写入数据库，在高并发的情况下数据库压力剧增，使得响应速度变慢。但是在使用消息队列之后，用户的请求数据发送给消息队列之后立即 返回，再由消息队列的消费者进程从消息队列中获取数据，异步写入数据库。由于消息队列服务器处理速度快于数据库（消息队列也比数据库有更好的伸缩性），因此响应速度得到大幅改善。
- 降低系统耦合性  

使用消息队列还可以降低系统耦合性。我们知道如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些。
![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97-%E8%A7%A3%E8%80%A6.png)


### AMQP消息模型
AMQP全称是：Advanced Message Queuing Protocol(高级消息队列协议)，是一个提供统一消息服务的应用层标准，其描述了一套模块化的组件以及这些组件之间进行连接的标准规则。
![image](https://note.youdao.com/yws/public/resource/abe5f2adfb9a2ce0d8d7ff8a887c3c80/xmlnote/2236FC1356AE40F8BF8B2FF72DF9A78F/6723)

### rabbitmq消息投递方式

- 1、基本消息模型(basic queues)

![图例](https://www.rabbitmq.com/img/tutorials/python-one-overall.png)

功能：一个生产者P发送消息到队列Q,一个消费者C接收。实现了基本的消息的生产和消费。一对一。

- 2、工作队列（work queues）

![image](https://www.rabbitmq.com/img/tutorials/python-two.png)

> **功能:**

一个生产者，多个消费者。写法与基本消息模型类似，只不过原来是一个消费者，现在是多个消费者。多个消费者处理队列中的数据。

> **特点**  

- 可以有多个消费者
- 一条消息只能被多个消费者中的一个消费。

- 3、发布/订阅模式 Publish/Subscribe
![image](https://www.rabbitmq.com/img/tutorials/python-three-overall.png)  

> **功能：**   
一个生产者发送的消息会被多个消费者获取。一个生产者、一个交换机、多个队列、多个消费者

> 与工作队列区别？
- 工作队列只有一个队列，而发布订阅有多个队列
- 工作队列一个消息只能被多个消费者中的一个消费，而发布订阅一个消息会被多个订阅的消费者消费。  
- 发布订阅比工作队列多出一个交换机概念，用来绑定消息发送到哪些消费者。
其实之前的两种模式也需要交换机，其使用默认交换，我们通过空字符串（“”）来识别。

- 4、路由模式（Routing）
![image](https://www.rabbitmq.com/img/tutorials/python-four.png)  

> 功能：生产者发送消息到交换机并且要指定路由key，消费者将队列绑定到交换机时需要指定路由key。只有当两个key相匹配时，消息才会发送到对应的消费者队列。即在广播的基础上有了路由的功能。 type 指定为direct。
- 5、主题订阅模式（topic）
![image](https://www.rabbitmq.com/img/tutorials/python-five.png)

> 说明：生产者P发送消息到交换机X，type=topic，交换机根据绑定队列的routing key的值进行通配符匹配；
符号#：匹配一个或者多个词 lazy.# 可以匹配 lazy.irs或者lazy.irs.cor
符号*：只能匹配一个词 lazy.* 可以匹配 lazy.irs或者lazy.cor  


### 如何确保MQ消息的可靠投递

> 什么是生产端的可靠性投递？ 
- 1.首先保障消息的成功发出
- 2.保障MQ节点的成功接收
- 3.发送端收到MQ节点(broker) 确认应答，也就是mq节点接收到消息给生产者一个应答
- 4.完善的消息进行补偿机制

> 业界主流的一些解决方案
- 1.消息落库，对消息状态进行打标 。在发送消息的时候把消息持久化到数据库中，然后设置一个状态，比如刚发送的状态叫发送中，比如收到了应答后我们改状态为已收到，比如用个数字码表示。但是对于没有响应的消息这时候我们用轮询抓取没有OK的消息，再重新做发送的操作，直到消息状态OK，但是我们需要对临界值设置下，比如轮询个几次。

![image](https://note.youdao.com/yws/public/resource/9bc6b757eee14e6531a57e766803151f/xmlnote/143B02DE14AB4C489480AFBD51A2138A/3790)
 

2.消息的延迟投递，做二次确认，回调检查
![image](https://note.youdao.com/yws/public/resource/9bc6b757eee14e6531a57e766803151f/xmlnote/68A3FD62E19F455FB25E0197CD9BD059/3798)
这里紫色的部分表示生产端，红色的表示消费端，绿色的是业务消息数据库
第一次消息的发送，先把消息入库（存在DB中），第一次一定是先把消息入库，才能把消息发出去。
紧接着第二条消息发送，不过这里是一条延迟消息检查，这里我们会设置一个延迟投递，可能需要几分钟后MQ中才会收到。为什么要这样，假设我们第一条和第二条延迟投递的都发送成功了，我们看第三步，我们消费端收到处理，处理完后其实我们还需要让消费端生成一条新的消息 ，然后我们通过第四步把消息投递到MQ。在这个过程中，灰色的部分用来监听消费者生成的消息，假设收到了消费者的确认，然后callback service对MGS DB进行持久化存储，可能接下来5分钟后我们的延迟消息发送到MQ了，然后我们的CallBack Service还需要监听延迟投递的消息，收到了然后检查MGS DB中是否有这条消息，如果有然后就不用做任何事情了。如果在第四步因为一些异常导致没有发送成功的确认，那么这个时候callback service需要做补偿，因为5分钟后监听延迟消息的时候去MSG DB中查询，发现没有，那需要主动发送一条RPC命令，再次重新通知生产者服务，生产者这时候再去BIZ DB中检查数据然后重新发送到MQ节点

## linux 
### linux 常用命令
- ls  查看目录下内容
- cd 切换目录
- mkdir 创建目录
- rm 删除文件和目录
- cp 复制2文件
- mv 剪切文件
- vi 编辑文件，新建文件
- grep 搜索内容
- find 搜索文档
- tar  打包，解压
- ps -ef |grep 搜索进程
- kill -9 杀死进行
- cat 查看文件
- tail 动态查看文件内容
### linux 权限操作
- chmod 修改权限
- chown 修改所有者
- chgrp 修改所属组

### linux 安装软件
- rpm rpm安装，需要下载rpm安装包
    - rpm -ivh 安装包  安装
    - rpm -e 卸载
- yum redhat提供的软件安装命令，类似于maven，从软件市场直接安装
- make  install  源码编译安装。下载软件源码，编译安装。
### linux 如何部署java web项目
 - 安装jdk
 - 安装tomcat
 - 安装数据库
 - 配置网络环境
 - 配置防火墙
 - 将项目war包放入tomcat webapps 目录启动tomcat
 - 访问测试
### linux 防火墙
- 启动： systemctl start firewalld
- 关闭： systemctl stop firewalld
- 查看状态：systemctl status firewalld 


### linux 常见操作整理 

#### 端口号被占用时，我们通过什么命令来查看？
- 通过netstat -pan | grep 端口号 查询端口号占用情况
![image](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlcy5jbmJsb2dzLmNvbS9jbmJsb2dzX2NvbS9idWdpbmdjb2RlLzExNDY5NTgvb19saW51eC1uZXRzdGF0MS5wbmc?x-oss-process=image/format,png)
- 通过第一步查询到的进程ID 进行进程信息查看ps -aux | grep pid 查看，进程程序名称
![image](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlcy5jbmJsb2dzLmNvbS9jbmJsb2dzX2NvbS9idWdpbmdjb2RlLzExNDY5NTgvb19saW51eC1uZXRzdGF0Mi5wbmc?x-oss-process=image/format,png)
- 直接通过 kill -9 进程ID 杀死对应进程。



## springcloud
    Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。
    springcloud中常见的子框架有
    - Eureka 服务发现框架
- Ribbon 进程内负载均衡器
- Open Feign 服务调用映射
- Hystrix 服务降级熔断器
- Zuul 微服务网关
- Config 微服务统一配置中心
- Bus 消息总线



### 定时任务

### activiti 工作流

### es搜索引擎


# jdk 源码解读

## String 

## HashMap

## ArrayList

## linkedList

......
