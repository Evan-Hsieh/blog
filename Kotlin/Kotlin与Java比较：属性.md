
### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 属性声明
* Java
Java声明属性的语法格式为：
```
[访问控制修饰符] 属性类型 属性名 [ = 初始化值];
```
> 其中以方括号[ ]表示可选的
例如，下面都是合法的属性声明：
```
private String str = "long";
String s = "short"
```

* Kotlin
在Kotlin中，声明属性有两种关键字，分别是var与val,前者有时称为“变量”，表示即可读又可写，而后者有时称为“值”，表示只可读。它们的声明语法的格式为：
```
var 属性名 [:属性类型] [ = 初始化器]
    [getter]
    [setter]

val 属性名 [:属性类型] [ = 初始化器]
    [getter]
```
初始化器可以理解为语句中赋值的部分，而getter与setter则被称为读访问器与写访问器，统称访问器。
当初始化器或访问器存在且类型能够不推断时，属性类型可以省略。

简单的例子为：
```
var name:String = "kotlin"  
val i = 1
```
在上例中，”kotlin"与1被称为初始化器。若存在初始化器，且类型能够被自动推断，则可以省略属性类型。

观察上面的语法格式可以发现，val与var相比，就是没有setter访问器。使用访问器但不使用初始化器的例子为：
```
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并赋值给其他属性
    }
```
该例子声明了一个var变量，是String类型。同时其getter方法返回的值为this.toString(),而setter操作是将传入的参数传给了setDataFromString方法。

> 可能有读者会有疑问，为何在Kotlin中，属性名是放在属性类型前面的，而Java是放在后面的。笔者也有类似的疑问，在查阅了一些资料以后，了解到前者是Pascal风格的声明方式，而后者是C风格的声明方式。Stack Overflow上的答主也对这个问题有着诸多讨论，有从可读性来解释的，有从编译器的角度来解释的，可以参考：[Why put the type after the variables name?](https://stackoverflow.com/questions/1712274/why-do-a-lot-of-programming-languages-put-the-type-after-the-variable-name)


### 简单的属性声明比较
* Java
```
String str = "hello";
```
其等价于

* Kotlin
```
var str:String = "hello"  
var str = "hello"
```
初始化器能够推断类型，则属性类型可以省略。同时，在该简单的例子中，省略了访问器，则Kotlin会生成默认的访问器。

### 带有访问器的属性声明比较
* Java
```
public class Data{

    String str = "hello";

    public getStr(){
        return "world";
    }
}
```
其部分等价的例子为：
* Kotlin
```
class Data{
    val str:String = "hello"
    get() = "world"
}
```
从上面的例子比较中可以看出，当获取一个变量的值时，在Java中是调用其getStr方法获取，而在Kotlin中，是通过其get()方法获取。

但需要注意的是，在Java中，调用getStr方法需要显式调用，而在Kotlin中是通过 . 就调用了。例如
```
var data = Data()
var anotherStr = data.str
```
即在Kotlin中，通过点.操作符，就可以代用getter方法，在被赋值时，就调用了setter方法。

### 改变属性访问范围
* Java
```
private String str = "abc";
```
* Kotlin
```
var str: String = "abc"
    private set // 此访问器私有的并且有默认实现
    private get // 此访问器私有的并且有默认实现
```
若要修改Kotlin属性的访问访问，可以在set或get前添加修饰符，如果后面没有覆盖其实现，它则保留默认实现。

### 幕后字段
* Java
在Java中，数据Bean常写作：
```
public class Person{
    private String name = "abc"
    public getName(){
        return name;
    }
    public setName(String name){
        this.name = name;
    }
}
```
* Kotlin
在Kotlin中，上面例子的对应写法却不是如下写法：
```
class Person{
    var name: String?=null
    get() = name
    set(value){
        name = value
    }
}
```
通过将这段代码转换为Java代码，可以得到:
```
public class Person{
    public String getName(){
        this.getName();
    }
    public void setName(String value){
        this.setName(value);
    }
}
```
从转换后的Java代码可以看出，对于getName与setName是无限递归的。为此，提出了幕后字段。

幕后字段即field关键字声明的字段，该字段表示该属性真正的值，上面一段Java代码对应的Kotlin代码应该为：
```
class Person{
    var name: String?=null
    get() = field
    set(value){
        field = value
    }
}
```