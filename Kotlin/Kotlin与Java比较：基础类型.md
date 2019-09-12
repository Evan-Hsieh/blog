
### 概述
Java中提供了8中基本类型，6种数字类型（四个整数型，两个浮点型），1种字符类型，还有一种布尔型。
Kotlin中所有东西都是对象，它的基本类型包括：布尔值、数字、字符、字符串和数组

### 基本类型

#### 布尔值
* Java
Java中使用boolean作为布尔型，有true和false两种取值。

* Kotlin
Kotlin中用Boolean表示布尔类型，其值分别为true与false.
支持与或非三种操作。若需要可空引用，则会被装箱。

#### 数字
* Java
支持6中数字类型，分别是byte、short、int、long、double、float.

* Kotlin
数字支持用若干内置类型表示，分别是：Double、Float、Long、Int、Short与Byte。

| 类型 | 位宽度 |
| :--: | :--:|
| Double | 64 |
| Float | 32 |
| Long | 64 |
| Int | 32 |
| Short | 16 |
| Byte | 8 |

数字字面量支持用下划线表示，让数字更易读
```
val oneMillion = 1_000_000
```
同时也支持传统符号表示的浮点数值：
```
123.5e10
```

在Java中，我们经常需要进行装箱和非装箱，例如：
```
Integer i1 = new Integer(1);
int i2 = 1;
```
但是，Kotlin不区分装箱和非装箱类型，在一般情况下，对于允许为空的类型，编译器会自动对其进行装箱。

##### 数字比较
* Java
Java中，使用两个等号 == 比较整数型数字。

注意，不能使用 == 比较浮点数字。 因为编程语言在计算是会有误差，故使用 == 比较是不可靠的。若在要求不是特别精确的场景，可以在比较两浮点数时，两数之差小于一个非常小的范围，即可认为相等。若要在商业场景中做精确计算，建议使用BigDecimal类。

* Kotlin
在 Kotlin 中，三个等号 === 表示比较对象地址，两个 == 表示比较两个值大小。
```
fun main(args: Array<String>) {
    val a: Int = 10000
    println(a === a) // true，值相等，对象地址相等

    // 创建了两个不同的对象
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a

    //值是相等的，都是10000
    println(boxedA === anotherBoxedA) //  false，值相等，对象地址不一样
    println(boxedA == anotherBoxedA) // true，值相等
}
```
> 当使用 == 比较对象时，若两者其一为null，则返回false。若两者均为null，则返回TRUE。

* Kotlin中字符不是数字。

#### 字符
* Java
Java中字符类型为char，使用单引号括起来的为字符。可以提升为int型。

* Kotlin
字符用Char表示,字符字面量要用单引号包围，如：'c'
不同于Java的是，字符不属于数值类型，是一个独立的数据类型。
```
fun check(c: Char) {
    if (c == 1) { // 错误：类型不兼容
        // ……
    }
}
```

#### 字符串
* Java
字符串类型为String，为不可变类型。


* Kotlin
字符串用String表示，索引字符串中的字符用s[i]. Kotlin字符串与Java一样可以用 + 拼接字符串。
在字符串中迭代字符：
```
for (c in str) {
    println(c)
}
```

字符串字面值分为：转义字符串与原始字符串
* 转义字符串
与Java字符串类似，在其中可以有诸如\n等转义字符
* 原始字符串
用三引号包围，内部没有转义，且能包含换行以及其他任意字符

字符串两种字面量都支持字符串模板，模板有两种类型：
* 变量型
```
val i = 10
println("i = $i") // 输出“i = 10”
```
* 表达式型
```
val s = "abc"
println("$s.length is ${s.length}") // 输出“abc.length is 3”
```

##### 字符串长度
```
val str = "kotlin very good"
 
// 1. 直接用length属性获取
println("str.length => ${str.length}")
 
// 2. 用count()函数获取
println("str.count() => ${str.count()}")
```
##### 字符串查找
* 查找第一个元素
```
    val str = "kotlin very good"
    println(str[0])
    println(str.get(0))
    println(str.first())
```
类似的可以通过last查找最后一个元素。其他还有firstOrNull()等函数。

* 查找对应元素下标
```
indexOf()
indexLastOf()
```



##### 字符串截取
```
val str = "Kotlin is a very good programming language"
 
println("s = ${str.substring(10)}")  // 当只有开始下标时，结束下标为length - 1
println(str.substring(0,15))
println(str.substring(IntRange(0,15)))
```
类似的还有subSequence()函数，其大致和subString()函数一样，但是其不提供只传递startIndex的情况

##### 字符串替换
replace()函数提供了4个重载函数。
* replace(oldChar,newChar,ignoreCase = false)
* replace(oldValue,newValue,ignoreCase = false)
* replace(regex,replacement)
* replace(regex: Regex, noinline transform: (MatchResult) -> CharSequence)

其他的还有诸如replaceBefore(), replaceBeforeLast()等函数

##### 验证字符串
* isEmpty() : 其源码是判断其length是等于0，若等于0则返回true,反之返回false。不能直接用于可空的字符串
* isNotEmpty() : 其源码是判断其length是否大于0，若大于0则返回true,反之返回false。不能直接用于可空的字符串
* isNullOrEmpty() : 其源码是判断该字符串是否为null或者其length是否等于0。
* isBlank() : 其源码是判断其length是否等于0,或者判断其包含的空格数是否等于当前的length。不能直接用于可空的字符串
* isNotBlank() : 其源码是对isBlank()函数取反。不能直接用于可空的字符串
* isNotOrBlank() : 其源码判断该字符串是否为null。或者调用isBlank()函数


#### 数组
* Java数组
可以通过如下方式声明数组
```
dataType[] arrayRefVar;   // 首选的方法
dataType arrayRefVar[];  // 效果相同，但不是首选方法
```
Java支持多维数组。
```
String str[][] = new String[3][4];
```
同时Java也提供java.util.Arrays类来操作数组，它提供的所有方法都是静态的。
具有以下功能：
1. 给数组赋值：通过 fill 方法。
2. 对数组排序：通过 sort 方法,按升序。
3. 比较数组：通过 equals 方法比较数组中元素值是否相等。
4. 查找数组元素：通过 binarySearch 方法能对排序好的数组进行二分查找法操作。


* Kotlin数组
用Array表示，定义了get、set与size方法。由于使用 [] 重载了 get 和 set 方法，所以我们可以通过下标很方便的获取或者设置数组对应位置的值。Array为不可变类型。
创建数组可通过arrayOf或Array构造函数。
```
fun main(args: Array<String>) {
    //[1,2,3]
    val a = arrayOf(1, 2, 3)
    //[0,2,4]
    val b = Array(3, { i -> (i * 2) })

    //读取数组内容
    println(a[0])    // 输出结果：1
    println(b[1])    // 输出结果：2
}
```
* 原生数组
Kotlin也提供了无装箱开销的原生数组，IntArray,ByteArray,ShortArray。与Array没有继承关系。使用intArrayOf()进行初始化。

