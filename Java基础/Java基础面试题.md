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



