# Gradle

## 1. 复用Maven仓库

> 配置系统环境变量

```properties
CRADLE_USER_HOME=E:\MavenRepository  # Maven仓库路径
M2_HOME=D:\apache-maven-3.6.0 # Maven路径
```

> 修改.gradle文件

```groovy
repositories {
    /**
     * 先让gradle从本地仓库找,找不到再从下面的mavenCentral()中央仓库去找jar包
     */
    mavenLocal()
    mavenCentral()
}
```

## 2. dependencies的scope

2017 年google 后，Android studio版本更新至3.0，更新中，连带着com.android.tools.build:gradle 工具也升级到了3.0.0，在3.0.0中使用了最新的Gralde 4.0 里程碑版本作为gradle的编译版本，该版本gradle编译速度有所加速，更加欣喜的是，完全支持Java8。
当然，对于Kotlin的支持，在这个版本也有所体现，Kotlin插件默认是安装的。

在com.android.tools.build:gradle 3.0 以下版本依赖在gradle 中的声明写法

compile fileTree(dir: 'libs', include: ['*.jar'])

    1

但在3.0后的写法为

implementation fileTree(dir: 'libs', include: ['*.jar'])
或
api fileTree(dir: 'libs', include: ['*.jar'])

    1
    2
    3

api 指令

    完全等同于compile指令，没区别，你将所有的compile改成api，完全没有错。

implement指令

    使用了该命令编译的依赖，它仅仅对当前的Moudle提供接口。
    优点：1. 加快编译速度。2. 隐藏对外不必要的接口。

provided（compileOnly）

    只在编译时有效，不会参与打包
    可以在自己的moudle中使用该方式依赖一些比如com.android.support，gson这些使用者常用的库，避免冲突。

apk（runtimeOnly）

    只在生成apk的时候参与打包，编译时不会参与，很少用。

testCompile（testImplementation）

    testCompile 只在单元测试代码的编译以及最终打包测试apk时有效。

debugCompile（debugImplementation）

    debugCompile 只在 debug 模式的编译和最终的 debug apk 打包时有效

releaseCompile（releaseImplementation）

    Release compile仅仅针对 Release 模式的编译和最终的 Release apk 打包。3

### 2.1 compile和 implementation的区别

compile子类可以继承，implementation子类无法继承

### 2.2 排除冲突依赖

```txt
compile("${lib.sbStarterWeb}") {
	exclude group: 'org.springframework', module: 'spring-webmvc'
}
```

