

### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 类的声明
* Java

使用class声明类，由类名、类头（类头为通过extends等关键字构成的类型参数等），类体（花括号包围的部分）组成。皆不可省略。

* Kotlin

使用class声明类，由类名，类头（类型参数与主构造函数等）与类体（花括号包围的部分）组成。类头和类体都可以省略。

### 构造函数
* Java
可以有0个或多个构造函数。构造方法名需与类名相同。构造方法没有返回值。多个构造函数之间可以重载，没有主次关系，都是平级的。不同于Kotlin，在Java的类头中没有构造函数。

* Kotlin
可以有0个或1个主构造函数以及0个或多个次构造函数。主构造函数就在类头中，次构造函数通过constructor关键字声明。一个简单的主构造函数的例子：
```
class Person constructor(firstName: String) { ... }
```
若主构造函数没有任何注解或者可见性修饰符，可以省略constructor关键字。添加次构造函数的例子为：
```
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```
在类头中(val name: String)即为主构造函数，类体中的constructor声明的函数为次构造函数。

> 如果一个非抽象类中没有任何主或次构造函数，会自动生成一个不带参数的主构造函数。

### 类的初始化
* Java
类的初始化通过构造函数进行。

* Kotlin
类的初始化通过初始化块进行。由于主构造函数中不能有任何代码，故初始化工作由init声明的初始化块完成，且初始化块可以认为是主构造函数的一部分，优先于次构造函数执行。
```
class Person(name: String) {
    val customerName = name.toUpperCase()
    init {
        println("Init block" + name)
    }

    constructor(name: String, i: Int) : this(name) {
        println("Constructor")
    }
}
```
在上面的代码中，init块中的代码会优先于次构造函数执行，并且在主构造函数中的参数可以在init块中使用，也可以在类体中声明的属性使用。

### 类的使用
* Java
通过new关键字来创建实例

* Kotlin
没有new关键字，直接使用类名与构造函数参数进行实例创建
```
val invoice = Invoice()
val customer = Customer("Joe Smith")
```

### 类的继承
对于没有直接声明继承基类的类，会隐式继承。
* Java
隐式继承自Object.

* Kotlin
隐式继承自Any，Any不是java.lang.Object.它除了 equals()、hashCode() 与 toString() 外没有任何成员。

若要显示继承一个类，其方法为：
* Java
通过extends关键字进行继承，被继承的基类要求没有final修饰。

* Kotlin
通过将基类类型放到子类类头的冒号之后，被继承的基类要求有open修饰。
```
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```
在Kotlin中，类默认是final的。

### 覆盖方法
* Java
覆盖方法要求子类方法与父类方法有相同的方法名、参数列表与返回值，且要求父类方法没有final修饰。

* Kotlin
覆盖方法要求子类方法与父类方法有相同的方法名、参数列表与返回值，且要求父类有open修饰，且父类方法也有open修饰，并子类同名方法需要有override修饰。
```
open class Base {
    open fun v() { ... }
    fun nv() { ... }
}
class Derived() : Base() {
    override fun v() { ... }
}
```
Kotlin中，若父类方法没有open修饰，则会报错。因为子类中不允许有同名方法存在的。对于子类中的overide修饰的方法，本身默认就是open的。如果希望它不被其子类方法覆盖，可以添加final关键字。
```
open class AnotherDerived() : Base() {
    final override fun v() { ... }
}
```