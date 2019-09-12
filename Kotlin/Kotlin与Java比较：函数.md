### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 函数声明
* Java
Java中没有专有的关键字用于声明函数，它是通过约定好的格式来声明函数的，即
```
[访问控制符] 返回值类型 方法名([参数列表]){
    方法体
}
```
即返回值类型，方法名，方法体必不可少，这样的形式组合在一起就是函数声明。如：
```
public int plus(int x, int y){
    return x + y;
}
```

* Kotlin
由于在Kotlin中语言的书写方式更为灵活，许多部分可以省略，为此需要通过关键字fun来定义函数。即编译器不用通过约定的书写格式去判断函数，而是通过关键字fun去判断，而书写格式则可以灵活多变。
```
fun 方法名([参数列表]) [:返回值类型][方法体]
```
简单的例子：
```
fun double(x: Int): Int {
    return 2 * x
}
```

> Kotlin编程规范：Note: In Kotlin, semicolons are optional, and therefore line breaks are significant. Omit semicolons whenever possible.
> [Kotlin Style Guide](https://developer.android.com/kotlin/style-guide)

### 函数参数
在程序语言中，函数参数可以分为两类，分别是位置参数与命名参数。

对于位置参数，即一个函数有多个参数时，它通过位置来判断哪一个值对应哪一个参数。Java函数仅支持位置参数。

对于命名参数，即通过参数的名字来判断哪一个参数值与参数对应。Kotlin既支持单独使用任意一种参数传递方式，也支持同时使用两种方式。

* Java
Java中使用位置参数调用函数的例子：
```
public int minus(int x, int y){
    return x - y;
}

// 调用时
minus(3,1);
```
即当调用minus(3,1)方法时，第一个参数值3对应第一个参数x,第二个参数值1对应第二个参数y。

* Kotlin
Kotlin中使用位置参数的例子：
```
fun minus(x: Int, y: Int): Int {
    return x - y
}

minus(3,1)
```
使用命名参数的例子:
```
minus(x=3,y=1) //结果为2
minus(y=1,x=3) //结果为2
```
由于使用了命名参数，所以位置可以任意调整。

### 默认参数
在Java中不支持默认参数，但在Kotlin中支持默认参数，这带来的好处就是可以书写应用更为广泛的方法，同时避免了使用重载，减少了代码量。

* Java
Java不支持默认参数，若要实现相应的功能，必须使用重载的方式，在一个方法的方法体中初始化默认值，然后将该默认值传递给另一个。例如
```
public float getGravity(float m, float g){
    return m * g;
}

public float getGravity(float m){
    float g = 9.8;
    return getGravity(m,g);
}
```
从上面的例子可以看到，第一个方法是传入物体质量与重力常量，来计算物体所受的重力。第二个方法重载了第一个方法，只传入了质量m，但在其中使用了重力常量g的默认值9.8。

* Kotlin
Kotlin中提供了极为简单的方式来实现默认参数，避免了重载带来的多余代码。
```
fun getGravity(m:Float, g:Float = 9.8){
    return m * g
}
```
通过该例子可以看到，使用默认参数非常简单，只需要在参数列表中添加‘=赋值表达式’即可。

对于继承的情况，覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值,子类的该方法中使用与基类该方法一样的默认值。不省略该默认参数值的话，IDE会报错：
```
open class A {
    open fun foo(i: Int = 10) { …… }
}

class B : A() {
    override fun foo(i: Int) { …… }  // 不能有默认值
}
```
如果一个默认参数在一个无默认值的参数之前，那么该默认值只能通过使用命名参数调用该函数来使用：
```
fun foo(bar: Int = 0, baz: Int) { …… }

foo(baz = 1) // 使用默认值 bar = 0
```
当一个函数调用混用位置参数与命名参数时，所有位置参数都要放在第一个命名参数之前。例如，允许调用 f(1, y = 2) 但不允许 f(x = 1, 2)。

### 函数无返回值
* Java
在Java中若无返回值，需要将返回值类型标记为void类型，如
```
public void fun(){...}
```
* Kotlin
Kotlin的函数不能没有返回值，若函数无需返回有用的值，则可以返回Unit类型，该类型只有一个值，即Unit，这个值不需要显式返回。
```
fun printHello(name: String?): Unit {
    // `return Unit` 或者 `return` 是可选的
}
```
返回类型Unit声明也是可选的。上面的代码等同于：
```
fun printHello(name: String?) { …… }
```

### 单表达式函数
* Java
在Java中不支持单表达式函数，更不能将函数赋值给变量。

* Kotlin
支持单表达式函数。当函数返回单个表达式时，可以省略花括号并且在 = 符号之后指定表达式即可：
```
fun double(x: Int): Int = x * 2
```
当返回值类型可由编译器推断时，显式声明返回类型是可选的
```
fun double(x: Int) = x * 2
```
> * 单表达式函数通过 = 号分隔
> * 单表达式函数若与花括号并用，返回的是lambda表达式
> * 单表达式不能与return混用
> * 花括号表示的函数若没有return语句，返回的是Unit
> * 花括号表示的函数若使用return语句，需要申明函数返回值(Unit的返回值除外)
下面是一些有趣的一些例子
```
fun double1(x : Int)  = x * 2 // 返回值符合预期

fun double2(x : Int)  = {x * 2} // 返回 () -> kotlin.Int

// fun double4(x : Int) = {return x * 2} // 报错

fun double3(x : Int) {x * 2} // 返回 kotlin.Unit

// fun double5(x : Int) {return x * 2} // 报错

fun double6(x : Int): Int {return x * 2} // 返回值符合预期
```


### 可变数量参数
#### 可变数量参数基础
可变数量参数即表示函数接受的参数数量是可以变化的。
* Java
通过在参数类型后添加...表示可变数量参数，如：
```
public static void dealArray(int... intArray){  
    for (int i : intArray){xxx}
}     
```
调用该函数时，可以有如下多种情形：
```
dealArray();  
dealArray(1);  
dealArray(1, 2, 3);  
```
即可以不传递参数，也可以传递1个或多个参数。

* Kotlin
通过在参数名前面添加关键字vararg来声明可变类型参数
```
fun <T> asList(vararg ts: T): List<T> {
    for (t in ts){...}
}
// 使用时
val list = asList(1, 2, 3)
```

#### 可变数量参数与数组
* Java
对于可变数量参数的函数，是兼容数组的，例如：
```
public static void dealArray(int... intArray){  //参数是可变数量的
    for (int i : intArray){xxx}
}     
// 使用时
int[] intArray = {1, 2, 3};        
dealArray(intArray);  //通过编译，正常运行  
```
但是反过来，对于数组参数的函数，是不兼容可变参数的，例如：
```
public static void dealArray(int[] intArray){  //参数是数组
    for (int i : intArray){xxx}
}     
// 使用时
dealArray(1,2,3);  //编译错误
```

* Kotlin
在Kotlin中，可变参数也是对数组兼容的。同时，它还支持对可变参数与数组的同时使用：
```
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```
即在数组名之前加上*，可以作为参数传递给拥有可变数量参数的函数。

#### 可变数量参数的位置
* Java
在Java中，可变数量参数必须位于参数列表的最后一项，如下例子则会报错：
```
public static void dealArray(int... intArray, int count) //编译报错
```
* Kotlin
Kotlin不强制要求可变数量参数位于末尾，但如果其vararg参数不是最后一个参数，则其后的其他参数需要用命名参数的形式来传递。

### 函数作用域
* Java
不同语言可定义函数的范围不同，Java中只能在类中定义一个函数，即成员函数，不管该函数是否是静态的。

* Kotlin
Kotlin中定义函数的范围更为广泛，可以是在顶层文件范围内，可以是类中定义成员函数，也可以是局部作用域中（如函数中定义函数），也可以是扩展函数。

顶层文件范围内定义函数，即函数可以在类的外部，与类“平级”：
```
class A(x: Int)

fun method(){...}
```
成员函数非常常见，但对于局部作用域定义函数，其例子为：
```
fun method() {
    fun innerMethod() {
    }
}
```
局部函数可以访问外部函数（即闭包）的局部变量，所以在上例中，visited 可以是局部变量：
```
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return    // 已存在该元素则返回false. 局部函数可以访问外部函数的visited变量。
        for (v in current.neighbors)
            dfs(v)
    }
    dfs(graph.vertices[0])
}
```


