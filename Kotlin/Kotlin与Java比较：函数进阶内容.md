### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 程序语言实体的等级
* 一等公民
在程序语言中，一等公民是指支持所有操作的实体，包括赋值，作为参数传递，作为返回值。例如Java与Kotlin中的对象都是一等公民。

* 二等公民
二等公民表示仅能作为参数传递的实体。

* 三等公民
既不能作为参数，也不能赋值或作为返回值的实体。

在Java 8之前，函数是三等公民，既不能作为参数传入，也不能赋值或作为返回值。在Java 8以及其后更高版本中，函数的地位提高了一些，接近于二等公民，其可以作为参数传递给函数。之所以说Java 8以后的函数只是接近于二等公民，是由于它的使用是有条件的（下文会详细说明）。

在Kotlin中，函数是一等公民，可以任意赋值、参数传递与作为返回值。

### 函数类型
函数如果能够赋值、作为参数传递与返回值，则也就是将其视为‘对象’了。‘对象’自然需要有相对应的类型，这就是函数类型。在Java 8中由于函数不是一等公民，自然也没有函数类型的概念。但是在Kotlin中，有函数类型的概念。下面关于函数类型的讨论，都仅与Kotlin有关。最简单的函数类型的例子为：
```
(Int,Int) -> Int
```
这就是一个函数类型。对于每一个函数类型，其可以对应不同的实例，即不同函数。
```
fun plus(x:Int, y:Int):Int = x + y
```
对于这个函数，就是上文中函数类型 (Int,Int) -> Int 的一个实例。

#### 函数类型若干表示法
* (A, B) -> C 表示接受类型分别为 A 与 B 两个参数并返回一个 C 类型值的函数类型。 参数类型列表可以为空，如 () -> A。但是 Unit 返回类型不可省略。
* 函数类型可以有一个额外的接收者类型，它在表示法中的点之前指定： 类型 A.(B) -> C 表示可以在 A 的接收者对象上以一个 B 类型参数来调用并返回一个 C 类型值的函数。 后面会做详细介绍。
* 挂起函数属于特殊种类的函数类型，它的表示法中有一个 suspend 修饰符 ，例如 suspend () -> Unit 或者 suspend A.(B) -> C。


#### 函数类型若干特点
* 函数类型表示法可以选择性地包含函数的参数名：(x: Int, y: Int) -> Int , 这些名称可用于表明参数的含义。
* 如需将函数类型指定为可空，请使用圆括号包围函数类型，并在末尾添加问号，例如：((Int, Int) -> Int)?
* 函数类型可以使用圆括号进行接合：(Int) -> ((Int) -> Unit)
* 箭头表示法是右结合的，(Int) -> (Int) -> Unit 与前述示例等价，但不等于 ((Int) -> (Int)) -> Unit。

有时使用函数类型时，名称比较长，难以书写，则可以通过使用类型别名给函数类型起一个别称：
```
typealias ClickHandler = (Button, ClickEvent) -> Unit
```
typealias是关键字，声明后可以使用ClickHandler表示对应的函数类型。

### Lambda表达式
* Java
在Java 8之后，也支持Lambda表达式，但是它的使用是有条件的。即只能在函数式接口（Funtional Interface）中使用。函数式接口是指只有一个抽象方法的接口，例如：
```
public interface MyFP {    //函数式接口
     int plus();
}
```
在该接口中，只有一个抽象方法。这就是函数式接口。但是，如果有一个以上的抽象方法，则不是函数式接口。例如：
```
public interface NotFP {    //不是函数式接口
     int plus();
     int minus();
}
```
在函数式接口中，可以添加@FunctionalInterface注解，保证该接口不会添加多余的抽象方法，同时函数式接口允许添加非抽象的方法，如静态方法：
```
@FunctionalInterface
interface MyFP {     //函数式接口
     //这是一个抽象方法
     int plus(int x, int y);
     //静态方法，不是抽象方法
     static void staticMethod() {
           System.out.println("接口里的静态方法！");
     }
```

对于一个函数式接口，可以传递一个Lambda表达式给它作为函数式接口的对象。例如一个方法接受该函数式接口的对象作为参数：
```
public static void process(MyFp myFP){
    //xxx
}
```
传递一个Lambda表达式给它作为函数式接口的对象：
```
process((x,y) -> {return x + y;})
```
这等同于创建了实现了同样抽象方法的类型实例，即等同于：
```
process(new MyFP(){
    @Override
    int plus(int x,int y){
        return x + y;
    }
});
```

在Java中，Lambda表达式具有多种形式，最基本的形式为：
```
(参数列表) -> {表达式或语句}
```
例如：
```
(int x, int y) -> {return x + y;}
(int x) -> {return x * 2;}
```
当表达式类型可以推断时，则可以省略变量类型：
```
(x, y) -> {return x + y;}
(x) -> {return x * 2;}
```
对于参数列表只有一个参数的情况，则可以省略圆括号，但是无参数情况下，不能省略：
```
x -> {return x * 2;}
() -> {print("finish");}
```
对于只有一条语句的情况，则可以省略大括号，即：
```
x -> return x * 2
() -> print("finish")
```
对于只有一条语句的情况下，返回值关键字return也可以省略
```
x -> x * 2
```
> Java的Lambda表达式可以使用外部函数的局部参数，但是需要是final的，且不更改它的值。

* Kotlin
在Kotlin中，其Lambda表达式与Java中的有一点不同。在如下例子中可以看出区别：
```
// Java
(int x,int y) -> {return x + y;}
// Kotlin
{x:Int, y:Int -> return x + y}
```
区别在于，Java中使用圆括号限定参数，用大括号限定语句。在Kotlin中，参数与语句都被包含在大括号中。这是形式的区别。另外，Kotlin中的Lambda表达式并不限定只能在函数式接口（只有一个抽象方法的接口）中使用。
> 注意，在Kotlin中，函数类型的表示是小括号把参数括起来，如(Int,Int) -> Int

Kotlin的Lambda表达式中，如果推断出其返回值不是Unit，则会以最后一个表达式作为lambda的返回值：
```
{x:Int, y:Int -> 
    x + y
    x } //x是返回值
```
在使用Lambda表达式时，有这样的例子：
```
// 该方法接受一个Int型的参数x, 同时接受一个函数类型为 (Int,Int) -> Int 的函数作为参数。
fun receive(cons:Int, method:(Int,Int) -> Int):Int{
    //...
}
// 使用Lambda表达式
receive(1,{x,y -> x + y})
```
如果函数接受的参数中，最后一个参数是函数类型，则在使用时，可以将Lambda表达式放到圆括号之外。对于上面的例子，可以改写为：
```
receive(1){x,y -> x + y}
```
如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略：
```
run { println("...") }
```

对于只有一个参数的Lambda表达式，有更为简便的约定，有例子如下：
```
fun receive(method:(Int) -> Boolean):Boolean{
    //...
}
```
即该方法接受一个函数，该函数的函数类型是 (Int) -> Boolean。在这种情况下，传入Lambda表达式时，可以省略 -> ,并且通过隐式对象it来表示这个Int型对象。例如为：
```
receive(it > 0)
receive{it > 0}
```
如果 lambda 表达式的参数未使用，那么可以用下划线取代其名称：
```
map.forEach { _, value -> println("$value!") }
```

### 匿名函数
匿名函数，即没有名称的函数。在Java中没有匿名函数的概念，故此部分的介绍都基于Kotlin。

在Kotlin中，匿名函数例子为：
```
fun(x: Int, y: Int): Int = x + y
```
这与普通看起来很像，也的确如此，它仅仅是将声明普通函数时的函数名省略了。匿名函数的返回类型推断机制与正常函数一样：对于具有表达式函数体的匿名函数将自动推断返回类型，而具有代码块函数体的返回类型必须显式指定（或者已假定为 Unit）。

匿名函数往往会与Lambda表达式一起讨论，一些网上的教材会将两者等同。在Kotlin官网教程中，对两者有如下描述：
> lambda 表达式与匿名函数是“函数字面值”，即未声明的函数， 但立即做为表达式传递

按照笔者的理解，匿名函数即就是没有名称的函数，Lambda表达式是匿名的，但是不等同于匿名函数。它们有如下若干区别：
* Lambda表达式是定义在大括号中的，但是匿名函数是类似于定义函数的方式来定义的
* Lambda表达式在参数末尾时，可以移动到圆括号之外，但是匿名函数不可以。
* 对于匿名函数中的return语句，其表示从匿名函数自身返回。但是Lambda表达式中的return语句表示从包含它的函数中返回。

### 闭包
按照Kotlin官网教程中的说法：
> Lambda 表达式或者匿名函数（以及局部函数和对象表达式） 可以访问其 闭包 ，即在外部作用域中声明的变量。 

* Java
若参考此定义，则Java也支持闭包。即在Lambda表达式中也能访问函数的变量，尽管不能修改它们。
```
static IntFunction<Integer> sumEx(int a) {
    //  定义局部变量
    final int ex = a * 10;
    return (value) -> {
        //  引用局部变量
        return ex + value;
    };
}
```

* Kotlin
Kotlin中可以修改闭包的变量
```
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### 带有接受者的函数类型
前面介绍了最为普通的函数类型如：(A,B) -> C ,除此之外，还有带有接收者的函数类型，例如 A.(B) -> C。
在这样的函数类型的函数中，可以隐式访问这个接受者对象，即this关键字。例如对于Int类有一个方法plus,以它作为接受者的函数类型的函数，可以直接调用plus方法：
```
val sum: Int.(Int) -> Int = { other -> plus(other) }
```

对于匿名函数，也支持类似的用法。匿名函数语法允许开发者直接指定函数字面值的接收者类型。 
```
val sum = fun Int.(other: Int): Int = this + other
```
这个例子与上面的例子类似。

带与不带接收者的函数类型非字面值可以互换，其中接收者可以替代第一个参数，反之亦然。例如，(A, B) -> C 类型的值可以传给或赋值给期待 A.(B) -> C 的地方，反之亦然：
```
val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
val twoParameters: (String, Int) -> String = repeatFun // OK

fun runTransformation(f: (String, Int) -> String): String {
    return f("hello", 3)
}
val result = runTransformation(repeatFun) // OK
```
> 请注意，默认情况下推断出的是没有接收者的函数类型，即使变量是通过扩展函数引用来初始化的。 如需改变这点，请显式指定变量类型。


### 函数类型实例化
前面介绍了若干函数类型，以及其实例。现对其进行总结，有如下若干方法可以获得函数类型实例：
#### 使用函数字面值的代码块
采用以下形式之一的代码块：
* lambda 表达式: { a, b -> a + b },
* 匿名函数: fun(s: String): Int { return s.toIntOrNull() ?: 0 }
* 带有接收者的函数字面值可用作带有接收者的函数类型的值。

#### 使用已有声明的可调用引用
* 顶层、局部、成员、扩展函数：::isOdd、 String::toInt，
* 顶层、成员、扩展属性：List<Int>::size，
* 构造函数：::Regex
这包括指向特定实例成员的绑定的可调用引用：foo::toString。

#### 使用实现函数类型接口的自定义类的实例
```
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```
