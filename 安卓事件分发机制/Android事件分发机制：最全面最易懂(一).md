
![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-427e24ec0260a5ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文介绍了安卓的事件分发机制。
若有错漏，烦请斧正。转载请注明出处。
* 作者：谢一 （Evan Xie）
* 邮箱：evanyixie@gmail.com

# 0. 前言
鉴于安卓分发机制较为复杂，故分为多个层次进行讲解，分别为基础篇、实践篇与高级篇。

* （一）基础篇：从基本概念入手，介绍了分发机制中的核心方法，通过分析其核心逻辑，总结其事件分发机制。
* （二）实践篇：该篇设计了简单与复杂的两个demo样例，从现象与应用的角度去讲解分发机制的核心内容，帮助读者从另一个角度理解事件分发机制。
* （三）高级篇：从源码角度去分析分发机制背后的原因，让读者对分发机制背后的本质有更为全面与深刻的理解。

# 1. 内容简介

本文内容为（一）基础篇，本篇主要对被分发的对象与参与分发的组件等基本概念做了介绍。同时，介绍了负责分发的核心方法。从这些方法的核心逻辑伪代码中，总结事件分发的规律。避免了许多文章直接给初学者讲解源码所带来的困惑。

本文深入浅出，通过阅读本文，可以帮助开发者对安卓事件分发机制有一个整体的了解，并且能够帮助开发者快速解决一些常见的实际问题，从而实现快速开发。

# 2. 被分发的对象
被分发的对象是那些？被分发的对象是用户触摸屏幕而产生的点击事件，事件主要包括：按下、滑动、抬起与取消。这些事件被封装成MotionEvent对象。

| 事件 | 触发场景 | 单次事件流中触发的次数 | 
| :---: | :---: | :---: |
| MotionEvent.ACTION_DOWN | 在屏幕按下时 | 1次 | 
| MotionEvent.ACTION_MOVE | 在屏幕上滑动时 | 0次或多次 | 
| MotionEvent.ACTION_UP | 在屏幕抬起时 | 0次或1次 | 
| MotionEvent.ACTION_CANCLE | 滑动超出控件边界时 | 0次或1次 | 

按下、滑动、抬起、取消这几种事件组成了一个事件流。事件流以按下为开始，中间可能有若干次滑动，以抬起或取消作为结束。

> 在安卓对事件分发的处理过程中，主要是对**按下**事件作分发，进而找到能够处理**按下**事件的组件。对于事件流中后续的事件（如滑动、抬起等），则直接分发给能够处理**按下**事件的组件。故本文讨论的内容则是主要针对**按下**事件的。


# 3. 分发事件的组件
分发事件的组件，也称为分发事件者，包括Activity、View和ViewGroup。它们三者的一般结构为：

![事件分发者结构](https://i.loli.net/2018/10/08/5bbb5665bb58f.png)

从上图中可以看出，Activity包括了ViewGroup，ViewGroup又可以包含多个View。

| 组件 | 特点 | 举例 | 
| :---: | :---: | :---: |
| Activity | 安卓视图类 | 如MainActivity | 
| ViewGroup | View的容器，可以包含若干View | 各种布局类 | 
| View | UI类组件的基类 | 如按钮、文本框 | 


# 4. 分发的核心方法
负责对事件进行分发的方法主要有三个，分别是：dispatchTouchEvent（）、onTouchEvent（）和onInterceptTouchEvent（）。它们并不存在于所有负责分发的组件中，其具体情况总结于下面的表格中：

| 组件 | dispatchTouchEvent | onTouchEvent | onInterceptTouchEvent |
| :---: | :---: | :---: | :---: |
| Activity | 存在 | 存在 | 不存在 |
| ViewGroup | 存在 | 存在 | 存在 |
| View | 存在 | 存在 | 不存在 |

从表格中看，dispatchTouchEvent（）,onTouchEvent（）方法存在于上文的三个组件中。而onInterceptTouchEvent（）为ViewGroup独有。这些方法的具体作用在下文作介绍。

> ViewGroup类中，实际是没有onTouchEvent()方法的，但是由于ViewGroup继承自View，而View拥有onTouchEvent()方法，故ViewGroup的对象也是可以调用onTouchEvent()方法的。故在表格中表明ViewGroup中存在onTouchEvent（）方法的。


# 5. 事件分发过程
这一小节是本文的核心内容，会从整体上对事件的分发过程作介绍。

> 对于事件分发过程从，笔者认为网上的一些教程中的观点是有误的。
> * 网上部分教程认为事件是从内部（如Button）开始分发的，这是有误的。
> * 网上部分教程常使用’向上‘、’向下‘传播等描述，但又未对‘何为上’、‘何为下’作解释。
> * 网上部分教程将Java的子类对象调用父类方法（向上转型）的过程也称为‘向上’传播，即将事件在组件之间的传播与程序语言多态特性混为一谈，让初学者费解。
> * 子类在覆写的方法中调用父类的同名方法，被称为’向上传播‘，这也是不对的。

为此在介绍分发过程之前，先对一些概念作定义：
* 向下传播：Activity包括Layout，事件从Activity向Layout传播被称作’向下传播‘。Layout包含若干View，事件从Layout向其子View传播，也被称为’向下传播‘。
* 向上传播：与’向下传播‘相反。
> ’向上转型‘不能称为传播，即子类对象调用父类方法，或在覆写的方法中调用父类方法，都不能称为传播。不能将面向对象程序语言中的概念与布局层次中的上下传播混为一谈。

## 分发方法dispatchTouchEvent（）
从方法的名称中可以看出该方法主要是负责分发，是安卓事件分发过程中的核心。事件是如何传递的，主要就是看该方法，理解了这个方法，也就理解了安卓事件分发机制。

在了解该方法的核心机制之前，需要知道一个结论：
* 如果某个组件的该方法返回TRUE,则表示该组件已经对事件进行了处理，不用继续调用其余组件的分发方法，即停止分发。
* 如果某个组件的该方法返回FALSE,则表示该组件不能对该事件进行处理，需要按照规则继续分发事件。在不复写该方法的情况下，除了一些特殊的组件，其余组件都是默认返回False的。后续有例子说明。

为何返回TRUE就不用继续分发，而返回FALSE就停止分发呢？为了解决这个疑问，需要看一看该方法的具体分发逻辑。为了便于理解，下面对dispatchTouchEvent方法进行简化，只保留最核心的逻辑。

###Activity的dispatchTouchEvent方法
```
// Activity中该方法的核心部分伪代码
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (child.dispatchTouchEvent(ev)) {
        return true;    //如果子View消费了该事件,则返回TRUE，让调用者知道该事件已被消费
    } else {
        return onTouchEvent(ev);    //如果子View没有消费该事件，则调用自身的onTouchEvent尝试处理。
    }
}
```
首先，从核心逻辑中看出，当事件传递给Activity后，它先将事件分发给子View处理。
* 如果经过子View层层传递或处理后，该事件被消费了（即返回了TRUE），则Activity的分发方法也返回TRUE，同样也表示该事件已经被消费了。
* 如果经过子View层层传递或处理后，该事件没有被消费（即返回了FALSE），则Activity的分发方法就不会返回TRUE了，而是调用onTouchEvent()去处理，看其实际的处理情况。
    * 如果onTouchEvent()消费了事件，那依然能返回TRUE（表示已消费事件），这个TRUE作为dispatchTouchEvent的返回值，让调用它的对象知道该Activity已经消费了事件。
    * 如果onTouchEvent()没有消费该事件，那就返回FALSE（表示未消费事件），这个FALSE作为dispatchTouchEvent的返回值，让调用它的对象知道该Activity没有消费事件，需要继续处理。



###ViewGroup的dispatchTouchEvent方法
```
// ViewGroup中该方法的核心部分伪代码
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (!onInterceptTouchEvent(ev)) {
        return child.dispatchTouchEvent(ev);    //不拦截，则传给子View进行分发处理
    } else {
        return onTouchEvent(ev);    //拦截事件，交由自身对象的onTouchEvent方法处理
    }
}
```
ViewGroup的该方法与Activity的类似，只是新添了一个onInterceptTouchEvent（）方法。当事件传入时，首先会调用onInterceptTouchEvent（）。
* 如果该方法返回了FALSE（表示不拦截），则交给子View去调用dispatchTouchEvent（）方法
* 如果该方法返回了TRUE（表示拦截），则直接交给该ViewGroup对象的onTouchEvent(ev)方法处理，具体是否能处理以onTouchEvent()的实际情况为准。

> 实际上，在onInterceptTouchEvent()返回TURE表示拦截时，实际调用的是super.dispatchTouchEvent()方法，即View的该方法，进而由该方法调用onTouchEvent().


###View的dispatchTouchEvent方法
```
// View中该方法的核心部分伪代码
public boolean dispatchTouchEvent(MotionEvent ev) {
    //如果该对象的监听成员变量不为空，则会调用其onTouch方法，
    if (mOnTouchListener != null && mOnTouchListener.onTouch(this, event)) {
        return true;    //若onTouch方法返回TRUE，则表示消费了该事件，则dispachtouTouchEvent返回TRUE，让其调用者知道该事件已被消费。
    }
    return onTouchEvent(ev);    //若监听成员为空或onTouch没有消费该事件，则调用对象自身的onTouchEvent方法处理。
}
```
从该方法的核心逻辑中可以看到，事件传递进来后，首先会对mOnTouchListener判空，如果之前Set了Listener，则会调用其onTouch方法。
* 若onTouch方法返回TRUE，则dispatchTouchEvent也会返回TRUE，表示消费该事件。
* 若onTouch方法返回FALSE，或者mOnTouchListener本来就是空，则调用自身的onTouchEvent()来处理，是否消费事件，可以由其返回值判断。

> 实际上，在View的onTouchEvent（）方法中，如果设置了onClickListener监听对象，则会调用其onClick()方法。

> 在同时设置了onTouchListener与onClickListener对象的情况下，正是由于View的dispacthTouchEvent方法会先调用mOnTouchListener的onTouch,才会调用onTouchEvent()方法，所以onTouchListener对象的onTouch方法是优先于onClickListener对象的onClick方法调用的。这里只简单描述结论，具体源码请查看本文对应的高级篇内容。

###小节：dispatchTouchEvent方法
回顾上面Activity、ViewGroup和View中的dispatchTouchEvent方法，它们大体都可以分为两部分，前一部分是交由子View的dispatchTouchEvent方法或onTouch方法进行处理，后一部分是交给自身的onTouchEvent方法处理。这样理解的话，就非常便于记忆了。

这个结构有点类似于递归的过程，就是组件的dispatchTouchEvent会自用子组件的同名方法，子组件一样会调用子子组件的同名方法，直到递归到底，然后在从递归底部返回上层。或者在进行递归调用时，传递到某个子View，其自身决定处理该事件，则事件交给其自身的onTouchEvent方法处理，如果onTouchEvent方法处理不了，再交由父组件的同名方法处理。

于是，就有了很多教程里的U型图。
![安卓分发事件U型图](https://i.loli.net/2018/11/03/5bdd6a80e06e3.png)

从U型图中可以发现，其实安卓事件分发的主体思路非常简单，即由父组件不断向子组件分发，若子组件能够处理，则立刻返回。若子组件都不处理，那传递到底层的子组件，再返回回来。这个过程类似上面说的递归的过程。

这里对这个U型图做一下说明，先看图中左上角，事件传到Activity，首先调用其dispatchTouchEvent方法，其会传递给子View处理，该子View（在图中是ViewGroup）会调用其dispatchTouchEvent方法，如果该方法被覆写直接返回TRUE,则立即返回Activity，表示已经消费事件。如果该方法没有被覆写或调用了super的同名方法，则会调用onInterceptTouchEvent方法，如果该方法返回TRUE拦截事件，则交给自身的onTouchEvent处理，如果该方法返回FALSE不拦截，则继续传给子子View（图中是View）的dispatchTouchEvent方法处理。此时，再看看这个U型图，该递归调用已经到底了，若在该方法中的onTouchListener方法不处理，则调用自身的onTouchEvent处理。若还是处理不了，则从递归底部向上返回，依次调用ViewGroup的、Activity的onTouchEvent方法。

> 实际上，用这个U型图来描述安卓的事件分发机制并不一定准确，因为同一对象的dispatchTouchEvent方法实际是包含了另外几个方法的（Activity与View只包含onTouchEvent),但是在这个图中，却是将几个方法分别画在不同的框中。所以通过该U型图来理解事件分发机智是不准确的。但是对于部分读者可能会有所帮助。要准确理解事件调用机制，还是应该回到上面，查看三个核心方法的核心逻辑，就能够准确理解。

> 强调说明，安卓事件分发的‘向上’与‘向下‘传播，不要与面向对象程序语言中基类与子类关系，或子类向上调用父类方法等概念搞混淆。对于安卓事件分发的‘向上’与‘向下‘传播，这里的上与下，是指在’递归‘调用过程中的上与下（也体现到U型图里的上与下）。这个概念，体现到布局中，就是外与内。即这里所说的事件’向下‘传播，等同于在布局上，由外向内传播，而’向上’传播，等同于在布局上，由内向外传播。

> 在面向对象程序语言中，对于子类覆盖父类方法，或子类调用父类方法，这些‘上’与‘下’的关系，在布局层面上并没有跨越布局层次，不要与事件传播的方向概念相混淆。


## 拦截方法onInterceptTouchEvent()
该方法是ViewGroup类对象所独有的，用于对事件进行提前拦截。在一般情况下，该方法是默认返回FALSE的，即不拦截。
如果自定义的ViewGroup希望拦截事件，不希望事件继续往子View传播，可以覆写该方法，返回TRUE，即可阻止向下的传播过程。

实际上，从上面的核心逻辑的伪代码中可以看出，在ViewGroup调用dispatchTouchEvent（）后，肯定会调用该方法，根据该方法的返回值来确定如何处理。若该方法返回True，则会将事件拦截掉，就给自身的onTouchEvent()处理。如果返回False,则继续传递给child执行分发流程。

## 处理方法onTouchEvent()
该方法主要对事件进行处理，若返回True表示已经处理了事件，若返回False则表示没有对事件进行处理，需要继续传递事件。一般情况下，默认为FALSE。


# 6. 总结
本文在介绍了事件分发基本概念的基础上，介绍了负责参与事件分发的核心方法，包括dispatchTouchEvent()、onInterceptTouchEvent（）与onTouchEvent（）方法。通过伪代码的形式介绍了这些方法的核心逻辑，重点分析了在Activity、ViewGroup与View中的dispatchTouchEvent()方法。它们三者中的该方法结构类似，都是先调用子View的同名方法或者listener方法，然后再调用自身的onTouchEvent()方法。

这些方法在调用关系中体现了一个类似‘递归’的调用过程，通过dispatchTouchEvent（）将事件传递下去，又通过onTouchEvent()将事件传递上来。中间的这一过程可以通过让onInterceptTouchEvent方法（对于ViewGroup），或者另外的负责分发的方法返回TRUE，均可以提前终止这一类似’递归‘的调用过程，进而让事件的处理符合我们的预期。


> 若有错漏，烦请斧正。转载请注明出处。
> * 作者：谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com