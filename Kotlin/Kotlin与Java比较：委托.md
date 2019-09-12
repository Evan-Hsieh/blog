### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 委托
委托是软件设计中的一个设计模式，即一个请求传递给一个对象，但是这个对象将这个请求交给另一个对象处理，这就是委托。在Java中需要通过一些模板方法去实现委托，即设计模式中的委托模式。在Kotlin中直接提供了语言的支持，通过极为简单的方式即可实现委托。

* Java
Java中的委托是一个对象A持有另一个对象B，对客户端可见的只有对象A。当调用A的某个方法时，实际上对象A是调用对象B的方法来执行的。这就是Java委托的基本思想。
下面的例子是有一个打印机的类Printer，其委托另一个真实的打印机的类去完成任务。
```
 class RealPrinter { // the "delegate"
     void print() { 
       System.out.print("something"); 
     }
 }
 
 class Printer { // the "delegator"
     RealPrinter p = new RealPrinter(); // create the delegate 
     void print() { 
       p.print(); // delegation
     } 
 }
 
 public class Main {
     // to the outside world it looks like Printer actually prints.
     public static void main(String[] args) {
         Printer printer = new Printer();
         printer.print();
     }
 }
```
在上面的例子中，可以发现两个类都有一个print方法，Print类持有RealPrinter的对象，当调用Printer类对象的print方法时，实际会调用RealPrenter的print方法，实现委托。

* Kotlin
通过关键字by来实现委托：
```
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```
Derived 的超类型列表中的关键字by以及其后的b，表示会在其内部存储b. 与上文Java的例子比较，b就对应于被委托的对象，这个对象会被存储，类似于Printer存储了RealPrinter的对象。并且，Kotlin编译器会给Derived对象自动生成b对象中所有对应的方法。（在Java例子中是显式实现对应方法的）

Kotlin发起委托的类也是可以有类体，类体中的方法可以覆写接口的方法。如果其覆写了接口的方法，那就不会调用被委托对象的方法了

interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()    //打印abc,而不是10
}