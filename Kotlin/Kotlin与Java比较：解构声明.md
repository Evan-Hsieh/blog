### 解构声明
在现代语言中常常拥有结构声明用语简化代码，例如JavaScript与Kotlin，但是在Java中却没有此特性。为此下文以Kotlin作为例子：
```
// 解构声明
val (name, age) = person
// 使用解构声明创建的变量
println(name)
println(age)
```
这种语法称为 解构声明。即能够同时创建多个变量。在这个例子中，创建了变量name与age，并且可以独立使用这两个变量。实际上，一个解构声明会被编译成如下代码：
```
val name = person.component1()
val age = person.component2()
```
其中的 component1() 和 component2() 是在约定好的函数。同样也可以有component3()与component4()。实际上，任何表达式都可以出现在解构声明的右侧，只要可以对它调用所需数量的 component 函数即可。 

### 循环中使用解构声明
解构声明也可以用到for循环中，例如：
```
for ((key, value) in map) { …… }
```
变量 key 和 value 的值取自对集合中的元素上调用 component1() 和 component2() 的返回值。
开发者之所以能够直接在map对应的for循环中使用解构声明，是因为map类型在标准库中实现了componentN()方法。即：
```
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```
从这一标准库的实现来看，为了能够在for循环中使用解构声明，需要做到：
* 通过提供一个 iterator() 函数将map表示为一个值的序列；
* 通过提供函数 component1() 和 component2() 来将每个元素呈现为一对。

### 函数返回值用于解构声明
Kotlin在其数据类中，提供了一种简便的方式对值进行返回，其返回值可以用于解构声明。
```
data class Result(val result: Int, val status: Status)
fun function(……): Result {
    // 各种计算

    return Result(result, status)
}

// 现在，使用该函数：
val (result, status) = function(……)
```
因为数据类自动声明 componentN() 函数，所以这里可以用解构声明。

### 解构声明中下划线的作用
如果在解构声明中你不需要某个变量，那么可以用下划线取代其名称：
```
val (_, status) = getResult()
```
对于以这种方式跳过的组件，不会调用相应的 componentN() 操作符函数。

### lambda表达式中的解构声明
lambda 表达式中的参数类型可以有许多种，如果这些参数类型为 Pair类型、Map.Entry 或任何其他具有相应 componentN 函数的类型，那么可以通过将它们放在括号中实现一个可以被解构的参数。
```
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```
注意声明两个参数和声明一个解构对来取代单个参数之间的区别：
```
{ a //-> …… } // 一个参数
{ a, b //-> …… } // 两个参数
{ (a, b) //-> …… } // 一个解构对
{ (a, b), c //-> …… } // 一个解构对以及其他参数
```
如果解构的参数中的一个组件未使用，那么可以将其替换为下划线，以避免编造其名称：
```
map.mapValues { (_, value) -> "$value!" }
```
可以指定整个解构的参数的类型或者分别指定特定组件的类型：
```
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }
map.mapValues { (_, value: String) -> "$value!" }
```








