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
    // 关键点2.1
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {    
        onUserInteraction();
    }
    // 关键点2.2
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;    
    }
    // 关键点2.3
    return onTouchEvent(ev);
}
```
从上面的源码可以看到，对于关键点2.1，当Activity的dispatchTouchEvent方法接收到按下Down事件后，首先调用onUserInteraction方法。该方法在事件分发给Activity时会被调用，且该方法一般为空。如果开发者希望知道用户与设备的交互情况，可以覆写该方法。

> 实际上，onUserInteraction方法主要是用于管理状态栏通知，以及在恰当的时候取消通知。与该方法相关的还有另一个方法，onUserLeaveHint。该方法作为Activity生命周期回调的一部分，会在用户将Activity放到后台时调用（如用户点击Home键），该方法会在onPause方法之前调用。

对于关键点2.2，getWindow()返回的是Window类对象，而Window类实际是一个抽象类，它有一个唯一的实现类PhoneWindow。实际上，getWindow()返回的是PhoneWindow类对象，后面调用方法也实际上是PhoneWindow对象的方法,该方法实现如下：
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
该类继承自FrameLayout，即是ViewGroup的子类。调用super父类的分发方法，即是调用了ViewGroup的分发方法（详细介绍见下文）。通过该方法，可以将事件分发给此ViewGroup内部的子View处理。至此，Activity的分发流程介绍完毕。

总的说来，getWindow().superDispatchTouchEvent()这一语句将事件从Activity传递到了其ViewGroup，再通过该ViewGroup传递给其内部的子View或ViewGroup。如果getWindow()的分发方法返回TRUE，即表示已经消费了该事件，那么会通过return返回TRUE,告知调用者已经消费事件。否则在源码中的关键点2.3处，调用onTouchEvent()方法进行处理。

#### Activity的onTouchEvent方法

Activity会将事件不断向下分发给其内部的ViewGroup或View，若内部的ViewGroup以及View都不消费该事件，则事件会层层传递回来，到达Activity的onTouchEvent方法。
> 这一事件传递流程的介绍，请参考本系列文章的《基础篇》与《实践篇》。本篇内容更为偏向源码层面的分析与解读。

onTouchEvent方法的源码如下：
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
该方法主要是判断点击事件是否为Down按下事件，并且在边界之外，并且还判断了一些标志位。若按下事件在边界外，且标志位为TRUE，否则返回FALSE。返回TRUE的话，正如上文所说，会在onTouchEvent中调用finish方法，并返回TRUE，否则也直接返回FALSE.

#### Activity分发方法小节
总的说来，Activity的事件分发逻辑还是比较简单的。在Activity的分发方法dispatchTouchEvent方法中，首先会在按下事件调用onUserInteraction方法，该方法一般为空。之后会调用PhoneWindow对象的分发方法，最后通过ViewGroup的分发方法将事件分发给Activity内部的ViewGroup或View。若它们不能处理，则会将事件传回来，调用Activity的onTouchEvent方法。在该方法中，会判断是否应该关闭Activity，否则则返回默认的FALSE。


### 3.ViewGroup核心分发方法
对于ViewGroup，负责分发事件的方法有：
* dispatchTouchEvent（）
* onInterceptTouchEvent()
* onTouchEvent()

上面这些方法并不是并列同级的关系，事实上，dispatch方法包含了后面两个方法。更为准确地说，ViewGroup中并不包含onTouchEvent方法，只是由于ViewGroup是View的子类，而后者拥有onTouchEvent方法，故也认为ViewGroup也存在该方法。下面对这三个核心方法分别做介绍。

#### ViewGroup的dispatchTouchEvent方法
由于该方法是事件分发机制中最长的一个，也是众多读者最难以理解的一个。网上的相关教程和总结也仅仅是讲源码的细节，并未对方法进行总结，难以让读者有一个整体的认识。为此，在介绍该方法之前，先对方法中的一些关键变量作介绍，然后再列出方法中的核心步骤与行为，在此基础之上再阅读源码，则会事半功倍。

#### 关键变量
* ev : MotionEvent类型，即需要分发的事件对象。
* action ： int型，表示事件类型
* actionIndex : int型，表示action位运算后的值
* handled : 布尔型，表示该方法最后的处理结果，TRUE表示已处理消费，FALSE表示未消费。
* intercepted : 布尔型，表示是否拦截事件
* canceled ：布尔型，表示是否取消事件
* mFirstTouchTarget ：TouchTarget类型，用于描述被点击的View。实际上，该对象是一个链表的头指针。


#### 核心步骤
* Step1: 初始化、过滤、重置等工作。
* Step2: 判断是否应该拦截，调用拦截方法，给是否已经拦截的标志位赋值。
* Step3: 判断是否取消事件。
* Step4: 开始主要对按下事件处理。（若事件被拦截、取消或者事件不是按下事件，则不执行Step4到Step6）
* Step5: 循环对ViewGroup的子View进行处理。
* Step6: 尝试获得新的touchTarget,并添加到mFirstTouchTarget指向的链表。若无新的target,则以链表末尾节点为target。
* Step7: 将事件分发给mFirstTouchTarget处理。
* Step8: 收尾工作。

> 读者看完这些核心步骤没有完全理解也没有关系，此处只是帮助读者对ViewGroup中的事件分发机制有一个整体全面的认识。待读完后续的详细介绍后，再对这些步骤进行比对，会有更深刻的认识。


#### 源码分析
下面开始对ViewGroup的分发方法的源码进行分析。由于Android 9.0中的该方法源码非常长，故只分析其核心逻辑。同时，为了让读者更易于阅读，不用前后查阅，这些源码会以分片的形式展示。并且在源码附近会添加注释，用于更直观地解释源码。
```
 //方法的入参为ev,即点击事件，所有的分发都是围绕它来进行的。该变量属于上文介绍的关键变量。
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
// 重要的局部变量，该变量值表示是否处理了该事件。该变量也是方法最后的返回值。该值属于上文介绍的关键变量之一。
boolean handled = false;
// 出于安全原因，会过滤点击事件。在该方法中会对event的一些标志位进行处理。
// 返回FALSE表示丢弃事件，返回TRUE表示继续处理,即执行流程会进入if语句内部。
if (onFilterTouchEventForSecurity(ev)) {
    final int action = ev.getAction();    // action变量表示事件类型
    // 在安卓源码中有大量的位操作，通过进行位操作限定标志位范围，再对其做判断
    // 此处位操作的作用是保留action的末尾8bit位，其余置0，作为actionMasked
    final int actionMasked = action & MotionEvent.ACTION_MASK;

    // 因为事件流是以按下事件开始的，为此当按下事件到来时，会做一些初始化工作。
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // 初始化工作主要就是清空target以及重置状态
        cancelAndClearTouchTargets(ev);
        resetTouchState();
    }
```
上面这部分操作即属于步骤Step1。在过滤以及初始化工作结束后，下面进入Step2 拦截事件的部分，其源码片段为：
```
// 该标志位表示是否拦截事件，此处声明该变量
final boolean intercepted;
// 若该事件是按下事件，或者事件已经被某个组件(target)处理过，则进入if子句。
if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {

    // 该变量表示是否禁止拦截，这个标志位由子View控制
    // 子View通过调用requestDisallowInterceptTouchEvent来禁用父View拦截
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    // 默认情况下，都是允许拦截的，此时会调用拦截方法
    if (!disallowIntercept) {
        // 调用拦截方法，并将是否拦截的结果赋值给变量
        intercepted = onInterceptTouchEvent(ev);
        // 将ev事件中的action值恢复，因为前面进行了位操作
        ev.setAction(action); // restore action in case it was changed
    } else {
        // 如果子View禁用拦截，会使得父View不会调用拦截方法。即令下面变量为false，后面会判断该变量。
        intercepted = false;
    }
} else {
    // 如果没有组件处理过事件，同时当前事件已经不是按下事件，则拦截事件
    // 也就是说，按下事件已经分发了，但是没有任何组件处理它，所以剩余的事件都拦截
    intercepted = true;
}

// 若拦截该事件，或者事件已经有目标组件进行处理，则进行正常的事件分发
// 此处的focus具体含义待查
if (intercepted || mFirstTouchTarget != null) {
    ev.setTargetAccessibilityFocus(false);
}
```
下面是Step3，获取是否取消事件的标志位，并且声明变量。
```
// 获取是否取消事件的标志位
final boolean canceled = resetCancelNextUpFlag(this)
        || actionMasked == MotionEvent.ACTION_CANCEL;

// 获取是否将事件分发给多个子View的标志位
final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;

// 声明后续会使用到的变量
TouchTarget newTouchTarget = null;
boolean alreadyDispatchedToNewTouchTarget = false;
```
查看后续源码:
```
// 此部分首先判断是否取消事件或者拦截事件，若都是否的话，则进入该if子句
// 需要注意的是，该if语句没有else分支。
// 即对于被拦截或取消的事件，不执行该if子句中的所有方法。不执行上文总结的Step4到6
if (!canceled && !intercepted) {
    // If the event is targeting accessibility focus we give it to the
    // view that has accessibility focus and if it does not handle it
    // we clear the flag and dispatch the event to all children as usual.
    // We are looking up the accessibility focused host to avoid keeping
    // state since these events are very rare.
    View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
            ? findChildWithAccessibilityFocus() : null;

    // 该if语句没有else分支
    // 此处对事件的类型等进行判断，主要是处理按下事件
    if (actionMasked == MotionEvent.ACTION_DOWN
            || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
            || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {

        // 此处获得事件的actonIndex与事件id.
        final int actionIndex = ev.getActionIndex(); // always 0 for down
        final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                : TouchTarget.ALL_POINTER_IDS;

        // Clean up earlier touch targets for this pointer id in case they
        // have become out of sync.
        removePointersFromTouchTargets(idBitsToAssign
```
查看后续源码，属于上文总结的Step5 循环对ViewGroup的子View进行处理。可以看到下面这部分源码在对一些变量进行初始化后，主要就是一个循环体：
```
// 获取子View数量，如果为0，或者已经有处理事件的target组件，则不会进入if子句
final int childrenCount = mChildrenCount;
if (newTouchTarget == null && childrenCount != 0) {
    final float x = ev.getX(actionIndex);
    final float y = ev.getY(actionIndex);

    // 获取子View的前序遍历的列表
    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
    final boolean customOrder = preorderedList == null
            && isChildrenDrawingOrderEnabled();
    final View[] children = mChildren;

    // 以‘从尾到头’的方式遍历这个前序遍历的列表
    for (int i = childrenCount - 1; i >= 0; i--) {
        final int childIndex = getAndVerifyPreorderedIndex(
                childrenCount, i, customOrder);
        final View child = getAndVerifyPreorderedView(
                preorderedList, children, childIndex);

        // If there is a view that has accessibility focus we want it
        // to get the event first and if not handled we will perform a
        // normal dispatch. We may do a double iteration but this is
        // safer given the timeframe.
        // 这里会进行判断，如果有一个View具有焦点，并且该View就是当前的子View，会对该View做第二次遍历
        if (childWithAccessibilityFocus != null) {
            if (childWithAccessibilityFocus != child) {
                continue;
            }
            childWithAccessibilityFocus = null;
            // 序号减1，之后会再做一次遍历处理。
            i = childrenCount - 1;
        }

        // pointerEvent，待查
        if (!canViewReceivePointerEvents(child)
                || !isTransformedTouchPointInView(x, y, child, null)) {
            ev.setTargetAccessibilityFocus(false);
            continue;
        }
```
下面是上文总结的Step6，即对touchTarget进行处理。
```
        // 在getTouchTarget方法中，其逻辑是如果mFirstTouchTarget表示的链表中某一个节点就是child,则返回它作为新的target。若找不到则返回空。
        newTouchTarget = getTouchTarget(child);
        if (newTouchTarget != null) {
            // newTouchTarget不为空则表示该child子View已经在其范围内接收到了事件
            // 为此需要给它一个新的pointer ID
            // Give it the new pointer in addition to the ones it is handling.
            newTouchTarget.pointerIdBits |= idBitsToAssign;
            //注意，此处是break,结束循环。
            break; 
        }
```
下面的代码为newTouchTarget为空时才能执行得到，
```
        resetCancelNextUpFlag(child);
        // 下面这个为方法为一个重要的方法，即将事件与child传递进行，会在其中对child的分发方法进行递归调用。也就是说，这一部分遍历子View的逻辑是在一个循环中，对View进行递归处理。
        // 如果该方法返回TRUE，表示子View已经消费了该事件爱你，则执行if子句的方法。核心是将child添加到链表中，获得newTouchTarget.
        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
            // Child wants to receive touch within its bounds.
            mLastTouchDownTime = ev.getDownTime();
            if (preorderedList != null) {
                // childIndex points into presorted list, find original index
                for (int j = 0; j < childrenCount; j++) {
                    if (children[childIndex] == mChildren[j]) {
                        mLastTouchDownIndex = j;
                        break;
                    }
                }
            } else {
                mLastTouchDownIndex = childIndex;
            }
            mLastTouchDownX = ev.getX();
            mLastTouchDownY = ev.getY();
            // 将child添加到mFirstTouchTarget为头结点的链表中，并且返回这个节点。
            newTouchTarget = addTouchTarget(child, idBitsToAssign);
            // 将已经分发的标志设置为TRUE，并且break结束循环。
            alreadyDispatchedToNewTouchTarget = true;
            break;
        }

        // The accessibility focus didn't handle the event, so clear
        // the flag and do a normal dispatch to all children.
        ev.setTargetAccessibilityFocus(false);
```
下面是for循环之后的代码：
```
        // 该if子句进入的条件是，该ViewGroup中没有一个子View能够处理该事件，并且已经有View能够处理该事件了。则需要将链表的最后一个节点作为newTouchTarget,并重新分配point ID.
        if (newTouchTarget == null && mFirstTouchTarget != null) {
            newTouchTarget = mFirstTouchTarget;
            while (newTouchTarget.next != null) {
                newTouchTarget = newTouchTarget.next;
            }
            newTouchTarget.pointerIdBits |= idBitsToAssign;
        }
```
结合前面对子View遍历的循环中的break,分析一下可能的几种情况：
* 情况一：在for循环中遍历时，能够处理事件的View已经在链表中，则跳出循环
* 情况二：在循环遍历时，该View的子View能够处理事件，为此将该View添加到链表，也跳出循环

从上面的分析中，可以知道，这些代码对View进行遍历也好，递归也好，就是要找出一个View能够处理该事件，并且将该View作为newTouchTarget,并且添加到mFirstTouchTarget的链表中。如果实在找不到这样一个View，也会将链表中最后一个节点作为newTouchTarget。总之，经过这些处理后，就是得到一个mFirstTouchTarget链表。

下面是上文总结的步骤 Step7: 将事件分发给mFirstTouchTarget处理。
```
    // 若链表为空，则对事件进行再次分发。
    if (mFirstTouchTarget == null) {
        // No touch targets so treat this as an ordinary view.
        handled = dispatchTransformedTouchEvent(ev, canceled, null,
                TouchTarget.ALL_POINTER_IDS);
    } else {
        // 若链表不为空，则遍历这个链表，将事件分发给除了newTouchTarget之外的节点。
        TouchTarget predecessor = null;
        TouchTarget target = mFirstTouchTarget;
        while (target != null) {
            final TouchTarget next = target.next;
            if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                handled = true;
            } else {
                final boolean cancelChild = resetCancelNextUpFlag(target.child)
                        || intercepted;
                if (dispatchTransformedTouchEvent(ev, cancelChild,
                        target.child, target.pointerIdBits)) {
                    handled = true;
                }
                if (cancelChild) {
                    if (predecessor == null) {
                        mFirstTouchTarget = next;
                    } else {
                        predecessor.next = next;
                    }
                    target.recycle();
                    target = next;
                    continue;
                }
            }
            predecessor = target;
            target = next;
        }
    }
```
总结Step 7,可能有几种情况：
* 情况一：链表不为空，给链表中不为newTouchTarget的View分发事件
* 情况二：链表为空，以null遍历子View

在分发方法的最后，是Step8的额外收尾工作：
```
        // 若需要的话，更新链表
        // Update list of touch targets for pointer up or cancel, if needed.
        if (canceled
                || actionMasked == MotionEvent.ACTION_UP
                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
            resetTouchState();
        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
            final int actionIndex = ev.getActionIndex();
            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
            removePointersFromTouchTargets(idBitsToRemove);
        }
    }

    // 若没有消费事件，且某对象不为空，调用该回调方法。
    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    // 返回最终结果
    return handled;
```




