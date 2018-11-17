---
title: Kotlin Introduction
author: 陈友平(123210)
date: 2017-09-19 14:00:00
categories:

- 最新资讯


tags:
- Kotlin
- 安卓支持编程语言
- 编程语言

---

# Kotlin Introduction

本文链接: https://wb.101.com/QVfiaq

本文基于 Kotlin 的版本为 1.1.2-5

本文作者 Q2hlbllvdXBpbmcoYm94LWd1QGhvdG1haWwuY29tKQ==

------

[TOC]

------

## Kotlin 是啥?
- kotlin 是一门 Jetbrains 开发的语言, 2016.2.15发布了[1.0版本][1]
- 它可以跑在 JVM 或 Android 的 Dalvik/ART 上及其它可以跑 Java 的环境上
- 它也可以被编译为JS跑在浏览器上(ps: 有 TS 谁用 Kotlin 啊)
- 有个实验性项目, 让 Kotlin 可以直接跑在硬件上. [Kotlin/Native][2]
- 相比于其它基于 JVM 的语言(Scala/Groovy/Clojure/JPython/JRuby 等等), Kotlin 更像是 Java 的一个改进版本, 学起来比较容易.

## Kotlin on Android ~~安利~~
- Kotlin 可以编译出符合 Java 6 标准的 class 文件. 这点对 Android 开发者很重要. 现在大多数的 App 最小支持的 API 版本一般是 15/16. 这两个版本只支持到 Java 6.
 - Kotlin 虽然支持到 Java 6, 但编译要用 JDK 8 ([The Kotlin compiler now requires JDK 8][3])
 - Android Kitkat 及以上支持 Java 7, Android Nougat 及以上支持 Java 8
 - 可以用 [Retrolambda][4] or [Jack][5] 把一些 Java 8 上的特性跑在只支持旧版的 Java 的虚拟机上
- 标准库很小
 - stdlib 方法数较少(~7000, 开启 minify 还可以更少)
 - 把一些功能拆分成多个库, 如: 反射之类的功能是单独的库
 - 大部分功能是直接用现有的 Java 库实现的
- Android 官方支持 [Android Announces Support for Kotlin][6]
- [Anko][7] 用 DSL 的方式写声明UI界面
- [Kotlin Android Extensions][8] no more findViewById()
- 第三方库

## 如何开始
- 下载一个 Android Studio 3.0 ~~(说好的2.4呢?)~~
 - ~~光速~~[内网下载][9]
- [Get Started with Kotlin on Android][10]

## 基本语法

### 变量与不变量
 - 不变量(只读?) ```val time = System.currentTimeMillis()```
 - 变量 ```var time = System.currentTimeMillis()```
 - 具有自动类型推导, 当无法推导出类型时在命名后加冒号和类型, 如: ```var time: Long = System.currentTimeMillis()```
 - 没有 Java 里的基本类型(byte/short/int/long float/double boolean char)

### 字符串
 - 普通字符串: ```val c = "normal string"```
 - 模板字符串: ```val c = "current time=${System.currentTimeMillis()}"``` ```val c = "current time=$time"```~~fuck off ```
 - 不转义字符串(用三个双引号, 用在写json/xml等): ```val c = """no-escape string"""```

### 数组
- ```arrayOf(1, 2, 3)```

### ```switch-case``` -> ```when```表达式
 - ps: ```if-else```, ```try-catch```也是表达式

### Null-safety
 - 可以用```Unit```替代```null```
 - 函数返回值为```void```地方改为返回```Unit```

### 函数
  - 在 Kotlin 的世界里, 函数是一等公民, 不要求写在类内部
  - 以关键字```fun```开始, 返回值写在后, 返回值不能为```void```
  - 参数可以指点默认值 ```fun log(tag: String = "99U", message: String) { ... }```,  ```log(message = "onCreate")```, 当这样调用时这个 log 用的 tag = 99u
  - 如果函数的最后一个参数是函数, 且调用方传为的值为 lambda 时, 要以把这个 lambda 写在括号外.

example0 基本函数
```
fun log(): Unit {
    Log.d("TAG", "a log")
}
```
example1 参数指定值的函数
```
fun log(tag: String = "99U", message: String) {
    Log.d(tag, message)
}
```

```
log(message = "onCreate")
```

example2 最后一个参数为函数, 且传为值为 lambda
```
fun runWhenNotFinished(activity: Activity, block: () -> Unit): Unit {
    if (!activity.isFinishing) {
        block()
    }
}
```

```
runWhenNotFinished(this) {
    val transaction = supportFragmentManager.beginTransaction()
    transaction.add(R.id.action0, ListFragment())
    transaction.commit()
}
```

**```block: () -> Unit```是函数类型, 代指无参, 返回值为```Unit```的函数**

### 类/接口
- 构造函数
 - 直接写在类名后的括号内```class MyThread(threadName: String) {...}```
 - 没有```extends``` 和 ```implements```, 继承和实现都是冒号, 多个父级用逗号隔开 ```class MainActivity : AppCompatActivity(), ToolbarManager```
- Properties vs fields
 - 在 Kotlin 的类里是没有 Field 概念的
- ```static``` vs companion object
 - 在 Kotlin 里不允许有 static 的成员
 - 对于 Java 里的 static 方法, 在 Kotlin 里用包方法替代
 - 对于 Java 里的 static 属性, 在 Kotlin 里放到 [Object declarations][11] 内. ps: ```Companion Object```可以用来实现单例
 - 编译时可以确定的值可以用```const```修饰
- 接口里的方法可以有实现(类似 Java 8 里的 default 方法)
- 接口可以声明抽象的 Property

example0
```
class Customer @Inject constructor(name: String, age: Int) {
    init {
        Log.d("Customer", "init name=$name, age=$age")
    }

    constructor(name: String) : this(name, -1) {
        Log.d("secondary Customer", "init name=$name, age=-1")
    }
}
```

example1
```
class App : Application() {
    companion object {
        const val URL = "http://sdp.nd"
    }
}
```
```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    Log.d("TAG", App.URL)
}
```

example2
```
interface ToolbarManager {
    val toolbar: Toolbar // 抽象Property
    fun initToolbar() {
        toolbar.inflateMenu(R.menu.menu_main)
        toolbar.setOnMenuItemClickListener {
            when (it.itemId) {
                R.id.action_settings -> {
                    val intent = Intent(toolbar.context, SettingActivity::class.java)
                    toolbar.context.startActivity(intent)
                }
                else -> toast("Unknown option")
            }
            true
        }
    }
}
```

## 语法糖

### 扩展函数

#### 具体类的扩展函数

扩展函数体内部的 this 代指 Receiver 实例

- 给 Activity 加一个扩展函数 logFinish()

```
package com.example.extensions

import android.app.Activity
import android.util.Log

fun Activity.logFinish(): Unit {
    Log.d("Activity=[" + this + "]", "finish!!")
}
```

```
import com.example.extensions.logFinish
...

class MainActivity : Activity() {
    ...
    override fun finish() {
        super.finish()
        logFinish()
    }
}
```

- 时间戳转换为时间字符串

```
package com.example.extensions

import java.text.DateFormat
import java.util.*

fun Long.toDateString() : String {
    val dateInstance = DateFormat.getDateInstance(DateFormat.MEDIUM, Locale.getDefault())
    return dateInstance.format(this)
}
```

```
import com.example.extensions.toDateString
...

toolbar.setTitle(1497427698000L.toDateString())
```


#### 泛型类的扩展函数
关于```this```~~的魔法~~

```
inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```

```
inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }
```

```
inline fun <T, R> T.run(block: T.() -> R): R = block()
inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```

```
inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```

| return&param | - | it(this) |
| :--------:   | :-----:  | :----:  |
| this | apply | also |
| result | run/with | let |

**ps: 由扩展函数可以延伸出 DSL**

### Delegation
把 Properties 的getter/setter用别的类(方法)来实现
```
val toolbar by lazy { find<Toolbar>(R.id.toolbar) }
```

### data
自动生成重复代码: ```equal()```之类的
```
data class Coordinates(val lon: Float, val lat: Float)
```

### 析构
```
val coo = Coordinates(66, 20)
val (lon, lat) = coo // lon == 66, lat == 20
```
```
for ((key, value) in map) {...}
```

## 小技巧
- 当不懂一个语法在 Kotlin 里不懂怎么写时, 先编写Java代码, 然后在AS里选 ```Code -> Convert Java File to Kotlin File```, AS 会自动把 Java 代码转换为 Kotlin 代码
- Show Kotlin Bytecode -> Decompile

## Musing
- immutable vs val
  - ps: immutable vs Unmodifiable [SO][12]
```
// 这是一个 val, 但它是可变的
val viewSize = if (contentView is ViewGroup) (contentView as ViewGroup).childCount else 0
```
- properties
 - 由于 Properties 是由 getter/setter 组成的一个语法糖, 所以在类里用val声明 Property 里, 只是声明只读, 而不是声明不可变.

## Java Annotations

与 Java Annotations 带来的能力相比 Kotlin 的优势究竟有多大? 更遑论有 Retrolambda, RxJava 之类的项目已经可以很好的把一些新版 Java 的特性或新的编程思想带到旧平台上了.
那从学习语言的角度上来说 Kotlin 可以给我们带来多少新鲜的东西呢? 花这时间是不是去学习 Javascript/Python/Lua 之类不同思想的语言更好呢?

- [Dagger 2][13] 依赖注入
- [Android Architecture Components][14] 在 I/O 2017 [video][15] 新推出的开发框架
- [JakeWharton/hugo][16] 日志
- [JakeWharton/ButterKnife][17] 自动生成与View相关的方法
- [google/auto][18] 自动生成不可变的模型类(有丰富的插件, 如: 自动实现```Parcelable```相关方法)


## 参考
- [Comparison to Java Programming Language][19]
- [Kotlin Reference][20]
- [Kotlin and Android][21]
- [Kotlin Weekly][22]
- [《Kotlin for android developers》中文版翻译][23]
- 关于扩展函数和 DSL 建议阅读: [Kotlin — A deeper look][24] 和 [Kotlin & Android: A Brass Tacks Experiment 系列文章][25]


  [1]: https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/
  [2]: https://blog.jetbrains.com/kotlin/2017/04/kotlinnative-tech-preview-kotlin-without-a-vm/
  [3]: https://blog.jetbrains.com/kotlin/2017/04/kotlin-1-1-2-is-out
  [4]: https://github.com/orfjackal/retrolambda
  [5]: https://developer.android.com/guide/platform/j8-jack.html
  [6]: https://android-developers.googleblog.com/2017/05/android-announces-support-for-kotlin.html
  [7]: https://github.com/Kotlin/anko
  [8]: https://kotlinlang.org/docs/tutorials/android-plugin.html
  [9]: http://reference.sdp.nd/android/blog/2017/05/31/dev-tool-repo/
  [10]: https://developer.android.com/kotlin/get-started.html
  [11]: https://kotlinlang.org/docs/reference/object-declarations.html#object-declarations
  [12]: https://stackoverflow.com/questions/8892350/immutable-vs-unmodifiable-collection
  [13]: https://google.github.io/dagger/
  [14]: https://developer.android.com/topic/libraries/architecture/index.html
  [15]: https://www.youtube.com/watch?v=FrteWKKVyzI
  [16]: https://github.com/JakeWharton/hugo
  [17]: https://github.com/JakeWharton/butterknife
  [18]: https://github.com/google/auto/tree/master/value
  [19]: https://kotlinlang.org/docs/reference/comparison-to-java.html
  [20]: https://kotlinlang.org/docs/reference/
  [21]: https://developer.android.com/kotlin/index.html
  [22]: http://us12.campaign-archive2.com/home/?u=f39692e245b94f7fb693b6d82&id=93b2272cb6
  [23]: https://www.gitbook.com/book/wangjiegulu/kotlin-for-android-developers-zh/details
  [24]: https://hackernoon.com/kotlin-a-deeper-look-8569d4da36f
  [25]: https://www.google.com/#q=Kotlin+%26+Android:+A+Brass+Tacks+Experiment