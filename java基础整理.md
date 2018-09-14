# java基础 #

## 数据类型 
### 包装类型： ###
---
八个基本类型  


- boolean/1
- byte/1
-  short/2
-  int/4
-  long/8
-  float/4
-  double/8
-  char/2  
  四种整数类型的精确度：long>int >short>byte  
两种浮点型的精确度：float>double  
隐式转换只能向上转型，利用+=等符号  
显示转换（强制类型转换）  

基本数据类型的包装类型，基本数据类型和包装类型之间的赋值使用自动装箱与拆箱完成  
`Integer x = 2;     // 装箱       
 int y = x;         // 拆箱`  
    
Integer和int 的比较：  

1. Integer是对象，int是基本数据类型
2. 通过Integer new 出来的integer类型永不相等  
`Integer i = new Integer(100); `   
`Integer j = new Integer(100);`  
`System.out.print(i == j); //false`
1. Integer变量和int变量比较时，只要两个变量的值是向等的，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）  
`Integer i = new Integer(100);`  
`int j = 100；`  
`System.out.print(i == j); //true`
1. 对于两个非new 出来的integer，如果两个变量之间的值在-128~127之间则比较结果为ture,否则为false  
  
String
--
String 被声明为 final，因此它不可被继承。

内部使用 char 数组存储数据，该数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。  
不可变得好处 
---


1. **可以缓存hash值：**  
因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。
2. **string pool需要:**  
如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。
1. **安全性**  
String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。
1. **线程安全**  
String 不可变性天生具备线程安全，可以在多个线程中安全地使用  
String, StringBuffer and StringBuilder
---
- String不可变
- StringBulider和StringBuffer可变
- String不可变，所以线程安全
- StringBulider不是线程安全的
- StringBuffer是线程安全的，其内部使用synchronized 进行同步
![](https://i.imgur.com/fVY2pKK.png)
![](https://i.imgur.com/Nv9IvWR.png)  

运算
---
java参数的传递是以值传递的，基本数据类型传递的是基本数据类型的值，对象传递的是其地址。  
 switch条件判断语句中不支持long数据类型，不支持浮点类型，但支持string类型  
  继承
---
访问权限：private，public，protect  

抽象类和接口有什么异同：
---  

抽象类：  
  
- 抽象类中可以有构造器
- 抽象类可以有具体方法
- 接口中的成员全是public
- 抽象类中可以定义成员变量
- 有抽象方法的类必须定义成抽象类，而抽象类未必要有抽象方法
- 一个类只能继承一个抽象类  

接口：  

- 抽象类的成员可以是public，private，protect
- 接口中的成员变量实际上都是常量一个类可以继承多个接口  
- 
相同：    

- 都不能够被实例化
- 都可以作为引用类型
- 一个类如果继承了某个抽象类或者实现了某个接口都需要对其中的抽象方法全部进行实现，否则该类仍然需要
被声明为抽象类  

Super()
--


- 访问父类构造器：可以用super函数访问父类的构造函数，从而委托父类完成一些初始化的工作。
- 访问父类的成员：如果子类重写了父类的中某个方法的实现，可以通过使用 super 关键字来引用父类的方法实现（不可再静态方法中使用）。   
 
## object通用方法 ##
---
equals()
---


1. equals的自反性：`x.equals(x); // true`   


1. equsal的对称性：`x.equals(y) == y.equals(x); // true`  


1. 传递性：`if (x.equals(y) && y.equals(z))  
x.equals(z); // true;`
 
1. 与null比较：对任何不适null的对象，x.euqals(null)都为空
>equals和==的比较：
>==的作用判断的两个对象的地址是否相等，即判断两个对象是否是同意对象（基本数据类型==比较的是值，引用类型比较的是内存地址）  
>equals比较的也是两个对象是否相等，但他有两种情况
>
- 情况一：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。  
  **比如String中的equals方法是被重写过的，所以比较的是值**   
  
hashcode()
---
hashcode返回散列值，其作用是获得哈希码，它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。
### 为什么要有hash code ###
#### 我们以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode： ####
当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals（）方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的Java启蒙书《Head fist java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。
#### hashCode（）与equals（）的相关规定 ####
1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个对象分别调用equals方法都返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）
#### 要是散列码冲突了怎么办？ ####


1. 开放寻址法：如果冲突了就去找下一个散列地址，只要散列表空间足够大
2. 在散列函数法：事先准备多个散列函数，一个有冲突了再去用下一个
3. 公共溢出法：为所有冲突的散列码找一块公共的溢出区来存放  

toString()
---
Object中的toString返回的是：默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。
public String toString()返回该对象的字符串表示。

clone()
---
要使用clone方法需要重写，且必要实现cloneable接口，不然会抛出CloneNotSupportedException异常
#### 浅拷贝 ####
拷贝对象和原始对象的引用类型引用同一个对象（实际上是同一个对象）。
#### 深拷贝 ####
拷贝对象和原始对象的引用类型引用不同对象（复制了一个新的对象）。

## 反射 ##
---
原理：每个类都有一个 Class 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。
类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。
通常借助一下四个类： 

- Class:类的对象
- Constructor：类的构造方法
- Field：类中的属性对象
- Method：类中的方法对象  
  
## 异常 ##
---
Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：  
  
- 受检异常 ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

## 泛型 ##
---
[http://www.importnew.com/24029.html](http://www.importnew.com/24029.html)

## 注解 ##
---
Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。  
[https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)  、
