
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
    var str:String = "hello"
    get() = "world"
}
```
从上面的例子比较中可以看出，当获取一个变量的值时，在Java中是调用其getStr方法获取，而在Kotlin中，是通过其get()方法获取。

但需要注意的是，在Java中，调用getStr方法需要显式调用，而在Kotlin中是通过 . 就调用了。例如
```
var data = Data()
var anotherStr = data.str
```
即在Kotlin中，通过点.操作符，就可以代用getter方法，这个值可以与成员的值不一致。
在被赋值时，就调用了setter方法。

### 改变属性访问范围
| 访问控制符 | Java类 | Kotlin类 | Kotlin顶层声明 |
| :--: | :--: | :--: | :--: |
| public | 所有地方可见 | 所有地方可见 | 所有地方可见 |
| protected | 子类可见 | 子类可见 | NA |
| private | 本类可见 | 本类可见 | 本文件可见 |
| internal | NA | 模块可见 | 模块可见 |
| default | 包可见 | 所有地方可见 | 所有地方可见 |

> 一个模块就是一组一起编译的Kotlin文件。，这可能是一个intellij IDEA模块、一个Eclipse项目、一个Maven或 Gradle项目或者一组使用调用ant任务进行编译的文件

* Java
```
private String str = "abc";
```
* Kotlin
```
protected var str: String = "abc"
    private set // 此访问器私有的并且有默认实现
    protected get // 此访问器子类可见并且有默认实现
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

幕后字段即field关键字声明的字段，该字段表示该属性真正的值，上面一段Java代码与正确的Kotlin代码的对应关系为：
Java代码：
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
Kotlin代码
```
class Person{
    var name: String?=null
    get() = field
    set(value){
        field = value
    }
}
```

#### 幕后字段定义
在Kotlin中, 如果属性至少一个访问器使用默认实现，那么Kotlin会自动提供幕后字段，用关键字field表示，幕后字段主要用于自定义getter和setter中，并且只能在getter 和setter中访问。

对于Kotlin自动提供幕后字段，有：
```
// 例子一
class Person {
    var name:String = ""
        get() = field 
        set(value) {
            field = value
        }
}
// 例子二
class Person {
    var name:String = ""
}
```
上面两个属性的声明是等价的，例子一中的getter和setter 就是默认的getter和setter。其中幕后字段field指的就是当前的这个属性，它不是一个关键字，只是在setter和getter的这个两个特殊作用域中有着特殊的含义，就像一个类中的this,代表当前这个类。

#### 幕后字段使用场景
用幕后字段，我们可以在getter和setter中做很多事，一般用于让一个属性在不同的条件下有不同的值，比如下面这个场景：
```
class Person(var gender:Gender){
    var name:String = ""
        set(value) {
            field = when(gender){
                Gender.MALE -> "Jake.$value"
                Gender.FEMALE -> "Rose.$value"
            }
        }
}

enum class Gender{
    MALE,
    FEMALE
}

fun main(args: Array<String>) {
    // 性别MALE
    var person = Person(Gender.MALE)
    person.name="Love"
    println("打印结果:${person.name}")
    //性别：FEMALE
    var person2 = Person(Gender.FEMALE)
    person2.name="Love"
    println("打印结果:${person2.name}")
}


// 打印结果:Jake.Love
// 打印结果:Rose.Love
```
如上，我们实现了name 属性通过gender 的值不同而行为不同。幕后字段大多也用于类似场景。

#### 拥有幕后字段条件
需要满足下面条件之一：
* 使用默认 getter / setter 的属性，一定有幕后字段。对于 var 属性来说，只要 getter / setter 中有一个使用默认实现，就会生成幕后字段；
* 在自定义 getter / setter 中使用了 field 的属性

没有幕后字段的例子
```
class NoField {
    var size = 0
    //isEmpty没有幕后字段
    var isEmpty
        get() = size == 0
        set(value) {
            size *= 2
        }
}
```
如上，isEmpty是没有幕后字段的，重写了setter和getter,没有在其中使用 field

### 幕后属性
有时候有这种需求，我们希望一个属性：对外表现为只读，对内表现为可读可写，我们将这个属性成为幕后属性。 如：
```
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 类型参数已推断出
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```
将_table属性声明为private,因此外部是不能访问的，内部可以访问，外部访问通过table属性，而table属性的值取决于_table，这里_table就是幕后属性。

> 其中?:为 Elvis Operator，当其前面的变量不会空时，返回该变量，若其为空时，返回右侧表达式的值。

幕后属性这中设计在Kotlin 的的集合Collection中用得非常多，Collection 中有个size字段，size 对外是只读的，size的值的改变根据集合的元素的变换而改变，这是在集合内部进行的，这用幕后属性来实现非常方便。

如Kotlin AbstractList中SubList源码：
```
private class SubList<out E>(private val list: AbstractList<E>, private val fromIndex: Int, toIndex: Int) : AbstractList<E>(), RandomAccess {
        // 幕后属性
        private var _size: Int = 0

        init {
            checkRangeIndexes(fromIndex, toIndex, list.size)
            this._size = toIndex - fromIndex
        }
        override fun get(index: Int): E {
            checkElementIndex(index, _size)
            return list[fromIndex + index]
        }

        override val size: Int get() = _size
    }
```
