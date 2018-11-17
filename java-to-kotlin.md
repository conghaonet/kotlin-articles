---
title: Kotlin For Android
author: 陈亮亮(291212)
date: 2018-03-02 19:00:00
categories:

- 经验总结


tags:

- kotlin

---

# Kotlin For Android

## 目录

[TOC]

## 1 介绍

### 1.1 Kotlin是什么

Kotlin是JetBrains开发的基于JVM的语言。

### 1.2 Kotlin能给什么

Kotlin是使用Java开发者的思维被创建出来的。对于Android开发者，Kotlin有如下特点：

- 易于学习，大部分语法同Java类似；
- 和Android Studio完美结合，开发者不需要对IDE做额外配置就能使用Kotlin。

Android开发者在Kotlin世界中属于一等公民。2017 Google IO，谷歌已经把Kotlin作为Android开发的官方语言。

相比Java7，Kotlin还具备哪些优势？

- 更加易表现

  简单的说，相同代码的逻辑，相比使用Java，Kotlin可以让你少写很多代码。

- 更加安全

  Kotlin是空安全的。Kotlin在编译期就处理了各种导致Null的情况，避免在运行时抛出NPT异常。

- 支持函数式编程

  Kotlin使用了很多函数式编程的概念，比如Lambda表达式，对Collections的处理方式。

- 可以扩展函数

  扩展函数意味着我们可以扩展一个类的能力，即使这个类我们并没有访问权限。

- 具有高度互操作性

  在项目中，我们可以同时使用Java和Kotlin编写代码，这两种语言之间具有完备的互操作性。

### 1.3 Code Time

#### 易表现

使用Kotlin可以避免开发者编写大量的模板代码。在平时Java开发过程中，IDE可以方便地帮我们生成许多通用的模版代码，但如果从语言层面上就支持，岂不是更好。毕竟，并不是每个人都有Uncle的水平，精通各种快速生成模版代码的快捷键。

```kotlin
data class Artist(
        var id: Long,
        var name: String,
        var url: String,
        var mbid: String
)
```

这个数据类，Kotlin会自动帮我们生成所有的属性、访问器以及一些基本方法。例如：toString()。

#### 空安全

Java开发中有一门必修课，叫防御式编程。外部模块返回给我们的数据，我们必须持谨慎的态度。通常拿到这个数据之前，需要做判空操作，只有非空的数据才能走接下来的处理逻辑，以避免在运行时抛NPT异常，保证程序的健壮性。

Kotlin是空安全的，我们可以通过［安全调用操作符(?)］明确地指定一个对象是否可以为空。需要明确，这里的空安全是在编译器就得到保证的。

```kotlin
// 这里不能通过编译. Artist 不能是null
var notNullArtist: Artist = null

// Artist 可以是 null
var artist: Artist? = null

// 无法编译, artist可能是null，我们需要进行处理
artist.print()

// 只要在artist != null时才会打印
artist?.print()

// 智能转换. 如果我们在之前进行了空检查，则不需要使用安全调用操作符调用
if (artist != null) {
  artist.print()
}

// 只有在确保artist不是null的情况下才能这么调用，否则它会抛出异常
artist!!.print()

// 使用Elvis操作符来给定一个在是null的情况下的替代值
val name = artist?.name ?: "empty"
```

#### 扩展方法

Kotlin允许我们给任何类添加函数。这种扩展现有类能力的方式，相比工具类更具有可读性。例如：我们想给Fragment增加一个显示Toast的函数。

```kotlin
fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) { 
    Toast.makeText(getActivity(), message, duration).show()
}
```

现在，我们可以这么调用：

```kotlin
fragment.toast("Hello world!")
```

#### 函数式编程

在UI编码过程中，如果我们要监听一个View的点击事件，使用Java语言的方式是创建一个匿名内部类，然后在匿名内部类的方法里，编写点击之后的处理逻辑。抛开语言层面，其实我们只是想表达，View点击之后需要做什么，而不是去创建一个匿名内部类。

```kotlin
view.setOnClickListener { toast("Hello world!") }
```

## 2 准备

### 2.1 IDE

Android Studio 3.0+。

###2.2 Plugin

如果只是想在现有的Android工程中支持Kotlin混合编程，并不需要额外配置。在工程中创建一个.kt文件，点击自动配置按钮，Android Studio就会自动帮我们完成配置。

## 3 初体验

### 3.1 创建项目

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxfli87b2j216w0lc0up.jpg)

创建一个新的Android工程时，勾选上Include Kotlin support，Android Studio会为我们完成相关配置，生成的模版Activity类也是Kotlin语言。

### 3.2 配置插件

如果现有的Android工程是通过Java语言编写的，现在想让这个工程也支持Kotlin语言编写，要怎么做？在Android Studio 3.0+中，只需在工程中创建一个Kotlin File，IDE就会提示我们需要进行Kotlin配置，按照步骤点击操作即可。

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxflg60zyj213c07itb6.jpg)

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxflh3zyvj20q00c2gna.jpg)

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxflh6lyjj20wk0dc0ul.jpg)

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxflgyhrcj20uc088di2.jpg)

![](http://ww1.sinaimg.cn/large/6f97245dgy1foxflhv3spj20x20l4whq.jpg)

整个配置包括3件事情：

- 声明`kotlin-gradle-plugin`；
- apply plugin `kotlin-android`；
- compile `kotlin-stdlib-jre7`。

### 3.3 代码转换

Kotlin Plugin包含了一个有趣的特性，能直接把Java代码转成Kotlin代码，入口在`Code` -> `Convert Java File to Kotlin File`。

## 4 语法

推荐在[try.kotlinlang.org](http://try.kotlinlang.org/)学习Kotlin语法。

### 4.1 类和函数

Kotlin中的类遵循一个简单的结构。同Java一样，如果你想定义一个类，你只需要使用`class`关键字。

```kotlin
class MainActivity{
	...
}
```

它有一个默认唯一的构造器。如果构造这个类需要额外的参数，只需要在类名后面写上相应的参数列表。

```kotlin
class Person(name: String, surname: String)
```

构造函数的函数体可以写在`init`块中：

```kotlin
class Person(name: String, surname: String) {
    init{
        ...
    }
}
```

默认任何类都是基础继承自`Any`（与java中的`Object`类似），但是我们可以继承其它类。所有的类默认都是不可继承的（final），所以我们只能继承那些明确声明`open`或者`abstract`的类：

```kotlin
open class Animal(name: String)
class Person(name: String, surname: String) : Animal(name)
```

当我们只有单个构造器时，我们需要在从父类继承下来的构造器中指定需要的参数。这是用来替换Java中的`super`调用的。

函数（Java中的方法）可以使用`fun`关键字就可以定义:

```kotlin
fun onCreate(savedInstanceState: Bundle?) {
}
```

如果你没有指定它的返回值，它就会返回`Unit`，与Java中的`void`类似，但是`Unit`是一个真正的对象。你当然也可以指定任何其它的返回类型：

```kotlin
fun add(x: Int, y: Int) : Int {
    return x + y
}
```

> Kotlin中，分号结尾不是必须的。不使用分号在实际编码过程中是一个不错的体验。

如果返回的结果可以使用一个表达式计算出来，你可以不使用括号而是使用等号：

```kotlin
fun add(x: Int,y: Int) : Int = x + y
```

可以给参数指定一个默认值使得它们变得可选。这里有一个例子，在Activity中创建了一个函数用来Toast一段信息：

```kotlin
fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, length).show()
}
```

第二个参数（length）指定了一个默认值。这意味着你调用的时候可以传入第二个值或者不传。

如果在Java中，要做到同样的调用逻辑，需要通过重载方法：

```kotlin
void toast(String message){
}

void toast(String message, int length){
    Toast.makeText(this, message, length).show();
}
```

这样可能体会不到太大的差别，但是如果有2个、3个、N个可选参数呢？在Java中重载方法的总数会以几何级增长。

### 4.2 变量和属性

和Java一样，在Kotlin中，**一切都是对象**，但是没有像Java中那样的原始基本类型。这样，我们就可以使用一致的方式来处理所有的可用类型。

#### 基本类型

在Kotlin中，基本类型integer，float或者boolean仍然存在，但是它们全部都会作为对象存在的。基本类型的名字和它们工作方式都是与Java非常相似的，但是有一些不同之处：

- 数字类型不会自动转型

  举个例子，我们不能给`Double`变量分配一个`Int`，必须要做一个明确的类型转换：

  ```kotlin
  val i:Int=7
  val d: Double = i.toDouble()
  ```

- 字符（char）不能作为数字处理

  如果要把字符作为数字处理，需要把他们显式地转换为一个数字：

  ```kotlin
  val c:Char='c'
  val i: Int = c.toInt()
  ```

- 位运算符

  ```java
  // Java
  int bitwiseOr = FLAG1 | FLAG2;
  int bitwiseAnd = FLAG1 & FLAG2;
  ```

  ```kotlin
  val bitwiseOr = FLAG1 or FLAG2
  val bitwiseAnd = FLAG1 and FLAG2
  ```

- 类型自动推断

  一个通用的Kotlin实践是省略变量的类型，让编译器自己去推断出具体的类型。

  ```kotlin
  val i = 12 // An Int
  val iHex = 0x0f // 一个十六进制的Int类型
  val l = 3L // A Long
  val d = 3.5 // A Double
  val f = 3.5F // A Float
  ```

- String可以像数组那样访问

  ```kotlin
  val s = "Example"
  val c = s[2] // 这是一个字符'a'
  // 迭代String
  val s = "Example"
  for(c in s){
      print(c)
  }
  ```

#### 变量

变量可以很简单地定义成可变(`var`)和不可变（`val`）的变量。这个与Java中使用的`final`很相似。

不可变的变量在Kotlin（和其它很多现代语言）中是一个很重要的概念。一个不可变的对象意味着它在实例化之后就不能再去改变它的状态了，这让编程更加具有健壮性和预估性。

在Java中，当模块把一个内部对象暴露给业务方之后，模块也就失去了对这个对象的控制权。业务方拿到这个对象之后，可以对它进行任意修改。如果模块的代码逻辑依赖这个对象的状态，很可能在业务方修改了这个对象之后，导致代码逻辑错误。为了解决这个问题，通常我们暴露给业务方的是一个新的对象，这个新对象持有相同的数据。

另外，不可变对象也可以说是线程安全的，因为它们无法去改变，也不需要去定义访问控制，因为所有线程访问到的对象都是同一个。

#### 属性

属性与Java中的字段是相同的，但是更加强大。属性做的事情是字段加上getter加上setter。下面通过一个例子来比较他们的不同之处。这是Java中字段安全访问和修改所需要的代码：

```java
public class Person {
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) { 
        this.name = name;
    }
}
...
Person person = new Person();
person.setName("name");
String name = person.getName();
```

在Kotlin中，只需要一个属性就可以了：

```kotlin
public class Person {
    var name: String = ""
}

...

val person = Person()
person.name = "name"
val name = person.name
```

如果没有任何指定，属性会默认使用getter和setter。当然它也可以修改为你自定义的代码，并且不修改存在的代码：

```kotlin
public classs Person {
    var name: String = ""
        get() = field.toUpperCase()
        set(value){
            field = "Name: $value"
        }
}
```

如果需要在getter和setter中访问这个属性自身的值，它需要创建一个`backing field`。可以使用`field`这个预留字段来访问，它会被编译器找到正在使用的并自动创建。需要注意的是，如果我们直接调用了属性，那我们会使用setter和getter而不是直接访问这个属性。`backing field`只能在属性访问器内访问。

### 4.3 数据类

数据类是一种非常强大的类，它可以让你避免创建Java中的用于保存状态但又操作非常简单的POJO的模版代码。它们通常只提供了用于访问它们属性的简单的getter和setter。

```
data class Forecast(val date: Date, val temperature: Float, val details: String)
```

####  额外的函数

数据类默认实现许多基本的函数，除了getter和setter函数，还有：

- equals()

  比较两个对象的属性来确保他们是相同的。

- hashCode()

  得到一个hash值，也是从属性中计算出来的。

- copy()

  拷贝一个对象，可以根据需要去修改里面的属性。

- 映射对象到变量中的函数

#### 复制数据类

如果我们使用不可修改的对象，就像我们之前讲过的，假如我们现在需要修改这个对象状态，必须要创建一个新的一个或者多个属性被修改的实例。这个任务是非常重复且不简洁的。举个例子，如果我们需要修改`Forecast`中的`temperature`（温度），我们可以这么做：

```kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val f2 = f1.copy(temperature = 30f)
```

#### 映射对象到变量

映射对象的每一个属性到一个变量中，这个过程就是Kotlin中的多声明，这就是为什么会有`componentX`函数被自动创建。继续使用上面的`Forecast`类举个例子：

```kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val (date, temperature, details) = f1
```

上面这个多声明会被编译成下面的代码：

```kotlin
val date = f1.component1()
val temperature = f1.component2()
val details = f1.component3()
```

这个特性背后的逻辑是非常强大的，它可以在很多情况下帮助我们简化代码。举个例子，`Map`类含有一些扩展函数的实现，允许它在迭代时使用key和value：

```kotlin
for ((key, value) in map) {
    Log.d("map", "key:$key, value:$value")
}
```

### 4.4 操作符

Kotin有一些固定数量象征性的操作符，我们可以在任何类中很容易地使用它们。方法是创建一个方法，方法名为保留的操作符关键字，这样就可以让这个操作符的行为映射到这个方法。

#### 操作符表

这里列举一系列包括`操作符`和`对应方法`的表。对应方法必须在指定的类中通过各种可能性被实现。

一元操作符：

| 操作符  | 函数             |
| ---- | -------------- |
| +a   | a.unaryPlus()  |
| -a   | a.unaryMinus() |
| !a   | a.not()        |
| a++  | a.inc()        |
| a--  | a.dec()        |

二元操作符：

| 操作符     | 函数               |
| ------- | ---------------- |
| a + b   | a.plus(b)        |
| a - b   | a.minus(b)       |
| a * b   | a.times(b)       |
| a / b   | a.div(b)         |
| a % b   | a.mod(b)         |
| a..b    | a.rangeTo(b)     |
| a in b  | b.contains(a)    |
| a !in b | !b.contains(a)   |
| a += b  | a.plusAssign(b)  |
| a -= b  | a.minusAssign(b) |
| a *= b  | a.timesAssign(b) |
| a /= b  | a.divAssign(b)   |
| a %= b  | a.modAssign(b)   |

数组操作符：

| 操作符                  | 函数                      |
| -------------------- | ----------------------- |
| a[i]                 | a.get(i)                |
| a[i, j]              | a.get(i, j)             |
| a[i_1, ..., i_n]     | a.get(i_1, ..., i_n)    |
| a[i] = b             | a.set(i, b)             |
| a[i, j] = b          | a.set(i, j, b)          |
| a[i_1, ..., i_n] = b | a.set(i_1, ..., i_n, b) |

等于操作符：

| 操作符    | 函数                            |
| ------ | ----------------------------- |
| a == b | a?.equals(b) ?: b === null    |
| a != b | !(a?.equals(b) ?: b === null) |

函数调用：

| 方法               | 调用                      |
| ---------------- | ----------------------- |
| a(i)             | a.invoke(i)             |
| a(i, j)          | a.invoke(i, j)          |
| a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n) |

#### 例子

Kotlin List是实现了数组操作符的，所以我们可以像Java中的数组一样访问List的每一项。除此之外，在可修改的List中，每一项也可以用一个简单的方式被直接设置：

```kotlin
val x = myList[2]
myList[2] = 4
```

前面提到到的ForecastList数据类，由很多其他额外的信息组成的。我们可以直接访问它的每一项而不是请求内部的List得到某一项。但是这里需要做一个完全不相关的事情，实现一个`size()`方法：

```kotlin
data class ForecastList(val city: String, val country: String,
                        val dailyForecast: List<Forecast>) {
    operator fun get(position: Int): Forecast = dailyForecast[position]
    fun size(): Int = dailyForecast.size
}
```

这样会使我们的`onBindViewHolder`更加简洁：

```kotlin
override fun onBindViewHolder(holder: ViewHolder,
        position: Int) {
    with(weekForecast[position]) {
        holder.textView.text = "$date - $description - $high/$low"
    }
}
```

### 4.5 流程控制

#### if

**在Kotlin中一切都是表达式**，也就是说一切都返回一个值。如果`if`条件不含有一个exception，那我们可以像使用方法那样使用它：

```kotlin
if(x>0){
    toast("x is greater than 0")
}else if(x==0){ 
    toast("x equals 0")
}else{
    toast("x is smaller than 0")
}

```

我们也可以把结果赋值给一个变量：

```kotlin
val res = if (x != null && x.size() >= days) x else null
```

这也说明Kotlin不需要像Java那样的三元操作符，只需要这样就能实现三元操作符一样的效果：

```kotlin
val z = if (condition) x else y
```

Kotlin的`if`表达式总是返回一个value。如果一个分支返回了Unit，那整个表达式也将返回Unit，它是可以被忽略的，这种情况下它的用法也就跟一般Java中的`if`条件一样了。

#### when

`when`表达式与Java中的`switch/case`类似，但是要强大得多。这个表达式会去试图匹配所有可能的分支直到找到满意的一项。然后它会运行右边的表达式。与Java的`switch/case`不同之处是参数可以是任何类型，并且分支也可以是一个条件。

对于默认的选项，我们可以增加一个`else`分支，它会在前面没有任何条件匹配时再执行。条件匹配成功后执行的代码也可以是代码块：

```kotlin
when (x){
    1 -> print("x == 1") 
    2 -> print("x == 2") 
    else -> {
        print("I'm a block")
        print("x is neither 1 nor 2")
    }
}
```

因为它是一个表达式，它也可以返回一个值。我们需要考虑什么时候作为一个表达式使用，它必须要覆盖所有分支的可能性或者实现`else`分支。否则它不会被编译成功：

```kotlin
val result = when (x) {
    0, 1 -> "binary"
    else -> "error"
}
```

如你所见，条件可以是一系列被逗号分割的值。但是它可以更多的匹配方式。比如，我们可以检测参数类型并进行判断：

```kotlin
when(view) {
    is TextView -> view.setText("I'm a TextView")
    is EditText -> toast("EditText value: ${view.getText()}")
    is ViewGroup -> toast("Number of children: ${view.getChildCount()} ")
    else -> view.visibility = View.GONE
}
```

在条件右边的代码中，参数会被自动转型，所以你不需要去明确地做类型转换。

它还让检测参数否在一个数组范围甚至是集合范围成为可能（我会在这章节的后面讲这个）：

```kotlin
val cost = when(x) {
    in 1..10 -> "cheap"
    in 10..100 -> "regular"
    in 100..1000 -> "expensive"
    in specialValues -> "special value!"
    else -> "not rated"
}
```

甚至参数可以来自不同的数据类型。它可以使用简单的`if/else`链替代：

```kotlin
valres=when{
    x in 1..10 -> "cheap"
    s.contains("hello") -> "it's a welcome!"
    v is ViewGroup -> "child count: ${v.getChildCount()}"
    else -> ""
}
```

#### for

Kotlin中的集合提供了各种各样的函数操作符供我们访问集合中的元素，因此一般情况下，我们很少需要去for一个集合去访问元素。但是for循环在一些情况下仍然是很有用的。提供一个迭代器它可以作用在任何东西上面：

```kotlin
for (item in collection) {
    print(item)
}
```

如果需要使用到index，可以使用`ranges`：

```kotlin
for (index in 0..viewGroup.getChildCount() - 1) {
    val view = viewGroup.getChildAt(index)
    view.visibility = View.VISIBLE
}
```

在我们迭代一个array或者list，一系列的index可以用来获取到指定的对象，所以上面的方式其实不是必要的：

```kotlin
for (i in array.indices)
    print(array[i])
```

#### while and do/while

使用方法和Java一样：

```kotlin
while(x > 0){ 
    x--
}

do{
    val y = retrieveData()
} while (y != null) // y在这里是可见的!
```

#### range

如果不去讲`ranges`的话，很难解释`control flow`。`ranges`表达式使用一个`..`操作符。

使用`ranges`可以以很多富有创造性的方式去简化代码。比如：

```kotlin
if(i >= 0 && i <= 10) 
    println(i)
```

转化成：

```kotlin
if (i in 0..10)
    println(i)
```

`ranges`被定义为可以被比较的任意类型，但是对于数字类型，比较器会通过转换它为简单的类似Java代码来避免额外开销的方式来优化它。数字类型的`ranges`也可以被迭代，编译器会将它们转换为与Java中使用index的for循环相同的字节码来进行优化：

```kotlin
for (i in 0..10)
    println(i)
```

`ranges` 默认是自增长的，所以如果是以下代码：

```kotlin
for (i in 10..0)
    println(i)
```

它就不会做任何事情。但是要想达到这个目的，可以使用`downTo`函数：

```kotlin
for(i in 10 downTo 0)
    println(i)
```

可以在`ranges`中使用`step`来指定自增长的步长：

```kotlin
for (i in 1..4 step 2) println(i)

for (i in 4 downTo 1 step 2) println(i)
```

如果想去创建一个open range（类似数学中的开区间），可以使用`until`函数：

```kotlin
for (i in 0 until 4) println(i)
```

这一行会打印从0到3，但是会跳过最后一个值。这也就是说`0 until 4 == 0..3`。在一个list中迭代时，使用`(i in 0 until list.size)`比`(i in 0..list.size - 1)`更加容易理解。

就如之前所提到的，使用`ranges`确实有富有创造性的方式。比如，一个简单的方式去从一个`ViewGroup`中得到一个Views列表可以这么做：

```kotlin
val views = (0..viewGroup.childCount - 1).map { viewGroup.getChildAt(it) }
```

### 4.6 集合和函数操作符

Kotlin中的集合结合相应的函数操作符非常强大。比如，如果我想去过滤一个list，不用去创建一个list，遍历这个list的每一项，然后如果满足一定的条件则放到一个新的集合中，而是直接食用filer函数并指明我想用的过滤器。用这种方式，我们可以节省大量的代码。

下面通过例子说明各个操作符的使用方法和使用时机。

#### 总数操作符

- any

  如果至少有一个元素符合给出的判断条件，则返回true。

  ```kotlin
  val list = listOf(1, 2, 3, 4, 5, 6)
  assertTrue(list.any { it % 2 == 0 })
  assertFalse(list.any { it > 10 })
  ```

- all

  如果全部的元素符合给出的判断条件，则返回true。

  ```kotlin
  assertTrue(list.all { it < 10 })
  assertFalse(list.all { it % 2 == 0 })
  ```

- count

  返回符合给出判断条件的元素总数。

  ```kotlin
  assertEquals(3, list.count { it % 2 == 0 })
  ```

- fold

  在一个初始值的基础上从第一项到最后一项通过一个函数累计所有的元素。

  ```kotlin
  assertEquals(25, list.fold(4) { total, next -> total + next })
  ```

- foldRight

  与`fold`一样，但是顺序是从最后一项到第一项。

  ```kotlin
  assertEquals(25, list.foldRight(4) { total, next -> total + next })
  ```

- forEach

  遍历所有元素，并执行给定的操作。

  ```kotlin
  list.forEach { println(it) }
  ```

- forEachIndexed

  与`forEach`，但是我们同时可以得到元素的index。

  ```kotlin
  list.forEachIndexed { index, value
          -> println("position $index contains a $value") }
  ```

- max

  返回最大的一项，如果没有则返回null。

  ```kotlin
  assertEquals(6, list.max())
  ```

- maxBy

  根据给定的函数返回最大的一项，如果没有则返回null。

  ```kotlin
  // The element whose negative is greater
  assertEquals(1, list.maxBy { -it })
  ```

- min

  返回最小的一项，如果没有则返回null。

  ```kotlin
  assertEquals(1, list.min())
  ```

- minBy

  根据给定的函数返回最小的一项，如果没有则返回null。

  ```kotlin
  // The element whose negative is smaller
  assertEquals(6, list.minBy { -it })
  ```

- none

  如果没有任何元素与给定的函数匹配，则返回true。

  ```kotlin
  // No elements are divisible by 7
  assertTrue(list.none { it % 7 == 0 })
  ```

- reduce

  与`fold`一样，但是没有一个初始值。通过一个函数从第一项到最后一项进行累计。

  ```kotlin
  assertEquals(21, list.reduce { total, next -> total + next })
  ```

- reduceRight

  与`reduce`一样，但是顺序是从最后一项到第一项。

  ```kotlin
  assertEquals(21, list.reduceRight { total, next -> total + next })
  ```

- sumBy

  返回所有每一项通过函数转换之后的数据的总和。

  ```kotlin
  assertEquals(3, list.sumBy { it % 2 })
  ```

除了总数操作符，还有：

- 过滤操作符
- 映射操作符
- 元素操作符
- 生产操作符
- 顺序操作符

**集合的函数操作符非常强大，熟悉使用它们可以消耗少量的代码完成许多复杂的数据处理逻辑。**这些操作符的详细使用说明和示例代码可以到官网查看。

### 4.7 接口

Kotlin中的接口比Java 7中要强大得多。如果你使用Java 8，它们非常相似。这里，我们抽象一个飞行动物的接口：

```kotlin
interface FlyingAnimal {
    fun fly()
}
```

鸟和蝙蝠都可以通过扇动翅膀的方式飞行。所以我们为它们创建两个类：

```kotlin
class Bird : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}

class Bat : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}
```

当两个类继承自一个接口，非常典型的是它们两者共享相同的实现。但是Java 7中的接口只能定义行为，但是不能去实现它。

Kotlin接口在某一方面它可以实现函数。它们与类唯一的不同之处是它们是无状态（stateless）的，所以属性需要子类去重写。类需要去负责保存接口属性的状态。

我们可以让接口实现`fly`函数：

```kotlin
interface FlyingAnimal {
    val wings: Wings
    fun fly() = wings.move()
}
```

就像提到的那样，类需要去重写属性：

```kotlin
class Bird : FlyingAnimal {
    override val wings: Wings = Wings()
}

class Bat : FlyingAnimal {
    override val wings: Wings = Wings()
}
```

现在鸟和蝙蝠都可以飞行了：

```kotlin
val bird = Bird()
val bat = Bat()

bird.fly()
bat.fly()
```

### 4.8 泛型

泛型编程是指在不指定代码中使用到的确切类型的情况下来编写算法。在Kotlin中，泛型甚至更加重要，因为经常使用扩展函数将会成倍增加我们泛型使用频率。

举个例子，我们可以创建一个指定泛型类：

```kotlin
class TypedClass<T>(parameter: T) {
    val value: T = parameter
}
```

这个类现在可以使用任何类型初始化，并且参数也会使用定义的类型，我们可以这么做：

```kotlin
val t1 = TypedClass<String>("Hello World!")
val t2 = TypedClass<Int>(25)
```

但是Kotlin很简单并且缩减了模版代码，如果编译器能够推断参数的类型，我们甚至也就不需要去指定它：

```kotlin
val t1 = TypedClass("Hello World!")
val t2 = TypedClass(25)
val t3 = TypedClass<String?>(null)
```

如第三个对象接收一个null引用，那仍然还是需要指定它的类型，因为它不能去推断出来。

我们可以像Java中那样在定义中指定的方式来增加类型限制。比如，如果我们想限制上一个类中为非null类型，我们只需要这么做：

```kotlin
class TypedClass<T : Any>(parameter: T) { 
    val value: T = parameter
}
```

如果你再去编译前面的代码，你将看到`t3`现在会抛出一个错误。可null类型不再被允许了。但是限制明显可以更加严厉。如果我们只希望`Context`的子类该怎么做？很简单：

```kotlin
class TypedClass<T : Context>(parameter: T) { 
    val value: T = parameter
}
```

现在所有继承`Context`的类都可以在我们这个类中使用。其它的类型是不被允许的。

泛型同样可以在函数中使用：

```kotlin
fun <T> typedFunction(item: T): List<T> {
    ...
}
```

### 4.9 访问修饰符

#### private

`private`修饰符是限制最高的修饰符。它表示它只能被自己所在的文件可见。所以如果给一个类声明为`private`，我们就不能在定义这个类之外的文件中使用它。

#### protected

这个修饰符只能被用在类或者接口中的成员上。一个包成员不能被定义为`protected`。定义在一个成员中，就与Java中的方式一样了：它可以被成员自己和继承它的成员可见（比如，类和它的子类）。

#### internal

如果是一个定义为`internal`的包成员的话，对所在的整个`module`可见。如果它是一个其它领域的成员，它就需要依赖那个领域的可见性了。比如，如果我们写了一个`private`类，那么它的`internal`修饰的函数的可见性就会限制与它所在的这个类的可见性。

我们可以访问同一个`module`中的`internal`修饰的类，但是不能访问其它`module`的。

> 什么是`module`
>
> > 根据Jetbrains的定义，一个`module`应该是一个单独的功能性的单位，它应该是可以被单独编译、运行、测试、debug的。根据我们项目不同的模块，可以在Android Studio中创建不同的`module`。在Eclipse中，这些`module`可以认为是在一个`workspace`中的不同的`project`。

#### public

你应该可以才想到，这是最没有限制的修饰符。**这是默认的修饰符**，成员在任何地方被修饰为`public`，很明显它只限制于它的领域。一个定义为`public`的成员被包含在一个`private`修饰的类中，这个成员在这个类以外也是不可见的。

### 4.10 Lambdas

Lambda表达式是一种很简单的方法，去定义一个匿名函数。Lambda是非常有用的，因为它们避免我们去写一些包含了某些函数的抽象类或者接口，然后在类中去实现它们。在Kotlin，我们把一个函数作为另一个函数的参数。

#### 简化Listener

我们用Android中非常典型的例子去解释它是怎么工作的：`View.setOnClickListener()`方法。如果我们想用Java的方式去增加点击事件的回调，我首先要编写一个`OnClickListener`接口：

```java
public interface OnClickListener {
    void onClick(View v);
}
```

然后我们要编写一个匿名内部类去实现这个接口：

```java
view.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
        Toast.makeText(v.getContext(), "Click", Toast.LENGTH_SHORT).show();
    }
})

```

我们将把上面的代码转换成Kotlin（使用了Anko的toast函数）：

```kotlin
view.setOnClickListener(object : OnClickListener {
    override fun onClick(v: View) {
        toast("Click")
    }
}
```

很幸运的是，Kotlin允许Java库的一些优化，Interface中包含单个函数可以被替代为一个函数。如果我们这么去定义了，它会正常执行：

```kotlin
fun setOnClickListener(listener: (View) -> Unit)

```

一个lambda表达式通过参数的形式被定义在箭头的左边（被圆括号包围），然后在箭头的右边返回结果值。在这个例子中，我们接收一个View，然后返回一个Unit（没有东西）。所以根据这种思想，我们可以把前面的代码简化成这样：

```kotlin
view.setOnClickListener({ view -> toast("Click")})

```

这是非常棒的简化！当我们定义了一个方法，我们必须使用大括号包围，然后在箭头的左边指定参数，在箭头的右边返回函数执行的结果。如果左边的参数没有使用到，我们甚至可以省略左边的参数：

```kotlin
view.setOnClickListener({ toast("Click") })
```

如果这个函数的最后一个参数是一个函数，我们可以把这个函数移动到圆括号外面：

```kotlin
view.setOnClickListener() { toast("Click") }
```

并且，最后，如果这个函数只有一个参数，我们可以省略这个圆括号：

```kotlin
view.setOnClickListener { toast("Click") }
```

比原始的Java代码简短了5倍多，并且更加容易理解它所做的事情。非常让人影响深刻。

## 5 使用Anko

[Anko](https://github.com/Kotlin/anko)是JetBrains开发的一个强大的库。它主要的目的是用来替代以前XML的方式来使用代码生成UI布局。此外，Anko还包含了很多的非常有帮助的函数和属性来避免我们写很多的模版代码。

## 6 参考

- [Kotlin Programming Language](https://kotlinlang.org/)
- [Kotlin for android Developers](https://antonioleiva.com/kotlin-android-developers-book/)