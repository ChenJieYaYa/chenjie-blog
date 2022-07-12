#### 1.4.二进制流的获取方式

* 文件系统读入.class文件（最常见）
* 读jar、zip等归档文件，提取类文件
* 实现放在数据库的二进制数据
* 使用协议通过网络加载
* 运行时生成class二进制信息

### 2.Linking（链接）

#### 2.1.Verification（验证）





若输入数据不是ClassFile，抛出异常ClassFormatError





验证的目的是保证加载的字节码合法，验证的步骤比较复杂，实际要验证的项目也很繁多，大概验证过程如下

![1657506041788](assets/1657506041788.png)

**其中格式验证会在加载阶段一起执行，验证通过后，类加载器才会将类的二进制数据加载到方法区**，其他验证会在方法区中进行，符号引用验证会在解析阶段执行

#### 2.2.Preparation（准备）

该阶段会为类的静态变量分配内存，并将其初始化为默认值，对于该阶段应注意以下几点

* Java不支持boolean类型，所以对于boolean类型内部实际是int，int默认值0所以对应于false
* 该阶段不包含static final类型的数据，因为final类型的数据在编译阶段就被分配
* 该阶段不会为类的实例变量初始化，因为实例变量随着对象一起被分配到Java堆中

#### 2.3.Resolution（解析）

该阶段将类、接口、字段和方法的符号引用转为直接引用

符号引用指字面量的引用，和VM的内部数据结构和内存布局无关，容易理解的是Class类文件通过常量池产生大量符号引用，**JVM为每个类都准备方法表，当调用某方法是只需要知道该方法在方法表中的偏移量即可直接调用，通过解析可将符号引用转变为目标方法在方法表中的位置**，从而使方法被成功调用

以println()为例，该方法被调用时，系统需要明确知道该方法的位置，`System.out.println()`方法的字节码为`invokevirtual #24 <java/io/PrintStream.println>`，对应的方法表如下

![1657507177812](assets/1657507177812.png)



## 3.Initialization（初始化）阶段

#### 3.1.概述

如果前面的步骤没有问题，表示类可以顺利的装载到系统中，此时类才会开始执行Java字节码，即**到达初始化阶段才真正开始执行类中定义的Java程序代码**

**初始化阶段的重要工作是执行类的初始化方法`<clinit>()`**，该方法只能由Java编译器生成，由JVM调用，程序员无法自定义该方法，更无法在程序中直接调用，该方法由静态成员赋值语句和static语句块合并产生

在加载类之前，JVM试图加载该类的父类，因此父类总是在子类之前被调用

#### 3.2.static final修饰的字段在哪个阶段被赋值？

**链接阶段的准备环节赋值**

* 对基本数据类型使用static final修饰，则显式赋值(直接赋值常量，而非调用方法)

* 对于String来说，如果使用字面量的方式赋值且使用static final修饰，则显式赋值通常是在链接阶段的准备环节进行

**初始化阶段`<clinit>()`中赋值**

* 排除上述的在准备环节赋值的情况之外的情况

#### 3.3.`<clinit>()`的线程安全性

对于`<clinit>()`方法的调用，JVM会在内部确保其多线程环境中的安全性，即确保其被正确地加锁、同步，若多线程同时初始化某类，那么**只有一个线程可以执行该类的`<clinit>()`**，其他线程阻塞等待，但这同时也会带来多线程场景下加锁再来的问题，如进行耗时操作可能造成阻塞、死锁等，并且这种死锁很难发现

若之前的线程成功加载类，则等在队列中的线程就没有机会再执行`<clinit>()`，当需要使用该类时，JVM会直接返回给它己经准备好的信息

#### 3.4.主动使用与被动使用

**主动使用**：Class只有在首次使用时才会被装载，JVM不会无条件地装载Class类型，JVM规定类或接口在初次使用前必须要进行初始化，此处指的“使用”是指主动使用，主动使用只有如下情况，如果出现如下情况则会对类进行初始化操作

* **创建类实例**，如new、克隆、反序列化等

* **调用静态方法**，即使用字节码`invokestatic`指令

* **调用静态字段**(final修饰特殊考虑)

* **使用反射类的方法**，如Class.forName()

* 初始化子类时，先初始化父类，但这对接口不适用

  > 初始化类，不会先初始化其实现的接口
  >
  > 初始化接口，不会先初始化其父接口

* 接口定义default方法，初始化接口实现类前先初始化该接口

* 虚拟机启动时，先初始化主类(main)


**被动使用**：除以上情况外的其他都属于被动使用，**被动使用不会引起类初始化**，也就是说，代码中出现的类不一定都被架加载或初始化，若不符合主动加载也不会被初始化

###  4.Using（使用）

任何类在使用前都要经过链接阶段(验证、准备、解析)，经历后就等着开发人员使用

### 5.Unloading（卸载）（NO）



> ==========**类加载相关**==========



## 十、JVM何时加载类？

主动加载时才会发生类的加载，主动加载的情况如下

- **创建类实例**，如new、克隆、反序列化等

- **调用静态方法**，即使用字节码`invokestatic`指令

- **调用静态字段**(final修饰特殊考虑)

- **使用反射类的方法**，如Class.forName()

- 初始化子类时，先初始化父类，但这对接口不适用

  > 初始化类，不会先初始化其实现的接口
  >
  > 初始化接口，不会先初始化其父接口

- 接口定义default方法，初始化接口实现类前先初始化该接口

- 虚拟机启动时，先初始化主类(main)

## 十一、类加载的详细过程？每个阶段的工作

> 字节码加载流程

## 十二、JVM的类加载器类型及加载它的目标路径？如何自定义类加载器加载指定目录下的class文件?

### 1.类加载器类型及目标路径

#### 1.1.类加载器类型及目标路径说明

**启动类加载器Bootstrap ClassLoader**，加载jre/lib下的核心jar包

![1657537137768](assets/1657537137768.png)

**扩展类加载器Extension ClassLoader**，加载jre/lib/ext下的核心jar包

![1657537192942](assets/1657537192942.png)

**应用程序类加载器Application ClassLoader**，加载classpath下的类库，加载程序所在的目录

#### 1.2.代码打印验证目标路径

①编写LoadPath

```Java
package ClassLoaderPath;


import java.net.URL;
import java.net.URLClassLoader;

public class LoadPath {
	public static void main(String[] args) {
        System.out.println("启动类的加载路径");
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();//C编写
        for (URL url : urls) {
            System.out.println(url);
        }
        System.out.println("----------------------------");
 
        //取得扩展类加载器
        URLClassLoader extClassLoader = (URLClassLoader) ClassLoader.getSystemClassLoader().getParent();
        System.out.println(extClassLoader);
        System.out.println("扩展类加载器的加载路径：");
        urls = extClassLoader.getURLs();
        for (URL url : urls) {
            System.out.println(url);
        }
        System.out.println("----------------------------");
 
 
        //取得应用程序类加载器
        URLClassLoader appClassLoader = (URLClassLoader) ClassLoader.getSystemClassLoader();
        System.out.println(appClassLoader);
        System.out.println("应用程序类加载器的加载路径：");
        urls = appClassLoader.getURLs();
        for (URL url : urls) {
            System.out.println(url);
        }
        System.out.println("----------------------------");
    }
}
```

②结果

```
启动类的加载路径
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/resources.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/rt.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/sunrsasign.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/jsse.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/jce.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/charsets.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/jfr.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/classes
----------------------------
sun.misc.Launcher$ExtClassLoader@2a139a55
扩展类加载器的加载路径：
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/access-bridge-64.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/cldrdata.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/dhf/
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/dnsns.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/jaccess.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/jfxrt.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/localedata.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/nashorn.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/sunec.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/sunjce_provider.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/sunmscapi.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/sunpkcs11.jar
file:/D:/develop/Java/jdk1.8.0_65/jre/lib/ext/zipfs.jar
----------------------------
sun.misc.Launcher$AppClassLoader@4e0e2f2a
应用程序类加载器的加载路径：
file:/E:/JAVASE_WordSpace/MyClassLoader/bin/
----------------------------
```

#### 1.3源码查看验证目标路径

Launcher为入口

![1657542294153](assets/1657542294153.png)

![1657542358739](assets/1657542358739.png)

![1657542390967](assets/1657542390967.png)

### 2.自定义类加载器

**实现原理**是通过类名找到对应的.class文件，然后将.class文件转为二进制数据，最后利用二进制数数据生成Class对象

**代码实现**需继承Java中的ClassLoader类，重写loadClass或者findClass方法均可，在重写的方法内调用父类的defineClass()将二进制数据转为Class对象

①定义需要被自定义加载器加载的类MyTest（E:\JVM\MyClassLoader）

```
public class MyTest {
    static {
        System.out.println("hello!!!");
    }
}
```

②编译MyTest.java得到.class文件

```
C:\Users\CJ>e:

C:\Users\CJ>cd E:\JVM\MyClassLoader

E:\JVM\MyClassLoader>javac MyTest.java
```

③自定义类加载器MyClassLoader（eclipse）

```Java
import java.io.BufferedInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
 
public class MyClassLoader extends ClassLoader{
    private String byteCodePath;//要加载的字节码文件的路径
    public MyClassLoader(String byteCodePath){
        this.byteCodePath = byteCodePath;
    }
	
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String fileName = byteCodePath + name + ".class";//拼接要加载的字节码文件的绝对路径
		
        BufferedInputStream in = null;
        ByteArrayOutputStream out = null;
        try {
            in = new BufferedInputStream(new FileInputStream(fileName));//输入流读取.class文件
            out = new ByteArrayOutputStream();
			
            int len = 0;
            byte[] data = new byte[1024];//1kb
            while((len = in.read(data)) != -1){
                out.write(data,0,len);
            }
            byte[] bytes = out.toByteArray();//获取到字节码的二进制流
            Class<?> aClass = defineClass(null, bytes, 0, bytes.length);// 调用父类方法获取Class对象
            return aClass;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            // 释放资源
            if (in != null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (out != null){
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
```

④定义测试类（eclipse）

```
package MyClassLoader;

public class Test {
    public static void main(String[] args)throws Exception {
		//1.构建自定义类加载器
        MyClassLoader myClassLoader = new MyClassLoader("E://JVM//MyClassLoader//"); //要加载的路径
        //2.通过.class文件获取Class对象
		Class<?> myTest = myClassLoader.findClass("MyTest");
		//注意主动使用加载器加载class文件不会触发类的初始化方法<clinit>()，所以需要通过创建对象的方式查看静态方法是否执行
		Object obj = myTest.getConstructor().newInstance(null);
    }
}
```

⑤结果在控制台输出`hello!!!`

## 十三、什么是双亲委派模型？有什么作用？

### 1.什么是双亲委派模型？

**双亲委派机制**指当类加载器收到类加载请求时，该类加载器首先会把请求委派给父类加载器，每个类加载器都是如此，只有父类加载器在自己的搜索范围内找不到指定类时，子类加载器才会尝试自己去加载

### 2.双亲委派工作过程

首先判断类是否加载，若未加载交给双亲委派器加载

* 当Application ClassLoader收到类加载请求时，他首先不会自己去尝试加载这个类，而是将这个请求委派给父类加载器Extension ClassLoader去完成

* 当Extension ClassLoader收到类加载请求时，他首先也不会自己去尝试加载这个类，而是将请求委派给父类加载器Bootstrap ClassLoader去完成

* 如果Bootstrap ClassLoader加载失败(在%JAVA_HOME%\lib中未找到所需类)，就会让Extension ClassLoader尝试加载

* 如果Extension ClassLoader也加载失败，就会使用Application ClassLoader加载

若双亲委派器都没有加载成功，会使用自定义加载器去尝试加载findClass

如果均加载失败，就会抛出ClassNotFoundException异常

![1657541161575](assets/1657541161575.png)

### 3.双清委派源码

```Java
protected Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            //首先检查这个classsh是否已经加载过
            Class<?> c = findLoadedClass(name);
            //c==null表示没有加载
            if (c == null) {
                try {
                    if (parent != null) {//如果有父类的加载器则让父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {//如果父类的加载器为空 则说明递归到bootStrapClassloader，因为这是C编
                        //bootStrapClassloader比较特殊无法通过get获取
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {}
                
                //如果bootstrapClassLoader仍然没有加载过，则递归回来尝试自己去加载class
                if (c == null) {
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
}
```

### 4.双清委派的作用

**保证安全性**：防止加载同一个.class，通过向上委托问一问是否加载过，加载过就不用再加载一遍

**保证唯一性**：核心.class不能被篡改，通过委托的方式不会篡改核心.class，试想若没有双亲委派机制，每个类加载器都自行加载，若用户编写了一个java.lang.Object的同名类放在ClassPath中，多个加载器都去加载Object，导致系统中Object各不相同，运行程序时出错

## 十四、类加载器如何确保类在JVM中的唯一性？两个类来自同一Class文件，被同一个JVM加载，这两个类一定相等吗？

### 1.类加载器如何确保类在JVM中的唯一性？

>  对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间

也就是说比较两个类是否相等，只有在这两个类被同一类加载器加载的前提下才有意义，即**类+类加载器才唯一确定一个Java类**

### 2.两个类来自同一Class文件，被同一个JVM加载，这两个类一定相等吗？

不一定，还需要判断这两个类是否属于同一个类加载器

> 此处的相等包括Class的equals()、isAssignableFrom()、isInstance()方法的返回结果与instanceof关键字的判断结果

①编写测试代码

```Java
package OnlyClassLoader;

import java.io.IOException;
import java.io.InputStream;

public class OnlyClassLoaderTest {
	public static void main(String[] args) throws Exception {
		ClassLoader myLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName=name.substring(name.lastIndexOf(".")+1)+".class";
                    InputStream is=getClass().getResourceAsStream(fileName);
                    if( is == null ){
                        return super.loadClass(name);
                    }
                    byte[] bytes = new byte[is.available()];
                    is.read(bytes); //通过自定义类加载器读取class文件的二进制流
                    return defineClass(name, bytes, 0,bytes.length);
                    
                } catch (IOException e) {
                    e.printStackTrace();
                    throw new ClassNotFoundException(name);
                }
            }
        };
        
        Object obj = myLoader.loadClass("OnlyClassLoader.OnlyClassLoaderTest").newInstance();
        System.out.println(obj.getClass());
        System.out.println(OnlyClassLoaderTest.class);
        System.out.println("------------------------");
        System.out.println("equals："+OnlyClassLoaderTest.class.equals(obj));
        System.out.println("isAssignableFrom："+OnlyClassLoaderTest.class.isAssignableFrom(obj.getClass()));
        System.out.println("isInstance："+OnlyClassLoaderTest.class.isInstance(obj));
        System.out.println(obj instanceof OnlyClassLoaderTest);
        System.out.println("------------------------");
        System.out.println(OnlyClassLoaderTest.class.getClassLoader());
        System.out.println(obj.getClass().getClassLoader());
	}
}
```

②输出结果

```
class OnlyClassLoader.OnlyClassLoaderTest
class OnlyClassLoader.OnlyClassLoaderTest
//这表明obj对象确实是OnlyClassLoader.OnlyClassLoaderTest实例出来的对象，来自同一个Class文件
------------------------
equals：false
isAssignableFrom：false
isInstance：false
false
//返回false是因为虚拟机中存在两个OnlyClassLoaderTest类，一个由应用程序类加载器加载，另一个由自定义类加载器加载，虽然二者都来自同一Class文件，但依然是两个独立的类，做对象所属类型检查时结果自然为false
------------------------
sun.misc.Launcher$AppClassLoader@4e0e2f2a
OnlyClassLoader.OnlyClassLoaderTest$1@6d06d69c
```

## 十五、Tomcat的类加载器有哪些？

### 1.为什么Tomcat需要自己的类加载器？

如果Tomcat类加载器机制和双亲委派机制一样会出现什么问题？

* **两个同名类无法被区分**：Tomcat的webapps目录下有两个应用，分别引入第三方jar：tool-1.0.jar和tool-2.0.jar，虽然jar的版本不同，但两个jar中都有MyTool.java类，若严格按照双亲委派机制可能导致两个应用中只有一个类会被加载，另一个已经加载过(全路径+类加载器相同)导致不会被加载，所以**要保证项目彼此隔离**
* **同一个类被多次加载**：两个项目都依赖Spring，当Spring的jar被加载到内存后，两个项目都加载一次Spring的jar，造成资源浪费，所以**要保证项目间能共享资源**
* Tomcat本身有类，也需要和项目的类进行隔离

①编写两个MyTool

```
package com.yc.MyTool;

class MyTool{	
	public static void say(){
		System.out.println("hello");
	}
	public static void main(String[] args) {
		say();
	}
}
```

```
package com.yc.MyTool;

public class MyTool {
	public static void say(){
		System.out.println("world");
	}
	public static void main(String[] args) {
		say();
	}
}
```

②将两个Tool打成jar

![1657593073891](assets/1657593073891.png)

![1657593344749](assets/1657593344749.png)

③创建普通Java项目，编写测试代码，导入两个jar，发现永远只输出hello

```
package MyToolTest;

import com.yc.MyTool.MyTool;

public class Test {
	public static void main(String[] args) {
		MyTool tool = new MyTool();
		tool.say();
	}
}
```

### 2.Tomcat的类加载器种类

![1657586925535](assets/1657586925535.png)

三个基础类加载器+每个web应用的web类加载器，默认情况下三个基础类加载器都是同一个(Common)

**Common ClassLoader**： 父类加载器是应用程序类加载器，是Tomcat顶层的公用类加载器，路径为common.loader，默认指向CATALINA_HOME/lib下的包，**负责加载Tomcat本身的类和Web应用都需要的类，如Servlet规范包等**

**Catalina ClassLoader**：父类加载器是Common加载器，路径为server.loader(默认为空)，**目的是隔离Tomcat本身的类和Web项目的类，负责加载Tomcat应用类，对Web应用不可见(解耦合)**

> 若需要共享采用父子关系，若需要隔离采用平行关系

**Shared ClassLoader**： 父类加载器是Common加载器，Shared ClassLoader作为WebApp ClassLoader的父加载器，路径为shared.loader(默认为空)，**负责加载Web应用共享的类，对Tomcat服务器不可见**

**WebApp ClassLoader**：父类加载器是Shared加载器，加载/WEB-INF/classes目录下未压缩的Class和资源文件以及/WEB-INF/lib下的jar包，**目的是隔离Web应用，区分类名相同的类，因为这些类所属的类加载器不同**

### 3.Tomcat类加载器设计优点

**共享性**：Common ClassLoader与Shared ClassLoader

**隔离性**：Catalina ClassLoader与WebApp ClassLoader

## 十六、双亲委派模型最大问题

**底层的类加载器无法加载底层的类**, 比如如下情况:

 javax.xml.parsers包中定义了xml解析的类接口,  Service Provider Interface SPI 位于rt.jar ，即接口在启动类加载器中，而SPI的实现类，通常是由用户实现的， 由应用程序类加载器加载

javax.xml.parsers.FactoryFinder中的解决代码

```
static private Class getProviderClass(String className, ClassLoader cl,boolean doFallback, boolean useBSClsLoader) throws ClassNotFoundException{
	try {
		//如果类加载器为空
    	if (cl == null) {
        	if (useBSClsLoader) {//启动类加载器
            	return Class.forName(className, true, FactoryFinder.class.getClassLoader());
        	} else {
            	cl = ss.getContextClassLoader();//获取上下文加载器
            	if (cl == null) {
                	throw new ClassNotFoundException();
             	} else {
                	return cl.loadClass(className);//使用上下文ClassLoader
             	}
         	}
     	} else {
        	return cl.loadClass(className);
    	}
	} catch (ClassNotFoundException e1) {
    	if (doFallback) {
        	// Use current class loader - should always be bootstrap CL
        	return Class.forName(className, true, FactoryFinder.class.getClassLoader());
    }
```

  更多可以参考理解：[jdbc的SPI加载方式](https://blog.csdn.net/syh121/article/details/120274044)

    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);

## 十七、破环双亲委派的场景

双亲委派模式是默认的模式，但并非必须，以下场景破坏双亲委派：

* Tomcat的WebappClassLoader先加载自己的Class，找不到再委托Parent
* OSGi的ClassLoader形成网状结构，根据需要自由加载Class

## 十八、什么是热替换？完成一个热替换的例子

**热替换**：在不重启服务的情况下更改的代码生效，热替换可以提升开发以及调试的效率，基于Java类加载器实现，热加载的不安全性导致其一般不会用于正式的生产环境

①被动态替换的类

```java
package HotReplace;

public class Demo {
	public void hot() {
        System.out.println("OldDemo1"); // A:old class print
        //System.out.println("NewDemo1"); // B:new class print
    }
}
```

②自定义类加载器MyClassLoader

```Java
package HotReplace;

import java.io.BufferedInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
 
public class MyClassLoader extends ClassLoader{
    private String byteCodePath;//要加载的字节码文件的路径
    public MyClassLoader(String byteCodePath){
        this.byteCodePath = byteCodePath;
    }
	
    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        String fileName = byteCodePath + name + ".class";//拼接要加载的字节码文件的绝对路径
		
        BufferedInputStream in = null;
        ByteArrayOutputStream out = null;
        try {
            in = new BufferedInputStream(new FileInputStream(fileName));//输入流读取.class文件
            out = new ByteArrayOutputStream();
			
            int len = 0;
            byte[] data = new byte[1024];//1kb
            while((len = in.read(data)) != -1){
                out.write(data,0,len);
            }
            byte[] bytes = out.toByteArray();//获取到字节码的二进制流
            Class<?> aClass = defineClass(null, bytes, 0, bytes.length);// 调用父类方法获取Class对象
            return aClass;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            // 释放资源
            if (in != null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (out != null){
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
```

③测试类

```Java
package HotReplace;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import MyClassLoader.MyClassLoader;

public class Test {
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchMethodException, SecurityException {
		while (true) {
            try {
            	//1.构建自定义类加载器
                MyClassLoader myClassLoader = new MyClassLoader("E://JAVASE_WordSpace//MyClassLoader//bin//HotReplace//"); //要加载的路径
                //2.通过.class文件获取Class对象
        		Class<?> myTest = myClassLoader.findClass("Demo");
        		//3.创建运行时类的实例
        		Object demo = myTest.getConstructor().newInstance(null);
        		//4.获取运行时类中指定的方法
        		Method m = myTest.getMethod("hot");
        		// 5. 调用指定的方法
                m.invoke(demo);
                
                Thread.sleep(5000);
            } catch (Exception e) {
                System.out.println("not find");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
            }
        }
	}
}
```

④放开Demo中的第一句，运行Test，隔一段时间再放开Demo中的第二句，输出结果如下

```
OldDemo1
OldDemo1
NewDemo1
```

## 十九、JVM是如何防止将非字节码文件加载到JVM中的?

**魔术字**

①编写一个简单的测试代码

```
class Hello{
	public static void main(String []args){
		System.out.println("hello");
	}
}
```

②生成字节码：`javac Hello.java`

③PowerShell窗口输入`format-hex Hello.class`

![1657613686434](assets/1657613686434.png)

## 二十、`<clinit>() `是类构造器方法，它与类的构造方法有什么区别？

`<init>()`：实例构造器方法，对非静态变量解析初始化，new对象时调用对象类的constructor方法时执`<init>()`，即实例化对象时调用

> 实例化的四种途径：new、Class或Constructor 对象的newInstance()、任意对象的clone()、ObjectInputStream的getObject()反序列化

`<clinit>()`：类构造器方法，对静态变量、静态代码块进行初始化，在类加载过程的初始化阶段JVM会执行`<clinit>()`，











=============================
从虚拟机层面来看，以下代码运行时

    public class ClassInitTest {
        private static int number = 10;//linking之prepare:number=0 -> initial:10 -> 20
    	static {
            number = 20;
            System.out.println(num);
        }
    
        public static void main(String[] args) {
            System.out.println(ClassInitTest.num);//2
            System.out.println(ClassInitTest.number);//10
        }
    }

 number值的变化历程?????

==============================
以下代码为输出类加载器的名字, 请给出你的结果:

```
public class ClassLoaderTest {
    public static void main(String[] args) {
    	ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
    	System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
    	
    	ClassLoader extClassLoader = systemClassLoader.getParent();
    	System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@1540e19d
    	
    	ClassLoader bootstrapClassLoader = extClassLoader.getParent();
    	System.out.println(bootstrapClassLoader);//null
    	
    	ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
    	System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
    	
    	ClassLoader classLoader1 = String.class.getClassLoader();
    	System.out.println(classLoader1);//null
	}
}
```

====================给出以下代码输出的各个类加载器的路径( 注意：分别用idea 测试和 直接在控制台下输出  测试两次，查看结果的不同，给出结论)======================

    public class ClassLoaderTest1 {
        public static void main(String[] args) {   
            System.out.println("**********启动类加载器**************");
            //获取BootstrapClassLoader能够加载的api的路径
            URL[] urLs = sun.misc.Launcher.getBootstrapClassPath().getURLs();
            for (URL element : urLs) {
                System.out.println(element.toExternalForm());
            }
            //从上面的路径中随意选择一个类,来看看他的类加载器是什么:引导类加载器
            ClassLoader classLoader = Provider.class.getClassLoader();
            System.out.println(classLoader);//null
    
            System.out.println("***********扩展类加载器*************");
            String extDirs = System.getProperty("java.ext.dirs");
            for (String path : extDirs.split(";")) {
                System.out.println(path);
            }
    
            //从上面的路径中随意选择一个类,来看看他的类加载器是什么:扩展类加载器
            ClassLoader classLoader1 = CurveDB.class.getClassLoader();
            System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@1540e19d
    	}
    }
======================================
获取ClassLoader 实例的途径

 1. 获取当前类的 ClassLoader ：clazz.getClassLoader()
 2. 获取当前线程上下文的ClassLoader：Thread.currentThread().getContextClassLoader()
 3. 获取系统的ClassLoader ： ClassLoader.getSystemClassLoader();
 4. 获取调用者的ClassLoader ： DriverManager.getCallerClassLoader()

 那么以下代码输出的类加载器将是什么：


        public class ClassLoaderTest2 {
            public static void main(String[] args) {
                try {        
                    //1.Class.forName().getClassLoader()
                    ClassLoader classLoader = Class.forName("java.lang.String").getClassLoader();
                    System.out.println(classLoader); // String 类由启动类加载器加载，我们无法获取
                //2.Thread.currentThread().getContextClassLoader()
                ClassLoader classLoader1 = Thread.currentThread().getContextClassLoader();
                System.out.println(classLoader1);
    
                //3.ClassLoader.getSystemClassLoader().getParent()
                ClassLoader classLoader2 = ClassLoader.getSystemClassLoader();
                System.out.println(classLoader2);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
    	}
    }
===================测试一下双亲委派模型在保证系统模型统一方面的作用================
创建一个类：   java.lang.String

    package java.lang;
    
    public class String {
        static{
            System.out.println("我是自定义的String类的静态代码块");
        }
    }
    
    public class StringTest {
        public static void main(String[] args) {
            java.lang.String str = new java.lang.String();
            System.out.println("hello");
    
            StringTest test = new StringTest();
            System.out.println(test.getClass().getClassLoader());
    	}
    }
以上代码的输出结果是什么?

如果修改一下代码如下: 

    package java.lang;
    
    public class String {
        static{
            System.out.println("我是自定义的String类的静态代码块");
        }
        
        public static void main(String[] args) {
            System.out.println("hello,String");
        }
    }
 输出会是什么? 为什么?   这种机制就叫沙箱安全机制，请试着解释此术语及作用?

如果在自定义的java.lang包下定义自己的类，代码如下，会发生什么，请试分析为什么?

```
package java.lang;

public class ShkStart {
    public static void main(String[] args) {
        System.out.println("hello!");
    }
}
```


=======================================
试分析基于SPI机制的  jdbc 驱动的加载流程.  着重分析，为什么jdbc的接口是通过Bootstrap ClassLoader加载  rt.jar包获取，但 jdbc驱动却无法通过Bootstrap classload加载，那么它是怎么加载进来的?

==========================================
jvm中只有主动使用类，才会加载类，那么加载类的七种情况有哪些?

```
1.创建类的实例。

例如：new Class();


2.访问某个类或接口的静态变量，或者给静态变量赋值



3.调用类的静态方法


4.反射：Class.forName("java.lang.String");


5.初始化一个类的子类


6.Java虚拟机启动时被标明为启动类的类（包含Main方法）
```

==================================================
类加载器作为jvm运行的第一阶段的组件的总结???











1. jvm运行时数据区的划分?
2. 根据jvm规范，这些数据区中哪些会出现 内存溢出异常，分别是什么场景下出现?
3. 这些数据区哪些是线程独有的，哪些是线程共享区?
4. 每个区存储的数据的特点?
5. 程序计数器是什么，它是线程独有的吗? 它是否有内存溢出问题.
6. 虚拟机栈上保存哪些数据?怎么放?虚拟机栈是线程独有的吗，它是否有内存溢出问题?虚拟机栈的优点?
7. 虚拟机栈的大小是否可动?是否会有异常出现?
8. 如何设置虚拟机栈大小?
9. 什么叫本地方法? 是否可以写一个例子来实现本地方法，以输出一个hello world?
10.什么叫本地方法栈?有什么作用?它是线程私有的吗? 它是否有可能抛出异常?
11. jvm规范一定强制要求实现本地方法栈吗?
12. 方法区是线程独有的吗?它是否有异常?它的作用?
13. 方法区的演进, jdk7及以前，它叫什么? jdk8开始，这又叫什么. 
14. 方法区或永久代的大小如何设置?



