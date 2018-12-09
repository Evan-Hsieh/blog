### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 数据类
在Java中没有专门的数据类，常常是通过JavaBean来作为数据类，但在Kotlin中提供了专门的数据类。

* Java
```
public class Data(int length){
    private int length;
    public setLength(int length){
        this.length = length;
    }
    public getLength(){
        return this.length;
    }
}
```
从上面的例子中可以看到，如果要使用数据类，需要手动写相应的setter/getter方法（尽管IDE也可以批量生成），但是从代码阅读的角度来说，在属性较多的情况下，诸多的seeter/getter方法还是不利于代码的阅读和维护。

* Kotlin
在Kotlin中，可以通过关键字data来生成数据类：
```
data class User(val name: String, val age: Int)
```
即在class关键字之前添加data关键字即可。编译器会根据主构造函数中的参数生成相应的数据类。

要声明一个数据类，需要满足：
* 主构造函数中至少有一个参数
* 主构造函数中所有参数需要标记为val或var
* 数据类不能是抽象、开发、密封和内部的

在使用数据类时，与普通类无异，通过向其主构造函数中传入参数值即可。若要数据类生成一个无参的构造函数，即在使用它的时候不用传入参数，需要在定义该数据类时，对参数指定默认值：
```
data class User(val name: String = "", val age: Int = 0)
```
若不希望所有的属性都作为数据类中的属性，可以将该属性放到类体中：
```
data class Person(val name: String) {
    var age: Int = 0
}
```

