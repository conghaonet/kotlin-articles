---
title: Kotlin之美——DSL篇
author: 张展(955908)
date: 2018-03-01 12:00:00
categories:

- Kotlin

tags:

- Kotlin

---

# Kotlin之美——DSL篇

![](http://upload-images.jianshu.io/upload_images/638283-4978a6313b725c62.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Kotlin DSL 把 Kotlin 的语法糖演绎得淋漓尽致，这些语法糖可谓好吃、好看又好玩，但是，仅痴迷于语法糖只会对语言的理解游离于表面，了解其实现原理，是我们阅读优秀源码、设计整洁代码和理解编程语言的必经之路，本文我们通过 DSL 来感受 Kotlin 之美。

# 理解 DSL

 DSL（domain specific language），即领域专用语言：专门解决某一特定问题的计算机语言，比如大家耳熟能详的 SQL 和正则表达式。

## 通用编程语言 vs DSL

通用编程语言（如 Java、Kotlin、Android等），往往提供了全面的库来帮助开发者开发完整的应用程序，而 DSL 只专注于某个领域，比如 SQL 仅支持数据库的相关处理，而正则表达式只用来检索和替换文本，我们无法用 SQL 或者正则表达式来开发一个完整的应用。

## API vs DSL

无论是通用编程语言，还是领域专用语言，最终都是要通过 API 的形式向开发者呈现。良好的、优雅的、整洁的、一致的 API 风格是每个优秀开发者的追求，而 DSL 往往具备**独特的代码结构和一致的代码风格**，从 SQL 和正则表达式的语法风格便可感受一二。

下文我们也将提到，Kotlin 构建的 DSL，代码风格更具表现力和想象力，也更加优雅。

## 内部 DSL

但是，如果为解决某一特定领域问题就创建一套独立的语言，开发成本和学习成本都很高，因此便有了内部 DSL 的概念。所谓内部 DSL，便是使用通用编程语言来构建 DSL。比如，本文提到的 Kotlin DSL，我们为 Kotlin DSL 做一个简单的定义：

**“使用 Kotlin 语言开发的，解决特定领域问题，具备独特代码结构的 API 。”**

下面，我们就来领略下千变万化的 Kotlin DSL 。


# 有趣的 Kotlin DSL

如果说 Kotlin 是一位魔术师，那么 DSL 便是其赖以成名，令人啧啧称赞的魔术作品，我们先来看下 Kotlin 在各个特定领域的有趣实现。

1. 日期

 ```
 val yesterday = 1.days.ago // 也可以这样写： val yesterday = 1 days ago
 val twoMonthsLater = 2 months fromNow
 ```
 
 以上日期处理的代码，真正做到见名知意，深谙代码整洁之道，更多细节可参考此库：[kxdate](https://github.com/yole/kxdate) 。
 
 如果不考虑规范，基于该库的设计思路，我们甚至可以设计出如下的 api：
 
 ```
 val yesterday = 1 天 前
 val twoMonthsLater = 2 月 后
 ```
 
**这个日期处理领域的 DSL 体现出来的代码结构是链式的，并且近似于我们日常使用的英语**
2. 单元测试

```
val str = "kotlin"
str should startWith("kot")
str.length shouldBe 6
```

与上述日期库的 api 风格类似，该单元测试的代码也是赏心悦目，更多细节可参考此库：[kotlintest](https://github.com/kotlintest/kotlintest) 。

基于该库的设计思路，我们甚至可以实现如下的代码风格，如同写英语句子一般简洁:

```
"kotlin" should start with "kot"
"kotlin" should have substring "otl"
```

**这个 DSL 的代码结构近似于我们日常使用的英语。**

3. HTML 构建器

```
fun createTable() = 
    table{
        tr{
            td{
                
            }
        }
    }
    
>>> println(createTable())
<table><tr><td></td></tr></table>
```

**这个 DSL 的代码结构使用了 lambda 嵌套，并且语义清晰，一目了然**。更多详情参考此库：[kotlinx.html](https://github.com/kotlin/kotlinx.html)。



4. SQL
```
(Users innerJoin Cities).slice(Users.name, Cities.name).
            select {(Users.id.eq("andrey") or Users.name.eq("Sergey")) and
                    Users.id.eq("sergey") and Users.cityId.eq(Cities.id)}.forEach {
            println("${it[Users.name]} lives in ${it[Cities.name]}")
        }
```

这类 SQL api 的风格，如果有用过 ORM 的框架，如 ActiveAndroid 或者 Realm 就不会陌生。以上代码来自于此库：[Exposed](https://github.com/JetBrains/Exposed) 。

5. Android 布局

[Anko Layouts]((https://github.com/Kotlin/anko)) 是一套帮助我们更简洁的开发和复用 Android 布局的 DSL ，它的代码风格如下：

```
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
       
        super.onCreate(savedInstanceState)
        verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { toast("Hello, ${name.text}!") }
            }
        }

    }
   
}
```
相比于笨重的 XML 布局方式，Anko DSL 显然是更先进和更高效的解决方案。

6. Gradle 构建

Gradle 的构建脚本是 groovy，对 Android 程序员有一定的学习成本，目前，Gradle 官方也提供了基于 Kotlin 的构建脚本：[Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) , 并提供了类 groovy 的代码风格：
```
dependencies {
    compile("com.android.support:appcompat-v7:27.0.1")
    compile("com.android.support.constraint:constraint-layout:1.0.2")
}
```

完整代码请参考：[build.gradle.kts](https://github.com/gradle/kotlin-dsl/blob/master/samples/hello-android/build.gradle.kts)。

**综上，Kotlin DSL 所体现的代码结构有如下特点：链式调用，大括号嵌套，并且可以近似于英语句子。**

# 实现原理
看了那么多 Kotlin DSL 的风格和使用场景，相较于刻板的、传统的 Java 而言，更加神奇和富有想象力。要理解 Kotlin DSL 这场魔术盛宴，就必须了解其背后用到的魔术道具——扩展函数、lambda、中缀调用和 invoke 约定。

## 扩展函数（扩展属性）

对于同样作为静态语言的 Kotlin 来说，扩展函数（扩展属性）是让他拥有类似于动态语言能力的法宝，即我们可以为任意对象动态的增加函数或属性。

比如，为 String 扩展一个函数： `lastChar()`:
```
package strings

fun String.lastChar(): Char = this.get(this.length - 1)
```

调用扩展函数：

```
>>> println("Kotlin".lastChar())
n
```

与 JavaScript 这类动态语言不一样，Kotlin 实现原理是： 提供静态工具类，将接收对象(此例为 String )做为参数传递进来,以下为该扩展函数编译成 Java 的代码

```
/* Java */
char c = StringUtilKt.lastChar("Java");
```

回顾前文讲到的日期的 DSL：

```
val yesterday = 1.days.ago
```

为配合扩展函数，我们先降低 api 的整洁程度，先实现一个扩展函数的版本：
```
val yesterday = 1.days().ago()
```

1 为 Int 类型，显然 Int 并没有 `days()` 函数，因此`days()` 为扩展函数，伪代码如下：

```
fun Int.days() = {//逻辑实现}
```

结合 Java8 的 Time api，此处将会涉及到两个扩展函数，完整实现如下：

```
fun Int.days() = Period.ofDays(this)
fun Period.ago() = LocalDate.now() - this
```

若要实现最终的效果，实际上就是将扩展函数修改为扩展属性的方式即可（扩展属性需提供getter或setter，本质上等同于扩展函数）：
```
val Int.days:Period
    get() = Period.ofDays(this)

val Period.ago:LocalDate
    get() = LocalDate.now() - this
```

代码虽少，却天马行空，妙趣横生。

## lambda
lambda 为 Java8 提供的新特性，于2014年3月18日发布。在2018年的今天我们依然无法使用或者要花很大的代价才能在 Android 编程中使用，而 Kotlin 则帮助我们解决了这一瓶颈，这也是我们拥抱 Kotlin 的原因之一。

lambda 是构建整洁代码的一大利器。

**1. lambda 表达式**

下图是 lambda 表达式，他总是用一对大括号包装起来，可以作为值传递给下节要提到的高阶函数。

![图片来自 Kotlin in Action](http://upload-images.jianshu.io/upload_images/638283-facd4dfcf991f635.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**2. 高阶函数**

关于高阶函数的定义，参考《Kotlin 实战》：
>高阶函数就是以另一个函数作为参数或返回值的函数

如果用 lamba 来作为高价函数的参数（此时为形参），就必须先了解如何声明一个函数的形参类型，如下：
![图片来自于 Kotlin in Action](http://upload-images.jianshu.io/upload_images/638283-ca1cffa4452f822c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相对于上一小节，我们应该弄清楚 lambda 作为实参和形参时的表现形式：

```java
// printSum 为高阶函数，定义了 lambda 形参
fun printSum(sum:(Int,Int)->Int){
        val result = sum(1, 2)
        println(result)
}

// 以下 lambda 为实参，传递给高阶函数 printSum
val sum = {x:Int,y:Int->x+y}
printSum(sum)

```

有了高阶函数，我们可以很轻易地做到**一个 lambda 嵌套另一个 lambda 的代码结构**。

**3. 大括号放在最后**

Kotlin 的 lambda 有个规约：如果 lambda 表达式是函数的最后一个实参，则可以放在括号外面，并且可以省略括号，如：

```
person.maxBy({ p:Person -> p.age })

// 可以写成
person.maxBy(){
    p:Person -> p.age
}

// 更简洁的风格：
person.maxBy{
    p:Person -> p.age
}

```
这个规约是 Kotlin DSL 实现嵌套结构的本质原因，比如上文提到的 anko Layout：

```
verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { toast("Hello, ${name.text}!") }
            }
        }
```
这里 verticalLayout 中 嵌套了 button，想必该库定义了如下函数：

```
fun verticalLayout( ()->Unit ){
    
}

fun button( text:String,()->Unit ){
    
}

```

verticalLayout 和 button 均是高阶函数，结合大括号放在最后的规约，就形成了 lambda 嵌套的语法结构。

**4. 带接收者的 lambda**

lambda 作为形参函数声明时，可以携带接收者，如下图：

  ![图片来自于 Kotlin in Action](http://upload-images.jianshu.io/upload_images/638283-1a6775d150b940c8.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

带接收者的 lambda 丰富了函数声明的信息，当传递该 lambda值时，将携带该接收者，比如：
  ```
// 声明接收者
fun kotlinDSL(block:StringBuilder.()->Unit){
    block(StringBuilder("Kotlin"))
}

// 调用高阶函数
kotlinDSL {
    // 这个 lambda 的接收者类型为StringBuilder
    append(" DSL")
    println(this)
}

 >>> 输出 Kotlin DSL
```

总而言之，lambda 在 Kotlin 和 Kotlin DSL 中扮演着很重要的角色，是实现整洁代码的必备语法糖。

## 中缀调用

Kotlin 中有种特殊的函数可以使用中缀调用，代码风格如下：
```
"key" to "value"

// 等价于
"key.to("value")

```

而 to() 的实现源码如下：
```
infix fun Any.to(that:Any) = Pair(this,that)
```

这段源码理解起来不难，infix 修饰符代表该函数支持中缀调用，然后为任意对象提供扩展函数 to，接受任意对象作为参数，最终返回键值对。

回顾下我们上文提到的不太规范的中文 api：
```
val yesteraty = 1 天 前
```
使用扩展函数和中缀调用便可实现：

```
object 前
infix fun Int.天(ago:前) = LocalDate.now() - Period.ofDays(this)

```
再比如上文提到的：

```
"kotlin" should start with "kot"

// 等价于
"kotlin".should(start).with("kot")
```

使用两个中缀调用便可实现，以下是伪代码：
```
object start
infix fun String.should(start:start):String = ""
infix fun String.with(str:String):String = ""
```
所以，中缀调用是实现类似英语句子结构 DSL 的核心。 

## invoke 约定

Kotlin 提供了 invoke 约定，可以让对象向函数一样直接调用，比如：

```
class Person(val name:String){
    operator fun invoke(){
        println("my name is $name")
    }
}

>>>val person = Person("geniusmart")
>>> person()
my name is geniusmart
```

回顾上文提到的 Gradle Kotlin DSL：
```
dependencies {
    compile("com.android.support:appcompat-v7:27.0.1")
    compile("com.android.support.constraint:constraint-layout:1.0.2")
}

// 等价于：
dependencies.compile("com.android.support:appcompat-v7:27.0.1")
dependencies.compile("com.android.support.constraint:constraint-layout:1.0.2")
```

这里，dependencies 是一个实例，既可以调用成员函数 compile，同时也可以直接传递 lambda 参数，后者便是采用了 invoke 约定，实现原理简化如下：
```
class Dependencies{

    fun compile(coordinate:String){
        println("add $coordinate")
    }

    operator fun invoke(block:Dependencies.()->Unit){
        block()
    }
}

>>>val dependencies = Dependencies()
>>>// 以两种方式分别调用 compile()
```

invoke 约定让对象调用函数的语法结构更加简洁。

# 总结

细细品味 Kotlin，你会发现她将代码整洁之道（Clean Code）和高效 Java 编程（Effective Java）中的部分精华融入到的语法和默认的规约中，因此她可以让开发者无形中写出整洁和高效的代码。

而更进一步， Kotlin DSL 则是对 Kotlin 所有语法糖的一个大融合，她的代码结构通常是链式调用、lambda 嵌套，并且接近于日常使用的英语句子，我们可以愉悦的使用 DSL 风格的 API，同时，也可以以此为思路，为社区贡献各种 Kotlin DSL。


Kotlin DSL 体现了代码的整洁之道，体现了天马行空的想象力，在 DSL 的点缀下，Kotlin 显示出整洁的美，自由的美。

Kotlin 有趣的外表之下，是一个更有趣的灵魂。

# 参考资料
* 《Kotlin 实战》

更多文章，请关注[我的博客](https://www.jianshu.com/u/9fa7fc2f3733)