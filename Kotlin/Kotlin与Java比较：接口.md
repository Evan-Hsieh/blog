
### 前言
Kotlin作为JVM系的语言，起源于Java又不同于Java。通过在语言层面比较两者的区别，可以使得开发者能够快速学习，融会贯通。

### 接口概况
* Java
在Java8之前，接口只能包含抽象方法和静态属性。但在Java8之后，在接口中不要求全部是抽象方法，而允许有具体方法的存在，具体分为：default方法与static方法。Java中接口支持多实现（继承）。

* Kotlin
在Kotlin的接口中，允许有抽象方法，也允许有具体方法。允许有抽象的属性，也允许提供访问器的属性。接口允许多实现（继承）。

### 接口定义
* Java
在Java8中，其可以定义的内容为：
```
 public interface JavaInterface {
    static final String str = "a"; //实际上static final都是冗余的。Java接口中属性都是静态常量
    String bar();    //抽象方法
    default String f(){    //默认方法
        return"this is default method";
    }
    static String foo{    //静态方法
        return"this is static method";
    }
```
从上面的例子中可以看到，Java8之后的接口允许定义默认方法与静态方法，默认方法的作用就是避免了实现该接口的重复代码。即如果有多个类都实现了该接口，并且某一个方法的实现是一样的，那此时就可以把这个方法提升到接口中，并以default修饰，设置为默认方法。静态方法与类中的静态方法一致。

* Kotlin
在Kotlin中，接口属性必须为抽象的或者提供访问器的。方法可以是抽象的，也可以有具体实现。
```
interface KotlinInterface {
    val prop: Int // 抽象的
    val propertyWithImplementation: String
        get() = "foo"    //提供访问器的属性

    fun bar()
    fun foo() {
      // 可选的方法体
    }
}
```
在接口声明的属性中，不能有幕后字段，即不能引用field.

### 接口实现
* Java
在Java中，实现方法不需添加额外关键字，只需添加方法体具体实现即可。同时建议添加@Override注解。
```
class JavaChild implements JavaInterface{
    String bar(){
        //xxx
    }
}
```

* Kotlin
Kotlin中实现属性或方法需要添加override关键字，但是与继承类不同，被实现的接口中的属性或方法不用添加open关键字，默认就是open的。
```
class KotlinChild : KotlinInterface {
    override val prop: Int = 29
    override fun bar() {
        // 方法体
    }
}
```

### 接口间继承
* Java
在Java中允许接口间继承，即接口B继承接口A，但是用类C去实现接口B。类C需要实现A与B中未实现的所有抽象方法。

* Kotlin
Kotlin中的接口间继承规则基本一致。接口间可以继承，最后的实现类只需要实现剩余的抽象方法即可。






