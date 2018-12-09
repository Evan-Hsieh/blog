
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
即通过自定义一个新的类去继承原有的类，并且在新的类中扩展方法。在这之后，还需要将初始化或声明原本类的地方都修改为这个新的类。如果涉及的地方较多，所消耗的时间和精力不容小觑。

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
