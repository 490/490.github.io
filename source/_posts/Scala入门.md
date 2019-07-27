---
title: Scala入门
date: 2019-06-19 16:30:21
tags: scala
---

Scala 是一门多范式（multi-paradigm）的编程语言，设计初衷是要集成面向对象编程和函数式编程的各种特性。
Scala 运行在Java虚拟机上，并兼容现有的Java程序。
Scala 源代码被编译成Java字节码，所以它可以运行于JVM之上，并可以调用现有的Java类库。

<!--more-->

```
object HelloWorld {
    def main(args: Array[String]): Unit = {
        println("Hello, world!")
    }
}

$ scalac HelloWorld.scala // 把源码编译为字节码
$ scala HelloWorld  // 把字节码放到虚拟机中解释运行

```
# 基础语法

*   **类名** - 对于所有的类名的第一个字母要大写。
    如果需要使用几个单词来构成一个类的名称，每个单词的第一个字母要大写。

    示例：_class MyFirstScalaClass_

*   **方法名称** - 所有的方法名称的第一个字母用小写。
    如果若干单词被用于构成方法的名称，则每个单词的第一个字母应大写。

    示例：_def myMethodName()_

*   **程序文件名** - 程序文件的名称应该与对象名称完全匹配(新版本不需要了，但建议保留这种习惯)。
    保存文件时，应该保存它使用的对象名称（记住Scala是区分大小写），并追加".scala"为文件扩展名。 （如果文件名和对象名称不匹配，程序将无法编译）。

    示例: 假设"HelloWorld"是对象的名称。那么该文件应保存为'HelloWorld.scala"

*   **def main(args: Array[String]) **- Scala程序从main()方法开始处理，这是每一个Scala程序的强制程序入口部分。



## 引用

Scala 使用 import 关键字引用包。
```

import java.awt.Color  // 引入Color 
import java.awt._ // 引入包内所有成员  
def handler(evt:  event.ActionEvent)  
{ 
  // java.awt.event.ActionEvent  ...  // 因为引入了java.awt，所以可以省去前面的部分 
}

```

import语句可以出现在任何地方，而不是只能在文件顶部。import的效果从开始延伸到语句块的结束。这可以大幅减少名称冲突的可能性。

如果想要引入包中的几个成员，可以使用selector（选取器）：
```
import java.awt.{Color,  Font}  // 重命名成员 
import java.util.{HashMap  =>  JavaHashMap}  // 隐藏成员  
import java.util.{HashMap  => _, _}  // 引入了util包的所有成员，但是HashMap被隐藏了
```
# class、object、trait

## class

在scala中，类名可以和对象名为同一个名字，该对象称为该类的伴生对象，类和伴生对象可以相互访问他们的私有属性，但是他们必须在同一个源文件内。类只会被编译，不能直接被执行，类的申明和主构造器在一起被申明，在一个类中，主构造器只有一个所有必须在内部申明主构造器或者是其他申明主构造器的辅构造器，主构造器会执行类定义中的所有语句。scala对每个字段都会提供getter和setter方法，同时也可以显示的申明，但是针对val类型，只提供getter方法，默认情况下，字段为公有类型，可以在setter方法中增加限制条件来限定变量的变化范围，在scala中方法可以访问改类所有对象的私有字段

## object

在scala中没有静态方法和静态字段，所以在scala中可以用object来实现这些功能，直接用对象名调用的方法都是采用这种实现方式，例如Array.toString。对象的构造器在第一次使用的时候会被调用，如果一个对象从未被使用，那么他的构造器也不会被执行；对象本质上拥有类（scala中）的所有特性，除此之外，object还可以一扩展类以及一个或者多个特质：例如，
abstract class ClassName（val parameter）{}
object Test extends ClassName(val parameter){}

注意：object不能提供构造器参数，也就是说object必须是无参的



## trait

在java中可以通过interface实现多重继承，在Scala中可以通过特征（trait）实现多重继承，不过与java不同的是，它可以定义自己的属性和实现方法体，在没有自己的实现方法体时可以认为它时java interface是等价的，在Scala中也是一般只能继承一个父类，可以通过多个with进行多重继承。

trait TraitA{}
trait TraitB{}
trait TraitC{}
object Test1 extends TraitA with TraitB with TraitC{}


# Scala 访问修饰符

Scala 访问修饰符基本和Java的一样，分别有：private，protected，public。
如果没有指定访问修饰符，默认情况下，Scala 对象的访问级别都是 public。
Scala 中的 private 限定符，比 Java 更严格，在嵌套类情况下，外层类甚至不能访问被嵌套类的私有成员。

## 私有(Private)成员

用 private 关键字修饰，带有此标记的成员仅在包含了成员定义的类或对象内部可见，同样的规则还适用内部类。
```java
class  Outer
{  
  class  Inner{  
    private  def f()
    {
        println("f")
    }  
    class  InnerMost
    { 
      f()  // 正确  
    }  
  }  
  (new  Inner).f()  //错误 
}
```


(new Inner).f( ) 访问不合法是因为 **f** 在 Inner 中被声明为 private，而访问不在类 Inner 之内。

但在 InnerMost 里访问 **f** 就没有问题的，因为这个访问包含在 Inner 类之内。

Java中允许这两种访问，因为它允许外部类访问内部类的私有成员。

* * *

## 保护(Protected)成员

在 scala 中，对保护（Protected）成员的访问比 java 更严格一些。因为它只允许保护成员在定义了该成员的的类的子类中被访问。而在java中，用protected关键字修饰的成员，除了定义了该成员的类的子类可以访问，同一个包里的其他类也可以进行访问。
```java
package p
{  
    class  Super
    {  
        protected  def f()  
        {println("f")}  
    } 
    class  Sub  extends  Super
    { 
        f() 
    } 
    class  Other
    { 
        (new  Super).f()  //错误 
    }  
}

```


上例中，Sub 类对 f 的访问没有问题，因为 f 在 Super 中被声明为 protected，而 Sub 是 Super 的子类。相反，Other 对 f 的访问不被允许，因为 other 没有继承自 Super。而后者在 java 里同样被认可，因为 Other 与 Sub 在同一包里。

* * *

## 公共(Public)成员

Scala中，如果没有指定任何的修饰符，则默认为 public。这样的成员在任何地方都可以被访问。

```java
class  Outer  
{  
    class  Inner  
    {  
        def f()  
        {
            println("f")  
        }  
        class  InnerMost  
        { 
            f()  // 正确  
        }  
    }  
    (new  Inner).f()  // 正确因为 f() 是 public  
}
```


* * *

## 作用域保护

Scala中，访问修饰符可以通过使用限定词强调。格式为:

private[x]  或  protected[x]

这里的x指代某个所属的包、类或单例对象。如果写成private[x],读作"这个成员除了对[…]中的类或[…]中的包中的类及它们的伴生对像可见外，对其它所有类都是private。

这种技巧在横跨了若干包的大型项目中非常有用，它允许你定义一些在你项目的若干子包中可见但对于项目外部的客户却始终不可见的东西。

```java
package bobsrocckets
{
    package navigation
    {
        private[bobsrockets] class Navigator
        {
             protected[navigation] def useStarChart(){}
             class LegOfJourney
             {
                 private[Navigator] val distance = 100
             }
              private[this] var speed = 200
         }
    }
    package launch
    {
        import navigation._
        object Vehicle
        {
            private[launch] val guide = new Navigator
        }
     }
}
```


上述例子中，类Navigator被标记为private[bobsrockets]就是说这个类对包含在bobsrockets包里的所有的类和对象可见。

比如说，从Vehicle对象里对Navigator的访问是被允许的，因为对象Vehicle包含在包launch中，而launch包在bobsrockets中，相反，所有在包bobsrockets之外的代码都不能访问类Navigator。

# Scala 方法与函数

Scala 有方法与函数，二者在语义上的区别很小。Scala 方法是类的一部分，而函数是一个对象可以赋值给一个变量。换句话来说在类中定义的函数即是方法。

Scala 中的方法跟 Java 的类似，方法是组成类的一部分。

Scala 中的函数则是一个完整的对象，Scala 中的函数其实就是继承了 Trait 的类的对象。

Scala 中使用 **val** 语句可以定义函数，**def** 语句定义方法。

```java
class Test{
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
}
```

Scala 方法声明格式如下：

`def functionName ([参数列表])  :  [return type]`

如果你不写等于号和方法主体，那么方法会被隐式声明为**抽象(abstract)**，包含它的类型于是也是一个抽象类型。

方法定义由一个 def 关键字开始，紧接着是可选的参数列表，一个冒号 : 和方法的返回类型，一个等于号 = ，最后是方法的主体。

Scala 方法定义格式如下：
```java
object add{
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```


如果方法没有返回值，可以返回为 Unit，这个类似于 Java 的 void, 实例如下：
```java
object Hello{
   def printMe( ) : Unit = {
      println("Hello, Scala!")
   }
}

```


以下是调用方法的标准格式：

`functionName(  参数列表  )`

如果方法使用了实例的对象来调用，我们可以使用类似java的格式 (使用 **.** 号)：

`[instance.]functionName(  参数列表  )`

## Scala 函数传名调用(call-by-name)


*   传值调用（call-by-value）：先计算参数表达式的值，再应用到函数内部；
*   传名调用（call-by-name）：将未计算的参数表达式直接应用到函数内部

```java
object Test 
{
   def main(args: Array[String]) 
   {
        delayed(time());
   }
   def time() = 
   {
      println("获取时间，单位为纳秒")
      System.nanoTime
   }
   def delayed( t: => Long ) = 
   {
      println("在 delayed 方法内")
      println("参数： " + t)
      t
   }
}

$ scalac Test.scala 
$ scala Test
在 delayed 方法内
获取时间，单位为纳秒
参数： 241550840475831
获取时间，单位为纳秒
```

## 默认参数值

Scala 可以为函数参数指定默认参数值，使用了默认参数，你在调用函数的过程中可以不需要传递参数，这时函数就会调用它的默认参数值，如果传递了参数，则传递值会取代默认值。实例如下：

```java
object Test 
{
   def main(args: Array[String]) 
   {
        println( "返回值 : " + addInt() );
   }
   def addInt( a:Int=5, b:Int=7 ) : Int = 
   {
      var sum:Int = 0
      sum = a + b
      return sum
   }
}
```

## Scala 高阶函数


```
object Test {
   def main(args: Array[String]) {

      println( apply( layout, 10) )

   }
   // 函数 f 和 值 v 作为参数，而函数 f 又调用了参数 v
   def apply(f: Int => String, v: Int) = f(v)

   def layout[A](x: A) = "[" + x.toString() + "]"
   
}
```

## Scala 匿名函数


箭头左边是参数列表，右边是函数体。
使用匿名函数后，我们的代码变得更简洁了。
下面的表达式就定义了一个接受一个Int类型输入参数的匿名函数:
`var inc =  (x:Int)  => x+1`
上述定义的匿名函数，其实是下面这种写法的简写：
`def add2 =  new  Function1[Int,Int]{ def apply(x:Int):Int  = x+1;  }  `
以上实例的 inc 现在可作为一个函数，使用方式如下：
`var x = inc(7)-1`

我们也可以不给匿名函数设置参数，如下所示：
`var userDir =  ()  =>  {  System.getProperty("user.dir")  }`




1、函数可作为一个参数传入到方法中，而方法不行。

2、在Scala中无法直接操作方法，如果要操作方法，必须先将其转换成函数。有两种方法可以将方法转换成函数：

`val f1 = m _`

在方法名称m后面紧跟一个空格和下划线告诉编译器将方法m转换成函数，而不是要调用这个方法。 也可以显示地告诉编译器需要将方法转换成函数：

`val f1:  (Int)  =>  Int  = m`

通常情况下编译器会自动将方法转换成函数，例如在一个应该传入函数参数的地方传入了一个方法，编译器会自动将传入的方法转换成函数。


3、函数必须要有参数列表，而方法可以没有参数列表

4、在函数出现的地方我们可以提供一个方法
在需要函数的地方，如果传递一个方法，会自动进行ETA展开（把方法转换为函数）

# Scala 闭包

闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量。

这里我们引入一个自由变量 factor，这个变量定义在函数外面。
这样定义的函数变量 multiplier 成为一个"闭包"，因为它引用到函数外面定义的变量，定义这个函数的过程是将这个自由变量捕获而构成一个封闭的函数。
```java
object Test 
{  
   def main(args: Array[String]) 
   {  
      println( "muliplier(1) value = " +  multiplier(1) )  
      println( "muliplier(2) value = " +  multiplier(2) )  
   }  
   var factor = 3  
   val multiplier = (i:Int) => i * factor  
}  


$ scalac Test.scala  
$  scala Test  
muliplier(1) value = 3  
muliplier(2) value = 6  

```

# Scala 字符串


```java
String 对象是不可变的，如果你需要创建一个可以修改的字符串，可以使用 String Builder 类，如下实例:

object Test
{
   def main(args: Array[String]) 
   {
      val buf = new StringBuilder;
      buf += 'a'
      buf ++= "bcdef"
      println( "buf is : " + buf.toString );
   }
}
```

 java.lang.String 中常用的方法，可以在 Scala 中使


# Scala 数组

```
var z:Array[String] = new Array[String](3)

或

var z = new Array[String](3)

```



定义了二维数组的实例：

`var myMatrix = ofDim[Int](3,3)`

```language
合并数组
      var myList1 = Array(1.9, 2.9, 3.4, 3.5)
      var myList2 = Array(8.9, 7.9, 0.4, 1.5)

      var myList3 =  concat( myList1, myList2)
```

下表中为 Scala 语言中处理数组的重要方法，使用它前我们需要使用 import Array._ 引入包。

序号	方法和描述
1	
def apply( x: T, xs: T* ): Array[T]

创建指定对象 T 的数组, T 的值可以是 Unit, Double, Float, Long, Int, Char, Short, Byte, Boolean。

2	
def concat[T]( xss: Array[T]* ): Array[T]

合并数组

3	
def copy( src: AnyRef, srcPos: Int, dest: AnyRef, destPos: Int, length: Int ): Unit

复制一个数组到另一个数组上。相等于 Java's System.arraycopy(src, srcPos, dest, destPos, length)。

4	
def empty[T]: Array[T]

返回长度为 0 的数组

5	
def iterate[T]( start: T, len: Int )( f: (T) => T ): Array[T]

返回指定长度数组，每个数组元素为指定函数的返回值。

以上实例数组初始值为 0，长度为 3，计算函数为a=>a+1：

scala> Array.iterate(0,3)(a=>a+1)
res1: Array[Int] = Array(0, 1, 2)
6	
def fill[T]( n: Int )(elem: => T): Array[T]

返回数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。

7	
def fill[T]( n1: Int, n2: Int )( elem: => T ): Array[Array[T]]

返回二数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。

8	
def ofDim[T]( n1: Int ): Array[T]

创建指定长度的数组

9	
def ofDim[T]( n1: Int, n2: Int ): Array[Array[T]]

创建二维数组

10	
def ofDim[T]( n1: Int, n2: Int, n3: Int ): Array[Array[Array[T]]]

创建三维数组

11	
def range( start: Int, end: Int, step: Int ): Array[Int]

创建指定区间内的数组，step 为每个元素间的步长

12	
def range( start: Int, end: Int ): Array[Int]

创建指定区间内的数组

13	
def tabulate[T]( n: Int )(f: (Int)=> T): Array[T]

返回指定长度数组，每个数组元素为指定函数的返回值，默认从 0 开始。

以上实例返回 3 个元素：

scala> Array.tabulate(3)(a => a + 5)
res0: Array[Int] = Array(5, 6, 7)
14	
def tabulate[T]( n1: Int, n2: Int )( f: (Int, Int ) => T): Array[Array[T]]

返回指定长度的二维数组，每个数组元素为指定函数的返回值，默认从 0 开始。
# Scala Collection

## Scala 使用 Option、Some、None，避免使用 Null

为了让所有东西都是对象的目标更加一致，也为了遵循函数式编程的习惯，Scala 鼓励你在变量和函数返回值可能不会引用任何值的时候使用 Option 类型。在没有值的时候，使用 None，这是 Option 的一个子类。如果有值可以引用，就使用 Some 来包含这个值。Some 也是 Option 的子类。 None 被声明为一个对象，而不是一个类，因为我们只需要它的一个实例。这样，它多少有点像 null 关键字，但它却是一个实实在在的，有方法的对象。

Option 类型的值通常作为 Scala 集合类型（List, Map 等）操作的返回类型。比如 Map 的 get 方法：

```java
scala> val capitals = Map("France"->"Paris", "Japan"->"Tokyo", "China"->"Beijing")
capitals: scala.collection.immutable.Map[String,String] = Map(France -> Paris, Japan -> Tokyo, China -> Beijing)

scala> capitals get "France"
res0: Option[String] = Some(Paris)

scala> capitals get "North Pole"
res1: Option[String] = None
```

Option 有两个子类别，Some 和 None。当程序回传 Some 的时候，代表这个函式成功地给了你一个 String，而你可以透过 get() 函数拿到那个 String，如果程序返回的是 None，则代表没有字符串可以给你。

在返回 None，也就是没有 String 给你的时候，如果你还硬要调用 get() 来取得 String 的话，Scala 一样是会抛出一个 NoSuchElementException 异常给你的。 我们也可以选用另外一个方法 getOrElse。这个方法在这个 Option 是 Some 的实例时返回对应的值，而在是 None 的实例时返回传入的参数。换句话说，传入 getOrElse 的参数实际上是默认返回值。


```java
scala> capitals get "North Pole" get
warning: there was one feature warning; re-run with -feature for details
java.util.NoSuchElementException: None.get
  at scala.None$.get(Option.scala:347)
  at scala.None$.get(Option.scala:345)
  ... 33 elided

scala> capitals get "France" get
warning: there was one feature warning; re-run with -feature for details
res3: String = Paris

scala> (capitals get "North Pole") getOrElse "Oops"
res7: String = Oops

scala> capitals get "France" getOrElse "Oops"
res8: String = Paris
```
通过模式匹配分离可选值，如果匹配的值是 Some 的话，将 Some 里的值抽出赋给 x 变量：
```java
def showCapital(x: Option[String]) = x match {
    case Some(s) => s
    case None => "?"
}

```


在 Scala 里 Option[T] 实际上是一个容器，就像数组或是 List 一样，你可以把他看成是一个可能有零到一个元素的 List。

当你的 Option 里面有东西的时候，这个 List 的长度是 1（也就是 Some），而当你的 Option 里没有东西的时候，它的长度是 0（也就是 None）。

如果我们把 Option 当成一般的 List 来用，并且用一个 for 循环来走访这个 Option 的时候，如果 Option 是 None，那这个 for 循环里的程序代码自然不会执行，于是我们就达到了「不用检查 Option 是否为 None 这件事。



