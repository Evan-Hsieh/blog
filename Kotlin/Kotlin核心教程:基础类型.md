
Kotlin中所有东西都是对象，它的基本类型包括：布尔值、数字、字符、字符串和数组

### 基本类型
#### 布尔值
Kotlin中用Boolean表示布尔类型，其值分别为true与false.
支持与或非三种操作。

#### 数字
数字支持用若干内置类型表示，分别是：Double、Float、Long、Int、Short与Byte。
数字字面量支持用下划线表示，让数字更易读
```
val oneMillion = 1_000_000
```
* Kotlin中字符不是数字。

#### 字符
字符用Char表示,字符字面量要用单引号包围，如：'c'

#### 字符串
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

#### 数组
* Kotlin数组
用Array表示，定义了get、set与size方法。Array为不可变类型。
创建数组可通过arrayOf或Array构造函数。
```
arrayOf(1,2,3)
```
* 原生数组
Kotlin也提供了无装箱开销的原生数组，IntArray,ByteArray,ShortArray。与Array没有继承关系。使用intArrayOf()进行初始化。

