---
title: Kotlin开发入门——基础语法
author: 杨扬(949177)
date: 2017-06-19 12:00:00
categories:

- 最新资讯


tags:
- Kotlin
- 安卓支持编程语言
- 编程语言 

---

[toc]

## 基本语法
### 函数
```Kotlin
fun add(x: Int, y: Int) : Int {
    return x + y
}
```
```Kotlin
fun add(x: Int,y: Int) : Int = x + y
```
#### 参数与默认值
```
fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, length).show()
}
```
#### 调用
```
toast(message = "Hello", length = Toast.LENGTH_SHORT)
```
### 变量
```
var s
s = "Example"
```
#### 不可变量
```
val s = "Example" // A String
val i = 23 // An Int
val actionBar = supportActionBar // An ActionBar in an Activity context
```
### 字符串模板
```
fun main(args: Array<String>) {
//sampleStart
    var a = 1
    // 模板中的简单名称：
    val s1 = "a is $a" 

    a = 2
    // 模板中的任意表达式：
    val s2 = "${s1.replace("is", "was")}, but now is $a"
//sampleEnd
    println(s2)
}
```
### 字符串内置表达式
```
//sampleStart
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
//sampleEnd
fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```
### 类型转换
```
fun getStringLength(obj: Any): Int? {
    // `obj` 在 `&&` 右边自动转换成 `String` 类型
    if (obj is String && obj.length > 0) {
      return obj.length
    }

    return null
}
```
### if表达式
```
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```
### when表达式
```
//sampleStart
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
}
//sampleEnd
fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}
```
### Range表达式
```
fun main(args: Array<String>) {
//sampleStart
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
//sampleEnd
}
```
### 集合
```
val items = listOf("apple", "banana", "kiwi")
val items = setOf("apple", "banana", "kiwi")
```
### Lambda表达式
```
fun main(args: Array<String>) {
    val fruits = listOf("banana", "avocado", "apple", "kiwi")
//sampleStart
    fruits
        .filter { it.startsWith("a") }
        .sortedBy { it }
        .map { it.toUpperCase() }
        .forEach { println(it) }
//sampleEnd
}
```
### Pojo
```
data class Customer(val name: String, val email: String)

val customer = Customer("test","test")

```
会为 Customer 类提供以下功能：
 - 所有属性的 getters （对于 var 定义的还有 setters）
 - equals()
 - hashCode()
 - toString()
 - List item
 - copy()



### 懒加载
```
val p: String by lazy {
    // 计算该字符串
}
```
### 单例
```
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ……
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ……
}
```
```
DataProviderManager.registerDataProvider(……)
```
## 空安全
强制检测
```
var a: String = "abc"
a = null // 编译错误
```
```
var b: String? = "abc"
b = null // ok
```
## 扩展函数
```
fun MutableList<Int>.swap(x: Int, y: Int) {
    val temp = this[x] // this 对应 list
    this[x] = this[y]
    this[y] = temp
}

fun String.isLong(): Boolean {
    try {
        java.lang.Long.parseLong(this)
        return true
    } catch(e: Exception) {
        return false
    }
}

fun main(args: Array<String>) {
    val l = mutableListOf(1, 2, 3)
    l.swap(0, 2)
    System.out.println(l)
    val s = "333d"
    val long = s.isLong()
    System.out.println(long)
}
```
