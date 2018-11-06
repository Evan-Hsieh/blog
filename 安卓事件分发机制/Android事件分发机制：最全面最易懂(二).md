![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何提升安卓水平？安卓开发者必须了解的事件分发机制。
最全面、最易懂的形式来讲解Android事件分发机制。

若有错漏，烦请斧正。转载请注明出处。
* 作者：谢一 （Evan Xie）
* 邮箱：evanyixie@gmail.com

# 0. 前言
鉴于安卓分发机制较为复杂，故分为多个层次进行讲解，分别为基础篇、实践篇与高级篇。

* （一）基础篇：从基本概念入手，介绍了分发机制中的核心方法，通过分析其核心逻辑，总结其事件分发机制。
* （二）实践篇：该篇设计了简单与复杂的两个demo样例，从现象与应用的角度去讲解分发机制的核心内容，帮助读者从另一个角度理解事件分发机制。
* （三）高级篇：从源码角度去分析分发机制背后的原因，让读者对分发机制背后的本质有更为全面与深刻的理解。

# 1. 内容简介

本文内容为（二）实践篇，本篇主要设计了两个demo样例，分别为基础样例与复杂样例。在基础样例中，只涉及单个的Activity、ViewGroup与View，对该样例中的事件实际分发情况进行分析，并总结其规律。在复杂样例中，涉及多个ViewGroup相互嵌套的情况，该样例更符合开发者在开发实际应用时所遇到的情况，由其现象得到的结论更具有借鉴意义。


# 2. 基本样例分析
经过（一）基础篇对分发过程的介绍，相信读者对安卓事件的分发过程已经有了一个基本的了解。下面以一个最基本的布局为例，看看事件分发过程是否真的如此。

![事件分发者结构](https://i.loli.net/2018/10/08/5bbb5665bb58f.png)

布局文件如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="50dp"
    tools:context=".lesson01.part01.OuterActivity">
    
    <com.evanxie.tutorial.lesson01.part01.LayoutX
        android:background="@color/blue"
        android:id="@+id/layout2"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.evanxie.tutorial.lesson01.part01.TextViewX
            android:background="@color/colorAccent"
            android:id="@+id/tv21"
            android:layout_margin="90dp"
            android:textSize="20sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="TextView" />

        <com.evanxie.tutorial.lesson01.part01.TextViewX
            android:background="@color/colorAccent"
            android:id="@+id/tv22"
            android:layout_margin="90dp"
            android:textSize="20sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="TextView" />
    </com.evanxie.tutorial.lesson01.part01.LayoutX>
</LinearLayout>
```

对于Activity，主要是覆写了如下方法，方便查看日志：
```
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.i(TAG, "dispatchTouchEvent: ");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.i(TAG, "onTouchEvent: ");
        return super.onTouchEvent(event);
    }
```

对于Layout(ViewGroup)，主要是覆写了如下方法：
```
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.i(TAG, "dispatchTouchEvent: " + getIdName());

        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.i(TAG, "onInterceptTouchEvent: " + getIdName());
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.i(TAG, "onTouchEvent: " + getIdName());
        return super.onTouchEvent(event);
    }

    private String getIdName() {
        return getResources().getResourceEntryName(getId());
    }
```

对于TextView，主要是覆写了如下方法：
```
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        Log.i(TAG, "dispatchTouchEvent: " + getText());
        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.i(TAG, "onTouchEvent: " + getText());
        return super.onTouchEvent(event);
    }

```

点击其中的TextView，并筛选其dispatchTouchEvent方法的日志，得到如下日志：
```
L0101: OuterActivity: dispatchTouchEvent: 
L0101: LayoutX: dispatchTouchEvent: layout2
L0101: TextViewX: dispatchTouchEvent: TextView
```

从该日志可以看出，事件是从Activity -> Layout(ViewGroup) -> TextView(View)进行传播的。从这里可以看出，有些文章中指出的，事件是由内部View（如button）开始传播的，这是有误的。

若取消前面的筛选，显示几个核心方法的日志，可以得到：
```
L0101: OuterActivity: dispatchTouchEvent: 
L0101: LayoutX: dispatchTouchEvent: layout2
L0101: LayoutX: onInterceptTouchEvent: layout2
L0101: TextViewX: dispatchTouchEvent: TextView
L0101: TextViewX: onTouchEvent: TextView
L0101: LayoutX: onTouchEvent: layout2
L0101: OuterActivity: onTouchEvent: 
```
从上面日志可以看出，Activity由于没有onInterceptTouchEvent()方法，所以在调用了dispatchTouchEvent()方法后，就调用了Layout的同名方法，但是在Layout却有所不同，它在调用dispatchTouchEvent()方法后，还调用了onInterceptTouchEvent()方法。

因为我们并没有改变方法后，还调用了onInterceptTouchEvent（）方法的返回值，故该事件继续分发，传递给TextView在调用dispatchTouchEvent（）处理。若不能继续分发，则会调用onTouchEvent()方法，同时，该方法也不能处理的话，则会向上传播，调用ViewGroup、Activity的onTouchEvent方法。

从这样的现象处罚，可以观察到如下结论：
* 结论5.1：事件分发的顺序是 Activity -> Layout(ViewGroup) -> TextView(View)
* 结论5.2：事件分发是一个递归调用的规程，通过dispatchTouchEvent（）进行递归调用。若考虑事件的整个传播过程，其分发的顺序是 Activity -> Layout(ViewGroup) -> TextView(View) -> Layout(ViewGroup) -> Activity.
* 结论5.3：事件若分发到ViewGroup，则首先会调用其onInterceptTouchEvent()方法，若该方法返回TURE，则事件不会继续向下传递，而是交由其自身onTouchEvent()处理，然后可能由onTouchEvent()由下往上传递回去。


# 3. 复杂样例分析
在实际的项目中，布局结果往往不会仅仅是上面基本样例那样简单。实际情况是可能会存在多个布局嵌套。故本节以一个较为复杂的样例来分析事件分发过程，理解该复杂样例的分发机制，有利于帮助开发者在实际项目中解决实际问题。

其Activity、Layout（ViewGroup）、与TextView的Java类代码与基本样例分析中的一致。唯独布局变得更为复杂，其布局示意图如下：
![0102复杂样例布局图](https://i.loli.net/2018/11/03/5bddb91472057.png)

对于该图中各空间的命名做如下约定：即 Name_X_Y。其中Name表示组件名，X表示层次，Y表示该层中第几个组件。对于TextView_2_3,表示位于第2层、第三个的TextView组件，对于Layout_3_4,表示处在第3层中第4个的Layout(ViewGroup)组件。最外层为第一层，向内一次递增。

为了更好地理解这些组件的关系，将它们绘制成树状图。其树状图如下：
![0102复杂样例布局树状图](https://i.loli.net/2018/11/03/5bddb926b895f.png)

从树状图中，可以清晰地看到该布局中有4层，第一层为一个Layout（ViewGroup），其内部有4个组件，分别是2个Layout（ViewGroup）与TextView。其余的组件关系也非常清晰。该复杂样例在设计时，考虑了ViewGroup多层嵌套的情况，通过该该样例的实验，可以对安卓事件分发机制有更深刻的理解，对于解决实际安卓应用的事件分发问题有一定的借鉴意义。

将该样例的代码运行起来后，点击TextView_4_3，设置筛选条件，只查看dispatchTouchEvent()方法的日志，得到相应的日志输出，结果如下
```
L0102: OuterActivity: dispatchTouchEvent: 
L0102: LayoutX: dispatchTouchEvent: Layout_1
L0102: LayoutX: dispatchTouchEvent: Layout_2_2
L0102: LayoutX: dispatchTouchEvent: Layout_3_4
L0102: TextViewX: dispatchTouchEvent: TextView_4_3
```
从该日志输出可以看出，分发事件从最外部的OuterActivity开始，然后传递第1层的Layout_1，然后依次传给Layout_2_2 -> Layout_3_4 -> TextView_4_3.

若将参与分发的组件用橙色标识出来，其传递的路径如下图所示：
![0102复杂样例布局树状点击图](https://i.loli.net/2018/11/03/5bddb934d6384.png)

从这样的结果可以看出，事件在复杂布局的情况下，并没有去遍历每一个子组件。

> 对于网络上的部分教程，其表示事件在传播时会遍历子View，这是比较模糊的。部分教程表示会ViewGroup会遍历其子View的dispatchTouchEvent()方法，那这可以说是有误的。

事实上，在一般情况下，ViewGroup并不会遍历其View的dispatchTouchEvent()方法，而像是”知道“被点击的子View的最短路径一样，通过该路径去分发事件，并让事件抵达被点击的组件。对于这个树状图，其余兄弟节点的组件的dispatchTouchEvent()方法并不会被调用。这是一个非常重要的结论。

从该现象出发，可以得到结论：
* 结论6.1：在一般情况下，可以认为事件分发是以‘最短路径’来分发的。

> 实际上，在源码中会判断点击的位置坐标是否处于其他组件的范围内，如果在点击位置的坐标在其范围之外，则不会去调用其dispatchTouchEvent()。这也就是上面结论的原因。除了一些较为特殊的情况，例如在同一层的组件存在重叠的情况下，其兄弟组件的dispatchTouchEvent()方法是可能被调用的。在一般情况下，就可以认为事件分发是以‘最短路径’来分发的。

若将日志的筛选条件去掉，点击TextView_4_3的日志如下所示：
```
L0102: OuterActivity: dispatchTouchEvent: 
L0102: LayoutX: dispatchTouchEvent: Layout_1
L0102: LayoutX: onInterceptTouchEvent: Layout_1
L0102: LayoutX: dispatchTouchEvent: Layout_2_2
L0102: LayoutX: onInterceptTouchEvent: Layout_2_2
L0102: LayoutX: dispatchTouchEvent: Layout_3_4
L0102: LayoutX: onInterceptTouchEvent: Layout_3_4
L0102: TextViewX: dispatchTouchEvent: TextView_4_3
L0102: TextViewX: onTouchEvent: TextView_4_3
L0102: LayoutX: onTouchEvent: Layout_3_4
L0102: LayoutX: onTouchEvent: Layout_2_2
L0102: LayoutX: onTouchEvent: Layout_1
L0102: OuterActivity: onTouchEvent: 
L0102: OuterActivity: dispatchTouchEvent: 
L0102: OuterActivity: onTouchEvent: 
```

从该日志可以印证前面的结论，事件从Activity想多个ViewGroup分发，最终抵达TextView，若没有组件能处理该事件，则通过onTouchEvent传递回去。

请注意日志的最后两行，似乎又有事件开始传递。实际上，前面一起的传递过程是‘按下’事件的传递过程，也是本文默认讨论的事件。最后两行是‘抬起’事件的传递过程。因为‘按下’事件在自上向下，又自下向上传递后，都没有组件能够处理，故默认Activity将该事件处理了。而对于‘按下’事件之后的其他事件，如‘抬起’事件，则默认交给处理了‘按下’事件的组件。

* 结论6.2：在一个事件序列（事件流）中，第一个事件（按下）被哪个组件处理了，那后续事件都会被直接交给这个组件处理。

再进行一次点击事件，此时不在点击TextView_4_3，而是点击布局图中仅仅属于Layout_2_2的绿色区域，得到的日志输出如下：
```
L0102: OuterActivity: dispatchTouchEvent: 
L0102: LayoutX: dispatchTouchEvent: Layout_1
L0102: LayoutX: onInterceptTouchEvent: Layout_1
L0102: LayoutX: dispatchTouchEvent: Layout_2_2
L0102: LayoutX: onInterceptTouchEvent: Layout_2_2
L0102: LayoutX: onTouchEvent: Layout_2_2
L0102: LayoutX: onTouchEvent: Layout_1
L0102: OuterActivity: onTouchEvent: 
```
从该现象可以看出，事件传递到Layout_2_2后，就不会再继续向下传播了，哪怕它还有许多子View。

* 结论6.3：当事件分发的被点击的组件时，则停止传播。不管这个组件是否还有子组件。

# 4. 总结
为了让读者更好地理解安卓事件调用机制，本文从实验的角度，设计了分别是较为基础的与较为复杂的两种样例，从该实验的现象出发，得出了若干结论。通过对这些结论的思考，可以帮助读者更深刻地理解安卓事件的分发机制，也可以帮助读者直接应用到实际项目中。此时思考（一）基础篇中的事件分发的核心方法逻辑，会发现其实安卓的事件分发机制原来如此简单。

* 结论1：事件分发的顺序是 Activity -> Layout(ViewGroup) -> TextView(View)
* 结论2：事件分发是一个递归调用的规程，通过dispatchTouchEvent（）进行递归调用。若考虑事件的整个传播过程，其分发的顺序是 Activity -> Layout(ViewGroup) -> TextView(View) -> Layout(ViewGroup) -> Activity.
* 结论3：事件若分发到ViewGroup，则首先会调用其onInterceptTouchEvent()方法，若该方法返回TURE，则事件不会继续向下传递，而是交由其自身onTouchEvent()处理，然后可能由onTouchEvent()由下往上传递回去。
* 结论4：在一般情况下，可以认为事件分发是以‘最短路径’来分发的。
* 结论5：在一个事件序列（事件流）中，第一个事件（按下）被哪个组件处理了，那后续事件都会被直接交给这个组件处理。
* 结论6：当事件分发的被点击的组件时，则停止传播。不管这个组件是否还有子组件。

> 若有错漏，烦请斧正。转载请注明出处。
> * 作者：谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com