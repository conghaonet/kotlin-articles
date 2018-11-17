---
title: Kotlin之美——高效篇
author: 张展(955908)
date: 2018-03-30 12:00:00
categories:

- Kotlin

tags:

- Kotlin

---

# Kotlin之美——高效篇

![俄罗斯圣彼得堡—Kotlin命名来自于其附近的一个岛屿](https://upload-images.jianshu.io/upload_images/638283-8aed6e4d053cda58.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Kotlin 作为后起之秀，站在巨人们的肩膀上是她得天独厚的优势，而这个巨人也包括—《Effective Java》，得益于这个巨人，Kotlin 到处散发着高效的味道，这篇文章让我们一起来领略下 Kotlin 的高效之道。

>第1条：考虑使用静态工厂方法代替构造器

在实例化对象的方式中，使用静态工厂方法相比构造器有几个好处:
1. 工厂方法拥有名字，易于开发者理解。
2. 不必在每次调用的时候都创建一个新对象，比如可以事先缓存好实例。
3. 可以返回原类型的任何子类型。

Kotlin 并没有 `static` 关键字，也没有静态成员的概念，取而代之的是『伴生对象』，因此，对于第一条准则，Kotlin 使用伴生对象关键字 `companion` 来定义静态工厂方法，代码风格如下：
```
class User private constructor(val account:String){

    companion object {

        fun newWeiboUser(email:String):User{
            return User(email)
        }

        fun newTelUser(tel:Long):User{
            return User(tel.toString())
        }
    }

}
```
调用方式类似 Java 中的静态方法：

```
val newTelUser = User.newTelUser(18888888888)
val weiBoUser = User.newWeiboUser("geniusmart")
```

>第3条：用私有构造器或者枚举类型强化Singleton属性

对于开发者而言，单例模式是最耳熟能详的设计模式，正如这第3条准则所述，单例模式有懒汉式、饿汉式、枚举等多种写法，其中前两者我们必须用私有构造器来禁止在单例之外的实例化。

Kotlin 对单例模式做了更彻底的精简，简直易如反掌，可以通过 `object` 关键字**声明一个单例类的同时创建一个实例**，如：
```
object singleton{//由于同时创建了实例，因此类名使用小写
    fun action(){
        println(this.hashCode())
    }
}
```
简单验证如下：
```
@Test
fun test(){
     val instance1 = singleton
     val instance2 = singleton
     assertEquals(instance1,instance2)
}
```
如果将 `object singleton` 转换成 Java，代码如下，大家可以感受下如何在声明一个单例类的同时创建一个实例：
```
public final class singleton {
   //在Java中使用singleton.INSTANCE来访问单例
   public static final singleton INSTANCE;
   private singleton() {
      INSTANCE = (singleton)this;
   }

   static {
      new singleton();
   }
}
```
Kotlin 让创建单例变得更高效。

>第13条：使类和成员的可访问性最小化

『封装』（也称之为信息隐藏）是面向对象的四大特性之一，体现在具体的实现层面便是四种访问权限：private、default、protected 和 public。

面向对象编程，我们的代码充满着类、成员属性和成员方法，这些都是我们对外的契约，如果类和成员都是可访问的，意味着我们后续的迭代版本都必须保持兼容，这显然是一项巨大的工程。

反之，充分利用好四种访问权限，将类和成员的可访问性控制到最小，更有利于程序的扩展。在这点上，Java 和 Kotlin 是大体一致的，但有细微区别：

![Kotlin 访问权限](https://upload-images.jianshu.io/upload_images/638283-044cc9f10f97f914.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. Kotlin 的默认访问权限为 `public`。
2. Kotlin 没有包级别访问权限。因为 Kotlin 认为包级别的访问权限很容易被破坏：只要使用者创建一个一模一样的包名即可访问，取代方案参照下一点。
3. Kotlin 新增了模块可见的访问权限 `internal`。
4. Kotlin 新增了顶层声明的类别（顶层函数和顶层属性，无需放在类中的属性和方法）。

关于 `internal`，举个栗子：假设工程里有两个 module，app 和 lib，app  依赖于 lib 工程，代码层级如下：

— app
——class Activity
— lib
—— internal class StringUtils

`StringUtils` 仅在 lib 工程中可视，app 工程中的 `Activity` 无法访问该类。

Kotlin 在访问权限的设计更彻底的贯彻了『使可访问性最小化』的准则。

>第14条：在公有类中使用访问方法而非公有域

```
public class Point {
    public double x;
    public double y;
}
```
如上代码，我们会直接调用 `public` 修饰的成员属性（即准则中的公有域），《Effective Java》 不建议这么用，取而代之的是将成员属性定义成私有的，并且提供 `public` 修饰的 `set` 和 `get` 方法。

原因很简单：**如果直接暴露成员属性，将来想改变其内部实现是不可能的，反之，如果是暴露方法，则可以在方法中轻易地修改实现。**

对于这条准则，Kotlin 在语法层面直接默认约束了：
```
class User{
    val num = 10//属性默认为private，且拥有public的getNum()
    var nickname = "geniusmart"//同上
}
```
调用属性的时，看似直接访问，实则访问的是 get 和 set 方法：
```
@Test
fun test(){
    val user = User()
    println(user.num)//实际上调用的是getNum()
    user.nickname = "Mr.Geniusmart"//实际上调用的是setNum()
    println(user.nickname)
}
```
如果哪一天，业务需要我们将所有昵称带上邮箱，此时亡羊补牢显得轻而易举：
```
class User{

    val num = 10
    var nickname = "geniusmart"
        get() = field.plus("@email.com")

}
``` 
Kotlin 的 setter 和 getter 规约完美吻合第14条准则。

>第16条：组合优先于继承（原书是复合优先于继承）

『组合优先于继承』是面向对象中非常重要的原则之一。继承破坏了封装性，父类必须暴露更多的细节让子类知道（比如使用 protected 访问权限），同时子类依赖于父类的实现，一旦父类改变，子类都会受影响。

举例说明，我们想对 `HashSet` 增加『计算新增元素个数』的能力，经过多年面向对象的熏陶，我们信誓旦旦的采用继承的方式：定义 HashSet 的子类，在子类中进行扩展:

```
class CountingSet: HashSet<String>() {

    var count = 0

    override fun add(element: String): Boolean {
        count++
        return super.add(element)
    }

    override fun addAll(elements: Collection<String>): Boolean {
        count+=elements.size
        return super.addAll(elements)
    }
}
```
然而事与愿违的是，父类的 `addAll()` 将会循环调用 `add()`，因此，计数器会成倍的增加计数，测试代码如下：
 ```
@Test
fun test(){

    val countingSet = CountingSet()
    countingSet.addAll(setOf("1","2","3"))
    println("countingSet.count=${countingSet.count}")//期望是3，实际上是6

}
```

这个例子告诉我们，继承是多么不可靠，子类与父类的耦合度太强，需要了解太多父类的实现。

『继承』不是最优解，相较而言，『组合』在这种场景下是更可靠的解决方案：
```
class CountintSetComposite(val countingSet : HashSet<String> ){

    var count = 0
    
    fun contains(element: String) {
        countingSet.contains(element)
    }

    fun add(element: String): Boolean {
        count++
        return countingSet.add(element)
    }

    // 庞大的工作量：声明HashSet的所有方法。。
}
```
但是，这里最大的问题在于：**我们必须将父类的所有方法都声明一遍，仅仅是为了扩展其中两个方法 add 和 addAll**。

Kotlin 再次体现了其追求高效的本质，『类委托』是 Kotlin 用来简化『组合』的利器：
```
class CountingSetBy(val countingSet: MutableCollection<String>):MutableCollection<String> by countingSet{

    var count = 0

    override fun add(element: String): Boolean {
        count++
        return countingSet.add(element)
    }

    override fun addAll(elements: Collection<String>): Boolean {
        count+=elements.size
        return countingSet.addAll(elements)
    }
}
```
此例中，MutableCollection（在 Kotlin 中作为 HashSet 的父接口）将其实现委托给 countingSet，我们只需要专注于需要扩展的方法即可。

注：准确来说，组合更多的目的是增加原始对象的能力，因此是『装饰』而非『代理』，而 Kotlin 的委托类在字面意思上更多的还是体现『代理』的味道。

>第17条：要么为继承而设计，并提供文档说明，要么就禁止继承

继承的缺点我们已经在上条准则领略到了，更进一步地，接下来这条准则告诉我们：如没有必要提供继承，则禁止。那么如何来禁止继承？其实很简单，将类定义为 final 类，退而求其次，如果类允许继承，则定义不允许重写的方法为 final 方法。

既然这是个更好的实践，为什么将其作为默认设计？Kotlin 便是这个思路的践行者，**Kotlin 中创建的类和方法默认都是 final 的**：
```
class Parent{
    fun action(){
        
    }
}

/* 
// 等价于：
public final class Parent {
   public final void action() {
   }
}
*/
```

如果经过深思熟虑，一定要提供继承和重写，则对类或方法增加 `open` 修饰符即可。

>第21条：用函数对象表示策略

关于这条准则，我们从策略模式讲起：
![策略模式类图](https://upload-images.jianshu.io/upload_images/638283-c7310aad09f7f48d.gif?imageMogr2/auto-orient/strip)

以 Java 的思维模式而言，首先要定义策略接口，及具体的策略实现类：
```
interface Strategy{
    fun action()
}

class StrategyA : Strategy{
    override fun action() {
        println("StrategyA")
    }
}

class StrategyB : Strategy{
    override fun action() {
        println("StrategyB")
    }
}

class Context(var strategy: Strategy){

    fun preform(){
        strategy.action()
    }
}
```
使用策略的代码如下：
```
val context1 = Context(StrategyA())
val context2 = Context(StrategyB())
val context3 = Context(object : Strategy{
    override fun action() {
        println("匿名内部类--StrategyC")
    }

})
context1.preform()
```
这些代码如同我们两点一线的工作一般毫无新意，Kotlin 的 lambda 表达式则激发了我们内心的一点涟漪：
```
class ContextKotlin{

    fun perform(strategy: ()->Unit){
        strategy()
    }
}

@Test
fun testAdavance(){
    val context = ContextKotlin()
    context.perform {
        println("StrategyA")
    }
    val strategyB = { println("strategyB")}
    context.perform(strategyB)
}
```
『用函数对象表示策略』，Kotlin 诠释得如此淋漓尽致。


>第22条：优先考虑静态成员类

在 Java 中，我们经常要把一个类定义在另外一个类的内部，该类被称之为内部类。内部类有四种：静态成员类、非静态成员类、匿名类和局部类。

该条款建议优先考虑静态成员类，原因在于静态成员类相比非静态成员类而言，不会持有外部类的引用，会带来几个好处：

1. 无需实例外部类就可以使用
2. 当外部类可以被垃圾回收时，不会因为内部类的持有而导致内存泄露。

Kotlin 在语法层面直接对该条款进行支持，静态成员类在 Kotlin 中称为『嵌套类』，默认的内部类便是嵌套类，比如：
```JAVA
class Outer {
    
    class Inner { // 默认便是静态成员类，等价于public static final class Inner

    }
}
```
这种『默认的规约』可以减少不必要的非静态成员类，当然如果经过深思熟虑，一定要使用非静态成员类，可以通过 `inner` 关键字来实现：
```
class Outer{

    class Inner{ // 静态成员类，等价于public final class Outer

    }

    inner class OtherInner{ // 非静态成员类
        
        fun action(){
            // 调用外部类实例
            this@Outer.toString()
        }
    }
}
```

>第36条：坚持使用 Override 注解

回顾上文提到的具备计数能力的 `HashSet`，采用继承的方式时，需要对 add 方法进行重写：
```
class CountingSet: HashSet<Any>() {

    var count = 0

    //1.正确的重写
    /*
    override fun add(element: Any): Boolean {
        count++
        return super.add(element)
    }
    */
    
    //2.错误的重写
    fun add(element: Int): Boolean {
        count++
        return super.add(element)
    }

}
```

看上文的第2个 add 方法，实际是重载而非重写，与我们的本意背道而驰，如果对该方法加上 `override 注解` ，编译器将提示我们问题所在，从而避免不必要的程序 bug。

**Kotlin 同样是这条准则的兢兢业业的践行者，因为在 Kotlin 中重写方法，必须必须必须强制加上 `override`。**


### 总结
Kotlin 与 《Effective Java》相映成辉，显得美不胜收。对照《Effective Java》，我们能更好地理解 Kotlin 的诸多语法的设计初衷。

长江后浪推前浪，前浪死在沙滩上，也许这是Kotlin的历史使命。

### 参考文章
* 《Effective Java 中文版第2版》
*  《Kotlin 实战》

更多文章，请关注[我的博客](https://www.jianshu.com/u/9fa7fc2f3733)