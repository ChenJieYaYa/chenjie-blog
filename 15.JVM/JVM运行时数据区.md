# JVM运行时数据区

## 一、JVM整体架构组成

**JVM由类装载器子系统、运行时数据区、执行引擎组成**，其整体架构图如下，本地方法接口属于OS

![请添加图片描述](assets/79da806c98364fe983fb20024264aca7.png)

**每个JVM只有一个Runtime实例**，即为运行时环境，相当内存结构的运行时数据区

JVM定义若干种程序运行期间会使用到的运行时数据区，其中一些会随着JVM启动而创建，随着JVM退出而销毁，另外一些则与线程对应的，随着线程的开始和结束而创建和销毁，如下图所示，灰色部分属于线程私有

![1657764061513](assets/1657764061513.png)

## 二、JVM内的线程

### 1.JVM内线程的创建与销毁

**线程是调度的基本单元，JVM允许一个应用内多线程并行执行**

HotSpot中**每个线程都与OS的本地线程直接映射**，当Java线程准备执行时OS本地线程同时创建，OS本地线程初始化成功后会调用Java程序中的`run()`，`run()`执行过程中遇到未捕获异常则Java程序将终止，即Java线程终止，当Java线程执行终止后OS本地线程也被回收，若此时回收的OS本地线程是JVM内的最后一个非守护线程则JVM也将终止

### 2.JConsole工具查看JVM线程

①编写死循环代码并编译运行

```Java
package com.yc.Test01_jconsole;

public class Test01 {
    public static void main(String[] args) {
        while (true) {
            Object obj = new Object();
        }
    }
}
```

②打开jconsole查看内部线程

![1657672309487](assets/1657672309487.png)

![1657765201739](assets/1657765201739.png)

![1657765214837](assets/1657765214837.png)

![1657765645448](assets/1657765645448.png)

### 3.JVM后台线程

使用jconsole工具发现JVM内包含许多后台线程，**其后台线程不包括调用`main()`的线程以及`main()`内部创建的线程**，主要的后台系统线程主要有如下几种

* **JVM线程**：JVM达到安全点才会出现
* **周期任务线程**：一般用于周期性操作的调度执行(比如中断)
* **GC线程**：为JVM中不同种类的垃圾收集行为提供支持
* **编译线程**：编译线程运行时将字节码编译成本地代码
* **信号调度线程**：该线程负责接收信号并发送给JVM

## 三、程序计数器PC

### 1.PC寄存器是什么？

PC寄存器是一块**很小的内存空间**(几乎可忽略不记)，也是**运行速度最快的存储区域**，用于**存储下一条指令的地址，即将要执行的代码由执行引擎读取下一条指令**，分支、循环、跳转、异常处理、线程恢复等基础功能都依赖PC

![1657767058131](assets/1657767058131.png)

JVM规范中，每个线程都有自己的PC寄存器，所以**PC寄存器是线程私有的**，其生命周期与线程的生命周期保持一致

**任何时刻一个线程只执行一个方法**，PC寄存器存储会存储当前线程正在执行的方法的JVM指令地址，若执行的方法是native方法，则是未指定值(undefined)

PC寄存器**没有GC(垃圾回收)**，也**没有OOM(OutOfMemoryError)**

> 虚拟机栈和本地方法栈没有GC有OOM，堆和方法区既有GC又有OOM

### 2.举例说明

①测试代码

```java
public class PCRegisterTest {
    public static void main(String[] args) {
        int i = 10;
        int j = 20;
        int k = i + j;

        String s = "nba";

        System.out.println(i);
        System.out.println(k);
    }
}
```

②将源代码编译生成字节码，后对其反编译

![1657673442469](assets/1657673442469.png)

![1657673512129](assets/1657673512129.png)

![1657768962517](assets/1657768962517.png)

### 3.常见面试题

#### 3.1.**使用PC寄存器存储字节码指令地址有什么用呢？**

CPU需要不断的切换各个线程，当切换回来时，**需要知道从何处开始继续执行**

JVM解释器需要通过改变PC寄存器的值来明确下一条指令

#### 3.2.为什么使用PC寄存器记录当前线程的执行地址呢？即PC寄存器为什么被设定为私有的？

**由于CPU时间片轮询限制，多线程在并发的特定的时间段内只执行其中某个线程的某条指令**，这样必然导致经常中断或恢复，为了**准确地记录各个线程正在执行的当前字节码指令地址**，最好的办法自然是**为每个线程都分配一个PC寄存器**，这样各个线程之间便可以进行独立计算，不会出现相互干扰的情况

> CPU时间片指CPU分配给各个程序的时间，每个线程被分配一个时间段，即时间片
>
> 宏观上同时打开多个应用程序，每个程序同时运行，微观上一个CPU一次只能处理程序要求的一部分，如何处理公平？一种方法就是引入时间片，每个程序轮流执行

## 四、虚拟机栈

### 1.虚拟机栈出现背景

由于跨平台性的设计，导致Java的指令都根据栈设计，优点是跨平台、指令集小、编译器容易实现，缺点是性能下降、实现同样的功能需要更多的指令

> 不同平台CPU架构不同，所以不能设计为基于寄存器的

### 2.虚拟机栈是什么？

在内存中，栈是运行单位，即**栈解决程序如何运行问题**，堆是存储单位，即**堆解决数据怎么存储问题**

虚拟机栈早期也叫Java栈，每个线程创建时都会创建一个虚拟机栈，所以**虚拟机栈是线程私有的**，其生命周期与线程的生命周期保持一致

PC寄存器**没有GC(垃圾回收)**，也**没有OOM(OutOfMemoryError)**

虚拟机栈**内部保存着一个个栈帧(Stack Frame)**，对应一次次Java方法调用，每个Java方法的调用都对应着压栈和入栈，虚拟机栈**保存方法的局部变量(8种基本数据类型、对象的引用地址)、部分结果，并参与方法的调用和返回**

虚拟机栈的速度仅次于PC寄存器

### 3.虚拟机栈中可能出现的异常

JVM规范允许虚拟机栈的大小是动态的或固定不变的

**StackOverflowError**：若采用固定不变的虚拟机栈，那么每个线程的虚拟机栈大小可在线程创建时独立创建，若线程请求分配的虚拟机栈容量大于虚拟机栈的最大容量则抛出此异常

**OutOfMemoryError**：若采用动态的虚拟机栈，在尝试扩展时无法申请到足够内存或创建新线程的同时没有足够内存创建虚拟机栈则抛出此异常

### 4.设置内存大小

**栈大小直接决定函数调用的最大可达深度**，可通过参数`-Xss`选项来设置线程的最大栈空间

IDEA中的设置方式：`Run → Edit Configurations → Application → 选中项目 → Configuration → VM options → -Xss2m`

![1657783374669](assets/1657783374669.png)

①测试代码

```Java
public class XSS {//一个简单的相互调用
    private static int count = 0;

    public static void recursion() {
        count++;
        recursion();
    }

    public static void main(String[] args) {
        try {
            recursion();
        } catch (Throwable e) {
            System.out.println(count);
            e.printStackTrace();
        }
    }
}
```

②直接运行结果如下

```
29509
java.lang.StackOverflowError
	at com.yc.Test03_Exception.XSS.recursion(XSS.java:16)
	at com.yc.Test03_Exception.XSS.recursion(XSS.java:16)
```

③设置内存大小为`2m`结果如下

```
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
Invalid thread stack size: -Xss:2m
```

### 5.栈的运行原理

每个线程都有自己的栈，栈中的数据都是以**栈帧(Stack Frame)**的格式存在，在该线程上执行的方法都对应一个栈帧，栈帧是内存区块，是数据集，维系着方法执行过程的各种数据信息

**JVM直接对虚拟机栈的操作只有两个，即出栈和入栈**，线程内同一时间只能有一个活动栈帧，即栈顶栈帧，被称为当前栈帧，当前栈帧对应的方法就是当前方法，定义该方法的类就是当前类，**不同线程的栈帧不允许相互引用**，**执行引擎运行的所有字节码指令只对当前栈帧进行操作**

> 若当前方法中还调用其他方法，则该被调用的方法对应新栈帧被创建，放在栈顶，成为新当前栈帧，该被调用方法返回结果之际会将结果给下一个栈帧，JVM丢弃该栈帧，下一个栈帧成为新当前栈帧
>
> 方法有两种返回方式，即正常return和抛出异常未捕获，不管哪一种方式返回当前栈帧都被弹出

![1657785122823](assets/1657785122823.png)

### 6.栈帧内部结构

#### 6.1.内部图解

![1657785351801](assets/1657785351801.png)

#### 6.2.局部变量表

局部变量表被定义为数字数组，用于**顺序存储方法参数和局部变量**，局部变量表属于线程私有，所以**不存在数据不安全问题**，方法调用结束后随着栈帧销毁，局部变量表也随之被销毁

局部变量表的大小在编译期间确定，保存于方法Code属性的`maximum local variables`数据项中，运行期间局部变量表的大小不会改变，以下出示案例，通过jclasslib工具查看分析

```java
import java.util.Date;

public class LocalVariablesTest {
    private int count = 0;

    public static void main(String[] args) {
        LocalVariablesTest test = new LocalVariablesTest();
        int num = 10;
        test.test1();
    }

    public void test1() {
        Date date = new Date();
        String name = "cba";
        System.out.println(date + name);
    }
}
```

![1657786823697](assets/1657786823697.png)

![1657786793812](assets/1657786793812.png)

![1657786925620](assets/1657786925620.png)

![1657787151211](assets/1657787151211.png)

![1657787263090](assets/1657787263090.png)

**局部变量表最基本的存储单元是Slot(变量槽)**，32位以内的数据占用一个Slot，64位占用两个Slot(long、double)，byte、short、char在存储前被转换为int，boolean也被转换为int，**JVM为局部变量表中的每个Slot都分配访问索引**，通过索引可成功访问到局部变量表中指定的局部变量值，当访问64位数据时，只需要使用前一个索引

若当前栈帧由构造方法`<init>`或实例方法(非静态)创建，该对象的this将被存于index为0的Slot，注意静态方法无this，其他参数按参数表顺序排列

局部变量表中Slot可重用，若一个局部变量过了其作用域，那么在其作用域之后产生的新局部变量就很有可能会复用该Slot，从而达到节省资源的目的

![1657788499968](assets/1657788499968.png)

> 栈帧中与**性能调优**关系最为密切的部分就是**局部变量表**，方法执行时JVM使用局部变量表完成方法的传递，**局部变量表中的变量也是重要的垃圾回收根节点**，只要被局部变量表中直接或间接引用的对象都不会被回收

#### 6.3.操作数栈

方法执行过程中根据字节码指令往操作数栈中写入或取出数据，**主要用于保存计算过程的中间结果，同时作为计算过程中变量的临时存储空间**，操作数栈使用**数组**实现，新栈帧被创建时操作数栈是空的，但是数组有长度，数组一旦创建其长度不可变

操作数栈所需的最大深度在编译器就定义好，保存于方法Code属性的`max_stack`数据项中

**操作数栈中元素可以是任意Java数据类型**，32位以内的数据占用一个栈单位深度，64位占用两个栈单位深度，操作数栈并非采用索引方式访问，**只可通过标准入栈和出栈完成一次数据访问**

若被调用的方法存在返回值，其返回值被压入当前栈帧的操作数栈，更新PC寄存器中下一条需要执行的字节码指令，接下来进行代码追踪👇

```Java
//以求和操作为例说明操作数栈的作用
public class OperandStackTest {
    public void testAddOperation() {
        byte i = 15;
        int j = 8;
        int k = i + j;
    }
}
```

将以上代码.class文件反编译后，字节码指令信息如下

```Java
 public void testAddOperation();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        15
         2: istore_1
         3: bipush        8
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: return
      LineNumberTable:
        line 16: 0
        line 17: 3
        line 18: 6
        line 19: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/atguigu/jvmstack/OperandStackTest;
            3       8     1     i   B
            6       5     2     j   I
           10       1     3     k   I
```









> [参考文章](https://blog.csdn.net/rrq_0324/article/details/109035773?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-3-109035773-blog-107146441.pc_relevant_multi_platform_whitelistv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-3-109035773-blog-107146441.pc_relevant_multi_platform_whitelistv2&utm_relevant_index=6)





## 1.jvm运行时数据区的划分?

本地方法栈

程序计数器

虚拟机栈，为了实现跨平台，Java指令集基于栈设计

```
栈是运行单位，堆是存储单位，每个线程都有自己的虚拟机栈（私有），内部保存这栈帧，生命周期和线程一致，对于站不存在垃圾回收GC，直接弹出，堆栈只有两个操作，入栈出栈，内存不够异常
```

堆区：新生代、老年代

元数据区：常量池、方法元信息、类元信息













1. 根据jvm规范，这些数据区中哪些会出现 内存溢出异常，分别是什么场景下出现?
2. 这些数据区哪些是线程独有的，哪些是线程共享区?
3. 每个区存储的数据的特点?
4. 程序计数器是什么，它是线程独有的吗? 它是否有内存溢出问题.
5. 虚拟机栈上保存哪些数据?怎么放?虚拟机栈是线程独有的吗，它是否有内存溢出问题?虚拟机栈的优点?
6. 虚拟机栈的大小是否可动?是否会有异常出现?
7. 如何设置虚拟机栈大小?
8. 什么叫本地方法? 是否可以写一个例子来实现本地方法，以输出一个hello world?
   10.什么叫本地方法栈?有什么作用?它是线程私有的吗? 它是否有可能抛出异常?
9. jvm规范一定强制要求实现本地方法栈吗?
10. 方法区是线程独有的吗?它是否有异常?它的作用?
11. 方法区的演进, jdk7及以前，它叫什么? jdk8开始，这又叫什么. 
12. 方法区或永久代的大小如何设置?



## 常见调优工具

![1657760578480](assets/1657760578480.png)

下载插件：https://blog.csdn.net/jushisi/article/details/109655175?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-109655175-blog-119964128.pc_relevant_multi_platform_whitelistv2&spm=1001.2101.3001.4242.1&utm_relevant_index=2

![1657761206483](assets/1657761206483.png)

![1657761180136](assets/1657761180136.png)

![1657761236631](assets/1657761236631.png)

![1657761400755](assets/1657761400755.png)

![1657761437834](assets/1657761437834.png)

运行项目，出现视图

![1657761863830](assets/1657761863830.png)

查看详细信息

![1657762108651](assets/1657762108651.png)

```
D:\develop\Java\jdk1.8.0_65\bin\java.exe -XX:+PrintGCDetails "-javaagent:D:\develop\IntelliJ IDEA 2020.1.2\lib\idea_rt.jar=14894:D:\develop\IntelliJ IDEA 2020.1.2\bin" -Dfile.encoding=UTF-8 -classpath D:\develop\Java\jdk1.8.0_65\jre\lib\charsets.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\deploy.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\access-bridge-64.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\cldrdata.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\dnsns.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\jaccess.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\jfxrt.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\localedata.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\nashorn.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunec.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunjce_provider.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunmscapi.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunpkcs11.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\zipfs.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\javaws.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jce.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jfr.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jfxswt.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jsse.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\management-agent.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\plugin.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\resources.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\rt.jar;E:\IdeaProjects\JVM\target\classes com.yc.Test03_visualVM.HeapInstance
[GC (Allocation Failure) [PSYoungGen: 32586K->5094K(37888K)] 32586K->29947K(123904K), 0.0070588 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 37829K->5115K(37888K)] 62681K->61379K(123904K), 0.0081835 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 5115K->0K(37888K)] [ParOldGen: 56263K->61203K(148480K)] 61379K->61203K(186368K), [Metaspace: 3780K->3780K(1056768K)], 0.0216438 secs] [Times: user=0.13 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 32768K->5117K(37888K)] 93971K->93777K(186368K), 0.0124481 secs] [Times: user=0.02 sys=0.11, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 37828K->5115K(52224K)] 126489K->126491K(200704K), 0.0068903 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 5115K->0K(52224K)] [ParOldGen: 121376K->126266K(244736K)] 126491K->126266K(296960K), [Metaspace: 3783K->3783K(1056768K)], 0.0178061 secs] [Times: user=0.13 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 46506K->5091K(52224K)] 172773K->172849K(296960K), 0.0243899 secs] [Times: user=0.09 sys=0.13, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 52116K->52091K(117760K)] 219874K->219850K(362496K), 0.0215176 secs] [Times: user=0.08 sys=0.16, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 114544K->63457K(125952K)] 282303K->282162K(370688K), 0.0387771 secs] [Times: user=0.05 sys=0.19, real=0.04 secs] 
[Full GC (Ergonomics) [PSYoungGen: 63457K->37328K(125952K)] [ParOldGen: 218704K->244617K(395264K)] 282162K->281946K(521216K), [Metaspace: 3788K->3788K(1056768K)], 0.0265873 secs] [Times: user=0.16 sys=0.08, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 99357K->99440K(181248K)] 343975K->344058K(576512K), 0.0707513 secs] [Times: user=0.05 sys=0.42, real=0.07 secs] 
[GC (Allocation Failure) [PSYoungGen: 180579K->117239K(198656K)] 425197K->425047K(593920K), 0.0831903 secs] [Times: user=0.11 sys=0.47, real=0.09 secs] 
[GC (Allocation Failure) [PSYoungGen: 198173K->136153K(232960K)] 505981K->505944K(628224K), 0.2245230 secs] [Times: user=0.17 sys=1.42, real=0.22 secs] 
[Full GC (Ergonomics) [PSYoungGen: 136153K->110689K(232960K)] [ParOldGen: 369791K->395004K(584704K)] 505944K->505693K(817664K), [Metaspace: 3788K->3788K(1056768K)], 0.3875346 secs] [Times: user=0.27 sys=0.78, real=0.39 secs] 
[GC (Allocation Failure) [PSYoungGen: 207245K->96533K(253952K)] 602249K->602273K(838656K), 0.0333949 secs] [Times: user=0.06 sys=0.08, real=0.03 secs] 
[Full GC (Ergonomics) [PSYoungGen: 96533K->18603K(253952K)] [ParOldGen: 505739K->583476K(817664K)] 602273K->602080K(1071616K), [Metaspace: 3788K->3788K(1056768K)], 0.0684669 secs] [Times: user=0.06 sys=0.16, real=0.07 secs] 
[GC (Allocation Failure) [PSYoungGen: 115371K->96279K(287744K)] 698848K->698359K(1105408K), 0.0171780 secs] [Times: user=0.13 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 226247K->129497K(290304K)] 828328K->827677K(1107968K), 0.0496736 secs] [Times: user=0.19 sys=0.17, real=0.05 secs] 
[GC (Allocation Failure) [PSYoungGen: 259219K->129691K(334848K)] 957399K->957336K(1162752K), 0.1049579 secs] [Times: user=0.20 sys=0.58, real=0.11 secs] 


[Full GC (Ergonomics) [PSYoungGen: 129691K->129460K(334848K)] [ParOldGen: 827644K->827511K(1135104K)] 957336K->956972K(1469952K), [Metaspace: 3788K->3788K(1056768K)], 0.7630309 secs] [Times: user=2.55 sys=0.66, real=0.76 secs] 
```

```
D:\develop\Java\jdk1.8.0_65\bin\java.exe -XX:+PrintGCDetails "-javaagent:D:\develop\IntelliJ IDEA 2020.1.2\lib\idea_rt.jar=2900:D:\develop\IntelliJ IDEA 2020.1.2\bin" -Dfile.encoding=UTF-8 -classpath D:\develop\Java\jdk1.8.0_65\jre\lib\charsets.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\deploy.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\access-bridge-64.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\cldrdata.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\dnsns.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\jaccess.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\jfxrt.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\localedata.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\nashorn.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunec.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunjce_provider.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunmscapi.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\sunpkcs11.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\ext\zipfs.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\javaws.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jce.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jfr.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jfxswt.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\jsse.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\management-agent.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\plugin.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\resources.jar;D:\develop\Java\jdk1.8.0_65\jre\lib\rt.jar;E:\IdeaProjects\JVM\target\classes com.yc.Test03_visualVM.HeapInstance
[GC (Allocation Failure) [PSYoungGen: 32525K->5099K(37888K)] 32525K->29875K(123904K), 0.0093917 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 37771K->5110K(37888K)] 62547K->61103K(123904K), 0.0078460 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 5110K->0K(37888K)] [ParOldGen: 55993K->61002K(144896K)] 61103K->61002K(182784K), [Metaspace: 3780K->3780K(1056768K)], 0.0157771 secs] [Times: user=0.09 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 32768K->5098K(37888K)] 93770K->93552K(182784K), 0.0065602 secs] [Times: user=0.00 sys=0.06, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 37499K->5091K(52736K)] 125953K->125966K(197632K), 0.0074200 secs] [Times: user=0.03 sys=0.06, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 5091K->0K(52736K)] [ParOldGen: 120875K->125774K(241152K)] 125966K->125774K(293888K), [Metaspace: 3783K->3783K(1056768K)], 0.0240476 secs] [Times: user=0.08 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 47615K->5094K(52736K)] 173390K->173550K(293888K), 0.0313306 secs] [Times: user=0.09 sys=0.14, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 52679K->46545K(108032K)] 221135K->221072K(349184K), 0.0277971 secs] [Times: user=0.08 sys=0.11, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 107337K->56802K(118272K)] 281864K->281715K(359424K), 0.0404652 secs] [Times: user=0.03 sys=0.11, real=0.04 secs] 
[Full GC (Ergonomics) [PSYoungGen: 56802K->40638K(118272K)] [ParOldGen: 224912K->240978K(395776K)] 281715K->281616K(514048K), [Metaspace: 3788K->3788K(1056768K)], 0.0260354 secs] [Times: user=0.13 sys=0.03, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 101801K->68070K(154624K)] 342779K->342890K(550400K), 0.0383453 secs] [Times: user=0.14 sys=0.20, real=0.04 secs] 
[GC (Allocation Failure) [PSYoungGen: 154208K->79869K(166400K)] 429028K->428833K(562176K), 0.0210915 secs] [Times: user=0.08 sys=0.00, real=0.02 secs] 
[Full GC (Ergonomics) [PSYoungGen: 79869K->32994K(166400K)] [ParOldGen: 348963K->395666K(586752K)] 428833K->428660K(753152K), [Metaspace: 3788K->3788K(1056768K)], 0.0457681 secs] [Times: user=0.09 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [PSYoungGen: 119474K->117518K(224768K)] 515140K->515173K(811520K), 0.0842301 secs] [Times: user=0.24 sys=0.22, real=0.08 secs] 
[GC (Allocation Failure) [PSYoungGen: 218795K->141309K(242688K)] 616451K->616304K(829440K), 0.0276550 secs] [Times: user=0.11 sys=0.00, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 242357K->100975K(302080K)] 717352K->717156K(919040K), 0.2265162 secs] [Times: user=0.13 sys=1.39, real=0.23 secs] 
[Full GC (Ergonomics) [PSYoungGen: 100975K->100806K(302080K)] [ParOldGen: 616180K->616081K(871936K)] 717156K->716887K(1174016K), [Metaspace: 3788K->3788K(1056768K)], 0.2569016 secs] [Times: user=0.41 sys=0.39, real=0.26 secs] 
[GC (Allocation Failure) [PSYoungGen: 237998K->137066K(306176K)] 854079K->854003K(1178112K), 0.1283725 secs] [Times: user=0.22 sys=0.66, real=0.13 secs] 
[GC (Allocation Failure) [PSYoungGen: 273598K->136483K(366080K)] 990534K->990351K(1238016K), 0.4650646 secs] [Times: user=0.13 sys=2.97, real=0.47 secs] 
[Full GC (Ergonomics) [PSYoungGen: 136483K->118807K(366080K)] [ParOldGen: 853868K->871223K(871936K)] 990351K->990030K(1238016K), [Metaspace: 3788K->3788K(1056768K)], 0.2819435 secs] [Times: user=0.24 sys=0.86, real=0.28 secs] 
[Full GC (Ergonomics) [PSYoungGen: 299751K->281545K(366080K)] [ParOldGen: 871223K->871221K(871936K)] 1170974K->1152766K(1238016K), [Metaspace: 8196K->8196K(1056768K)], 3.9181715 secs] [Times: user=15.47 sys=4.13, real=3.92 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281544K(366080K)] [ParOldGen: 879979K->879978K(880128K)] 1161557K->1161523K(1246208K), [Metaspace: 8196K->8196K(1056768K)], 0.2491159 secs] [Times: user=0.33 sys=0.25, real=0.25 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281544K->281544K(366080K)] [ParOldGen: 879978K->879978K(880128K)] 1161523K->1161523K(1246208K), [Metaspace: 8196K->8196K(1056768K)], 0.1460397 secs] [Times: user=0.02 sys=0.25, real=0.15 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281557K(366080K)] [ParOldGen: 899363K->899362K(900096K)] 1180941K->1180920K(1266176K), [Metaspace: 8196K->8196K(1056768K)], 0.1231576 secs] [Times: user=0.00 sys=0.25, real=0.12 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281558K(366080K)] [ParOldGen: 917513K->917512K(918016K)] 1199091K->1199070K(1284096K), [Metaspace: 8196K->8196K(1056768K)], 0.1199277 secs] [Times: user=0.13 sys=0.31, real=0.12 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281570K(366080K)] [ParOldGen: 931055K->931055K(931328K)] 1212633K->1212625K(1297408K), [Metaspace: 8196K->8196K(1056768K)], 0.1156576 secs] [Times: user=0.05 sys=0.34, real=0.12 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281570K(366080K)] [ParOldGen: 931055K->931055K(931328K)] 1212633K->1212625K(1297408K), [Metaspace: 8196K->8196K(1056768K)], 0.1209292 secs] [Times: user=0.02 sys=0.34, real=0.12 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281570K(366080K)] [ParOldGen: 944687K->944686K(945152K)] 1226265K->1226256K(1311232K), [Metaspace: 8196K->8196K(1056768K)], 0.1369434 secs] [Times: user=0.16 sys=0.41, real=0.14 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281570K(366080K)] [ParOldGen: 960084K->960084K(960512K)] 1241663K->1241654K(1326592K), [Metaspace: 8196K->8196K(1056768K)], 0.0230092 secs] [Times: user=0.05 sys=0.06, real=0.02 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281570K->281570K(366080K)] [ParOldGen: 960084K->960084K(960512K)] 1241654K->1241654K(1326592K), [Metaspace: 8196K->8196K(1056768K)], 0.0110208 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281570K(366080K)] [ParOldGen: 960576K->960576K(961024K)] 1242155K->1242147K(1327104K), [Metaspace: 8196K->8196K(1056768K)], 0.0102843 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 281570K->281463K(366080K)] [ParOldGen: 960576K->960523K(961024K)] 1242147K->1241986K(1327104K), [Metaspace: 8196K->8164K(1056768K)], 2.2795142 secs] [Times: user=13.20 sys=2.19, real=2.28 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281578K->281463K(366080K)] [ParOldGen: 961770K->961770K(962048K)] 1243349K->1243234K(1328128K), [Metaspace: 8164K->8164K(1056768K)], 0.0803552 secs] [Times: user=0.00 sys=0.00, real=0.08 secs] 
[Full GC (Ergonomics) [PSYoungGen: 281570K->281506K(366080K)] [ParOldGen: 976914K->976914K(977408K)] 1258485K->1258421K(1343488K), [Metaspace: 8164K->8164K(1056768K)], 0.0261823 secs] [Times: user=0.02 sys=0.00, real=0.03 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 281506K->281506K(366080K)] [ParOldGen: 976914K->976914K(977408K)] 1258421K->1258421K(1343488K), [Metaspace: 8164K->8164K(1056768K)], 0.0241203 secs] [Times: user=0.02 sys=0.00, real=0.03 secs] 
[Full GC (Ergonomics) Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.yc.Test03_visualVM.HeapInstance.<init>(HeapInstance.java:13)
	at com.yc.Test03_visualVM.HeapInstance.main(HeapInstance.java:18)
[PSYoungGen: 281578K->0K(366080K)] [ParOldGen: 976919K->8199K(834048K)] 1258497K->8199K(1200128K), [Metaspace: 8178K->8178K(1056768K)], 0.0792789 secs] [Times: user=0.11 sys=0.03, real=0.08 secs] 
Heap
 PSYoungGen      total 366080K, used 0K [0x00000000d6200000, 0x00000000fa700000, 0x0000000100000000)
  eden space 181760K, 0% used [0x00000000d6200000,0x00000000d6200080,0x00000000e1380000)
  from space 184320K, 0% used [0x00000000ed380000,0x00000000ed380000,0x00000000f8780000)
  to   space 196608K, 0% used [0x00000000e1380000,0x00000000e1380000,0x00000000ed380000)
 ParOldGen       total 834048K, used 8199K [0x0000000082600000, 0x00000000b5480000, 0x00000000d6200000)
  object space 834048K, 0% used [0x0000000082600000,0x0000000082e01f40,0x00000000b5480000)
 Metaspace       used 8178K, capacity 8280K, committed 8448K, reserved 1056768K
  class space    used 970K, capacity 1010K, committed 1024K, reserved 1048576K

Process finished with exit code 1

```

