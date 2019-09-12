
### 前言
Gradle基于Groovy语言,采用DSL的构建工具。gradle脚本中可以包含Groovy的任何元素。

### 架构
Gradle结构与任务围绕着两个额概念：
* project： 每一个构建都围绕着一个或多个project。project不一定代表诸如安卓应用、web应用等实体，也可以代表行为。
* task：每一个project由一个或多个task组成

### 项目
对于简单的脚本：
```
version = '1.0.0.GA'

configurations {
    ...
}
```
version与configurations都是org.gradle.api.Project的一部分。一般情况下，脚本中的元素都由一个隐式的实例，即project。对于找不到定义的元素，可以查看project的API。

### 任务
要了解gradle可以先从task着手。通过task关键字声明任务，其格式为：
```
task 任务名[(参数)] {

}
```
其中参数可选。除了静态声明的方式，也可以使用动态声明的方式创建任务：
```
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```
运行时，使用命令：
```
> gradle -q task1
I'm task number 1
```

#### 任务依赖
通过depensOn来配置依赖：
```
task intro(dependsOn: hello) << {
    println "I'm Gradle"
}

task intro(dependsOn: 'hello') << {
    println "I'm Gradle"
}
```
本例中前者需要对被依赖的任务先进行声明，后者则不需要（因为gradle在执行阶段前有一个配置阶段）。同时gradle允许多个依赖：
```
task0.dependsOn task2, task3
```
#### 排除依赖
假设A依赖B，使用-x参数来排除依赖
```
 gradle A -x B
```
则B不会被执行


#### 任务顺序
```
task hello << {
    println 'Hello Earth'
}
hello.doFirst {
    println 'Hello Venus'
}
hello.doLast {
    println 'Hello Mars'
}
hello << {
    println 'Hello Jupiter'
```
执行结果：
```
gradle -q hello
Hello Venus
Hello Earth
Hello Mars
Hello Jupiter
```

#### 默认任务
```
defaultTasks 'clean', 'run'    //定义默认任务

task clean << {
    println 'Default Cleaning!'
}
task run << {
    println 'Default Running!'
}
```
在使用时
```
> gradle -q
Default Cleaning!
Default Running!
```



### 属性
属性语法：
```
<obj>.<name>                // Get a property value
<obj>.<name> = <value>      // Set a property to a new value
"$<name>"                   // Embed a property value in a string
"${<obj>.<name>}"           // Same as previous (embedded value)
```
#### 标准属性
| 属性名 | 类型 | 默认值 |
| :---: | :---: | :---: |
| project | Project | Project 实例对象 |
| name | String | 项目目录的名称 |
| path | String |项目的绝对路径 | 
| description | String | 项目描述 |
| projectDir | File | 包含构建脚本的目录 |
| build | File | projectDir/build |
| group| Object |未具体说明 |
| version| Object | 未具体说明 |
| ant | AntBuilder | Ant实例对象 |


#### 使用属性
通过给任务执行属性，然后可以访问该属性
```
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty
}
```
任务中的隐含属性name.
```
task hello << {
    println 'Hello world!'
}
hello.doLast {
    println "Greetings from the $hello.name task."
}
```

> 对于脚本中没有明确的元素，其可能属于如下某种情况：
> * Project的属性
> * 定义在其他某处的project的属性
> * 任务的实例
> * block中的某个隐式对象的属性
> * 局部变量

### 声明变量
局部变量
```
def dest = "dest"

task copy(type: Copy) {
        form "source"
        into dest
}
```
扩展属性
```
ext {
    springVersion = "3.1.0.RELEASE"
    emailNotification = "build@master.org"
}

sourceSets.all { ext.purpose = null }

sourceSets {
    main {
        purpose = "production"
    }
    test {
        purpose = "test"
        }
    plugin {
        purpose = "production"
    }
}
task printProperties << {
    println springVersion
    println emailNotification
    sourceSets.matching { it.purpose == "production" }.each { println it.name }
}
```
每一个gradle都默认有project对象。上面的例子中，通过ext模块向project添加了两个扩展属性。名为purpose的属性被添加到source set.这些扩展属性被添加后，它们就像预定义的属性一样可以被读取，更改值.


### 声明方法
方法的语法为：
```
<obj>.<name>()              // Method call with no arguments
<obj>.<name>(<arg>, <arg>)  // Method call with multiple arguments
<obj>.<name> <arg>, <arg>   // Method call with multiple args (no parentheses)
```
注意第三种，即参数不为0时，可以省略圆括号。gradle允许声明方法，在任务中使用.

> Blocks也是方法，常作为某种类型的最后一个参数

### 块
语法格式为：
```
<obj>.<name> {
     ...
}

<obj>.<name>(<arg>, <arg>) {
     ...
}
```
例如：
```
configurations {
    assets
}
sourceSets {
    main {
        java {
            srcDirs = ['src']
        }
    }
}
```
这一语法格式允许开发者一次性定义构建元素的多个方面。关于它有两个重要的特点：
* 它们被以方法的形式实现，并且拥有具体的签名。通过这些签名，可以查到该block执行什么方法。
* 它们可以更改代理

#### 块签名
以笔者的理解，签名即为方法名与参数列表。通过签名可以唯一识别方法。同时，也可以通过块名以及其签名（或参数）来识别块。对于一个块方法：
* 至少包含一个参数
* 最后一个参数类型为 groovy.lang.Closure或者org.gradle.api.Action.

例如，下面这个块即与Project.copy(Action)的签名匹配。
```
copy {
    into "$buildDir/tmp"
    from 'custom-resources'
}

```

### 生命周期
配置阶段和执行阶段

### 仓库
```
repositories{

}
```


### 依赖
```
denpendencies{

}
```

### 插件
* 脚本插件
```
apply from: 'other.gradle'
```
* 二进制插件
```
apply plugin: 'java'
```