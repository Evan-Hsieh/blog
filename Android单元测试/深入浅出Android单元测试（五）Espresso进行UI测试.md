

![安卓机器人](https://i.loli.net/2018/11/21/5bf57844a4872.jpg)
需要对Android界面UI进行测试？本文对以最易懂的方式介绍Espresso框架。


> 若有错漏，烦请斧正。转载请注明出处。欢迎关注程序引力
> * 作者：程序引力 | 谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com


### Espresso核心概念
在Espresso中，主要有两个对象，分别是：
* ViewInteraction：表示在Espresso中的一个特定的View
* DataInteraction：表示在Espresso中一个特定的Data对象

为了构造这两种对象，Espresso提供了两种方法，分别是：
* onView:构造ViewInteraction对象,该方法接受Matcher对象作为参数,例如
```
onView(withId(R.id.editTextView))
```
* onData:构造DataInteraction对象,该方法接受Matcher对象作为参数,例如
```
onData(is(instanceof(DataBean.class))
```
通过上面的方法就可以获得相应的Interaction对象，得到这个对象后才可以进行后面的测试。

这两个对象均有两个方法，分别是：
* perform(): 执行对象的某个动作。该方法接受ViewAction的对象作为动作参数
* check()：校验动作执行后的结果。该方法接受ViewAssertion的对象作为校验参数

### Espresso核心流程
#### 对View进行UI测试的核心流程
```
onView() -> perform() -> check()
```
即创建ViewInteration对象，执行UI动作，最后校验动作结果是否符合预期
写成链式调用的形式，就是：
```
onView().perform().check()
```

#### 对Data进行UI测试的核心流程
```
onData() -> DataOptions -> perform() -> check()
```
即创建DataInteration对象，之后执行一些数据处理，然后执行UI动作，最后校验动作结果是否符合预期
写成链式调用的形式，就是：
```
onView().someMethod().perform().check()
```

### Espresso简单示例
#### View示例
```
onView(withId(R.id.my_view))            // withId(R.id.my_view)是一个匹配器Matcher
        .perform(click())               // click()是一个动作ViewAction
        .check(matches(isDisplayed())); // matches(isDisplayed())是一个校验器
```
从上面的简单实例中，可以看出也是链式调用，该调用分为三步：
* 通过withId方法传入一个view的ID后，得到一个Matcher对象，并将该对象传给onView获得ViewInteraction对象。
* 调用ViewInteraction对象的perform方法，并传入一个ViewAction，例如一个点击操作click就是一个ViewAction.
* 调用ViewInteraction对象的check方法对结果进行校验。其中matches是一种校验器。





### 实践
#### 引入依赖
在模块（如app)的build.gradle中添加
```
dependencies {
    // Espresso 核心包
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
    // Espresso 同时依赖JUnit与AndroinUnitRunner
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
}
```
> 为了保证UI测试结果可靠，Android建议在开发者模式中将设备的动画都关闭，包括窗口动画缩放、过度动画缩放与动画程序时长调整。








### 附录
Espresso相关包：
* espresso-core： 核心包，以及View的动作、匹配等api依赖库
* espresso-web：提供了对webview的支持
* espresso-idling-resource: 提供了对在需要同步等待后台场景的UI测试支持
* espresso-contrib：外部库，增加了对RecycleView，DatePicker等支持
* espresso-intents： 提供了模拟传入intent场景的支持
* espresso-remote： 提供了对Espresso多进程功能的支持