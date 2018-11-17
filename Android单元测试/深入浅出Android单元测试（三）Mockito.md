![欢迎关注程序引力](https://upload-images.jianshu.io/upload_images/14342329-acd327916ea5ec23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对Android有依赖的单元测试如何写？怎样脱离真机与模拟器？本文将会对Java测试框架mockito做详细介绍。

> 若有错漏，烦请斧正。转载请注明出处。
> * 作者：程序引力 | 谢一 （Evan Xie）
> * 邮箱：evanyixie@gmail.com

# 前言
在单元测试基础篇中，介绍了单元测试的基础以及JUnit以及AndroidJUnintRunner，对于测试不依赖于Android的Java代码，可以仅使用JUnit进行测试。但若需要测试的代码依赖于Android，则需要结合AndroidJUnitRunner并运行在真机或模拟器上才行。

不知开发者有没有一个疑问？能否有一种方式可以让对Android存在依赖的测试代码也直接运行于本地的JVM中？答案是肯定的。mockito则提供了这样的能力，能够脱离真机与模拟器运行对安卓有依赖的测试代码。

# Mockito概述
在实际的工程中，待测试的代码往往不是孤立存在的，其中的方法或变量可能依赖外部的变量。为此，为了测试某一部分代码，需要将其他被依赖的代码关联进来。更为可能的情况是，这些被依赖的代码可能还存在更多的依赖，这就导致开发者为了测试某一小部分的代码，而不得不与庞大的其他代码相关联。为了解决这个问题，Mockito应运而生。

Mockito是一款Java测试框架，它通过构造一些‘假’的对象，来将测试代码与其依赖隔离，进而提高了测试的易用性与运行效率。

Mockito主要有两个作用：
* 验证某个对象的行为。
* 验证某个对象的行为的调用次数。

Mockito优点：
* 能够将待测试代码与其依赖进行隔离
* 让一些对Android存在一定依赖的测试代码，运行在本地JVM上

适合使用Mockito的场景：
* 待测试代码对Android有较小的依赖
* 开发者希望待测试代码与其依赖隔离开


# 引入Mockito
使用Mockito前需要引入相应的库，除了JUnit框架外，还需在模块目录（如APP）的build.gradle中，添加如下依赖：
```
dependencies {
    testImplementation 'junit:junit:4.12'
    testImplementation 'org.mockito:mockito-core:1.10.19'
    androidTestImplementation 'org.mockito:mockito-core:1.10.19'
}
```
同步后即可引入Mockito。

# Mockito的基本对象
Mockito提供了两种基本对象，分别是mock对象与spy对象。
* mock对象：完全虚构的对象，除了自定义的行为外，无其他行为。
* spy对象：部分虚构的对象，除了自定义行为外，其他行为参考真实对象的行为。

可以这么理解，spy对象是在真实对象的基础上，自定义了部分行为，其他行就是原本真实对象的行为。而mock对象是完全虚构，完全自定义的，对于没有定位的行为，就仅有默认值。

# 撰写测试代码总体步骤
撰写基于Mockito框架的测试代码，主要步骤为：
* 构造mock/spy对象
* 定义对象的行为
* 运行测试代码
* 校验测试代码运行结果与预期结果

#  Mock对象简单用法
## 构造mock对象
构造mock对象的方式有两类，一类是通过mock方法进行创建，另一类是通过@Mock注解的方式创建。对于后者，又分为三种不同的方式
### 通过mock方法创建对象：
```
@Before
public void setUp() throws Exception {
    mockedList = mock(ArrayList.class);
}
```
对于setUp()方法，由于添加了@Before注解，故该方法在所有@Test方法之前执行。在该方法中，将需要被构造的类传入mock方法中，能够创建出mock对象。

### 通过@Mock注解的方式创建对象
* 方法1：使用前调用iniMocks方法：
```
@Mock
private ArrayList mockedList;

@Before
public void setUp() throws Exception {
    MockitoAnnotations.initMocks(this);
}
```
* 方法2：通过@RunWith注解：
```
@RunWith(MockitoJUnitRunner.class)
public class MockitoJUnitRunnerTest {
    @Mock
    AccountData accountData;
}
```
* 方法3：通过MockitoRule
```
public class MockitoRuleTest {
    @Mock
    AccountData accountData;
    @Rule
    public MockitoRule mockitoRule = MockitoJUnit.rule();
}

```

不管通过何种方式，创建出mock对象，是执行后续测试流程的前提。
## 定义mock对象行为
```
when(mockedList.get(0)).thenReturn("first");
```
定义mock对象行为的语法非常通俗易懂，其语法为当(when)调用什么时，返回(thenReturn)什么。对于上面的例子，即当调用mockedList.get(0)时，返回“first".

## 运行测试代码
构造mock对象，并且定义了该对象的行为后，即可运行该对象的相应方法与其他需要被测试的方法。在该简单的例子中，该步骤的代码为：
```
String res = mockedList.get(0);
```
## 校验测试代码运行结果与预期结果
得到运行结果后，可以通过断言与预期结构比较，得到测试结果。
```
assertEquals(res,"first")
```
## 本例完整代码
```
@RunWith(MockitoJUnitRunner.class)
public class MockitoTest {
    @Mock
    ArrayList mockedList;    //创建mock对象
    
    @Before
    public void setUp(){
        when(mockedList.get(0)).thenReturn("first");    //定义mock行为
    }
    
    @Test
    public void testMockito(){
        String res = (String)mockedList.get(0);    //调用mock对象方法
        assertEquals(res, "first");    //比较实际结果与预期
    }
}
```



# Mock对象存在安卓依赖的用法
对于存在Android依赖的情况，其测试代码的基本思路也与上文中的例子是一致的。也是在创建对象后，定义对象行为，然后调用对象方法并与预期比较。在下面的例子中，就不分布介绍了，具体看代码右侧的注释。

以安卓官网一个的例子看：
```
@RunWith(MockitoJUnitRunner.class)    //声明使用Mockito
public class UnitTestSample {
    private static final String FAKE_STRING = "HELLO WORLD";

    @Mock
    Context mMockContext;     //mock构造一个context对象

    @Test
    public void readStringFromContext_LocalizedString() {
        // 定义该mock对象的行为，即通过id或者字符串
        when(mMockContext.getString(R.string.hello_world)).thenReturn(FAKE_STRING);
        
        // 将该mockContext传入某个类，安卓场景中一般都会传入context
        ClassUnderTest myObjectUnderTest = new ClassUnderTest(mMockContext);

        // 运行测试代码，获取字符串
        String result = myObjectUnderTest.getHelloWorldString();

        // 校验实际结果与预期结果是否一致
        assertThat(result, is(FAKE_STRING));
    }
}
```

# Verify语句的用法
在前面的例子中，主要是定义mock对象的行为后，验证其行为与预期是否一致。但存在着另外一种情况，开发者希望验证mock对象的某个方法是否调用，以及调用的其他情况。那么可以使用Mockito提供的verify方法。

verify语句使用的形式为：
```
verify(mock对象).方法(参数)
```
如果该mock对象的该方法被调用过（调用时传入的参数也一样），则该测试通过。


## verify的简单例子
```
public class MockitoTest {
    @Mock
    ArrayList mockedList;

    @Before
    public void setUp(){
        //when(mockedList.get(0)).thenReturn("first");    //即使注释掉定义定位的代码
    }

    @Test
    public void testMockito(){
        String res = (String)mockedList.get(0);    //调用mock对象的get方法
        verify(mockedList).get(0);    //验证通过
        verify(mockedList).get(1);    //验证不通过
    }
}
```
在这个例子中，尽管没有定义mock对象的行为，但只要调用了该方法（此时get返回null)，调用verify方法也能使验证通过。若参数不一致，则验证不通过。

## verify方法的其他用法
除了简单验证mock对象的某个方法是否调用，verify还可以验证该方法的调用情况。
```
 verify(mockedList).get(0);    //验证方法是否调用，且参数传入的是0
 verify(mockedList, times(1)).get(0);    //验证方法是否调用且只调用了1次
 verify(mockedList, never()).get(0);    //验证方法是否没有被调用过
 verify(mockedList, atLeast(2)).get(0);    //验证方法是否调用且调用2次以上
 verify(mockedList, atMost(5)).get(0);    //验证方法是否调用且最多调用5次
 verify(mockedList).get(anyInt());    //验证方法是否调用，且传入的参数是任意整型数
```

# spy对象的用法
spy对象与mock对象的区别在于：
* mock对象：通过类进行创建，是完全虚构的对象，除了自定义的行为外，无其他行为。
* spy对象：通过对象进行创建，是部分虚构的对象，除了自定义行为外，其他行为参考真实对象的行为。

也就是说，对于spy对象，除了自定义的行为外，均把它当做原本的那个对象处理。简单的例子如下：
```
@RunWith(MockitoJUnitRunner.class)
public class AssertEquals {
    @Test
    public void testMockito(){
        ArrayList<Integer> integers = new ArrayList<>();
        ArrayList<Integer> spyList = spy(integers);    //通过真实的对象创建spy对象

        spyList.add(1);    //真实对象添加了一个元素
        System.out.println(spyList.get(0));    //查看其元素值，返回1

        when(spyList.size()).thenReturn(100);    //定义其行为，虚构其size为100
        System.out.println(spyList.size());    //查看其size,返回100

        when(spyList.get(0)).thenReturn(2);    //定义其行为，虚构其第一个元素值
        System.out.println(spyList.get(0));    //返回2，即虚构的行为会覆盖原有真实的行为
    }
}
```
从上面的例子中可知，spy仅仅是对真实对象的行为进行部分虚构，虚构的部分可以覆盖原来的部分。

# 总结
Android开发者应该了解到，在撰写单元测试之前，需要考虑自己的测试场景是什么，依赖是什么，需求是什么。如果需要测试的代码对Android有一定的依赖，同时又希望这样的单元测试是运行在本地JVM中，则可以选择mockito框架进行测试。


Mockito框架的使用方法也很简单，主要分为四步：
* 构造mock/spy对象
* 定义mock对象的行为
* 运行测试代码
* 校验测试代码运行结果与预期结果

只要记住这四步，同时查阅Mockito的API文档，了解它支持定义哪些行为，即可写出符合实际需要的单元测试用例。

> 若你喜欢本文或觉得有所帮助，请点赞或关注。
> 你的支持是对笔者最大的鼓励与肯定。比芯~