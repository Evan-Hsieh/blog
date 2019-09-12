
### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 扩展方法
在编程实践中，开发者往往希望增加某个类对象的行为，这时就涉及到了扩展。常见的扩展方式可以通过继承与被称为装饰器的设计模式。以继承方式为例，在Java中是这样实现的：

* Java
例如希望在一个只有加法的计算器中，新增乘法，那通过继承的方式是这样的：
```
public class NewCalculator extends Calculator{
    public boolean multi(int x, int y) {
        return x * y;
    }
}
```
通过自定义一个新的类去继承原有的类，并且在新的类中扩展方法。在这之后，还需要将初始化或声明原本类的地方都修改为这个新的类。如果涉及的地方较多，所消耗的时间和精力不容小觑。

同样，例如在Android开发中，需要为某个布局的方法添加一行日志，需要新建一个自定义的类继承该布局，并覆写方法。对应的xml布局文件也要修改。引用该布局对应类的地方也要做修改。由此可见，想要在Java中进行扩展是十分耗时耗力的。



* Kotlin
为了解决Java中扩展的麻烦，Kotlin提供了更为容易的扩展机制。其不仅支持方法扩展，还支持属性扩展。下面以方法扩展为例子：
```
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```
扩展方法的语法为：
```
fun 类名.扩展方法名（参数列表）{
    //方法体
}
```
调用它的例子为：
```
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // “swap()”内部的“this”得到“l”的值
```

### 扩展方法是静态解析的
* Java
在Java中，通过继承或装饰器实现的扩展，都是动态解析的。即子类对象会调用子类的方法：
```
class Calculator{
    public boolean add(int x, int y) {
        return x + y;
    }
}

class NewCalculator extends Calculator{
    public boolean add(int x, int y) {
        return x + y + 1;
    }
}

class Executor{
    public void exec(Calculator cal){
        cal.add(1,1);
    }
}

public static main(String[] args){
    new Executor().exec(new NewCalculator()); //结果为3
}
```
即子类对象，肯定会调用子类中的覆盖方法。

* Kotlin
在Kotlin中，扩展方法却不是动态的，而是静态解析的。其例子为：
```
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```
按照以理解Java代码的思路，应该打印d,但实际上，该例子最终会输出c。原因是printFoo方法的参数类型是C，故扩展方法对应的也是C的扩展方法。即扩展方法是静态解析的。

### 扩展声明为成员
* Java

* Kotlin
一般的情况下，是在顶层文件中声明扩展。但是。在一个类内部你可以为另一个类声明扩展。在这样的扩展内部，有多个 隐式接收者 —— 其中的对象成员可以无需通过限定符访问。扩展声明所在的类的实例称为 分发接收者，扩展方法调用所在的接收者类型的实例称为 扩展接收者。

```
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
     fun printPort() { print(port) }

     fun Host.printConnectionString(p: Int) {
         printHostname()   // calls Host.printHostname()
         print(":")
         printPort()   // calls Connection.printPort()
     }

     fun connect() {
         /*……*/
         host.printConnectionString(port)   // calls the extension function
     }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    //Host("kotl.in").printConnectionString(443)  // error, the extension function is unavailable outside Connection
}
```
若某个扩展的声明时在类的内部（即作为类的成员），它同样可以声明被open，并且被子类覆盖。下面的例子展示了函数的分发对于分发接收者类型是虚拟的（多态的），但对于扩展接收者类型是静态的。

```
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // 调用扩展函数
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   // "Base extension function in BaseCaller"
    BaseCaller().call(Derived())   // "Base extension function in BaseCaller"
    DerivedCaller().call(Base())  // "Base extension function in DerivedCaller" - dispatch receiver is resolved virtually
    DerivedCaller().call(Derived())  // "Base extension function in DerivedCaller" - extension receiver is resolved statically
}
```
BaseCaller和DerivedCaller是分发者，其调用的具体是哪个扩展方法是多态的。但是Base和Derived作为扩展接受者，其扩展方法的调用是静态的。这个区别需要注意。

