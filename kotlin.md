---
title: Kotlin 快速上手(整理)
author: 林春伟(912025)
date: 2016-05-25 12:00:00
categories:

 
- 最新资讯 
 

tags: 

- Kotlin 
- 最新资讯
- 安卓支持编程语言
- 编程语言 
---

# Kotlin 快速上手
## 写在前面
2017 Google I/O 大会落幕，Google 正式宣布官方支持 Kotlin。Kotlin 是一个以 Apache License 开源的，基于 JVM 的新的编程语言， 由 JetBrains（目前广受欢迎的 Java IDE IntelliJ 的提供商，Android Studio 基于此）开发。它可以编译成Java字节码，也可以编译成JavaScript，方便在没有JVM的设备上运行。

本文只介绍它的基础语法以及在 Android 中的初步使用，旨在快速上手，关于该语言的高级特性不做阐述。  


### 为什么选择 Kotlin？
从 [官网](https://kotlinlang.org/) 我们可以了解到 Kotlin 具备以下特性：

- 简洁 - 大大减少样板代码的数量
- 安全 - 避免空指针异常等整个类的错误
- 互操作性 - 充分利用 JVM、Android 和浏览器的现有库
- 工具友好 - 可用任何 Java IDE 或者使用命令行构建

每个特性可参见具体官方示例。

### Android 中的 Kotlin 看起来是什么样的？
![Android 中的 Kotlin 看起来是什么样的？](images/kotlin_android_lookslike.png)  

可以看到 Kotlin 可以简化函数调用，同时也支持 Lambda 表达式；具体语法会在下文中体现。
  

## AS 配置

在最新发布的 [**Android Studio 3.0 Canary 1**](https://developer.android.com/studio/preview/index.html) 特性中已经支持使用 Kotlin。该版本是 preview 版，可以和稳定版同时安装。使用稳定版的同学可以通过安装插件的方式来支持 Kotlin（**安装完成后需要重新生效**）： 

![Android Studio 安装 Kotlin 插件](images/kotlin_plugin_install.png)  

安装生效后可以看到已经支持了 Kotlin 相关文件的创建：

![Android Studio 创建 Kotlin 类](images/kotlin_new_file.png)  


另外，不管是 Android Studio 3.0 版本还是早期装了 Kolin 插件的版本，都可以使用 **Converting Java code to Kotlin** 进行 java 代码的自动转换。

![Converting Java code to Kotlin*](images/kotlin_convert_file.png)  

**注意：千万不要直接使用 Converting Java code to Kotlin 来转换已有的成熟项目，毕竟涉及到具体业务，工具并不会太智能，该转换工具仅作为简单 java 类的转换，或者为了对比 Kotlin 和 Java 的语法区别而使用。如果需要把以后项目转向 Kotlin开发，则需要我们对各个类进行手动重构。**

### Gradle 配置

Android Studio 3.0 在创建项目时提供了启用 Kotlin 支持的选项，如果是早期版本则需要对工程进行 Kotlin 的配置。

如果还未配置工程，在编辑 Kotlin 类时会跳出 **"Kotlin not configured"** 提示，按提示配置最新版本后，build.gradle 文件就会自动更新。能看到新增了 apply plugin: 'kotlin-android' 及其依赖。

![Kotlin config *](images/kotlin_config.png)  

在工程根目录的 build.gradle 中 depandencies增加了一行：

```
dependencies {
	classpath 'com.android.tools.build:gradle:2.3.1' 
	classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version" //Kotlin 插件配置
}
```

在对应的 module 的 build.gradle 文件增加:
```
apply plugin: 'kotlin-android' //配置 Kotlin 插件
dependencies {
	compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version" //Kotlin 依赖
}
```

做完这些就可以开始使用 Kotlin 来开发我们的 Android 应用程序了。构建的应用程序，可以在虚拟机或连接的设备上运行。 所有这些工作与 Java 并无区别。 你可以发布应用程序，并以类似于使用 Java 编写的 Android 应用程序的方式进行签名。

Kotlin有着极小的运行时文件体积：整个库的大小约 859KB（1.1.2-2 版本）。这意味着 Kotlin 对 apk 文件大小影响微乎其微。

就对比 Kotlin 与 Java 所编写的程序而言，Kotlin 编译器所生成的字节码看上去几乎毫无差异。

## 基本语法

### 1. 变量
#### 1.1 变量声明

Kotlin 中用 var 声明变量，用 val 声明常量。在变量名后通过“ : String ”来声明类型为 String，Kotlin 支持类型推断，声明时可省略类型。如下代码，变量 id 的类型 int 可以根据值推断出来。另外，Kotlin 语句结尾的 “;” 是可选的。
```
val LOG = "MainActivity"    // val 声明常量
var num = 1                  // var 声明变量
var str : String  = "hello" // 显式标明类型
```
#### 1.2 如何延迟初始化成员变量

Java 定义的类成员变量如果不初始化，那么基本类型被初始化为其默认值，比如 int 初始化为 0，boolean 初始化为 false，非基本类型的成员则会被初始化为 null。
```
public class Hello{ 
	private String name; 
} 
```
类似的代码在 Kotlin 当中直译为：
```
class Hello{ 
	private var name: String? = null 
} 
```
使用了可空类型，副作用就是后面每次你想要用 name 的时候，都需要判断其是否为 null。如果不使用可空类型，需要加 lateinit 关键字：
```
class Hello{ 
	private lateinit var name: String 
} 
```
lateinit 是用来告诉编译器，name 这个变量后续会妥善处置的。

对于 final 的成员变量，Java 要求它们必须在构造方法或者构造块当中对他们进行初始化：
```
public class Hello{ 
	private final String name = "Peter"; 
} 
```
也就是说，如果我要想定义一个可以延迟到一定实际再使用并初始化的 final 变量，这在 Java 中是做不到的。

Kotlin 有办法，使用 lazy 这个 delegate 即可：
```
class Hello{ 
	private val name by lazy{ 
    	NameProvider.getName()  
	} 
}
```
只有使用到 name 这个属性的时候，lazy 后面的 Lambda 才会执行，name 的值才会真正计算出来。

### 2. 类

#### 2.1 类的声明

Kotlin 中类的写法较 Java 来说也更为简洁，“:” 符号相当于 Java 中的 extend 和 implements。
```
class MainActivity : AppCompatActivity() , View.OnClickListener{

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```
#### 2.2 如何实例化类

Kotlin 构造对象时，不需要像 new 这样的关键字，只需这样：
```
var hello = Hello()
```
### 2.3 类的构造器

Kotlin 中的类可以有一个 主构造器 (primary constructor), 以及一个或多个 次构造器 (secondary constructor). 主构造器是类头部的一部分, 位于类名称(以及可选的类型参数)之后.
```
class Person constructor(firstName: String) {
}
```
如果主构造器没有任何注解(annotation), 也没有任何可见度修饰符, 那么 constructor 关键字可以省略:
```
class Person(firstName: String) {
}
```
主构造器中不能包含任何代码. 初始化代码可以放在 初始化代码段 (initializer block) 中, 初始化代码段使用 init 关键字作为前缀:
```
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```
注意, 主构造器的参数可以在初始化代码段中使用. 也可以在类主体定义的属性初始化代码中使用:
```
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```
实际上, Kotlin 有一种简洁语法, 可以通过主构造器来定义属性并初始化属性值:
```
class Person(val firstName: String, val lastName: String, var age: Int) {
  // ...
}
```
与通常的属性一样, 主构造器中定义的属性可以是可变的(var), 也可以是只读的(val).

如果构造器有注解, 或者有可见度修饰符, 这时 constructor 关键字是必须的, 注解和修饰符要放在它之前:
```
class Customer public @Inject constructor(name: String) { ... }
```
次级构造器(secondary constructor)

类还可以声明 次级构造器 (secondary constructor), 使用 constructor 关键字作为前缀:
```
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```
如果类有主构造器, 那么每个次级构造器都必须委托给主构造器, 要么直接委托, 要么通过其他次级构造器间接委托. 委托到同一个类的另一个构造器时, 使用 this 关键字实现:
```
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```
如果一个非抽象类没有声明任何主构造器和次级构造器, 它将带有一个自动生成的, 无参数的主构造器. 这个构造器的可见度为 public. 如果不希望你的类带有 public 的构造器, 你需要声明一个空的构造器, 并明确设置其可见度:
```
class DontCreateMe private constructor () {
}
```
注意: 在 JVM 中, 如果主构造器的所有参数都指定了默认值, 编译器将会产生一个额外的无参数构造器, 这个无参数构造器会使用默认参数值来调用既有的构造器. 有些库(比如 Jackson 或 JPA) 会使用无参数构造器来创建对象实例, 这个特性将使得 Kotlin 比较容易与这种库协同工作.
```
class Customer(val customerName: String = "")
```
### 2.4 类的继承

Kotlin 中所有的类都有一个共同的超类 Any, 如果类声明时没有指定超类, 则默认为 Any:
```
class Example // 隐含地继承自 Any
```
Any 不是 java.lang.Object; 尤其要注意, 除 equals(), hashCode() 和 toString() 之外, 它没有任何成员.

要明确声明类的超类, 我们在类的头部添加一个冒号, 冒号之后指定超类:
```
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```
如果类有主构造器, 那么可以(而且必须)在主构造器中使用主构造器的参数来初始化基类.

如果类没有主构造器, 那么所有的次级构造器都必须使用 super 关键字来初始化基类, 或者委托到另一个构造器, 由被委托的构造器来初始化基类. 注意, 这种情况下, 不同的次级构造器可以调用基类中不同的构造器:
```
class MyView : View {
    constructor(ctx: Context) : super(ctx) {
    }
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs) {
    }
}
```
类上的 **open** 注解(annotation) 与 Java 的 final 正好相反: 这个注解表示允许从这个类继承出其他子类. 默认情况下, Kotlin 中所有的类都是 final 的, 这种设计符合 Effective Java, 一书中的第 17 条原则: 允许继承的地方, 应该明确设计, 并通过文档注明, 否则应该禁止继承.

### 3. 函数
#### 3.1 函数声明

使用 **fun** 关键字来声明函数，如下代码声明了 **sum** 函数用于计算两个参数的和，返回值类型写在后面，注意 Kotlin 中的 基础类型首字母都是大写的：
```
fun sum(a: Int, b:Int): Int {
	return a+b
}
```
这个时候你通过传入参数 sum(1, 2) 就可以得到计算后的值，你也可以通过设定参数的**默认值**来时间函数调用：
```
fun sum(a: Int = 1, b:Int = 2): Int {
	return a+b
}
```
如果需要声明**变参函数**可以使用 **vararg** 关键字：
```
fun getFirst(vararg a: Int): Int {
	return a[0]
}
```
在 Kotlin 中像上面这些简单函数可以**简写**成：
```
fun getFirst(vararg a: Int): Int = a[0]
fun sum(a: Int = 1, b:Int = 2): Int = a+b
```
#### 3.2 扩展函数

很多时候，Framework提供给我们的API往往都时比较原子的，调用时需要我们进行组合处理，因为就会产生了一些Util类，一个简单的例子，我们想要更快捷的展示Toast信息，在Java中我们可以这样做。
```
public static void longToast(Context context, String message) {
	Toast.makeText(context, message, Toast.LENGTH_LONG).show();
}
```
但是Kotlin的实现却让人惊奇，我们只需要重写扩展方法就可以了，比如这个longToast方法**扩展到所有的Context对象中**，如果不去追根溯源，可能无法区分是Framework提供的还是自行扩展的。
```
fun Context.longToast(message: String) {
	Toast.makeText(this, message, Toast.LENGTH_LONG).show()
}
applicationContext.longToast("hello world")
```
注意：Kotlin的方法扩展并不是真正修改了对应的类文件，而是在编译器和IDE方面做得处理。使我们看起来像是扩展了方法。


### 4. 空值检测

在开头的提到过，Kotlin 的特性中有一项是安全，它并避免空指针异常的错误。例如这句代码 `println(files?.size)`，只会在`files`不为空时执行。另外非空判断还可以执行语句块：

```
//当data不为空的时候，执行语句块
data?.let{
	//... 
}

//相反的，以下代码当data为空时才会执行
data?:let{
	//...
}
```

### 5. 可见度修饰符

类, 对象, 接口, 构造器, 函数, 属性, 以及属性的设值方法, 都可以使用可见度修饰符.(属性的取值方法永远与属性本身的可见度一致, 因此不需要控制其可见度.) Kotlin 中存在 4 种可见度修饰符: `private`, `protected`, `internal` 以及 `public`. 如果没有明确指定修饰符, 则使用默认的可见度`public`。

对于类内部的声明:

- private 表示只在这个类(以及它的所有成员)之内可以访问;
- protected — 与 private 一样, 另外在子类中也可以访问;
- internal — 在 本模块之内, 凡是能够访问到这个类的地方, 同时也能访问到这个类的 internal 成员;
- public — 凡是能够访问到这个类的地方, 同时也能访问这个类的 public 成员.。

Java 使用者 请注意: 在 Kotlin 中, 外部类(outer class)不能访问其内部类(inner class)的 private 成员。

如果你覆盖一个 protected 成员, 并且没有明确指定可见度, 那么覆盖后成员的可见度也将是 protected。

### 6. 语句
#### 6.1 in关键字的使用
```
判断一个对象是否在某一个区间内，可以使用in关键字

//如果存在于区间(1,Y-1)，则打印OK
if (x in 1..y-1) 
  print("OK")

//如果x不存在于array中，则输出Out
if (x !in 0..array.lastIndex) 
  print("Out")

//打印1到5
for (x in 1..5) 
  print(x)

//遍历集合(类似于Java中的for(String name : names))
for (name in names)
  println(name)

//如果names集合中包含text对象则打印yes
if (text in names)
  print("yes")
```
#### 6.2 when表达式

类似于 Java 中的 switch，但是 Kotlin 更加智能，可以自动判断参数的类型并转换为响应的匹配值。
```
fun cases(obj: Any) { 
  when (obj) {
    1       -> print("第一项")
    "hello" -> print("这个是字符串hello")
    is Long -> print("这是一个Long类型数据")
    !is String -> print("这不是String类型的数据")
    else    -> print("else类似于Java中的default")
  }
}
```

#### 6.3 如何声明数组

Java 的数组非常简单，当然也有些抽象，毕竟是编译期生成的类：
```
 String[] names = new String[]{"Kyo", "Ryu", "Iory"}; 
 String[] emptyStrings = new String[10]; 
```
Kotlin 的数组其实更真实一些，看上去更让人容易理解：
```
 val names: Array<String> = arrayOf("Kyo", "Ryu", "Iory") 
 val emptyStrings: Array<String?> = arrayOfNulls(10) 
```
注意到，Array T 即数组元素的类型。另外，String? 表示可以为 null 的 String 类型。

数组的使用基本一致。需要注意的是，为了避免装箱和拆箱的开销，Kotlin 对基本类型包括 Int、Short、Byte、Long、Float、Double、Char 等基本类型提供了定制版数组类型，写法为 XArray，例如 Int 的定制版数组为 IntArray，如果我们要定义一个整型数组，写法如下：
```
val ints = intArrayOf(1, 3, 5) 
```



### 参考链接
- [Kotlin 官网](https://kotlinlang.org/)
- [掘金 Kotlin 专栏](https://juejin.im/post/591dd9f544d904006c9fbb96)
- [Kotlin on GitHub](https://github.com/JetBrains/kotlin)
- [Kotlin 中文文档](http://www.kotlincn.net/docs/reference/)

PS: 上文部分内容摘自参考链接，非全部原创。整理这边文档的目的是为了让组员能够对 Kotlin 有个基础的认识，关于 Kotlin 的使用方法只做以上的初步介绍，具体的用法还需要大家继续深入学习。


--------------------------
by 林春伟（912025）





