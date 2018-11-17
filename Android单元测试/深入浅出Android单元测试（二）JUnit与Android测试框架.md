![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

想学习单元测试无从下手，本文对以最易懂的方式介绍单元测试。

> 若有错漏，烦请斧正。转载请注明出处。
> * 作者：程序引力 | 谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com


# JUnit
JUnit为Java最为著名的单元测试框架，在Java以及安卓开发中应用广泛。一般情况下，我们并不需要手动对JUnit进行配置。在Android Studio中创建工程时，IDE会自动在app/src/test目录下自动创建测试代码，并在build.gradle中生成相应的依赖。若需要手动添加，可以打开模块（如app）目录中的build.gradle,添加如下依赖即可。
```
dependencies {
    // Required for local unit tests (JUnit 4 framework)
    testImplementation 'junit:junit:4.12'
}
```

## 手动创建测试类
若要创建自己的测试代码，在app模块下的build.gradle中添加上述依赖后，只需要打开测试的类（例如选择MainActivity），然后执行：
* Ctrl + Shift + T （或 cmd + shift + T )弹出对话框；
* 选择Create New Test，或者直接点击回车；
* 更改或默认测试代码类名；
* 勾选需要被测试的方法（如onCreate)，点击OK；
* 选择生成测试代码的目录，对于纯JUnit测试代码选择app/srt/test目录；
* 点击OK，即可生成测试代码。

在app/src/test目录下，可以打开生成的测试代码。以测试OuterActivity的onCreate()方法为例，生成的测试代码主体为：
```
public class MainActivityTest {
    @Test
    public void onCreate() {
    }
}
```
会发现其中有一个注解@Test，该注解表示下面的方法为需要进行单元测试的方法。类似的，JUnit还有其他的注解，其含义如下：
| 注解   | 作用   |
| :---: | :---: | 
| @Test | 表示该方法需要进行单元测试 |
| @Test(timeout=5) | 表示测试方法执行要在5毫秒以内 |
| @Before | 在所有测试方法之前执行，常做初始化工作|
| @After | 在所有测试方法之后，常做收尾工作 | 
| @BeforeClass | 在所有测试类的方法之前执行 | 
| @AfterClass | 在所有测试类的方法之后执行 |
| @Ignore | 忽略该测试方法 |

上面的注解都只能作用于public void且无参的方法，而@BeforeClass与@AfterClass则还要求是public static void的方法。

了解这些注解还不够，还需要知道单元测试中用到的断言。最为常见的为assertEquals,其用法为
```
@Test
public void testAdd() {
    int x = 2;
    assertEquals(4, x + 2); // 通过
    assertEquals(4, x + 1); // 不通过
}
```
当执行该单元测试时，对于assertEquals，其会比较传入的两个参数值。若相等则通过，若不相等则抛出异常，测试不通过。对于同一个@Test注解的方法中，若有多个assertEquals,只要有一个不通过，则整个方法的单元测试就不通过。

除了assertEquals,还有其他断言语句，主要有如下几类：
| 语句   | 作用   |
| :---: | :---: | 
| assertTrue | 查看传入的布尔值是否为TRUE|
| assertNull | 查看传入的对象是否为NULL|
| assertEquals | 比较两个值是否相等 |
| assertArrayEquals | 比较两个集合的值是否相等 |
| assertSame | 比较两个对象是否是同一个 |
| assertThat | 查看一个表达式是否满足 |

上面的语句除了assertThat以外，都有与之匹配的“相反”作用的语句，如assertTrue有assertFalse、assertNull有assertNotNull与之配对。

## 撰写单元测试的基本步骤和思路
在了解了这些断言语句后，对于写单元测试代码就一点也不难了。大致可以分为如下几步：
* 创建测试类代码：参考上文的方法。
* 撰写初始化代码：检查是否有需要进行的初始化工作，若需要的话，在相应的初始化方法上添加@Before或@BeforeClass注解。
* 撰写待测试方法：对需要测试的方法前加上@Test注解，并在该方法中调用实际需要被测试的代码
* 撰写断言语句：通过断言语句判断实际结果与预期的值是否一致
* 撰写收尾工作代码：检查是否有需要进行收尾的工作，如关闭流，关闭文件等。可在相应方法上添加@After或@AfterClass注解。
* 运行测试类：右击测试类或测试类中的某个方法，选择Run即可运行测试。

> 时刻记住，其实测试代码与普通的Java代码没有本质什么区别


# AndroidUnitRunner
通过上文的介绍可知，若仅使用JUnit，则只能测试纯Java代码。但是在安卓开发中，许多代码或多或少都会有安卓依赖，这时则可以通过AndroidUnitRunner来实现测试代码。

AndroidUnitRunner的测试代码需要运行在真机或模拟器上，故该测试也被称作设备测试（Instrumented Test）。在使用AndroidUnitRunner前，需要在build.grdle中添加依赖
```
dependencies {
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:0.5'
} 
```
使用上文同样的方法创建测试类代码，唯一有区别的地方在于选择测试代码目录时，选择app/src/androidTest目录，则会在该目录下生成相应测试类代码。以创建工程时生成的AndroidTest为例，生成的代码如下：
```
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getTargetContext();
        assertEquals("com.evanxie.tutorial", appContext.getPackageName());
    }
}
```
与仅使用JUnit的本地单元测试方法不同，该测试类中新增了一个注解@RunWith，表示测试代码的执行环境。对于RunWith(AndroidJUnit4.class)表示运行环境是AndroidJUnit4.class提供的。从执行环境也能看出来，该测试类AndroidUnitRunner提供的。

其中@Test注解的方法为需要进行单元测试的方法。在该方法中，通过InstrumentationRegistry即可获取当前应用的上下文Context，拿到context对象，就可以做许多与安卓相关的操作了。在本例中是通过该对象可以获取应用包名，然后通过断言判断是否预期一致。

## InstrumentationRegistry
通过上面的例子可以知道，要进行与安卓库有关的测试，获取应用的context以及相关对象是非常重要的。而获取的方式即为通过InstrumentationRegistry获取。

InstrumentationRegistry作为对外暴露的接口，允许调用者获取设备信息，包括对设备的引用、应用的上下文以及参数等。InstrumentationRegistry提供的都是静态方法，包括：

| 静态方法  | 作用   |
| :---: | :---: | 
| getInstrumentation() | 获取设备实例 |
| getContext() | 获取设备包的上下文 |
| getTargetContext() | 获取设备上运行的应用上下文|
| getArguments() | 获取设备参数Bundle | 
| registerInstance | 记录存储实例 |

值得注意的是，这些静态方法中有getContext()与getTargetContext()。那它们有何区别呢？

参考StackOverflow上有关问题的解释，得知在运行设备测试（即本节讨论的这类测试）时，实际运行在设备中的应用有两个，一个是待测试的应用，另一个是执行测试逻辑的应用。
* 待测试应用：即开发者开发的应用，需要被单元测试代码进行测试
* 执行测试逻辑的应用：自动生成的应用，用于执行测试步骤与逻辑

对于getTargetContext()方法，即获取的是待测试应用的上下文。对于getContext()方法，获取的是执行测试逻辑的应用上下文。

# 总结
对Android本地测试与设备测试的两种主要框架做了简要介绍，分别是JUnit与AndroidUnitRunner。在讲解AndroidStudio中创建测试类的方法后，概括了撰写测试代码的记录思路，以及JUnit的主要注解。同时，介绍了AndroidUnitRunner框架的使用方法，以及InstrumentationRegistry主要接口，进而让开发者能够有能力撰写基本的测试用例。

> 若你喜欢本文或觉得有所帮助，请点赞或关注。
> 你的支持是对笔者最大的鼓励与肯定。比芯~