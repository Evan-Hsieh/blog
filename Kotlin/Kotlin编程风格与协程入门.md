
![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


笔者近日有幸参与到与Kotlin核心作者Hadi Hariri的研讨会。Hariri先生作为JetBrains开发者推广团队的领导，在研讨会中分享了Kotlin的若干编程风格，以及协程与并发编程的内容。其中分享的知识与经验均来自于Hariri先生。

### 前言
2017年Google宣布Kotlin在Android开发上正式受到官方支持，使得更多的Android开发者迁移到Kotlin。截止到2018年冬，Kotlin 1.3 已经发布，也带来了许多新特性。本文将会以4种编程风格作为背景，引出Kotlin中有关协程特性的介绍。

### 编程风格
不管对于怎样的程序语言，开发者在使用过程中会形成各种各样的编程风格（Programming Style),或者对于一些新型语言，在设计之初就会参考这些风格。

对于异步编程的场景，主要有4类风格：
#### 1. 异步线程
在实践中，最普通的方式就是在需要并发编程的场景中，手动创建线程，然后启动线程，这也是最为基础也是最为原始的方式。但是它也有着一些缺点：
* 创建线程的数量有限制：由于一些平台的限制或硬件的限制，允许创建的最大线程数是固定的。
* 创建线程开销较大：创建线程有着较大的系统资源开销。
* 部分场景不允许创建线程：JS/web

#### 2. 回调函数
回调函数是一种常用的异步编程风格，将函数或者持有函数的对象作为参数传递给函数，然后在其执行完毕后，调用回调函数。但是，回调函数也有着一些缺点，例如对于Javascript的一个毁掉函数的例子：
```
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```
从代码中可以看出，回调函数中可能传入另一个回调函数，依次嵌套许多层，这会让代码难以阅读和维护。这也被称为“回调地狱”（callback hell).

同时，在这样的嵌套的回调中，如果发生异常，很难去处理异常，这也是另一个缺点。

#### 3. Future/Promise/Rx风格
这种风格表示通过定义一个接口，然后返回这个接口，再一次调用这个接口中的方法来完成任务的一种编程风格。另外，如Rx系列支持的链式调用也是属于这一种风格。以RxJava的调用方式为例：
```
Observable.create()
.observeOn()
.subscribOn()
.subscrib()
```
该例子省略了许多参数。但从该例子中也看得出来，该编程风格与回调函数的不同是将嵌套的多个方法，变成‘链式’调用了。
该方法虽然没有了‘回调地狱’的问题，但是其返回值不是真正执行业务的方法的返回值，而是一个接口。为此，引出了下一种编程风格。

#### 4.Kotlin 协程
对于Kotlin协程，需要先使用关键字suspend声明可挂起函数：
```
suspend fun doSomething(foo: Foo): Bar { ... }
suspend fun doSomethingElse(foo: Foo): Bar { ... }

```
为此，在使用launch方法，既可以启动协程
```
launch {
    val bar = doSomething(foo())
    val bar2 = doSomethingElse(foo())
}
```
上面两个方法经过编译处理后，会运行在异步的协程中。

