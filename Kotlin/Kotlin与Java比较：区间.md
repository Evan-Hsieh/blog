### 前言
区间来自于数学的概念，只一段可以比较的值的范围。程序语言也不同程度地支持区间。
* 支持用变量表示区间
* 支持判断区间范围
* 支持迭代区间
* 支持对区间的操作

### 用变量表示区间
在Java中不支持用变量表示一个区间，但在Kotlin中支持。
* Kotlin

闭区间：通过..操作符表示：
```
//闭区间[2,1024]
val aRange: IntRange = 2..1024
val bRange = 2..1024
```
半闭区间：通过util操作符表示：
```
//半闭区间[0,1024) = [0,1023]
val cRange: IntRange = 0 until 1024
```

### 判断是否在区间范围
* Java
Java通过对多个表达式进行逻辑运算来判断：
```
if(0 < x && x < 100)
```

* Kotlin
通过in或!in操作符来实现：
```
if (i in 1..10) { // 等同于 1 <= i && i <= 10
    println(i)
}
```

### 迭代区间元素
* Java
对于Java集合类型,通过冒号来实现：
```
List<String> items = new ArrayList<>();
items.add("A");
items.add("B");

for(String item : items){
    System.out.println(item);
}
```

* Kotlin
通过in来实现：
```
for (i in 1..4) print(i)
```
另外在循环中也支持downTo、util、step
```
for (i in 4 downTo 1) print(i)    //4321
for (i in 1..4 step 2) print(i)    //13
for (i in 4 downTo 1 step 2) print(i)  // 42
for (i in 1 until 10) println(i)   //i in [1, 10) 排除了 10
```

### 对区间进行操作
在Java中不支持将区间保存在变量中，故也不能对其进行操作。但在Kotlin中，区间可以被理解为是数列（Progression），对于一个数列自然可以用一个变量表示，并且对其进行操作。

* Kotlin中reverse方法
属于*Progression数列类的方法，用于反转数列

* Kotlin中的downTo方法
为任何整型类型定义，表示区间有端点。

* Kotlin中的step方法
同样属于*Progression数列类的方法。表示设置步长。要求步长必须为正数。

通过对一个数列设置step后，其返回的数列的last值与原始数列可能不同,以便保持不变式 (last - first) % step == 0 成立。
```
(1..12 step 2).last == 11  // 值为 [1, 3, 5, 7, 9, 11] 的数列
(1..12 step 3).last == 10  // 值为 [1, 4, 7, 10] 的数列
(1..12 step 4).last == 9   // 值为 [1, 5, 9] 的数列
```

> rangeTo属于*Range类方法