---
title: Java虚拟机的执行机制
date: 2019-03-10 23:17:20
tags: [Java,JVM]
---
# 类文件结构

Class 文件是一组以 8 位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在 Class 文件中，中间没有任何分隔符。Java 虚拟机规范规定 Class 文件采用一种类似 C 语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：无符号数和表，我们之后也主要对这两种类型的数据类型进行解析。
<!--more-->
- **无符号数：** 无符号数属于基本数据类型，以 u1、u2、u4、u8 分别代表 1 个字节、2 个字节、4 个字节和 8 个字节的无符号数，可以用它来描述数字、索引引用、数量值或 utf-8 编码的字符串值。
- **表：** 表是由多个无符号数或其他表为数据项构成的复合数据类型，名称上都以 `_info` 结尾。

![image](http://490.github.io/images/20190314_085611.png)

## 魔数与版本号

Class 文件的头 8 个字节是魔数和版本号，其中头 4 个字节是魔数，也就是 `0xCAFEBABE`，它可以用来确定这个文件是否为一个能被虚拟机接受的 Class 文件（这通过扩展名来识别文件类型要安全，毕竟扩展名是可以随便修改的）。

后 4 个字节则是当前 Class 文件的版本号，其中第 5、6 个字节是次版本号，第 7、8 个字节是主版本号。


## 常量池

![image](http://490.github.io/images/20190314_085651.png)

![image](http://490.github.io/images/20190314_085746.png)

从第 9 个字节开始，就是常量池的入口，常量池是 Class 文件中：

- 与其他项目关联最多的的数据类型；
- 占用 Class 文件空间最大的数据项目；
- Class 文件中第一个出现的表类型数据项目。

常量池的开始的两个字节，也就是第 9、10 个字节，放置一个 u2 类型的数据，标识常量池中常量的数量 cpc (constant_pool_count)，这个计数值有一个十分特殊的地方，就是它是从 1 开始而不是从 0 开始的，也就是说如果 cpc = 22，那么代表常量池中有 21 项常量，索引值为 1 ~ 21，第 0 项常量被空出来，为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”时，将让这个索引值指向 0 即可。

常量池中记录的是代码出现过的所有 token（类名，成员变量名等，也是我们接下来要修改的地方）以及符号引用（方法引用，成员变量引用等），主要包括以下两大类常量：

- **字面量：** 接近于 Java 语言层面的常量概念，包括
	- 文本字符串
	- 声明为 final 的常量值
- **符号引用：** 以一组符号来描述所引用的目标，包括
	- 类和接口的全限定名
	- 字段的名称和描述符
	- 方法的名称和描述符

常量池中的每一项常量都通过一个表来存储。目前一共有 14 种常量，不过麻烦的地方就在于，这 14 种常量类型每一种都有自己的结构，我们在这里只详细介绍两种：CONSTANT_Class_info 和 CONSTANT_Utf8_info。

CONSTANT_Class_info 的存储结构为：

```java
... [ tag=7 ] [ name_index ] ...
... [  1位  ] [     2位    ] ...
```

其中，tag 是标志位，用来区分常量类型的，tag = 7 就表示接下来的这个表是一个 CONSTANT_Class_info，name_index 是一个索引值，指向常量池中的一个 CONSTANT_Utf8_info 类型的常量所在的索引值，CONSTANT_Utf8_info 类型常量一般被用来描述类的全限定名、方法名和字段名。它的存储结构如下：

```java
... [ tag=1 ] [ 当前常量的长度 len ] [ 常量的符号引用的字符串值 ] ...
... [  1位  ] [        2位        ] [         len位         ] ...
```

## 访问标志

 常量池之后的两个字节代表访问标志，即这个class是类还是接口，是否为public等的信息。不同的含义有不同的标志值（没有用到的标志位一律为0。），具体信息如下：
 
 ![image](http://490.github.io/images/20190314_085909.png)

## 类索引、父类索引、接口索引集合

* 类索引占两个字节，分别指向常量池中的CONSTANT_Class_info类型的常量，这个类型的常量结构见常量池中的图表，其中包含一个指向全限定名常量项的索引。
* 因为java只允许单继承，所以只有一个父类，具体内容同上-类索引。
- 接口索引开始两个字节用来表示接口的数量，之后的每两个字节表示一个接口索引，用法同类索引与父类索引。

## 字段表集合

字段用于描述接口或者类中声明的变量，包括类级变量以及实例变量，但不包括局部变量。

字段域的开始两个字节表示字段数量，之后为紧密排列的字段结构体数据，其结构如下：

![image](http://490.github.io/images/20190314_090048.png)

其中的字段和方法的描述符，对于字段来说用来描述**字段的数据类型**；而对于方法来说，描述的就是**方法的参数列表（包括数量、类型以及顺序）和返回值**，这个描述顺序也是固定的，必须是参数列表在前，返回值在后，参数列表必须放在一组小括号内。同时为了节省空间，各种数据类型都使用规定的一个字母来表示，具体如下：

![image](http://490.github.io/images/20190314_090101.png)

对象使用L加上对象的全限定名来表示，而数组则是在每一个维度前添加一个`"["`来描述。属性表在之后进行介绍。

## 方法表集合

class文件中对方法的描述与以前对字段的描述几乎采用了完全一致的方式，唯一的区别就是访问类型不完全一致。


## 属性表集合

java7中预定义了21项属性，具体内容限于篇幅不再列出。对于每个属性的结构，没有特别严格的要求，并且可以自定义属性信息，jvm运行时会忽略不认识的属性。符合规范的属性表基本结构如下：

![image](http://490.github.io/images/20190314_090227.png)

其中前两个字节为指向常量池中的CONSTANT_Utf8_info类型的属性名称，之后4个字节表示属性值所占用的位数，最后就是具体属性了。 其中有一个比较重要的名称为「Code」的属性为方法的代码，即字节码指令。Code属性表结构如下：

![image](http://490.github.io/images/20190314_090246.png)



# 虚拟机的类加载机制


## 类加载的时机

JVM 会在程序第一次主动引用类的时候，加载该类，被动引用时并不会引发类加载的操作。也就是说，JVM 并不是在一开始就把一个程序就所有的类都加载到内存中，而是到不得不用的时候才把它加载进来，而且只加载一次。那么什么是主动引用，什么是被动引用呢？

- **主动引用**
	- 遇到 new、getstatic、putstatic、invokestatic 字节码指令，例如：
		- 使用 new 实例化对象；
		- 读取或设置一个类的 static 字段（被 final 修饰的除外）；
		- 调用类的静态方法。
	- 对类进行反射调用；
	- 初始化一个类时，其父类还没初始化（需先初始化父类）；
		- 这点类与接口具有不同的表现，接口初始化时，不要求其父接口完成初始化，只有真正使用父接口时才初始化，如引用父接口中定义的常量。
	- 虚拟机启动，先初始化包含 main() 函数的主类；
	- JDK 1.7 动态语言支持：一个 java.lang.invoke.MethodHandle 的解析结果为 REF_getStatic、REF_putStatic、REF_invokeStatic。
- **被动引用**
	- 通过子类引用父类静态字段，不会导致子类初始化；
	- `Array[] arr = new Array[10];` 不会触发 Array 类初始化；
	- `static final VAR` 在编译阶段会存入调用类的常量池，通过 `ClassName.VAR` 引用不会触发 ClassName 初始化。

也就是说，只有发生主动引用所列出的 5 种情况，一个类才会被加载到内存中，也就是说类的加载是 lazy-load 的，不到必要时刻是不会提前加载的，毕竟如果将程序运行中永远用不到的类加载进内存，会占用方法区中的内存，浪费系统资源。



## 类的显式加载和隐式加载

- **显示加载：**
	- 调用 `ClassLoader#loadClass(className)` 或 `Class.forName(className)`。
	-  两种显示加载 .class 文件的区别：
		- `Class.forName(className)` 加载 class 的同时会初始化静态域，`ClassLoader#loadClass(className)` 不会初始化静态域；
		- Class.forName 借助当前调用者的 class 的 ClassLoader 完成 class 的加载。
- **隐式加载：**
	- new 类对象；
	- 使用类的静态域；
	- 创建子类对象；
	- 使用子类的静态域；
	- 其他的隐式加载，在 JVM 启动时：
		- BootStrapLoader 会加载一些 JVM 自身运行所需的 Class；
		- ExtClassLoader 会加载指定目录下一些特殊的 Class；
		- AppClassLoader 会加载 classpath 路径下的 Class，以及 main 函数所在的类的 Class 文件。



## 类加载的过程

### 类的生命周期

```
加载 --> 验证 --> 准备 --> 解析 --> 初始化 --> 使用 --> 卸载
       |<------- 连接 ------->|
|<------------- 类加载 ---------------->|
```

类的生命周期一共有 7 个阶段，其中前五个阶段较为重要，统称为类加载，第 2 ~ 4 阶段统称为连接，加载和连接中的三个过程开始的顺序是固定的，但是执行过程中是可以交叉执行的。接下来，我们将对类加载的 5 个阶段进行一一讲解。

### 加载

#### 加载的 3 个阶段

- 通过类的全限定名获取二进制字节流（将 .class 文件读进内存）；
- 将字节流的静态存储结构转化为运行时的数据结构；
- 在内存中生成该类的 Class 对象；
	- HotSpot 虚拟机把这个对象放在方法区，非 Java 堆。

#### 分类

- **非数组类**
	- 系统提供的引导类加载器
	- 用户自定义的类加载器
- **数组类**
	- 不通过类加载器，由 Java 虚拟机直接创建
	- 创建动作由 newarray 指令触发，new 实际上触发了 `[L全类名` 对象的初始化
	- 规则
		- 数组元素是引用类型
			- 加载：递归加载其组件
			- 可见性：与引用类型一致
		- 数组元素是非引用类型
			- 加载：与引导类加载器关联
			- 可见性：public

### 验证

- **目的：** 确保 .class 文件中的字节流信息符合虚拟机的要求。
- **4 个验证过程：**
	- 文件格式验证：是否符合 Class 文件格式规范，验证文件开头 4 个字节是不是 “魔数” `0xCAFEBABE`
	- 元数据验证：保证字节码描述信息符号 Java 规范（语义分析）
	- 字节码验证：程序语义、逻辑是否正确（通过数据流、控制流分析）
	- 符号引用验证：对类自身以外的信息（常量池中的符号引用）进行匹配性校验
- 这个操作虽然重要，但不是必要的，可以通过 `-Xverify:none` 关掉。

### 准备

- **描述：** 为 static 变量在方法区分配内存。
- static 变量准备后的初始值：
	- `public static int value = 123;`
		- 准备后为 0，value 的赋值指令 putstatic 会被放在 `<client>()` 方法中，`<client>()`方法会在初始化时执行，也就是说，value 变量只有在初始化后才等于 123。
	- `public static final int value = 123;`
		- 准备后为 123，因为被 `static final` 赋值之后 value 就不能再修改了，所以在这里进行了赋值之后，之后不可能再出现赋值操作，所以可以直接在准备阶段就把 value 的值初始化好。

### 解析

- **描述：** 将常量池中的 “符号引用” 替换为 “直接引用”。
	- 在此之前，常量池中的引用是不一定存在的，解析过之后，可以保证常量池中的引用在内存中一定存在。
	- 什么是 “符号引用” 和 “直接引用” ？
		- 符号引用：以一组符号描述所引用的对象（如对象的全类名），引用的目标不一定存在于内存中。
		- 直接引用：直接指向被引用目标在内存中的位置的指针等，也就是说，引用的目标一定存在于内存中。

### 初始化

- **描述：** 执行类构造器 `<client>()` 方法的过程。
- **`<client>()` 方法**
	- 包含的内容：
		- 所有 static 的赋值操作；
		- static 块中的语句；
	- `<client>()` 方法中的语句顺序：
		- 基本按照语句在源文件中出现的顺序排列；
		- 静态语句块只能访问定义在它前面的变量，定义在它后面的变量，可以赋值，但不能访问。
	- 与 `<init>()` 的不同：
		- 不需要显示调用父类的 `<client>()` 方法；
		- 虚拟机保证在子类的 `<client>()` 方法执行前，父类的 `<client>()` 方法一定执行完毕。
			- 也就是说，父类的 static 块和 static 字段的赋值操作是要先于子类的。
	- 接口与类的不同：
		- 执行子接口的 `<client>()` 方法前不需要先执行父接口的 `<client>()` 方法（除非用到了父接口中定义的 public static final 变量）；
	- 执行过程中加锁：
		- 同一时刻只能有一个线程在执行 `<client>()` 方法，因为虚拟机要保证在同一个类加载器下，一个类只被加载一次。
	- 非必要性：
		- 一个类如果没有任何 static 的内容就不需要执行 `<client>()` 方法。

*注：初始化时，才真正开始执行类中定义的 Java 代码。*



## 类加载器

### 如何判断两个类 “相等”

- **“相等” 的要求**
	- 同一个 .class 文件
	- 被同一个虚拟机加载
	- 被同一个类加载器加载
- **判断 “相等” 的方法**
	- `instanceof` 关键字
	- Class 对象中的方法：
		- `equals()`
		- `isInstance()`
		- `isAssignableFrom()`

### 类加载器的分类

- **启动类加载器（Bootstrap）**：Bootstrp加载器是用C++语言写的，它是在Java虚拟机启动后初始化的，它主要负责加载`%JAVA_HOME%/jre/lib`,`-Xbootclasspath`参数指定的路径以及`%JAVA_HOME%/jre/classes`中的类。

- **扩展类加载器（Extension）**：`<JAVA_HOME>/lib/ext`、`java.ext.dirs `系统变量指定的路径
- **应用程序类加载器（Application）**： `-classpath` 参数

### 双亲委派模型

- **工作过程**
	- 当前类加载器收到类加载的请求后，先不自己尝试加载类，而是先将请求委派给父类加载器
		- 因此，所有的类加载请求，都会先被传送到启动类加载器
	- 只有当父类加载器加载失败时，当前类加载器才会尝试自己去自己负责的区域加载
- **实现**
	- 检查该类是否已经被加载
	- 将类加载请求委派给父类
		- 如果父类加载器为 null，默认使用启动类加载器
		- `parent.loadClass(name, false)`
	- 当父类加载器加载失败时
		- catch ClassNotFoundException 但不做任何处理
		- 调用自己的 findClass() 去加载
			- 我们在实现自己的类加载器时只需要 `extends ClassLoader`，然后重写 `findClass()` 方法而不是 `loadClass()` 方法，这样就不用重写 `loadClass()` 中的双亲委派机制了
- **优点**
	- Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。
	- 其次是考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改。
   ![image](http://490.github.io/images/20190311_083801.png)

## 破坏双亲委派

### 为什么需要破坏双亲委派？

因为在某些情况下父类加载器需要委托子类加载器去加载class文件。受到加载范围的限制，父类加载器无法加载到需要的文件，以Driver接口为例，由于Driver接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如mysql的就写了`MySQL Connector`，那么问题就来了，DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况。

### 破坏双亲委派的实现

我们结合Driver来看一下在spi（Service Provider Inteface）中如何实现破坏双亲委派。

先从DriverManager开始看，平时我们通过DriverManager来获取数据库的Connection：

```
String url = "jdbc:mysql://localhost:3306/testdb";
Connection conn = java.sql.DriverManager.getConnection(url, "root", "root"); 
```


[参考](http://www.cnblogs.com/joemsu/p/9310226.html#_caption_2)





# 虚拟机字节码执行引擎


## 运行时栈帧结构

![image](http://490.github.io/images/20190314_084016.png)

### 局部变量表

- 存放方法参数和方法内部定义的局部变量；
	- Java 程序编译为 class 文件时，就确定了每个方法需要分配的局部变量表的最大容量。
- 最小单位：Slot；
	- 一个 Slot 中可以存放：boolean，byte，char，short，int，float，reference，returnAddress (少见)；
	- 虚拟机可通过局部变量表中的 reference 做到：
		- 查找 Java 堆中的实例对象的起始地址；
		- 查找方法区中的 Class 对象。

#### 局部变量表的空间分配

![image](http://490.github.io/images/20190314_084054.png)

#### Slot 的复用

**定义：** 如果当前位置已经超过某个变量的作用域时，例如出了定义这个变量的代码块，这个变量对应的 Slot 就可以给其他变量使用了。但同时也说明，只要其他变量没有使用这部分 Slot 区域，这个变量就还保存在那里，这会对 GC 操作产生影响。

**对 GC 操作的影响：**

```java
public static void main(String[] args) {
    {
    	byte[] placeholder = new byte[64 * 1024 * 1024];
    }
    System.gc();
}
```

`-verbose:gc` 输出：

```
[GC (System.gc())  68813K->66304K(123904K), 0.0034797 secs]
[Full GC (System.gc())  66304K->66204K(123904K), 0.0086225 secs]  // 没有被回收
```

进行如下修改：

```java
public static void main(String[] args) {
    {
    	byte[] placeholder = new byte[64 * 1024 * 1024];
    }
    int a = 1; // 新加一个赋值操作
    System.gc();
}
```

`-verbose:gc` 输出：

```
[GC (System.gc())  68813K->66320K(123904K), 0.0017394 secs]
[Full GC (System.gc())  66320K->668K(123904K), 0.0084337 secs]  // 被回收了
```

**第二次修改后，placeholder 能被回收的原因？**

- placeholder 能否被回收的关键：局部变量表中的 Slot 是否还存在关于 placeholder 的引用；
- 出了 placeholder 所在的代码块后，还没有进行其他操作，所以 placeholder 所在的 Slot 还没有被其他变量复用，也就是说，局部变量表的 Slot 中依然存在着 placeholder 的引用；
- 第二次修改后，int a 占用了原来 placeholder 所在的 Slot，所以可以被 GC 掉了。



### 操作数栈

- 元素可以是任意 Java 类型，32 位数据占 1 个栈容量，64 位数据占 2 个栈容量；
- Java 虚拟机的解释执行称为：基于栈的执行引擎，其中 “栈” 指的就是操作数栈；



### 动态连接

- 指向运行时常量池中该栈帧所属方法的引用；
- 为了支持方法调用过程中的动态连接，什么是动态连接会在下一篇文章进行讲解，先知道有这么个东西就行。



### 方法返回地址

- **两种退出方法的方式：**
	- 遇到 return；
	- 遇到异常。
- **退出方法时可能执行的操作：**
	- 恢复上层方法的局部变量表和操作数栈；
	- 把返回值压入调用者栈帧的操作数栈；
	- 调整 PC 计数器指向方法调用后面的指令。



## 方法调用

Java 的方法的执行分为两个部分：

- 方法调用：确定被调用的方法是哪一个；
- 基于栈的解释执行：真正的执行方法的字节码。

在本节中我们将对方法调用进行详细的讲解，我们知道，一切方法的调用在 Class 文件中存储的都是常量池中的符号引用，而不是方法实际运行时的入口地址（直接引用），直到类加载的时候，甚至是实际运行的时候才回去会去确定要被运行的方法的直接引用，而确定要被运行的方法的直接引用的过程就叫做方法调用。

### 方法调用字节码指令

Java 虚拟机提供了 5 个职责不同的方法调用字节码指令：

- `invokestatic`：调用静态方法；
- `invokespecial`：调用构造器方法、私有方法、父类方法；
- `invokevirtual`：调用所有虚方法，除了静态方法、构造器方法、私有方法、父类方法、final 方法的其他方法叫虚方法；
- `invokeinterface`：调用接口方法，会在运行时确定一个该接口的实现对象；
- `invokedynamic`：在运行时动态解析出调用点限定符引用的方法，再执行该方法。

除了 `invokedynamic`，其他 4 种方法的第一个参数都是被调用的方法的符号引用，是在编译时确定的，所以它们缺乏动态类型语言支持，因为动态类型语言只有在运行期才能确定接收者的类型，即变量的类型检查的主体过程在运行期，而非编译期。

> final 方法虽然是通过 `invokevirtual` 调用的，但是其无法被覆盖，没有其他版本，无需对接收者进行多态选择，或者说多态选择的结果是唯一的，所以属于非虚方法。



### 解析调用

解析调用，正如其名，就是 **在类加载的解析阶段，就确定了方法的调用版本** 。我们知道类加载的解析阶段会将一部分符号引用转化为直接引用，这一过程就叫做解析调用。因为是在程序真正运行前就确定了要调用哪一个方法，所以 **解析调用能成立的前提就是：方法在程序真正运行前就有一个明确的调用版本了，并且这个调用版本不会在运行期发生改变。**

符合这两个要求的只有以下两类方法：

- 通过 `invokestatic` 调用的方法：静态方法；
- 通过 `invokespecial` 调用的方法：私有方法、构造器方法、父类方法；

这两类方法根本不可能通过继承或者别的方式重写出来其他版本，也就是说，在运行前就可以确定调用版本了，十分适合在类加载阶段就解析好。它们会在类加载的解析阶被解析为直接引用，即确定调用版本。



### 分派调用

在介绍分派调用前，我们先来介绍一下 Java 所具备的面向对象的 3 个基本特征：封装，继承，多态。

其中多态最基本的体现就是重载和重写了，重载和重写的一个重要特征就是方法名相同，其他各种不同：

- 重载：发生在同一个类中，入参必须不同，返回类型、访问修饰符、抛出的异常都可以不同；
- 重写：发生在子父类中，入参和返回类型必须相同，访问修饰符大于等于被重写的方法，不能抛出新的异常。

相同的方法名实际上给虚拟机的调用带来了困惑，因为虚拟机需要判断，它到底应该调用哪个方法，而这个过程会在分派调用中体现出来。其中：

- 方法重载 —— 静态分派
- 方法重写 —— 动态分派


#### 静态分派（方法重载）

在介绍静态分派前，我们先来介绍一下什么是变量的静态类型和实际类型。

** 变量的静态类型和实际类型**

```java
public class StaticDispatch {
    static abstract class Human {
    }

    static class Man extends Human {
    }

    static class Woman extends Human {
    }

    public void sayHello(Human guy) {
    	System.out.println("Hello guy!");
    }

    public void sayHello(Man man) {
    	System.out.println("Hello man!");
    }

    public void sayHello(Woman woman) {
    	System.out.println("Hello woman!");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch sr = new StaticDispatch();
        sr.sayHello(man);
        sr.sayHello(woman);
        /* 输出：
        Hello guy!
        Hello guy!
        因为是根据变量的静态类型，也就是左面的类型：Human 来判断调用哪个方法，
        所以调用的都是 public void sayHello(Human guy)
        */
    }
}

/* 简单讲解 */
// 使用
Human man = new Man();

// 实际类型发生变化
Human man = new Man();
man = new Woman();

// 静态类型发生变化
sr.sayHello((Man) man);   // 输出：Hello man!
sr.sayHello((Woman) man); // 输出：Hello woman!
```

其中 `Human` 称为变量的静态类型，`Man` 称为变量的实际类型。

在重载时，编译器是通过方法参数的静态类型，而不是实际类型，来判断应该调用哪个方法的。

**通俗的讲，静态分派就是通过方法的参数（类型 & 个数 & 顺序）这种静态的东西来判断到底调用哪个方法的过程。**

**重载方法匹配优先级，例如一个字符 'a' 作为入参**

- 基本类型
	- char
	- int
	- long
	- float
	- double
- Character
- Serializable（Character 实现的接口）
	- 同时出现两个优先级相同的接口，如 Serializable 和 Comparable，会提示类型模糊，拒绝编译。
- Object
- char...（变长参数优先级最低）

#### 动态分派（方法重写）

动态分派就是在运行时，根据实际类型确定方法执行版本的分派过程。

```java
public class DynamicDispatch {
    static abstract class Human {
  	  protected abstract void sayHello();
    }

    static class Man extends Human {
        protected void sayHello() {
        	System.out.println("Hello man");
        }
    }

    static class Woman extends Human {
        protected void sayHello() {
        	System.out.println("Hello woman");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        man.sayHello();
        woman.sayHello();
        man = woman;
        man.sayHello();
        /* 输出
        Hello man
        Hello woman
        Hello woman
        */
    }
}
```

字节码分析：

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
        stack=2, locals=3, args_size=1
        0: new           #2      // class com/jvm/ch8/DynamicDispatch$Man
        3: dup
        4: invokespecial #3      // Method com/jvm/ch8/DynamicDispatch$Man."<init>":()V
        7: astore_1
        8: new           #4      // class com/jvm/ch8/DynamicDispatch$Woman
        11: dup
        12: invokespecial #5     // Method com/jvm/ch8/DynamicDispatch$Woman."<init>":()V
        15: astore_2
        16: aload_1		        // 把刚创建的对象的引用压到操作数栈顶，
        					   // 供之后执行sayHello时确定是执行哪个对象的sayHello
        17: invokevirtual #6    // 方法调用
        20: aload_2             // 把刚创建的对象的引用压到操作数栈顶，
                                // 供之后执行sayHello时确定是执行哪个对象的sayHello
        21: invokevirtual #6    // 方法调用
        24: aload_2
        25: astore_1
        26: aload_1
        27: invokevirtual #6    // Method com/jvm/ch8/DynamicDispatch$Human.sayHello:()V
        30: return
```

通过字节码分析可以看出，`invokevirtual` 指令的运行过程大致为：

- 去操作数栈顶取出将要执行的方法的所有者，记作 C；
- 查找此方法：
	- 在 C 中查找此方法；
	- 在 C 的各个父类中查找；
	- 查找过程：
		- 查找与常量的描述符和简单名称都相同的方法；
		- 进行访问权限验证，不通过抛出：IllegalAccessError 异常；
		- 通过访问权限验证则返回直接引用；
- 没找到则抛出：AbstractMethodError 异常，即该方法没被实现。

动态分派在虚拟机种执行的非常频繁，而且方法查找的过程要在类的方法元数据中搜索合适的目标，从性能上考虑，不太可能进行如此频繁的搜索，需要进行性能上的优化。

**常用优化手段：** 在类的方法区中建立一个虚方法表。

- 虚方法表中存放着各个方法的实际入口地址，如果某个方法没有被子类方法重写，那子类方法表中该方法的入口地址 = 父类方法表中该方法的入口地址；
- 使用这个方法表索引代替在元数据中查找；
- 该方法表会在类加载的连接阶段初始化好。

**通俗的讲，动态分派就是通过方法的接收者这种动态的东西来判断到底调用哪个方法的过程。**

> 总结一下：静态分派看左面，动态分派看右面。

#### 单分派与多分派

除了静态分派和动态分派这种分派分类方式，还有一种根据宗量分类的方式，可以将方法分派分为单分派和多分派。

> 宗量：方法的接收者 & 方法的参数。

Java 语言的静态分派属于多分派，根据 **方法接收者的静态类型** 和 **方法参数类型** 两个宗量进行选择。

Java 语言的动态分派属于单分派，只根据 **方法接收者的实际类型** 一个宗量进行选择。



### 动态类型语言支持

**什么是动态类型语言？** 

就是类型检查的主体过程在运行期，而非编译期的编程语言。

**动/静态类型语言各自的优点？**

- 动态类型语言：灵活性高，开发效率高。
- 静态类型语言：编译器提供了严谨的类型检查，类型相关的问题能在编码的时候就发现。

**Java虚拟机层面提供的动态类型支持：**

- `invokedynamic` 指令
- java.lang.invoke 包

#### java.lang.invoke 包

**目的：** 在之前的依靠符号引用确定调用的目标方法的方式之外，提供了 MethodHandle 这种动态确定目标方法的调用机制。

 **MethodHandle 的使用**

- 获得方法的参数描述，第一个参数是方法返回值的类型，之后的参数是方法的入参：

```java
	MethodType mt = MethodType.methodType(void.class, String.class);
```

- 获取一个普通方法的调用：

```java
	/**
	 * 需要的参数：
	 * 1. 被调用方法所属类的类对象
	 * 2. 方法名
	 * 3. MethodType 对象 mt
	 * 4. 调用该方法的对象
	 */
	MethodHandle.lookup().findVirtual(receiver.getClass(), "方法名", mt).bindTo(receiver);
```

- 获取一个父类方法的调用：

```java
	/**
	 * 需要的参数：
	 * 1. 被调用方法所属类的类对象
	 * 2. 方法名
	 * 3. MethodType 对象 mt
	 * 4. 调用这个方法的类的类对象
	 */
	MethodHandle.lookup().findSpecial(GrandFather.class, "方法名", mt, getClass());
```

- 通过 `MethodHandle mh` 执行方法：

```java
	/* 
	invoke() 和 invokeExact() 的区别：
	- invokeExact() 要求更严格，要求严格的类型匹配，方法的返回值类型也在考虑范围之内
	- invoke() 允许更加松散的调用方式
	*/
	mh.invoke("Hello world");
	mh.invokeExact("Hello world");
```

使用示例：

```java
public class MethodHandleTest {
    static class ClassA {
        public void println(String s) {
            System.out.println(s);
        }
    }

    public static void main(String[] args) throws Throwable {
        /*
        obj的静态类型是Object，是没有println方法的，所以尽管obj的实际类型都包含println方法，
        它还是不能调用println方法
         */
        Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
        /*
        invoke()和invokeExact()的区别：
        - invokeExact()要求更严格，要求严格的类型匹配，方法的返回值类型也在考虑范围之内
        - invoke()允许更加松散的调用方式
         */
        getPrintlnMH(obj).invoke("Hello world");
        getPrintlnMH(obj).invokeExact("Hello world");
    }

    private static MethodHandle getPrintlnMH(Object receiver) 
        	throws NoSuchMethodException, IllegalAccessException {
        /* MethodType代表方法类型，第一个参数是方法返回值的类型，之后的参数是方法的入参 */
        MethodType mt = MethodType.methodType(void.class, String.class);
        /*
        lookup()方法来自于MethodHandles.lookup，
        这句的作用是在指定类中查找符合给定的方法名称、方法类型，并且符合调用权限的方法句柄
        */
        /*
        因为这里调用的是一个虚方法，按照Java语言的规则，方法第一个参数是隐式的，代表该方法的接收者，
        也即是this指向的对象，这个参数以前是放在参数列表中进行传递，现在提供了bindTo()方法来完成这件事情
        */
        return MethodHandles.lookup().findVirtual(receiver.getClass(), "println", mt).bindTo(receiver);
    }
}

```

**MethodHandles.lookup 中 3 个方法对应的字节码指令：**

- `findStatic()`：对应 invokestatic
- `findVirtual()`：对应 invokevirtual & invokeinterface
- `findSpecial()`：对应 invokespecial

**MethodHandle 和 Reflection 的区别**

- 本质区别：它们都在模拟方法调用，但是
	- Reflection 模拟的是 Java 代码层次的调用；
	- MethodHandle 模拟的是字节码层次的调用。
- 包含信息的区别：
	- Reflection 的 Method 对象包含的信息多，包括：方法签名、方法描述符、方法的各种属性的Java端表达方式、方法执行权限等；
	- MethodHandle 对象包含的信息比较少，既包含与执行该方法相关的信息。

#### `invokedynamic` 指令

Lambda 表达式就是通过 `invokedynamic` 指令实现的。


JAVA类装载方式，有两种:1.隐式装载， 程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中。 2.显式装载， 通过class.forname()等方法，显式加载需要的类
<!--more-->
类加载的动态性体现:一个应用程序总是由n多个类组成，Java程序启动时，并不是一次把所有的类全部加载后再运行，它总是先把保证程序运行的基础类一次性加载到jvm中，其它类等到jvm用到的时候再加载，这样的好处是节省了内存的开销，因为java最早就是为嵌入式系统而设计的，内存宝贵，这是一种可以理解的机制，而用到时再加载这也是java动态性的一种体现.
![image](http://490.github.io/images/20190311_083329.png)


# 基于栈的字节码解释执行引擎

这个栈，就是栈帧中的操作数栈。

## 解释执行

先通过 javac 将代码编译成字节码，虚拟机再通过加载字节码文件，解释执行字节码文件生成机器码，解释执行的流程如下：

```
词法分析 -> 语法分析 -> 形成抽象语法树 -> 遍历语法树生成线性字节码指令流
```


## 指令集分类

### 基于栈的指令集

- **优点：**

	- 可移植：寄存器由硬件直接提供，程序如果直接依赖这些硬件寄存器，会不可避免的受到硬件的约束；
	- 代码更紧凑：字节码中每个字节对应一条指令，多地址指令集中还需要存放参数；
	- 编译器实现更简单：不需要考虑空间分配问题，所需的空间都在栈上操作。

- **缺点：** 
  - 执行速度稍慢
  - 完成相同的功能，需要更多的指令，因为出入栈本身就产生相当多的指令；
  - 频繁的栈访问导致频繁的内存访问，对于处理器而言，内存是执行速度的瓶颈。

- **示例：** 两数相加

```
	iconst_1  // 把常量1入栈
	iconst_1
	iadd      // 把栈顶两元素出栈相加，结果入栈
	istore_0  // 把栈顶值存入第0个Slot中
```

### 基于寄存器的指令集

**示例：** 两数相加

```
mov  eax, 1
add  eax, 1
```



## 执行过程分析

```java
public class Architecture {
    /*
    calc函数的字节码分析：
    public int calc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
    stack=2, locals=4, args_size=1 // stack=2，说明需要深度为2的操作数栈
                                   // locals=4，说明需要4个Slot的局部变量表
    0: bipush 100                  // 将单字节的整型常数值push到操作数栈
    2: istore_1                    // 将操作数栈顶的整型值出栈并存放到第一个局部变量Slot中
    3: sipush 200
    6: istore_2
    7: sipush 300
    10: istore_3
    11: iload_1                    // 将局部变量表第一个Slot中的整型值复制到操作数栈顶
    12: iload_2
    13: iadd                       // 将操作数栈中头两个元素出栈并相加，将结果重新入栈
    14: iload_3
    15: imul                       // 将操作数栈中头两个元素出栈并相乘，将结果重新入栈
    16: ireturn                    // 返回指令，结束方法执行，将操作数栈顶的整型值返回给此方法的调用者
    */
    
    public int calc() {
        int a = 100;
        int b = 200;
        int c = 300;
        return (a + b) * c;
    }

    public static void main(String[] args) {
        Architecture architecture = new Architecture();
        architecture.calc();
    }
}
```

