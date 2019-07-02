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

## 数据库驱动为什么用反射

反射我们知道是对一个类的主动使用，会触发类的初始化过程，在jvm定义中，对类加载的前几个步骤在什么情况下执行没有具体规定，但是对初始化过程做了一下规定，凡是主动对一个类的使用，就会触发初始化，既然初始化触发，那么“加载，连接（验证，准备，解析（不一定在这一步）），初始化”肯定都执行了。 
此外，对一个类的初始化，首先会看他的父类有没有初始化，如果没有，还要先进行父类的初始化。 
所谓初始化，就是调用其静态代码块，为静态变量赋值。 来看看`com.mysql.jdbc.Driver`在初始化过程中究竟做了那些事。

```java
static 
{
     try {
          // 放入到一个copyonwritearraylist中
          java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {throw new RuntimeException("Can't register driver!");}
}
```

由此可见，Driver通过静态代码块把自己注册到`DriverManger`中去了，这也是下一步 
`Connection conn = DriverManager.getConnection(url, username, password);` 
能够获取连接的原因，看看具体代码

```java
for(DriverInfo aDriver : registeredDrivers) 
{
      // If the caller does not have permission to load the driver then skip it.
      if(isDriverAllowed(aDriver.driver, callerCL)) 
      {
           try {
                println("    trying " + aDriver.driver.getClass().getName());
                // 通过具体注册的driver的connect方法获取连接
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) 
                {
                    println("getConnection returning"+aDriver.driver.getClass().getName());
                    return (con);
                 }
            } catch (SQLException ex) {
                    if (reason == null) {
                        reason = ex;}}
      } else {
            println("skipping: " + aDriver.getClass().getName());
      }
}

```

本质上是调用了mysql.Driver的connect方法，通过建立到数据库的socket连接，来完成接下来sql的执行。 
JDBC只是jdk提出的一种java连接数据库的规范，提供了一些接口和抽象方法，并没有提供到某个具体数据库的实现，由各个数据库厂家来实现JDBC，比如上面提到的mysql.Driver就是一种具体实现。

**工厂模式**，反射的作用就是，**无论你使用哪种数据库**（数据库类型）只需要把数据库的驱动名称传过来就能穿件对象，而制定类只能创建你制定的数据库对象。

以上对JDBC连接数据库的具体源码做了一个粗略的分析，实际上可以看出来，只要是对`com.mysql.jdbc.Driver `的主动使用都会触发那个注册操作，为什么一定要使用反射呢？因为反射是运行时根据全类名动态生成的Class对象，完全可以把这个全类名写在xml或者properties中去，不仅从代码上解耦和，而且需要更换数据库时，不需要进行代码的重新编译。

## 内省 (Introspector)

Introspector 是操作 javaBean 的 API，用来访问某个属性的 getter/setter 方法。
对于一个标准的 javaBean 来说，它包括属性、get 方法和 set 方法，这是一个约定俗成的规范。为此 sun 提供了 Introspector 工具包，来使开发者更好或者更灵活的操作 javaBean。

核心类是 `Introspector`, 它提供了的 `getBeanInfo` 系类方法，可以拿到一个 JavaBean 的所有信息。

通过 `BeanInfo` 的 `getPropertyDescriptors` 方法和 `getMethodDescriptors` 方法可以拿到 javaBean 的字段信息列表和 getter 和 setter 方法信息列表。

`PropertyDescriptors` 可以根据字段直接获得该字段的 getter 和 setter 方法。

`MethodDescriptors` 可以获得方法的元信息，比如方法名，参数个数，参数字段类型等。


```java
public class User 
{
    private String name;
    private int age;
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

```
    @Test
    public void test1() throws Exception 
    {
        // 获取整个Bean的信息
        // BeanInfo beanInfo= Introspector.getBeanInfo(user.getClass());
        // 在Object类时候停止检索，可以选择在任意一个父类停止
        BeanInfo beanInfo = Introspector.getBeanInfo(User.class, Object.class);
        // 获取所有的属性描述
        PropertyDescriptor[] pds = beanInfo.getPropertyDescriptors();
        for (PropertyDescriptor propertyDescriptor : pds) 
        {
            System.out.println(propertyDescriptor.getName());
        }
        for (MethodDescriptor methodDescriptor : beanInfo.getMethodDescriptors()) 
        {
            System.out.println(methodDescriptor.getName());
            // Method method = methodDescriptor.getMethod();
        }
    }
```


```
    @Test
    public void test2 () throws Exception 
    {
        User user = new User("jack", 21);
        String propertyName = "name";
        PropertyDescriptor namePd = new PropertyDescriptor(propertyName, User.class);
        System.out.println("名字：" + namePd.getReadMethod().invoke(user));
        namePd.getWriteMethod().invoke(user, "tom");
        System.out.println("名字：" + namePd.getReadMethod().invoke(user));
        System.out.println("========================================");
        String agePropertyName = "age";
        PropertyDescriptor agePd = new PropertyDescriptor(agePropertyName, User.class);
        System.out.println("年龄：" + agePd.getReadMethod().invoke(user));
        agePd.getWriteMethod().invoke(user, 22);
        System.out.println("年龄：" + agePd.getReadMethod().invoke(user));
    }

名字：jack
名字：tom
========================================
年龄：21
年龄：22
```







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


## Tomcat

 [解析Tomcat内部结构和请求过程](https://www.cnblogs.com/zhouyuqin/p/5143121.html)

### 结构

Tomcat是一个JSP/Servlet容器。其作为Servlet容器，有三种工作模式：独立的Servlet容器、进程内的Servlet容器和进程外的Servlet容器。

Tomcat是一个基于组件的服务器，它的构成组件都是可配置的，其中最外层的是Catalina servlet容器，其他组件按照一定的格式要求配置在这个顶层容器中。 Tomcat的各种组件都是在Tomcat安装目录下的/conf/server.xml文件中配置的。

```xml
<Server>    //顶层类元素，可以包括多个Service   
    <Service> //顶层类元素，可包含一个Engine，多个Connecter
        <Connector>  //连接器类元素，代表通信接口
                <Engine>  //容器类元素，为特定的Service组件处理客户请求，要包含多个Host
                        <Host> //容器类元素，为特定的虚拟主机组件处理客户请求，可包含多个Context
                                <Context> //容器类元素，为特定的Web应用处理所有的客户请求
                                </Context>
                        </Host>
                </Engine>
        </Connector>
    </Service>
</Server>
```

![image](http://490.github.io/images/20190331_134940.png)

由上图可看出Tomca的心脏是两个组件：Connecter和Container。一个Container可以选择多个Connecter，多个Connector和一个Container就形成了一个Service。Service可以对外提供服务，而Server服务器控制整个Tomcat的生命周期。

Service 和 Server 管理它下面组件的生命周期。 
Tomcat 中组件的生命周期是通过 Lifecycle 接口来控制的，组件只要继承这个接口并实现其中的方法就可以统一被拥有它的组件控制了，这样一层一层的直到一个最高级的组件就可以控制 Tomcat 中所有组件的生命周期，这个最高的组件就是 Server，而控制 Server 的是 Startup，也就是您启动和关闭 Tomcat。

### Tomca的两大组件：Connecter和Container

 **Connecter组件**

一个Connecter将在某个指定的端口上侦听客户请求，接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 对象传给处理Engine(Container中的一部分)，从Engine出获得响应并返回客户。 
Tomcat中有两个经典的Connector，一个直接侦听来自Browser的HTTP请求，另外一个来自其他的WebServer请求。Cotote HTTP/1.1 Connector在端口8080处侦听来自客户Browser的HTTP请求，Coyote JK2 Connector在端口8009处侦听其他Web Server的Servlet/JSP请求。 
Connector 最重要的功能就是接收连接请求然后分配线程让 Container 来处理这个请求，所以这必然是多线程的，多线程的处理是 Connector 设计的核心。

**Container组件**

Container的体系结构如下：

![image](http://490.github.io/images/20190331_135144.png)

Container是容器的父接口，该容器的设计用的是典型的责任链的设计模式，它由四个自容器组件构成，分别是Engine、Host、Context、Wrapper。这四个组件是负责关系，存在包含关系。通常一个Servlet class对应一个Wrapper，如果有多个Servlet定义多个Wrapper，如果有多个Wrapper就要定义一个更高的Container，如Context。 
Context 还可以定义在父容器 Host 中，Host 不是必须的，但是要运行 war 程序，就必须要 Host，因为 war 中必有 web.xml 文件，这个文件的解析就需要 Host 了，如果要有多个 Host 就要定义一个 top 容器 Engine 了。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。

*   Engine 容器 
    Engine 容器比较简单，它只定义了一些基本的关联关系
*   Host 容器 
    Host 是 Engine 的字容器，一个 Host 在 Engine 中代表一个虚拟主机，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。它的子容器通常是 Context，它除了关联子容器外，还有就是保存一个主机应该有的信息。
*   Context 容器 
    Context 代表 Servlet 的 Context，它具备了 Servlet 运行的基本环境，理论上只要有 Context 就能运行 Servlet 了。简单的 Tomcat 可以没有 Engine 和 Host。Context 最重要的功能就是管理它里面的 Servlet 实例，Servlet 实例在 Context 中是以 Wrapper 出现的，还有一点就是 Context 如何才能找到正确的 Servlet 来执行它呢？ Tomcat5 以前是通过一个 Mapper 类来管理的，Tomcat5 以后这个功能被移到了 request 中，在前面的时序图中就可以发现获取子容器都是通过 request 来分配的。
*   Wrapper 容器 
    Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper 是最底层的容器，它没有子容器了，所以调用它的 addChild 将会报错。 
    Wrapper 的实现类是 StandardWrapper，StandardWrapper 还实现了拥有一个 Servlet 初始化信息的 ServletConfig，由此看出 StandardWrapper 将直接和 Servlet 的各种信息打交道。

Tomcat 还有其它重要的组件，如安全组件 security、logger 日志组件、session、mbeans、naming 等其它组件。这些组件共同为 Connector 和 Container 提供必要的服务。


### Tomcat Server处理一个HTTP请求的过程

![image](http://490.github.io/images/20190331_135019.png)

1、用户点击网页内容，请求被发送到本机端口8080，被在那里监听的Coyote HTTP/1.1 Connector获得。 
2、Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应。 
3、Engine获得请求localhost/test/index.jsp，匹配所有的虚拟主机Host。 
4、Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为localhost的Host获得请求/test/index.jsp，匹配它所拥有的所有的Context。Host匹配到路径为/test的Context（如果匹配不到就把该请求交给路径名为“ ”的Context去处理）。 
5、path=“/test”的Context获得请求/index.jsp，在它的mapping table中寻找出对应的Servlet。Context匹配到URL PATTERN为*.jsp的Servlet,对应于JspServlet类。 
6、构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet（）或doPost（）.执行业务逻辑、数据存储等程序。 
7、Context把执行完之后的HttpServletResponse对象返回给Host。 
8、Host把HttpServletResponse对象返回给Engine。 
9、Engine把HttpServletResponse对象返回Connector。 
10、Connector把HttpServletResponse对象返回给客户Browser。


# Java 序列化和反序列化
[补充阅读](java的io#java对象操作)

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

另外要注意安全问题，序列化也会将 private 数据保存下来，必要的时候可以把敏感数据用 transient 关键字修饰，防止其被序列化。一旦变量被 transient 修饰，变量将不再是对象持久化的一部分，在对象反序列化后，transient 修饰的变量被设为初始值，即 int 型数据的值为 0，对象型数据为 null。

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

## 代理模式（JDK代理）

[设计模式](设计模式#代理模式)

 **  JDK动态代理所用到的代理类在程序调用到代理类对象时才由JVM真正创建，JVM根据传进来的 业务实现类对象 以及 方法名 ，动态地创建了一个代理类的class文件并被字节码引擎执行，然后通过该代理类对象进行方法调用。**我们需要做的，只需指定代理类的预处理、调用后操作即可。

1：首先，定义业务逻辑接口
```java
public interface BookFacade 
{ 
    public void addBook();  
} 
```
2：然后，实现业务逻辑接口创建业务实现类

```java
public class BookFacadeImpl implements BookFacade 
{   
    @Override  
    public void addBook() 
    {  
        System.out.println("增加图书方法。。。");  
    }  
}
```
3：最后，实现 调用管理接口InvocationHandler  创建动态代理类

```java
public class BookFacadeProxy implements InvocationHandler 
{  
    private Object target;//这其实业务实现类对象，用来调用具体的业务方法 
    /** 
     * 绑定业务对象并返回一个代理类  
     */  
    public Object bind(Object target) 
    {  
        this.target = target;  //接收业务实现类对象参数
       //通过反射机制，创建一个代理类对象实例并返回。用户进行方法调用时使用
       //创建代理对象时，需要传递该业务类的类加载器（用来获取业务实现类的元数据，在包装方法是调用真正的业务方法）、接口、handler实现类
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),  
                target.getClass().getInterfaces(), this); 
    }  
    /** 
     * 包装调用方法：进行预处理、调用后处理 
     */  
    public Object invoke(Object proxy, Method method, Object[] args)  throws Throwable 
    {  
        Object result=null;  
        System.out.println("预处理操作——————");  
        //调用真正的业务方法  
        result=method.invoke(target, args);  
        System.out.println("调用后处理——————");  
        return result;  
    }  
}
```


 4：在使用时，首先创建一个业务实现类对象和一个代理类对象，然后定义接口引用（这里使用向上转型）并用代理对象.bind(业务实现类对象)的返回值进行赋值。最后[通过接口引用对象](#通过接口引用对象)调用业务方法即可。（接口引用真正指向的是一个绑定了业务类的代理类对象，所以通过接口方法名调用的是被代理的方法们）


```java
public static void main(String[] args) 
{  
        BookFacadeImpl bookFacadeImpl=new BookFacadeImpl();
        BookFacadeProxy proxy = new BookFacadeProxy();  
        BookFacade bookfacade = (BookFacade) proxy.bind(bookFacadeImpl);  
        bookfacade.addBook();  
}
```

   JDK动态代理的代理对象在创建时，需要使用业务实现类所实现的接口作为参数（因为在后面代理方法时需要根据接口内的方法名进行调用）。如果业务实现类是没有实现接口而是直接定义业务方法的话，就无法使用JDK动态代理了。并且，如果业务实现类中新增了接口中没有的方法，这些方法是无法被代理的（因为无法被调用）。





## CGLIB代理

CGLIB（Code Generator Library）是一个强大的、高性能的代码生成库。其被广泛应用于AOP框架（Spring、dynaop）中，用以提供方法拦截操作。

** cglib是针对类来实现代理的，原理是对指定的业务类生成一个子类，并覆盖其中业务方法实现代理。因为采用的是继承，所以不能对final修饰的类进行代理。** 

1：首先定义业务类，无需实现接口（当然，实现接口也可以，不影响的）

```java
public class BookFacadeImpl1 
{ 
    public void addBook() 
    {  
        System.out.println("新增图书...");  
    }  
}
```
 2：实现 MethodInterceptor方法代理接口，创建代理类

```java
public class BookFacadeCglib implements MethodInterceptor 
{  
    private Object target;//业务类对象，供代理方法中进行真正的业务方法调用
    //相当于JDK动态代理中的绑定
    public Object getInstance(Object target) 
    {  
        this.target = target;  //给业务对象赋值
        Enhancer enhancer = new Enhancer(); //创建加强器，用来创建动态代理类
        enhancer.setSuperclass(this.target.getClass());  //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this); 
       // 创建动态代理类对象并返回  
       return enhancer.create(); 
    }
    // 实现回调方法 
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable 
    { 
        System.out.println("预处理——————");
        proxy.invokeSuper(obj, args); //调用业务类（父类中）的方法
        System.out.println("调用后操作——————");
        return null; 
    }
```

  3：创建业务类和代理类对象，然后通过  代理类对象.getInstance(业务类对象)  返回一个动态代理类对象（它是业务类的子类，可以用业务类引用指向它）。最后通过动态代理类对象进行方法调用。

```java
public static void main(String[] args) 
{      
        BookFacadeImpl1 bookFacade=new BookFacadeImpl1()；
        BookFacadeCglib  cglib=new BookFacadeCglib();  
        BookFacadeImpl1 bookCglib=(BookFacadeImpl1)cglib.getInstance(bookFacade);  
        bookCglib.addBook();  
}
```

 静态代理是通过在代码中显式定义一个业务实现类一个代理，在代理类中对同名的业务方法进行包装，用户通过代理类调用被包装过的业务方法；

    JDK动态代理是通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法；

    CGlib动态代理是通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法进行代理；


# Java注解

## 深入理解JAVA注解

　　要深入学习注解，我们就必须能定义自己的注解，并使用注解，在定义自己的注解之前，我们就必须要了解Java为我们提供的元注解和相关定义注解的语法。

### 元注解（meta-annotation）：

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：
　　　　1.@Target,
　　　　2.@Retention,
　　　　3.@Documented,
　　　　4.@Inherited
　　这些类型和它们所支持的类在java.lang.annotation包中可以找到。下面我们看一下每个元注解的作用和相应分参数的使用说明。

**@Target：**

@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

　**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**
　**　取值(ElementType)有：**

　　　　1.CONSTRUCTOR:用于描述构造器
　　　　2.FIELD:用于描述域
　　　　3.LOCAL_VARIABLE:用于描述局部变量
　　　　4.METHOD:用于描述方法
　　　　5.PACKAGE:用于描述包
　　　　6.PARAMETER:用于描述参数
　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

　　使用实例：

```java
@Target(ElementType.TYPE)
public @interface Table 
{
    /**
     * 数据表名称注解，默认值为类名称
     * @return
     */
    public String tableName() default "className";
}
@Target(ElementType.FIELD)
public @interface NoDBColumn {
}
```


注解Table 可以用于注解类、接口(包括注解类型) 或enum声明,而注解NoDBColumn仅可用于注解类的成员变量。

**@Retention：**

　**　@Retention**定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

　　**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

　**　取值（RetentionPoicy）有：**

　　　　1.SOURCE:在源文件中有效（即源文件保留）
　　　　2.CLASS:在class文件中有效（即class保留）
　　　　3.RUNTIME:在运行时有效（即运行时保留）

Retention meta-annotation类型有唯一的value作为成员，它的取值来自java.lang.annotation.RetentionPolicy的枚举类型值。具体实例如下：

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column 
{
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField"; 
    public boolean defaultDBValue() default false;
}
```


Column注解的的RetentionPolicy的属性值是RUTIME,这样注解处理器可以通过反射，获取到该注解的属性值，从而去做一些运行时的逻辑处理

**@Documented:**

**@Documented**用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Column 
{
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField"; 
    public boolean defaultDBValue() default false;
}
```

**@Inherited：**

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

实例代码：

```java
@Inherited
public @interface Greeting 
{
    public enum FontColor{ BULE,RED,GREEN};
    String name();
    FontColor fontColor() default FontColor.GREEN;
}
```

### 自定义注解：

　　使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。

**定义注解格式：**
　　public @interface 注解名 {定义体}

**注解参数的可支持数据类型：**

　　　　1.所有基本数据类型（int,float,boolean,byte,double,char,long,short)
　　　　2.String类型
　　　　3.Class类型
　　　　4.enum类型
　　　　5.Annotation类型
　　　　6.以上所有类型的数组

　　Annotation类型里面的参数该怎么设定: 
- 第一,只能用public或默认(default)这两个访问权修饰.例如,String value();这里把方法设为defaul默认类型；　 　
- 第二,参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和 String,Enum,Class,annotations等数据类型,以及这一些类型的数组.例如,String value();这里的参数成员就为String;　　
- 第三,如果只有一个参数成员,最好把参数名称设为"value",后加小括号.例:下面的例子FruitName注解就只有一个参数成员。

　　简单的自定义注解和使用注解实例：


```java
package annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 水果名称注解
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName 
{
    String value() default "";
}

/**
 * 水果颜色注解
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor 
{
    /**
     * 颜色枚举
     */
    public enum Color{ BULE,RED,GREEN};
    /**
     * 颜色属性
     * @return
     */
    Color fruitColor() default Color.GREEN;
}
```

```java
package annotation;
import annotation.FruitColor.Color;
public class Apple 
{
    @FruitName("Apple")
    private String appleName;
    @FruitColor(fruitColor=Color.RED)
    private String appleColor;
    public void setAppleColor(String appleColor) 
    {
        this.appleColor = appleColor;
    }
    public String getAppleColor() 
    {
        return appleColor;
    }
    public void setAppleName(String appleName) 
    {
        this.appleName = appleName;
    }
    public String getAppleName() 
    {
        return appleName;
    }
    public void displayName()
    {
        System.out.println("水果的名字是：苹果");
    }
}
```


**注解元素的默认值：**

注解元素必须有确定的值，要么在定义注解的默认值中指定，要么在使用注解时指定，非基本类型的注解元素的值不可为null。因此, 使用空字符串或0作为默认值是一种常用的做法。这个约束使得处理器很难表现一个元素的存在或缺失的状态，因为每个注解的声明中，所有元素都存在，并且都具有相应的值，为了绕开这个约束，我们只能定义一些特殊的值，例如空字符串或者负数，一次表示某个元素不存在，在定义注解时，这已经成为一个习惯用法。例如：


```java
package annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
/**
 * 水果供应者注解
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider 
{
    /**
     * 供应商编号
     * @return
     */
    public int id() default -1;
    /**
     * 供应商名称
     * @return
     */
    public String name() default "";
    /**
     * 供应商地址
     * @return
     */
    public String address() default "";
}
```
定义了注解，并在需要的时候给相关类，类属性加上注解信息，如果没有响应的注解信息处理流程，注解可以说是没有实用价值。如何让注解真真的发挥作用，主要就在于注解处理方法，下一步我们将学习注解信息的获取和处理！

## 注解的使用

 第一步：新建一个annotation，名字为：MyAnnotation.java。

```java
package com.dragon.test.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation
{
    String hello () default "hello";
    String world();
}
```

第二步：建立一个MyTest.java 来使用上面的annotation。

```java
package com.dragon.test.annotation;
public class MyTest
{
    @MyAnnotation(hello = "Hello,Beijing",world = "Hello,world")
    public void output() {
        System.out.println("method output is running ");
    }
}
```

第三步：用反射机制来调用注解中的内容

```java
package com.dragon.test.annotation;
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
public class MyReflection
{
    public static void main(String[] args) throws Exception
    {
        // 获得要调用的类
        Class<MyTest> myTestClass = MyTest.class;
        // 获得要调用的方法，output是要调用的方法名字，new Class[]{}为所需要的参数。空则不是这种
        Method method = myTestClass.getMethod("output", new Class[]{});
        // 是否有类型为MyAnnotation的注解
        if (method.isAnnotationPresent(MyAnnotation.class))
        {
            // 获得注解
            MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
            // 调用注解的内容
            System.out.println(annotation.hello());
            System.out.println(annotation.world());
        }
        System.out.println("----------------------------------");
        // 获得所有注解。必须是runtime类型的
        Annotation[] annotations = method.getAnnotations();
        for (Annotation annotation : annotations)
        {
            // 遍历所有注解的名字
            System.out.println(annotation.annotationType().getName());
        }
    }
}

Hello,Beijing
Hello,world
----------------------------------
com.dragon.test.annotation.MyAnnotation
```


# 通过接口引用对象


对于参数类型，要优先使用接口而不是类。通俗地讲，应该优先使用接口而不是类来引用对象。如果有合适的接口类型存在，那么对于参数、返回值、变量和域来说，就应该使用接口类型来声明。只有当你利用构造函数创建某个对象的时候，才真正引用这个对象的类。

考虑Vector的情形，它是List接口的一个实现，在声明变量的时候应该养成这样的习惯：

```
//Good - use interface as type
List<？> list= new Vector<？>();
```

而不是像这样的声明：

```
//Bad - use class as type
Vector<?> list= new Vector<?>();
```

优点：
1. 假如一个类实现了多个接口,那么用接口类型来定义它的引用变量的话,一眼就可以明白,这里是需要这个类的哪些方法。
2. 程序更加灵活。当你决定更换实现时，只需要改变构造器中类的名称。其他使用list地方的代码根本不需要改动。第一个声明可以被改变为：
```
List<?> list= new ArrayList<?>();
```


注意：
list只能使用ArrayList已经实现了的List接口中的方法，ArrayList中那些自己的、没有在List接口中定义的方法是不可以被访问到的。List.add其实是List接口的方法，但是调用ArrayList方法如clone（）方法是调用不到的。

适合于用类来引用对象的情形：
1. 如果没有合适的接口存在，可以用类来引用对象。
例如，考虑值类（String、BigInteger）很少用多个实现编写，他们通常是final的，并且很少有对应的接口。使用这种值类作为参数、变量、域或者返回值类型就比较合适。
2. 对象属于一个框架，而框架的基本类型是类，不是接口。（对象属于基于类的框架）
例如java.util.TimerTask抽象类。应该用相关的基类（往往是抽象类）来引用对象，而不是它的实现类。
3. 类实现了接口，但是它提供了接口中不存在的额外方法。
例如LinkedHashMap，程序依赖于这些额外的方法，这种类就应该只被用来引用它的实例。

以上这些例子并不全面，而只是代表了一些“适合于用类来引用对象”的情形

总结：给定的对象是否具有适当的接口应该是很明显的。如果是，用接口引用对象就会使程序更加灵活；如果不是，则使用类层次结构中提供了必要功能的最基础的类。

