![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安卓开发者必须了解的事件分发机制。本文将从源码角度，以最全面、最易懂的形式来讲解Android事件分发机制。

若有错漏，烦请斧正。转载请注明出处。
* 作者：程序引力 | 谢一 （Evan Xie）
* 邮箱：evanyixie@gmail.com

### 0. 前言
鉴于安卓分发机制较为复杂，故分为多个层次进行讲解，分别为基础篇、实践篇与高级篇。

* （一）基础篇：从基本概念入手，介绍了分发机制中的核心方法，通过分析其核心逻辑，总结其事件分发机制。
* （二）实践篇：该篇设计了简单与复杂的两个demo样例，从现象与应用的角度去讲解分发机制的核心内容，帮助读者从另一个角度理解事件分发机制。
* （三）高级篇：从源码角度去分析分发机制背后的原因，让读者对分发机制背后的本质有更为全面与深刻的理解。

### 1. 内容简介

本文内容为（三）高级篇，将从源码角度分析事件分发机制。该部分源码来自于目前最新的Android Pie (即Android 9.0)，API 28的源码。

建议读者在阅读了本系列的基础篇与实践篇的文章后，再阅读本篇内容，会更容易理解。

### 2. Activity核心分发方法
对于Activity，负责参与分发的方法有：
* dispatchTouchEvent（）
* onTouchEvent()

这两个方法不是并列同级的关系，实际上前者是包含后者的。

#### Activity的dispatchTouchEvent
```
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {    
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;    
    }
    return onTouchEvent(ev);
}
```
从上面的源码可以看到，当Activity的dispatchTouchEvent方法接收到按下Down事件后，首先调用onUserInteraction方法。该方法在事件分发给Activity时会被调用，且该方法一般空。如果开发者希望知道用户与设备的交互情况，可以覆写该方法。

> 实际上，onUserInteraction方法主要是用于管理状态栏通知，以及在恰当的时候取消通知。与该方法相关的还有另一个方法，onUserLeaveHint。该方法作为Activity生命周期回调的一部分，会在用户将Activity放到后台时调用（如用户点击Home键），该方法会在onPause方法之前调用。

对于上面源码中的getWindow(),返回的是Window类对象，而Window类实际是一个抽象类，它有一个唯一的实现类PhoneWindow。实际上，getWindow()返回的是PhoneWindow类对象，后面调用方法也实际上是PhoneWindow对象的方法,该方法实现如下：
```
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```
在该方法中，调用了mDecor的同名方法，mDecor对象属于DecorView，该类是PhoneWindow类的内部类。其源码实现如下：
```
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```
该类继承自FrameLayout，即是ViewGroup的子类。调用super父类的分发方法，即是调用了ViewGroup的分发方法（详细介绍见下文）。通过该方法，可以将事件分发给此ViewGroup内部的子View处理。

总的说来，getWindow().superDispatchTouchEvent()这一语句将事件从Activity传递到了其内部的ViewGroup。如果其分发方法返回TRUE，即表示已经消费了该事件，故也返回TRUE。否则调用onTouchEvent()方法。

#### Activity的onTouchEvent方法

Activity会将事件不断向下分发给起内部的ViewGroup与View，若内部的ViewGroup以及View都不消费该事件，事件会不断的传递回来，到达Activity的onTouchEvent方法。该方法的源码如下：
```
public boolean onTouchEvent(MotionEvent event) {
    if (mWindow.shouldCloseOnTouch(this, event)) {
        finish();
        return true;
    }
    return false;
}
```
在该方法中，首先调用mWindow成员的shouldCloseOnTouch方法，该方法不是PhoneWindows对象的方法，而是抽象类Window自身的方法，其表示是否要在该touch事件后关闭窗口。如果返回TRUE，则调用Activity的finish方法，并返回TRUE表示消费事件。否则返回FALSE。

在Window类的shouldCloseOnTouch方法中，其源码如下：
```
public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    final boolean isOutside = event.getAction() == MotionEvent.ACTION_DOWN && isOutOfBounds(context, event)
        || event.getAction() == MotionEvent.ACTION_OUTSIDE;

    if (mCloseOnTouchOutside && peekDecorView() != null && isOutside) {
        return true;
    }
    return false;
}
```
该方法主要是判断点击事件是否为Down按下事件，并且在边界之外，并且还判断了一些标志位。若按下事件在边界外，且标志位为TRUE，则返回TURE。


#### Activity分发方法小节
总的说来，Activity的事件分发逻辑还是比较简单的。在Activity的分发方法dispatchTouchEvent方法中，首先会在按下事件调用onUserInteraction方法，该方法一般为空。之后会调用PhoneWindow对象的分发方法，最后通过ViewGroup的分发方法将事件分发给Activity内部的ViewGroup或View。若它们不能处理，则会将事件传回来，调用Activity的onTouchEvent方法。在该方法中，会判断是否应该关闭Activity，否则则返回默认的FALSE。


### 3.ViewGroup核心分发方法
对于ViewGroup，负责分发事件的方法有：
* dispatchTouchEvent（）
* onInterceptTouchEvent()
* onTouchEvent()

上面这些方法并不是并列同级的关系，事实上，dispatch方法包含了后面两个方法。而准确地说，ViewGroup中并不包含onTouchEvent方法，只是由于ViewGroup是View的子类，而后者拥有onTouchEvent方法，故也认为ViewGroup也存在该方法。

#### ViewGroup的dispatchTouchEvent方法
由于Android 9.0的中ViewGroup的该方法源码非常长，故只分析其核心逻辑。同时，为了让读者更易于阅读，不用前后查阅，这些源码会以分片的形式展示。并且在源码附近会添加注释，用于更直观地解释源码。
```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 输入一致性校验
    // 若该类的isInstrumentationEnabled方法为TRUE，则会在View类中初始化该对象。
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);  
    }

    // 通过点击事件判断是否应该被有焦点的View处理事件，如果同时存在拥有焦点的View，则设置为FALSE。
    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
        ev.setTargetAccessibilityFocus(false);
    }


```
上面是dispatchTouchEvent方法入口会执行的逻辑，不是特别重要，可以重点关注后面的源码。
```
//重要的局部变量，该变量值表示是否处理了该事件。该变量也是方法最后的返回值。
boolean handled = false;
//出于安全原因，会过滤点击事件。返回FALSE表示丢弃事件，返回TRUE表示继续处理。
if (onFilterTouchEventForSecurity(ev)) {
    final int action = ev.getAction();
    //保留action的末尾8bit位，其余置0，作为actionMask
    final int actionMasked = action & MotionEvent.ACTION_MASK;

    // 因为按下事件是整个事件流的开始，为此当按下事件到来时，会做一些初始化工作。
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // 初始化工作主要就是清空target以及重置状态
        cancelAndClearTouchTargets(ev);
        resetTouchState();
    }
```




