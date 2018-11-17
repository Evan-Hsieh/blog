
Kotlin中的控制流由if,when,for,while所支持。

### if
if可以作为语句（块），也可以作为表达式。
* 语句块
```
// 传统用法
var max = a 
if (a < b) max = b

// With else 
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}
```

* 表达式
```
// 作为表达式
val max = if (a > b) a else b
// 代码块最后的值作为表达式的值
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```
> 若使用if作为表达式，则必须有else分支。

### when
when用于代替switch,其参数会与分支参数顺序比较。when同样可以被作为语句或表达式。
```
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个块
        print("x is neither 1 nor 2")
    }
}
```
同时支持同样处理方式的分支合并，分支表达式也可以是任意表达式（不必是常量）：
```
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    parseInt(s) -> print("s encodes x")
    else -> print("otherwise")
}
```
when也可以用来代替if-else链。即不提供参数，分支表达式以布尔表达式处理。
```
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```
对于Kotlin1.3,支持将when语句复制给变量。

