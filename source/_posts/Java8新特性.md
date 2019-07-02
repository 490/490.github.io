---
title: Java8新特性
date: 2019-04-25 16:42:09
tags: Java
---

Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本。 Oracle 公司于 2014 年 3 月 18 日发布 Java 8 ，它支持函数式编程，新的 JavaScript 引擎，新的日期 API，新的Stream API 等。

<!--more-->

# Lambda 表达式

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

## 语法

lambda 表达式的语法格式如下：

(parameters) -> expression 或 (parameters) ->{  statements; }

以下是lambda表达式的重要特征:

*   **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
*   **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
*   **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
*   **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

## Lambda 表达式实例

Lambda 表达式的简单例子:

```java
// 1. 不需要参数,返回值为 5 
 ()  ->  5  
// 2. 接收一个参数(数字类型),返回其2倍的值 
x ->  2  * x 
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y)  -> x – y 
// 4. 接收2个int型整数,返回他们的和  
(int x,  int y)  -> x + y 
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s)  ->  System.out.print(s)
```


# 方法引用

方法引用通过方法的名字来指向一个方法。
方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
方法引用使用一对冒号 :: 。
下面，我们在 Car 类中定义了 4 个方法作为例子来区分 Java 中 4 种不同方法的引用。

```java
package com.runoob.main;
@FunctionalInterface
public interface Supplier<T> 
{
    T get();
}
class Car 
{
    //Supplier是jdk1.8的接口，这里和lamda一起使用了
    public static Car create(final Supplier<Car> supplier) 
    {
        return supplier.get();
    }
    public static void collide(final Car car) 
    {
        System.out.println("Collided " + car.toString());
    }
    public void follow(final Car another) 
    {
        System.out.println("Following the " + another.toString());
    }
    public void repair() 
    {
        System.out.println("Repaired " + this.toString());
    }
}
```

*   **构造器引用：**它的语法是Class::new，或者更一般的Class< T >::new实例如下：
`final  Car  car = Car.create(  Car::new  ); final  List< Car > cars = Arrays.asList(  car  );`

*   **静态方法引用：**它的语法是Class::static_method，实例如下：
`cars.forEach(  Car::collide  );`

*   **特定类的任意对象的方法引用：**它的语法是Class::method实例如下：
`cars.forEach(  Car::repair  );`

*   **特定对象的方法引用：**它的语法是instance::method实例如下：
`final  Car  police = Car.create(  Car::new  ); cars.forEach(  police::follow  );`
## 方法引用实例

```java
import java.util.List;
import java.util.ArrayList;
public class Java8Tester 
{
   public static void main(String args[])
   {
      List names = new ArrayList();
      names.add("Google");
      names.add("Runoob");
      names.add("Taobao");
      names.add("Baidu");
      names.add("Sina");
      names.forEach(System.out::println);
   }
}
实例中我们将 System.out::println 方法作为静态方法来引用。
执行以上脚本，输出结果为：
$ javac Java8Tester.java 
$ java Java8Tester
Google
Runoob
Taobao
Baidu
Sina
```

# Optional 类
https://www.cnblogs.com/zhangboyu/p/7580262.html

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

我们从一个简单的用例开始。在 Java 8 之前，任何访问对象方法或属性的调用都可能导致 _NullPointerException_：

String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();

在这个小示例中，如果我们需要确保不触发异常，就得在访问每一个值之前对其进行明确地检查：
```java
if (user != null)
{
    Address address = user.getAddress();
    if (address != null) 
    {
        Country country = address.getCountry();
        if (country != null) 
        {
            String isocode = country.getIsocode();
            if (isocode != null) 
            {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```

你看到了，这很容易就变得冗长，难以维护。

为了简化这个过程，我们来看看用 _Optional_  类是怎么做的。从创建和验证实例，到使用其不同的方法，并与其它返回相同类型的方法相结合，下面是见证 _Optional  _奇迹的时刻。

# Stream

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

```java
List<Integer> transactionsIds = 
widgets.stream()
             .filter(b -> b.getColor() == RED)
             .sorted((x,y) -> x.getWeight() - y.getWeight())
             .mapToInt(Widget::getWeight)
             .sum();
```

Stream（流）是一个来自数据源的元素队列并支持聚合操作

*   元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
*   **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
*   **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

*   **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
*   **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。


## 生成流


*   **stream()** − 为集合创建串行流。
*   **parallelStream()** − 为集合创建并行流。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```



## Stream 完整实例

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;
import java.util.Map;
 
public class Java8Tester 
{
   public static void main(String args[])
   {
      System.out.println("使用 Java 7: ");
        
      // 计算空字符串
      List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
      System.out.println("列表: " +strings);
      long count = getCountEmptyStringUsingJava7(strings);
        
      System.out.println("空字符数量为: " + count);
      count = getCountLength3UsingJava7(strings);
        
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      // 删除空字符串
      List<String> filtered = deleteEmptyStringsUsingJava7(strings);
      System.out.println("筛选后的列表: " + filtered);
        
      // 删除空字符串，并使用逗号把它们合并起来
      String mergedString = getMergedStringUsingJava7(strings,", ");
      System.out.println("合并字符串: " + mergedString);
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
        
      // 获取列表元素平方数
      List<Integer> squaresList = getSquares(numbers);
      System.out.println("平方数列表: " + squaresList);
      List<Integer> integers = Arrays.asList(1,2,13,4,15,6,17,8,19);
        
      System.out.println("列表: " +integers);
      System.out.println("列表中最大的数 : " + getMax(integers));
      System.out.println("列表中最小的数 : " + getMin(integers));
      System.out.println("所有数之和 : " + getSum(integers));
      System.out.println("平均数 : " + getAverage(integers));
      System.out.println("随机数: ");
        
      // 输出10个随机数
      Random random = new Random();
        
      for(int i=0; i < 10; i++)
      {
         System.out.println(random.nextInt());
      }
        
      System.out.println("使用 Java 8: ");
      System.out.println("列表: " +strings);
        
      count = strings.stream().filter(string->string.isEmpty()).count();
      System.out.println("空字符串数量为: " + count);
        
      count = strings.stream().filter(string -> string.length() == 3).count();
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      filtered = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.toList());
      System.out.println("筛选后的列表: " + filtered);
        
      mergedString = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.joining(", "));
      System.out.println("合并字符串: " + mergedString);
        
      squaresList = numbers.stream().map( i ->i*i).distinct().collect(Collectors.toList());
      System.out.println("Squares List: " + squaresList);
      System.out.println("列表: " +integers);
        
      IntSummaryStatistics stats = integers.stream().mapToInt((x) ->x).summaryStatistics();
        
      System.out.println("列表中最大的数 : " + stats.getMax());
      System.out.println("列表中最小的数 : " + stats.getMin());
      System.out.println("所有数之和 : " + stats.getSum());
      System.out.println("平均数 : " + stats.getAverage());
      System.out.println("随机数: ");
        
      random.ints().limit(10).sorted().forEach(System.out::println);
        
      // 并行处理
      count = strings.parallelStream().filter(string -> string.isEmpty()).count();
      System.out.println("空字符串的数量为: " + count);
   }
    
   private static int getCountEmptyStringUsingJava7(List<String> strings)
   {
      int count = 0;
      for(String string: strings)
      {
         if(string.isEmpty())
         {
            count++;
         }
      }
      return count;
   }
   private static int getCountLength3UsingJava7(List<String> strings)
   {
      int count = 0;
      for(String string: strings)
      {
         if(string.length() == 3)
         {
            count++;
         }
      }
      return count;
   }
    
   private static List<String> deleteEmptyStringsUsingJava7(List<String> strings)
   {
      List<String> filteredList = new ArrayList<String>();
      for(String string: strings)
      {
         if(!string.isEmpty())
         {
             filteredList.add(string);
         }
      }
      return filteredList;
   }
    
   private static String getMergedStringUsingJava7(List<String> strings, String separator)
   {
      StringBuilder stringBuilder = new StringBuilder();
      for(String string: strings)
      {
         if(!string.isEmpty())
         {
            stringBuilder.append(string);
            stringBuilder.append(separator);
         }
      }
      String mergedString = stringBuilder.toString();
      return mergedString.substring(0, mergedString.length()-2);
   }
    
   private static List<Integer> getSquares(List<Integer> numbers)
   {
      List<Integer> squaresList = new ArrayList<Integer>();
      for(Integer number: numbers)
      {
         Integer square = new Integer(number.intValue() * number.intValue());
         if(!squaresList.contains(square))
         {
            squaresList.add(square);
         }
      }
      return squaresList;
   }
    
   private static int getMax(List<Integer> numbers)
   {
      int max = numbers.get(0);
      for(int i=1;i < numbers.size();i++)
      {
         Integer number = numbers.get(i);
         if(number.intValue() > max)
         {
            max = number.intValue();
         }
      }
      return max;
   }
   private static int getMin(List<Integer> numbers)
   {
      int min = numbers.get(0);
      for(int i=1;i < numbers.size();i++)
      {
         Integer number = numbers.get(i);
         if(number.intValue() < min)
         {
            min = number.intValue();
         }
      }
      return min;
   }
    
   private static int getSum(List numbers)
   {
      int sum = (int)(numbers.get(0));
      for(int i=1;i < numbers.size();i++)
      {
         sum += (int)numbers.get(i);
      }
      return sum;
   }
   private static int getAverage(List<Integer> numbers)
   {
      return getSum(numbers) / numbers.size();
   }
}
```


