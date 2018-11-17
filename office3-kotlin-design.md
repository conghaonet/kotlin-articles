---
title: Kotlin中的设计模式
author: 王跃杰(250750)
date: 2018-03-01 12:00:00
categories:

- Kotlin
- 设计模式

tags:

- Kotlin
- 设计模式

---

# Kotlin中的设计模式

学习完 Kotlin 语法知识后，直接开始项目开发也许还有难度，先通过用 Kotlin 实现设计模式会是一个比较有效的过渡

## 内容

* [行为型](#行为型)
    * [观察者](#观察者)
	* [策略](#策略)
	* [命令](#命令)
	* [状态](#状态)
	* [责任链](#责任链)
	* [访问者](#访问者)
* [创建型](#创建型)
	* [建造者](#建造者)
	* [工厂模式](#工厂模式)
	* [单例](#单例)
	* [抽象工厂](#抽象工厂)
* [结构型](#结构型)
	* [适配器](#适配器)
	* [装饰器](#装饰器)
	* [门面模式](#门面模式)
	* [代理](#代理)

行为型
==========

> 在软件工程中，行为型设计模式为设计模式的一种类型，用来识别对象之间的常用交流模式并加以实现。如此，可以在交流时增强灵活性。
>
>**Source:** [wikipedia.org](https://en.wikipedia.org/wiki/Behavioral_pattern)

观察者
--------

定义对象间的一种一对多的依赖关系,当一个对象的状态发生改变时, 所有依赖于它的对象都得到通知并被自动更新

#### 范例

```kotlin
interface TextChangedListener {
    fun onTextChanged(newText: String)
}

class PrintingTextChangedListener : TextChangedListener {
    override fun onTextChanged(newText: String) = println("Text is changed to: $newText")
}

class TextView {

    var listener: TextChangedListener? = null

    var text: String by Delegates.observable("") { prop, old, new ->
        listener?.onTextChanged(new)
    }
}
```

#### 使用

```kotlin
val textView = TextView()
textView.listener = PrintingTextChangedListener()
textView.text = "Lorem ipsum"
textView.text = "dolor sit amet"
```

#### 输出

```
Text is changed to: Lorem ipsum
Text is changed to: dolor sit amet
```

#### 分析

关键在于使用 `Delegates.observable()` 对属性监听，通过 `by` 语法巧妙地队被观测对象进行代理，代码十分简洁

策略
-----------

定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化

#### 范例

```kotlin
class Printer(val stringFormatterStrategy: (String) -> String) {
    fun printString(string: String) = println(stringFormatterStrategy.invoke(string))
}

val lowerCaseFormatter: (String) -> String = { it.toLowerCase() }

val upperCaseFormatter = { it: String -> it.toUpperCase() }
```

#### 使用

```kotlin
val lowerCasePrinter = Printer(lowerCaseFormatter)
lowerCasePrinter.printString("LOREM ipsum DOLOR sit amet")

val upperCasePrinter = Printer(upperCaseFormatter)
upperCasePrinter.printString("LOREM ipsum DOLOR sit amet")

val prefixPrinter = Printer({ "Prefix: " + it })
prefixPrinter.printString("LOREM ipsum DOLOR sit amet")
```

#### 输出

```
lorem ipsum dolor sit amet
LOREM IPSUM DOLOR SIT AMET
Prefix: LOREM ipsum DOLOR sit amet
```

#### 分析

代码中用到量 Kotlin 函数文本语法，将策略以函数的方式保存到变量中传递给调用者，灵活地变换策略

命令
-------

将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可取消的操作

#### 范例:

```kotlin
interface OrderCommand {
    fun execute()
}

class OrderAddCommand(val id: Long) : OrderCommand {
    override fun execute() = println("adding order with id: $id")
}

class OrderPayCommand(val id: Long) : OrderCommand {
    override fun execute() = println("paying for order with id: $id")
}

class CommandProcessor {

    private val queue = ArrayList<OrderCommand>()

    fun addToQueue(orderCommand: OrderCommand): CommandProcessor
            = apply { queue.add(orderCommand) }

    fun processCommands(): CommandProcessor = apply {
        queue.forEach { it.execute() }
        queue.clear()
    }
}
```

#### 使用

```kotlin
CommandProcessor()
    .addToQueue(OrderAddCommand(1L))
    .addToQueue(OrderAddCommand(2L))
    .addToQueue(OrderPayCommand(2L))
    .addToQueue(OrderPayCommand(1L))
    .processCommands()
```

#### 输出

```
adding order with id: 1
adding order with id: 2
paying for order with id: 2
paying for order with id: 1
```

状态
------

允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它的类

#### 范例

```kotlin
sealed class AuthorizationState

class Unauthorized : AuthorizationState() // may be an object: object Unauthorized : AuthorizationState()

class Authorized(val userName: String) : AuthorizationState()

class AuthorizationPresenter {

    private var state: AuthorizationState = Unauthorized()

    fun loginUser(userLogin: String) {
        state = Authorized(userLogin)
    }

    fun logoutUser() {
        state = Unauthorized()
    }

    val isAuthorized: Boolean
        get() = when (state) {
            is Authorized -> true
            is Unauthorized -> false
        }

    val userLogin: String
        get() = when (state) {
            is Authorized -> (state as Authorized).userName
            is Unauthorized -> "Unknown"
        }

    override fun toString() = "User '$userLogin' is logged in: $isAuthorized"
}
```

#### 使用

```kotlin
val authorization = AuthorizationPresenter()
authorization.loginUser("admin")
println(authorization)
authorization.logoutUser()
println(authorization)
```

#### 输出

```
User 'admin' is logged in: true
User 'Unknown' is logged in: false
```

#### 分析

使用密封(sealed)类定义了状态类的之类，类似于 Java 中的枚举，但比 Java 的枚举有新的特性。例如，枚举要求一个类只有一个对象，而密封类允许一个类有多个对象，因此可以用来保存其它信息

责任链
-----------------------

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止

#### 范例

```kotlin
interface MessageChain {
    fun addLines(inputHeader: String): String
}

class AuthenticationHeader(val token: String?, var next: MessageChain? = null) : MessageChain {

    override fun addLines(inputHeader: String): String {
        token ?: throw IllegalStateException("Token should be not null")
        return "$inputHeader Authorization: Bearer $token\n".let { next?.addLines(it) ?: it }
    }
}

class ContentTypeHeader(val contentType: String, var next: MessageChain? = null) : MessageChain {

    override fun addLines(inputHeader: String): String
            = "$inputHeader ContentType: $contentType\n".let { next?.addLines(it) ?: it }
}

class BodyPayload(val body: String, var next: MessageChain? = null) : MessageChain {

    override fun addLines(inputHeader: String): String
            = "$inputHeader $body\n".let { next?.addLines(it) ?: it }
}
```

#### 使用

```kotlin
val authenticationHeader = AuthenticationHeader("123456")
val contentTypeHeader = ContentTypeHeader("json")
val messageBody = BodyPayload("{\"username\"=\"dbacinski\"}")

val messageChainWithAuthorization = messageChainWithAuthorization(authenticationHeader, contentTypeHeader, messageBody)
val messageWithAuthentication = messageChainWithAuthorization.addLines("Message with Authentication:\n")
println(messageWithAuthentication)

fun messageChainWithAuthorization(authenticationHeader: AuthenticationHeader, contentTypeHeader: ContentTypeHeader, messageBody: BodyPayload): MessageChain {
    authenticationHeader.next = contentTypeHeader
    contentTypeHeader.next = messageBody
    return authenticationHeader
}
```

#### 输出

```
Message with Authentication:
Authorization: Bearer 123456
ContentType: json
{"username"="dbacinski"}
```

访问者
-------

表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作

#### 范例

```kotlin
interface ReportVisitable {
    fun accept(visitor: ReportVisitor)
}

class FixedPriceContract(val costPerYear: Long) : ReportVisitable {
    override fun accept(visitor: ReportVisitor) = visitor.visit(this)
}

class TimeAndMaterialsContract(val costPerHour: Long, val hours: Long) : ReportVisitable {
    override fun accept(visitor: ReportVisitor) = visitor.visit(this)
}

class SupportContract(val costPerMonth: Long) : ReportVisitable {
    override fun accept(visitor: ReportVisitor) = visitor.visit(this)
}

interface ReportVisitor {
    fun visit(contract: FixedPriceContract)
    fun visit(contract: TimeAndMaterialsContract)
    fun visit(contract: SupportContract)
}

class MonthlyCostReportVisitor(var monthlyCost: Long = 0) : ReportVisitor {
    override fun visit(contract: FixedPriceContract) {
        monthlyCost += contract.costPerYear / 12
    }

    override fun visit(contract: TimeAndMaterialsContract) {
        monthlyCost += contract.costPerHour * contract.hours
    }

    override fun visit(contract: SupportContract) {
        monthlyCost += contract.costPerMonth
    }
}
```

#### 使用

```kotlin
val projectAlpha = FixedPriceContract(costPerYear = 10000)
val projectBeta = SupportContract(costPerMonth = 500)
val projectGamma = TimeAndMaterialsContract(hours = 150, costPerHour = 10)
val projectKappa = TimeAndMaterialsContract(hours = 50, costPerHour = 50)

val projects = arrayOf(projectAlpha, projectBeta, projectGamma, projectKappa)

val monthlyCostReportVisitor = MonthlyCostReportVisitor()
projects.forEach { it.accept(monthlyCostReportVisitor) }
println("Monthly cost: ${monthlyCostReportVisitor.monthlyCost}")
```

#### 输出

```
Monthly cost: 5333
```

创建型
==========

> 在软件工程中，创建型模式是处理对象创建的设计模式，试图根据实际情况使用合适的方式创建对象。基本的对象创建方式可能会导致设计上的问题，或增加设计的复杂度。创建型模式通过以某种方式控制对象的创建来解决这些问题。
>
>**Source:** [wikipedia.org](http://en.wikipedia.org/wiki/Creational_pattern)


建造者
----------

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

#### 范例

```kotlin
// Let's assume that Dialog class is provided by external library.
// We have only access to Dialog public interface which cannot be changed.

class Dialog() {

    fun showTitle() = println("showing title")

    fun setTitle(text: String) = println("setting title text $text")

    fun setTitleColor(color: String) = println("setting title color $color")

    fun showMessage() = println("showing message")

    fun setMessage(text: String) = println("setting message $text")

    fun setMessageColor(color: String) = println("setting message color $color")

    fun showImage(bitmapBytes: ByteArray) = println("showing image with size ${bitmapBytes.size}")

    fun show() = println("showing dialog $this")
}

//Builder:
class DialogBuilder() {
    constructor(init: DialogBuilder.() -> Unit) : this() {
        init()
    }

    private var titleHolder: TextView? = null
    private var messageHolder: TextView? = null
    private var imageHolder: File? = null

    fun title(init: TextView.() -> Unit) {
        titleHolder = TextView().apply { init() }
    }

    fun message(init: TextView.() -> Unit) {
        messageHolder = TextView().apply { init() }
    }

    fun image(init: () -> File) {
        imageHolder = init()
    }

    fun build(): Dialog {
        val dialog = Dialog()

        titleHolder?.apply {
            dialog.setTitle(text)
            dialog.setTitleColor(color)
            dialog.showTitle()
        }

        messageHolder?.apply {
            dialog.setMessage(text)
            dialog.setMessageColor(color)
            dialog.showMessage()
        }

        imageHolder?.apply {
            dialog.showImage(readBytes())
        }

        return dialog
    }

    class TextView {
        var text: String = ""
        var color: String = "#00000"
    }
}
```

#### 使用

```kotlin
//Function that creates dialog builder and builds Dialog
fun dialog(init: DialogBuilder.() -> Unit): Dialog {
    return DialogBuilder(init).build()
}

val dialog: Dialog = dialog {
	title {
    	text = "Dialog Title"
    }
    message {
        text = "Dialog Message"
        color = "#333333"
    }
    image {
        File.createTempFile("image", "jpg")
    }
}

dialog.show()
```

#### 输出

```
setting title text Dialog Title
setting title color #00000
showing title
setting message Dialog Message
setting message color #333333
showing message
showing image with size 0
showing dialog Dialog@5f184fc6
```

工厂模式
-----------------

定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method 使一个类的实例化延迟到其子类

#### 范例

```kotlin
interface Currency {
    val code: String
}

class Euro(override val code: String = "EUR") : Currency
class UnitedStatesDollar(override val code: String = "USD") : Currency

enum class Country {
    UnitedStates, Spain, UK, Greece
}

class CurrencyFactory {
    fun currencyForCountry(country: Country): Currency? {
        when (country) {
            Country.Spain, Country.Greece -> return Euro()
            Country.UnitedStates          -> return UnitedStatesDollar()
            else                          -> return null
        }
    }
}
```

#### 使用

```kotlin
val noCurrencyCode = "No Currency Code Available"

val greeceCode = CurrencyFactory().currencyForCountry(Country.Greece)?.code() ?: noCurrencyCode
println("Greece currency: $greeceCode")

val usCode = CurrencyFactory().currencyForCountry(Country.UnitedStates)?.code() ?: noCurrencyCode
println("US currency: $usCode")

val ukCode = CurrencyFactory().currencyForCountry(Country.UK)?.code() ?: noCurrencyCode
println("UK currency: $ukCode")
```

#### 输出

```
Greece currency: EUR
US currency: USD
UK currency: No Currency Code Available
```

单例
------------

保证一个类仅有一个实例，并提供一个访问它的全局访问点

#### 范例:

```kotlin
object PrinterDriver {
    init {
        println("Initializing with object: $this")
    }

    fun print() = println("Printing with object: $this")
}
```

#### 使用

```kotlin
println("Start")
PrinterDriver.print()
PrinterDriver.print()
```

#### 输出

```
Start
Initializing with object: PrinterDriver@6ff3c5b5
Printing with object: PrinterDriver@6ff3c5b5
Printing with object: PrinterDriver@6ff3c5b5
```

#### 分析

使用 `object` 语法实现的单例代码十分简洁，并且还是延迟加载，仅当第一次调用其属性或方法时才创建对象

抽象工厂
-------------------

提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类

#### 范例

```kotlin
interface Plant

class OrangePlant : Plant

class ApplePlant : Plant

abstract class PlantFactory {
    abstract fun makePlant(): Plant

    companion object {
        inline fun <reified T : Plant> createFactory(): PlantFactory = when (T::class) {
            OrangePlant::class -> OrangeFactory()
            ApplePlant::class  -> AppleFactory()
            else               -> throw IllegalArgumentException()
        }
    }
}

class AppleFactory : PlantFactory() {
    override fun makePlant(): Plant = ApplePlant()
}

class OrangeFactory : PlantFactory() {
    override fun makePlant(): Plant = OrangePlant()
}
```

#### 使用

```kotlin
val plantFactory = PlantFactory.createFactory<OrangePlant>()
val plant = plantFactory.makePlant()
println("Created plant: $plant")
```

#### 输出

```kotlin
Created plant: OrangePlant@4f023edb
```

#### 分析

`PlantFactory.createFactory` 涉及到很多语法知识，包括静态方法、内联函数、具体化泛型等

结构型
==========

> 在软件工程中，结构型设计模式是借由一以贯之的方式来了解元件间的关系，从而简化设计的一种设计模式。
>
>**Source:** [wikipedia.org](http://en.wikipedia.org/wiki/Structural_pattern)

适配器
----------

将一个类的接口转换成另外一个客户希望的接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

#### 范例

```kotlin
interface Temperature {
    var temperature: Double
}

class CelsiusTemperature(override var temperature: Double) : Temperature

class FahrenheitTemperature(var celsiusTemperature: CelsiusTemperature) : Temperature {

    override var temperature: Double
        get() = convertCelsiusToFahrenheit(celsiusTemperature.temperature)
        set(temperatureInF) {
            celsiusTemperature.temperature = convertFahrenheitToCelsius(temperatureInF)
        }

    private fun convertFahrenheitToCelsius(f: Double): Double = (f - 32) * 5 / 9

    private fun convertCelsiusToFahrenheit(c: Double): Double = (c * 9 / 5) + 32
}

```

#### 使用

```kotlin
val celsiusTemperature = CelsiusTemperature(0.0)
val fahrenheitTemperature = FahrenheitTemperature(celsiusTemperature)

celsiusTemperature.temperature = 36.6
println("${celsiusTemperature.temperature} C -> ${fahrenheitTemperature.temperature} F")

fahrenheitTemperature.temperature = 100.0
println("${fahrenheitTemperature.temperature} F -> ${celsiusTemperature.temperature} C")
```

#### 输出

```
36.6 C -> 97.88000000000001 F
100.0 F -> 37.77777777777778 C
```

装饰器
------------

动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活

#### 范例

```kotlin
interface CoffeeMachine {
    fun makeSmallCoffee()
    fun makeLargeCoffee()
}

class NormalCoffeeMachine : CoffeeMachine {
    override fun makeSmallCoffee() = println("Normal: Making small coffee")

    override fun makeLargeCoffee() = println("Normal: Making large coffee")
}

//Decorator:
class EnhancedCoffeeMachine(val coffeeMachine: CoffeeMachine) : CoffeeMachine by coffeeMachine {

    // overriding behaviour
    override fun makeLargeCoffee() {
        println("Enhanced: Making large coffee")
        coffeeMachine.makeLargeCoffee()
    }

    // extended behaviour
    fun makeCoffeeWithMilk() {
        println("Enhanced: Making coffee with milk")
        coffeeMachine.makeSmallCoffee()
        println("Enhanced: Adding milk")
    }
}
```

#### 使用

```kotlin
    val normalMachine = NormalCoffeeMachine()
    val enhancedMachine = EnhancedCoffeeMachine(normalMachine)

    // non-overridden behaviour
    enhancedMachine.makeSmallCoffee()
    // overriding behaviour
    enhancedMachine.makeLargeCoffee()
    // extended behaviour
    enhancedMachine.makeCoffeeWithMilk()
```

#### 输出

```
Normal: Making small coffee

Enhanced: Making large coffee
Normal: Making large coffee

Enhanced: Making coffee with milk
Normal: Making small coffee
Enhanced: Adding milk
```

门面模式
---------

为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用

#### 范例

```kotlin
class ComplexSystemStore(val filePath: String) {

    init {
        println("Reading data from file: $filePath")
    }

    val store = HashMap<String, String>()

    fun store(key: String, payload: String) {
        store.put(key, payload)
    }

    fun read(key: String): String = store[key] ?: ""

    fun commit() = println("Storing cached data: $store to file: $filePath")
}

data class User(val login: String)

//Facade:
class UserRepository {
    val systemPreferences = ComplexSystemStore("/data/default.prefs")

    fun save(user: User) {
        systemPreferences.store("USER_KEY", user.login)
        systemPreferences.commit()
    }

    fun findFirst(): User = User(systemPreferences.read("USER_KEY"))
}
```

#### 使用

```kotlin
val userRepository = UserRepository()
val user = User("dbacinski")
userRepository.save(user)
val resultUser = userRepository.findFirst()
println("Found stored user: $resultUser")
```

#### 输出

```
Reading data from file: /data/default.prefs
Storing cached data: {USER_KEY=dbacinski} to file: /data/default.prefs
Found stored user: User(login=dbacinski)
```

代理
------------------

为其他对象提供一种代理以控制对这个对象的访问

#### 范例

```kotlin
interface File {
    fun read(name: String)
}

class NormalFile : File {
    override fun read(name: String) = println("Reading file: $name")
}

//Proxy:
class SecuredFile : File {
    val normalFile = NormalFile()
    var password: String = ""

    override fun read(name: String) {
        if (password == "secret") {
            println("Password is correct: $password")
            normalFile.read(name)
        } else {
            println("Incorrect password. Access denied!")
        }
    }
}
```

#### 使用

```kotlin
val securedFile = SecuredFile()
securedFile.read("readme.md")

securedFile.password = "secret"
securedFile.read("readme.md")
```

#### 输出

```
Incorrect password. Access denied!
Password is correct: secret
Reading file: readme.md
```

小结
==========

Kotlin 语法更为灵活、特性也比 Java 更丰富，通过实现设计模式代码可发现其最大的优点是代码更为简洁明了。除此之外，例如封闭类实现的状态模式、函数文本实现的策略模式等，让编码添加了更多可能

参考文档
==========

[Design-Patterns-In-Kotlin](https://github.com/dbacinski/Design-Patterns-In-Kotlin)
