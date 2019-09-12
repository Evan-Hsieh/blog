
### 问题背景
模块化在安卓开发中应用广泛，开发者可以将自己工程中的模块编译成aar提供给其他工程使用，也可以使用其他工程提供的模块aar. 但是这些aar可能有自己的依赖（如jar包）等。当外部工程在依赖aar时，可能无法依赖到aar的依赖。

### 模块被自身工程使用的情况
对于被依赖的模块，如果其是在自身的工程中，如果在编译时提示找不到相关的类或资源，常见的解决方案是：
* 将compileOnly修改为implementation或api
* 若是单元测试无法依赖，需添加androidTestImplementation
即解决思路就是修改build.gradle中的编译关键字，让其依赖在编译后，也能被打包到apk中。

### 模块被其他工程使用的情况
一般情况下，模块包括资源文件的话，需要将其编译成aar文件，才能被其他工程所依赖。但是会发现，不管是否使用implementation或者api关键字，其相关依赖都不会被编译到aar文件中。导致在依赖aar时，会告警找不到aar中的类。

常见的解决方案为：
* 将依赖复制到工程中
若模块中的依赖不是动态依赖，即是将依赖文件放置在本地路径下。可以将这些依赖复制到使用模块的工程中。不推荐该方法。

* 复制依赖到aar中
在对应模块的build.gradle中添加task
```
task copyLibs(type: Copy) {
    from configurations.compile
    into 'libs'
}
```
执行该任务后，相应的依赖会被包含到aar中。

* 在外部工程中添加依赖
例如在aar对应的模块中，其依赖是
```
implementation xxx
```
只需将这句命令复制到使用aar的工程的build.gradle中，即可实现对其的依赖。