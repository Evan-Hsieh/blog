### 前言
面向对象的语言中大多有关键字this，用于表示对象本身。但不同语言对于其支持的用法不尽相同。

### Java
根据《Thinking in Java》中提到的，Java中关于this的用法主要就是三种：
* this用于传递当前对象
* this用作调用成员变量
* this用于调用构造方法

#### this用于传递当前对象
* 传递到方法参数中
这是最为常见的用法之一，在某个类中需要将其传递给某一个方法中，例如：
```
class Person{
    public void eat(Apple apple){
        Apple peeled = apple.getPeeled();
        System.out.println("Yummy");
    }
}
class Peeler{
    static Apple peel(Apple apple){
        //....remove peel
        return apple;
    }
}
class Apple{
    Apple getPeeled(){
        return Peeler.peel(this);
    }
}
public class This{
    public static void main(String args[]){
        new Person().eat(new Apple());
    }
}
```
在该例子中，Apple类的对象通过this将自己的对象传递给Peeler.peel方法，在该方法中可以获得Apple对象并对其进行处理。

* 作为方法返回值
除了作为方法参数，this表示的对象还可以作为返回值，在链式调用或建造者设计模式中均有应用：
```
public class Tool{

    class Builder{
        setA(A a){
            //...
            return this;
        }
        
        setB(B b){
            //...
            return this;
        }
    }
}
```
在上面例子中是建造者设计模式的局部代码，其中的内部类Builder拥有多个方法，每个方法的返回值均是this，可以让调用者实现’链式‘调用，即
```
new Builder()
    .setA(a)
    .setB(b);
```

#### this用于调用成员变量
对于JavaBean类，其中的成员变量与方法形参可能同名，为了将其区分，可以通过this:
```
class Demo{
    String str;
    public void setStr(String str){
        this.str = str
    }
}
```

#### this用于调用构造方法
```
class Demo{
    Demo(){
        this("default");
    }
    Demo(String str){
        //xxx
    }
}
```

### Kotlin
在Kotlin中this表达的含义会更为广泛一些，除了表示类对象，还是表示接受者，具体来说：
* 在类的成员中，this 指的是该类的当前对象。
* 在扩展函数或者带有接收者的函数字面值中， this 表示在点左侧传递的 接收者 参数。

#### 标签限定符
如果 this 没有限定符，它指的是最内层的包含它的作用域。要引用其他作用域中的 this，需要使用标签限定符。

要访问来自外部作用域的this（一个类 或者扩展函数， 或者带标签的带有接收者的函数字面值）我们使用this@label，其中 @label 是一个代指 this 来源的标签。
```
class A { // 隐式标签 @A
    inner class B { // 隐式标签 @B
        fun Int.foo() { // 隐式标签 @foo
            val a = this@A // A 的 this
            val b = this@B // B 的 this

            val c = this // foo() 的接收者，一个 Int
            val c1 = this@foo // foo() 的接收者，一个 Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit 的接收者
            }

            val funLit2 = { s: String ->
                // foo() 的接收者，因为它包含的 lambda 表达式
                // 没有任何接收者
                val d1 = this
            }
        }
    }
}
```