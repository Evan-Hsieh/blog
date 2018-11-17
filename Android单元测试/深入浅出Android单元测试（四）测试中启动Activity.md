![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开发者有时希望能够在单元测试中显示界面，那应该如何做呢？

> 若有错漏，烦请斧正。转载请注明出处。
> * 作者：程序引力 | 谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com

单元测试主要是针对后台逻辑的测试，但一些业务与界面相关的话，就不仅仅涉及到后台代码了，可能需要启动Activity，显示界面。本文将介绍若干在单元测试中启动Activity的方式。

# 添加依赖
单元测试启动Activity必然就依赖Android库了，自然会考虑使用AndroidJunitRunner，为此需要在模块（如app）的build.gradle中添加依赖：
```
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
```

# 撰写测试代码
在AndroidStudio中，通过Ctrl + Shift + T 可以对制定类创建测试类，或在新建module时自动生成的测试类中，撰写如下代码：
```
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() {
        Context appContext = InstrumentationRegistry.getTargetContext();
        Intent intent = new Intent(appContext,AnotherActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        appContext.startActivity(intent);
    }
}
```
从代码中可以看出，通过InstrumentationRegistry获取到context，在初始化一个intent，跳转到AnotherActivity,然后通过context的startActivity方法跳转。

# 通过AndroidTestRule跳转
AndroidTestRule作为安卓基础类库，提供了对单个Activity进行测试的能力。为了使用它，一般需要在build.gradle中添加依赖：
```
androidTestImplementation 'com.android.support.test:rule:1.0.2'
```
对于AndroidTestRule，提供了多个构造方法，其中最全面的构造方法提供3个参数传入，分别是
| 参数 | 描述 |  
| :---: | :---: | 
| activityClass | 需要跳转的class | 
| initialTouchMode | 是否touch模式 | 
| launchActivity | 是否每个测试方法都启动Activity | 

下面是示例代码：
```
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getTargetContext();
        
        Intent intent = new Intent(appContext,OuterActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

        ActivityTestRule<OuterActivity> activityTestRule = new ActivityTestRule<>(OuterActivity.class, false, false);
        activityTestRule.launchActivity(intent);
    }
}
```
与通过context直接启动Activity不同，该方法是通过初始化一个ActivityTestRule对象，再通过该对象去启动Activity。



