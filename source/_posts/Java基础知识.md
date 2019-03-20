---
title: Java基础知识
date: 2019-03-14 13:07:49
tags: Java
---
零散的面试常考基础知识
<!--more-->

# String 和 StringBuffer、StringBuilder 的区别是什么？

## 可变性

简单的来说：String 类中使用 final 关键字字符数组保存字符串， `private final char value[] `，所以 String 对象是不可变的。而StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串 char[]value 但是没有用 final 关键字修饰，所以这两种对象都是可变的。 StringBuilder 与 StringBuffer 的构造方法都是调用父类构造方法也就是 AbstractStringBuilder 实现的，大家可以自行查阅源码。
 AbstractStringBuilder.java
 
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    AbstractStringBuilder() {
    } 
    AbstractStringBuilder(int capacity) {
      value = new char[capacity];
    }
}
```

## 线程安全性 
String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的，如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有synchronized关键字。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。

## 性能

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。 StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StirngBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。


## 对于三者使用的总结： 
1. 操作少量的数据 = String
2. 单线程操作字符串缓冲区下操作大量数据 = StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据 = StringBuffer


# 关于 final 关键字的一些总结

final关键字主要用在三个地方：变量、方法、类。 
1. 对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
2. 当用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。 
3. 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为fianl

# Object类的常见方法总结

```java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。

public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
```

# Java的异常体系

![image](http://490.github.io/images/20190313_073019.png)

## Error（错误）

是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。

这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（VirtualMachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。 

## Exception（异常）

### unchecked Exception（RuntimeException）

RuntimeException 异常由Java虚拟机抛出。NullPointerException（要访问的变量没有引用任何对象时，抛出该异常）、ArithmeticException（算术运算异常，一个整数除以0时，抛出该异常）和 ArrayIndexOutOfBoundsException （下标越界异常）。

**对未检查的异常(unchecked exception )的几种处理方式**：
1. 捕获
2. 继续抛出
3. 不处理

### checked Exception（非RuntimeException）

对于不是你犯的错，我们统称为非RuntimeException，也叫checked Exception。
**对检查的异常的几种处理方式**：

1. 继续抛出，消极的方法，一直可以抛到java虚拟机来处理
2. 用try...catch捕获

对于检查的异常必须处理，或者必须捕获或者必须抛出

异常处理完成以后，Exception对象会在下一个垃圾回收过程中被回收掉。

注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。 

### Throwable类常用方法

`public string getMessage()`:返回异常发生时的详细信息 
`public string toString()`:返回异常发生时的简要描述
`public string getLocalizedMessage()`:返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以声称本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与getMessage（）返回的结果相同 
 `public void printStackTrace()`:在控制台上打印Throwable对象封装的异常信息

### 异常处理总结 

- try 块：用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。
- catch 块：用于处理try捕获到的异常。
- finally 块：无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行。

在以下4种特殊情况下，finally块不会被执行： 

1. 在finally语句块中发生了异常。 
2. 在前面的代码中用了System.exit()退出程序。 
3. 程序所在的线程死亡。 
4. 关闭CPU


# 获取用键盘输入常用的的两种方法

方法1：通过 Scanner

```java
Scanner input = new Scanner(System.in);
String s = input.nextLine();
input.close();
```

方法2：通过 BufferedReader

```java
BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```

# 接口和抽象类的区别是什么

1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），抽象类可以有非抽象的方法
2. 接口中的实例变量默认是 final 类型的，而抽象类中则不一定 
3. 一个类可以实现多个接口，但最多只能实现一个抽象类 
4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定 
5. 接口不能用 new 实例化，但可以声明，但是必须引用一个实现该接口的对象 。从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

备注:在JDK8中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现两个接口，接口中定义了一样的默认方法，必须重写，不然会报错。

# 深拷贝与浅拷贝

[Java深拷贝浅拷贝](http://blog.csdn.net/xiaxia__/article/details/41652057)
将一个对象的引用复制给另外一个对象，一共有三种方式。第一种方式是直接赋值，第二种方式是浅拷贝，第三种是深拷贝。

- 直接赋值：A a1 = a2，复制的是引用，也就是说a1和a2指向的是同一个对象。因此，当a1变化的时候，a2里面的成员变量也会跟着变化。 
- 浅：属性不一样，是独立的，对象（方法）什么的一样，是同一份。clone()主要做了些什么，创建一个新对象，然后将当前对象的非静态字段复制到该新对象，如果字段是值类型的，那么对该字段执行复制；如果该字段是引用类型的话，则复制引用但不复制引用的对象。因此，原始对象及其副本引用同一个对象。
- 深：都不一样

```java
//奥义在于深拷贝在super.clone之后还得把属性什么的new一个，然后做和原本的构造函数给属性赋值一样的操作，这样属性就也不一样了
@Override
protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
}
```


new Integer(123) 与 Integer.valueOf(123) 的区别在于，new Integer(123) 每次都会新建一个对象，而 Integer.valueOf(123) 可能会使用缓存对象，因此多次使用 Integer.valueOf(123) 会取得同一个对象的引用。


# 反射、泛型

## [泛型](https://blog.csdn.net/s10461/article/details/53941091)

泛型，即“参数化类型”。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

```java
List arrayList = new ArrayList();
arrayList.add("aaaa");

List<String> arrayList = new ArrayList<String>();
//arrayList.add(100); 在编译阶段，编译器就会报错
```

**泛型只在编译阶段有效**。见“反射”，用方法的反射绕过编译

**通配符的出现是为了指定泛型中的类型范围**。

1.  `<?>`被称作无限定的通配符。
2.  `<? extends T>`被称作有上限的通配符。
3.  `<? super T>`被称作有下限的通配符。

泛型类、泛型接口、泛型方法

泛型擦除： Java 编译器生成的字节码文件不包含有泛型信息，泛型信息将在编译时被擦除，这个过程称为泛型擦除。其主要过程为 
1. 将所有泛型参数用其最左边界（最顶级的父类型）类型替换； 
2. 移除 all 的类型参数。


## 反射

`javap` 原生的 看class文件
![image](http://490.github.io/images/20190310_232512.png)
![image](http://490.github.io/images/20190310_232518.png)
![image](http://490.github.io/images/20190310_232524.png)

类是java.lang.class类的实例对象

```java
//第一种表示方式--->实际在告诉我们任何一个类都有一个隐含的静态成员变量class
		Class c1 = Foo.class;
		//第二中表达方式  已经知道该类的对象通过getClass方法
		Class c2 = foo1.getClass();
		/*官网 c1 ,c2 表示了Foo类的类类型(class type) 万事万物皆对象，类也是对象，是Class类的实例对象
		 * 这个对象我们称为该类的类类型 */
		//不管c1  or c2都代表了Foo类的类类型，一个类只可能是Class类的一个实例对象
		System.out.println(c1 == c2);相等
		//第三种表达方式
		Class c3 = null;
		try {
			c3 = Class.forName("com.imooc.reflect.Foo");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println(c2==c3);相等
		//我们完全可以通过类的类类型创建该类的对象实例---->通过c1 or c2 or c3创建Foo的实例对象
		try {
			Foo foo = (Foo)c1.newInstance();//需要有无参数的构造方法
			foo.print();
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

![image](http://490.github.io/images/20190310_232551.png)
new创建对象是静态加载类，编译时刻就需要加载所有可能用到的类

```java
Class c = obj.getClass();
/** 成员变量也是对象
* java.lang.reflect.Field
* Field类封装了关于成员变量的操作
* getFields()方法获取的是所有的public的成员变量的信息
* getDeclaredFields获取的是该类自己声明的成员变量的信息 */
//Field[] fs = c.getFields();
Field[] fs = c.getDeclaredFields();
```
![image](http://490.github.io/images/20190310_232638.png)

1.  要获取一个方法就是获取类的信息，获取类的信息首先要获取类的类类型.
2.  获取方法 名称和参数列表来决定  getMethod获取的是public的方法、getDelcaredMethod自己声明的方法

```java
"Method m = c.getMethod(""print"", int.class,int.class);
//方法的反射操作  
//a1.print(10, 20);方法的反射操作是用m对象来进行方法调用 和a1.print调用的效果完全相同
//方法如果没有返回值返回null,有返回值返回具体的返回值
//Object o = m.invoke(a1,new Object[]{10,20});
Object o = m.invoke(a1, 10,20);
```


Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

-   **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
-   **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
-   **Constructor** ：可以用 Constructor 创建新的对象。

**反射的优点：**

- **可扩展性** ：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
- **类浏览器和可视化开发环境** ：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
- **调试器和测试工具** ： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

**反射的缺点：**

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- **性能开销** ：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
- **安全限制** ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
- **内部暴露** ：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。



# Java Web
## Servlet的生命周期与工作原理
Servlet运行在Servlet容器中，其生命周期由容器来管理。Servlet的生命周期通过javax.servlet.Servlet接口中的init()、service()和destroy()方法来表示,Servlet的生命周期包含了下面4个阶段：
1. 加载和实例化
2. 初始化
3. 请求处理
4. 服务终止

![image](http://490.github.io/images/20190311_101001.png)

1.  Web Client 向Servlet容器（Tomcat）发出Http请求
2.  Servlet容器接收Web Client的请求
3.  Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
4.  Servlet容器创建一个HttpResponse对象
5.  Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给HttpServlet 对象。
6.  HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
7.  HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
8.  Servlet容器把HttpServlet的响应结果传给Web Client。


# Java 序列化和反序列化

序列化是指 把 Java 对象字节序列化的过程，就是说将原本保存在 内存 中的对象，以字节序列的形式，保存到 硬盘 (或数据库等) 中。当需要使用时，再 反序列化 恢复到内存中使用。


## 如何实现

只要对象实现了 Serializable 接口，这个对象就可以通过如下方法进行序列化和反序列化 ( **注意：** Serializable 接口仅仅是一个标记接口，里面没有任何方法 )：

*   **序列化：**

```java
// 创建一个 OutputStream 流并将其封装在一个 ObjectOutputStream 对象内。
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("worm.out"));
// 调用 ObjectOutputStream 对象的 writeObject() 方法，即可将对象 wa 序列化。
out.writeObject(wa);
```

*   **反序列化：**

```java
// 创建一个 InputStream 流并将其封装在一个 ObjectInputStream 对象内。
ObjectInputStream in = new ObjectInputStream(new FileInputStream("worm.out"));
// 调用 ObjectInputStream 对象的 readObject() 方法，可获得一个向上转型的 Object 对象引用，然后将获得的 Object 对象向下转型即可。
Worm newWa = (Worm) in.readObject();
```


*   **注意：**
    *   将一个对象从它的序列化状态恢复出来所需要的必要条件：保证 Java JVM 能够找到相关的 .class 文件，否则会抛出 ClassNotFoundException 异常。
    *   被 static 修饰的字段是无法被序列化的，因为它根本就不保存在对象中，而是保存在方法区中，如果想要序列化 static 值，必须自己手动去实现，并手动调用方法，一般会在类中加上 `serializeStaticState(ObjectOutputStream os)` 和 `deserializeStaticState(ObjectInputStream os)` 这两个方法用来序列化 static 字段。

```java
        class Square implements Serializable 
        {
            private static int color = RED;
            public static void serializeStaticState(ObjectOutputStream os) throws IOException {
                os.writeInt(color); // 在这里
            }
            public static void deserializeStaticState(ObjectInputStream os) throws IOException {
                color = os.readInt();
            }
        }

        // 在序列化和反序列化 Square 对象时的时候需要手动调用：
        // 序列化：
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("square.out"));
        Square.serializeStaticState(out); // 在 writeObject 前先把 static 变量的值写道 Output 流中
        out.writeObject(sq);
        // 反序列化：
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("square.out"));
        Square.deserializeStaticState(in); // 在 readObject 前先把 static 变量的值从 Input 流中取出来
        Square newSq = (Square) in.readObject();
```

    *   另外要注意安全问题，序列化也会将 private 数据保存下来，必要的时候可以把敏感数据用 transient 关键字修饰，防止其被序列化。一旦变量被 transient 修饰，变量将不再是对象持久化的一部分，在对象反序列化后，transient 修饰的变量被设为初始值，即 int 型数据的值为 0，对象型数据为 null。

另一种实现序列化和反序列化的方法：实现 ExternalSerializable 接口。

## 序列化的控制：Externalizable 接口

Externalizable 接口继承自 Serializable 接口，同时增加了两个方法：`writeExternal()`和`readExternal()`，这两个方法会在序列化和反序列化还原的过程中被自动调用，我们可以在`writeExternal()`中将来自对象的重要信息写入，然后在`readExternal()`中恢复数据。(默认是不写入任何成员对象的)

*   **对比 transient 关键字：**
    *   Externalizable 接口：选择要进行序列化的字段进行序列化操作。
    *   transient 关键字：选择不要进行序列化的字段取消序列化操作。
*   **对比 Serializable 接口：**
    *   Externalizable 接口：会调用普通的默认构造器，因此必须有 public 的默认构造器，否则会抛出异常。相当于新 new 了一个对象，然后把`writeExternal()`中进行序列化的成员变量进行重新赋值。
    *   Serializable 接口：对象完全以它存储的二进制位为基础来构造，不调用构造器。

## Externalizable 接口的替代方法

Externalizable 接口使用起来较为麻烦，我们可以实现 Serializable 接口，并添加 (是"添加"，既不是"覆盖"也不是"实现") 如下两个方法：

```java
private void writeObject(ObjectOutputStream stream) throws IOException { ... }
private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException { ... }
```

ObjectOutputStream 和 ObjectInputStream 对象的 writeObject() 和 readObject() 方法会调用我们的对象的 writeObject() 和 readObject() 方法。

**既然如此我们为什么不为这两个方法写个接口呢？因为这两个方法是 private 的，而接口中定义的东西都是默认 public 的，所以只能是"添加"这两个方法了。**

在我们的 writeObject() 和 readObject() 方法中，可以调用 defaultWriteObject() 和 defaultReadObject() 方法来选择执行默认的 writeObject() 和 readObject() 方法，比较方便好用。

**注意：** 如果我们打算使用默认机制写入对象的非 transient 部分，那么必须调用 defaultWriteObject() 作为 writeObject() 的第一个操作，调用 defaultReadObject() 作为 readObject() 的第一个操作。

## 常见的序列化协议

*   **XML：** XML是一种常用的序列化和反序列化协议，具有跨机器，跨语言等优点，并且可读性强。
*   **JSON：**
    *   这种 Associative array 格式非常符合工程师对对象的理解。
    *   它保持了XML的人眼可读（Human-readable）的优点。
    *   相对于XML而言，序列化后的数据更加简洁。
    *   它具备 Javascript 的先天性支持，所以被广泛应用于 Web browser 的应用常景中，是 Ajax 的事实标准协议。
    *   与 XML 相比，其协议比较简单，解析速度比较快。
    *   松散的 Associative array 使得其具有良好的可扩展性和兼容性。

# Java 动态代理

动态代理是干啥的？增强对象的！并且是终极加强版的对象增强工具！为啥，请看下表：

| **增强对象方式** | **被增强对象** | **增强内容** |
| --- | --- | --- |
| 继承 | 固定 | 固定 |
| 装饰者模式 | 不固定 | 固定 |
| 动态代理 | 不固定 | 不固定 |

通过上表可以看出，动态代理最为灵活！

现在我们就要实现这个超强的对象加强机器，它就像一个有三个卡槽的对象加强机器一样，给它装配好合适的卡之后，它就能按你给它装配了什么卡生产出相应的增强对象出来，十分的灵活。如下图：

![image](http://490.github.io/images/20190315_200511.png)

之后只要我们创建一个 ProxyFactory factory，然后在里面放好 targetObject (这个是必须有的)，再放上 beforeAdvice 和 afterAdvice (这两个是可有可没有的，要根据对象加强的需要)，最后调用 factory.createProxy() 就能生产出来一个被加强的对象了。

**需求分析：**

我们有一个 Eat 接口，实现了这个接口的类都能吃饭，然后我们我们现在想要对所有实现这个接口的类的 eat() 方法进行加强，在吃之前要说：我开动啦，吃完之后要说：我吃饱啦。

**实现**

先准备 3 个接口和两个实现了 Eat 接口的类：

```java
public interface Eat {
    public void eat();
}
public interface BeforeAdvice {
    public void before();
}
public interface AfterAdvice {
    public void after();
}
public class Man implements Eat {
    @Override
    public void eat() {
        System.out.println("Man在吃饭...");
    }
}
public class Animal implements Eat {
    @Override
    public void eat() {
        System.out.println("Animal在吃饭...");
    }
}
```

然后我们就可以通过动态代理实现这样的一个对象加强器了：

```java
public class DynamicProxyFactory 
{
    private Object targetObject;
    private BeforeAdvice beforeAdvice;
    private AfterAdvice afterAdvice;

    public Object createProxy() 
    {
        // 类加载器比较好得到，用当前类的类加载器就行
        ClassLoader loader = this.getClass().getClassLoader();
        // 要实现的接口也很好得到，就是我们要加强的接口
        Class[] interfaces = targetObject.getClass().getInterfaces();
        // 处理器就是调用代理对象的方法时会执行的内容，除了个别类似 getClass() 这种方法，
        // 其他的方法调用了并不会执行，而是执行 invoke 这个方法
        // 所以我们要是想加强原方法只要让它在 invoke 里执行，再加上需要扩展的部分，就达到了增强的效果
        InvocationHandler handle = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable 
            {
                if (beforeAdvice != null)
                {
                    beforeAdvice.before();
                }
                Object result = method.invoke(targetObject, args);
                if (afterAdvice != null) 
                {
                    afterAdvice.after();
                }
                return result;
            }
        };
        // 得到一个实现了interfaces中接口的类
        // 这个方法需要三个参数：类加载器，要实现的接口，处理器
        return Proxy.newProxyInstance(loader, interfaces, handle);
    }
    public Object getTargetObject() {
        return targetObject;
    }
    public void setTargetObject(Object targetObject) {
        this.targetObject = targetObject;
    }
    public BeforeAdvice getBeforeAdvice() {
        return beforeAdvice;
    }
    public void setBeforeAdvice(BeforeAdvice beforeAdvice) {
        this.beforeAdvice = beforeAdvice;
    }
    public AfterAdvice getAfterAdvice() {
        return afterAdvice;
    }
    public void setAfterAdvice(AfterAdvice afterAdvice) {
        this.afterAdvice = afterAdvice;
    }
}
```

这个 DynamicProxyFactory 的使用方法如下：

```java
// 创建工厂
DynamicProxyFactory factory = new DynamicProxyFactory();
// 设置工厂要进行增强的对象
// factory.setTargetObject(new Man());
factory.setTargetObject(new Animal());
// 设置工厂要在原方法前进行的增强内容
factory.setBeforeAdvice(new BeforeAdvice() {
                        @Override
                        public void before() {
                            System.out.println("我开动啦");
                        }
                    });

// 设置工厂要在原方法后进行的增强内容
factory.setAfterAdvice(new AfterAdvice() {
                      @Override
                      public void after() {
                          System.out.println("我吃饱啦");
                      }
                  });

// 通过createProxy得到增强后的方法
Eat proxyObject = (Eat) factory.createProxy();
proxyObject.eat();
```

