# 兼容性概念
在论述兼容性时，不同的人可能对于先前与先后兼容存在完全相反的理解。为此，参考了安卓官方的定义，得到两者的概念。

## 向前兼容性
根据安卓文档，向前兼容指兼容更新的版本。

> 因为几乎所有对框架 API 的更改都是新增更改，所以使用 API 任何给定版本（其 API 级别所指定版本）开发的 Android 应用均向前兼容更新版本的 Android 平台以及更高 API 级别。 应用应该能够在所有后期版本的 Android 平台上运行，除非在个别情况下应用使用的某个 API 部分后来由于某种原因被移除。

## 向后兼容性
根据安卓文档，向后兼容指兼容较老的版本

> Android 应用不一定向后兼容比其编译时所针对的目标版本更久远的 Android 平台版本。由于早期版本的平台不包括新 API，因此使用新 API 的应用无法运行在这些平台上。

## minSdkVersion
### 安装时
如果系统的API level低于android:minSdkVersion设定的值，那么android系统会阻止用户安装这个应用。
例子：
将android:minSdkVersion设置为11， 并且将该应用安装在android 2.3的手机上（对应API level 9），则会提示：
Failure[INSTALL_FAILED_OLDER_SDK]

### 编译时
如果指明了这个属性，并且在项目中使用了高于这个API level的API， 那么会在编译时报错。

若指定的minSdkVersion为8，调用Activity.getActionBar()和ActionBar.getHeight()方法需要API level 11， 则会报错。

> 由此可见，minSdkVersion不仅在程序安装时起作用，也会在项目构建时起作用。

## targetSdkVersion
targetSdkVersion这个属性是在程序运行时期起作用的，系统根据这个属性决定要不要以兼容模式运行这个程序。


