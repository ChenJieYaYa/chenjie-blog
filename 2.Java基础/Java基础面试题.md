# Java基础面试题

## 1.Java语言的特点是什么？

* 学习起来较简单

* 面向对象，三大特性封装、继承、多态
* 与平台无关，JVM的跨平台特性
* 支持多线程、支持网络编程
* 编译与解释并存

## 2.JVM&JRE&JDK

**JVM**是运行字节码的Java虚拟机，是Java实现跨平台的关键

**JDK**包含开发环境JRE与大量开发包，提供给开发人员使用，能够创建和编译程序

**JRE**是Java运行环境，提供给使用者使用，不适用于开发环境，即运行已经编译好的程序，包含JVM、Java类库、Java命令和其他基础构建

## 3.什么是字节码？有什么好处？

字节码是运行于JVM中的代码，即`.class`文件，字节码不面向任何特定的处理器，只面向JVM，Java使用字节码在一定程度上解决传统解释型语言执行效率低的问题，同时保留解释型语言可移植的特点

## 4.为什么说Java语言编译与解释并存？

**编译型**语言通过编译器一次性将源代码翻译成机器码后执行，其执行速度快，但开发效率低，如C、C++、Go；**解释型**语言通过解释器一行行将源代码解释成机器码后执行，其执行速度慢，但开发效率高，如Python、JavaScript、PHP

`.class->机器码` 中JVM类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢；而且有些方法和代码块是经常需要被调用的，即热点代码，所以后面引进JIT编译器，JIT属于运行时编译，当JIT编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用，这就是常说的**Java 是编译与解释共存的语言** 

## 5.Java和C++的区别？

**相同点**是二者都是面向对象的语言，拥有面向对象三特征封装、继承、多态

**不同点**如下

* Java不提供指针直接访问内存，程序内存更加安全
* Java只支持单继承，而C++支持多继承
* Java只支持方法重载，而C++支持方法和操作符重载
* Java存在自动内存管理垃圾回收机制(GC)，不用手动释放内存

## 6.标识符和关键字的区别？

**标识符**简单来说就是一个名字，由`大小写字母，数字字符，$，_`组成，**关键字**则是被赋予特殊含义的标识符，Java内的关键字如下，其`true`、`false`、`null`是字面值，不是关键字，但同时不可作为标识符

|         分类         |                            关键字                            |
| :------------------: | :----------------------------------------------------------: |
|       访问控制       |                  private、protected、public                  |
| 类，方法和变量修饰符 | abstract、class、extends、final、implements、interface、native、new、static、strictfp、synchronized、transient、volatile、enum |
|       程序控制       | break、continue、return、do、while、if、else、for、instanceof、switch、case、default、assert |
|       错误处理       |              try、catch、throw、throws、finally              |
|        包相关        |                       import、package                        |
|       基本类型       |     boolean、byte、char、double、float、int、long、short     |
|       变量引用       |                      super、this、void                       |
|        保留字        |                         goto、const                          |

## 7.成员变量与局部变量的区别？

## 8.成员变量与静态变量的区别？

## 9.静态方法为什么不能调用非静态成员?

静态方法属于类，在类加载的时候就会分配内存，可以通过类名直接访问，而非静态成员属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问，在类的非静态成员不存在的时候静态成员就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作

## 10.重载和重写有什么区别？

|   区别点   | 重载方法 |                           重写方法                           |
| :--------: | :------: | :----------------------------------------------------------: |
|  发生范围  | 同一个类 |                             子类                             |
|  参数列表  | 必须修改 |                         一定不能修改                         |
|  返回类型  |  可修改  |      子类方法返回值类型应比父类方法返回值类型更小或相等      |
|    异常    |  可修改  | 子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等； |
| 访问修饰符 |  可修改  |            一定不能做更严格的限制（可以降低限制）            |
|  发生阶段  |  编译期  |                            运行期                            |

方法的重写要遵循**两同两小一大**，即方法名和形参列表相同，子类返回值类型和异常更小或相等，子类权限访问修饰符更大或相等

## 11.重载会优先匹配固定参数还是可变参数的方法呢？

优先匹配**固定参数**的方法，因为固定参数的方法匹配度更高

## 12.Java内有哪几种基本数据类型？

| 基本类型  | 位数 | 字节 | 默认值  |                  取值范围                  |
| :-------: | :--: | :--: | :-----: | :----------------------------------------: |
|  `byte`   |  8   |  1   |    0    |                 -128 ~ 127                 |
|  `short`  |  16  |  2   |    0    |               -32768 ~ 32767               |
|   `int`   |  32  |  4   |    0    |          -2147483648 ~ 2147483647          |
|  `long`   |  64  |  8   |   0L    | -9223372036854775808 ~ 9223372036854775807 |
|  `char`   |  16  |  2   | 'u0000' |                 0 ~ 65535                  |
|  `float`  |  32  |  4   |   0f    |           1.4E-45 ~ 3.4028235E38           |
| `double`  |  64  |  8   |   0d    |     4.9E-324 ~ 1.7976931348623157E308      |
| `boolean` |  1   |      |  false  |                true、false                 |

## 13.基本类型和包装类型的区别？

八种基本数据类型对应的包装类分别为`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean`

- 包装类型可用于泛型，而基本类型不可以
- 基本数据类型的局部变量存放在JVM栈中的局部变量表中，基本数据类型的非静态成员变量存放在JVM堆中；包装类型属于对象类型，几乎所有对象实例都存在于堆中
- 相比于对象类型， 基本数据类型占用的空间非常小

> **为什么说是几乎所有对象实例都存于堆中呢？** 
>
> JVM会对对象进行逃逸分析，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

## 14.包装类型的缓存机制了解么？

`Byte`、`Short`、`Integer`、`Long`默认创建数值[-128，127]的相应类型的缓存数据，`Character`创建数值在[0，127]范围的缓存数据，`Boolean`直接返回`True`或`False`，若超出缓存范围才会创建新对象，两种浮点数类型的包装类`Float`、`Double`并没有实现缓存机制

**Integer 缓存源码：**

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```

**Character 缓存源码:**

```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache(){}
    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```

**Boolean 缓存源码：**

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

下面的代码的输出结果是 `true` 还是 `false` 呢？**包装类对象之间值的比较全部使用equals**

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Integer i1 = 40;//调用Integer.valueOf(40)，发生装箱，先看缓存中有没有
Integer i2 = new Integer(40);//new会创建新对象
System.out.println(i1==i2);// 输出 false

Float i1 = 333f;
Float i2 = 333f;
System.out.println(i1 == i2);// 输出 false

Double i1 = 1.2;
Double i2 = 1.2;
System.out.println(i1 == i2);// 输出 false
```

## 15.自动装箱与拆箱了解吗？

**装箱**是将基本类型用它们对应的引用类型包装起来，实际调用`valueOf()`；**拆箱**是将包装类型转换为基本数据类型，实际调用`xxxValue()`，如下

```java
Integer i = 10;  //装箱，等价Integer i = Integer.valueOf(10)
int n = i;   //拆箱，等价int n = i.intValue()
```

**如果频繁拆装箱的话，也会严重影响系统的性能，我们应该尽量避免不必要的拆装箱操作**

## 16.为什么浮点数运算的时候会有精度丢失的风险？如何解决？

浮点数运算代码精度丢失和计算机保存浮点数的机制有很大关系，计算机是二进制的，且计算机表示一个数的宽度是有限的，无限循环的小数存储在计算机时只能被截断，所以就会导致小数精度发生损失的情况，如下

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

`BigDecimal`可以实现对浮点数的运算，不会造成精度丢失

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.1 */
System.out.println(y); /* 0.1 */
System.out.println(Objects.equals(x, y)); /* true */
```



