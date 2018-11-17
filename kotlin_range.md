---
title: Kotlin的Range
author: 张波(261512)
date: 2018-10-30 20:00:00
categories:

- 基础知识


tags:
- Kotlin Range

---
# Kotlin的for循环
kotlin的for循环和java的略微有些不同，下面列出常用的两类for循环方式。  
1. 遍历1-100
    - 正序
        ```
        for (index in 1..100){
            print(index) //输出1到100
        }
        ```
    - 倒序
        ```
        for (index in 100 downTo 1){
            print(index)
        }
        ```
    - 添加步长
        ```
        for (index in 1..100 step 2){
            print(index) //输出1..3..5......
        }
        ```
    - 不包含末尾元素
        ```
        for (index in 1 until 100){
            println(index) //输出1到99
        }
        ```
<!-- more -->
2. 遍历数组/列表
    - 取出元素
        ```
        val array = arrayOf("a", "b", "c")
        for (e in array){
            println("元素=$e")
        }
        ```
    - 取出元素和下标
        ```
        val array = arrayOf("a", "b", "c")
        for ((index,e) in array.withIndex()){
            println("下标=$index----元素=$e")
        }
        ```
    - 只取出下标
        ```
        val array = arrayOf("a", "b", "c")
        for (index in array.indices){
            println("index=$index")//输出0，1，2
        }
        ```
```in```和```..```都是Kotlin的操作符。
在for循环中，```in```操作符会调用对应获取迭代器、```next()```和```hasNext()```。而```..```操作符返回一个Range，即集合，这个概念和Java中不太一样。

# 什么是Range
- Range是Kotlin相对Java新增的一种表达式，它表示的是值的范围，类似于数学中的区间。
- Range中实现了迭代的功能。
- Range的表达式是像这样子的：```1..20```，其中```..```是运算符，它表示一个闭区间[1, 20]。而右开区间用```until```表示：```1 until 20```，即[1, 20)。
- Range表达式一般是和```in```操作符一起使用，在for循环中```in```操作符调用获取迭代器、```next()```和```hasNext()```进行迭代操作；不再for循环时，```in```调用的是```contains()```函数，表示元素是否包含在该集合内，例如：
    ```
    if (2 in 1..20){ //相当于判断 2 >= 1 && 2 <= 20
        ...
    }
    ```
- 对于一些整形的range（```IntRange```、```LongRange```、```CharRange```）是可以进行迭代的，它们可以和for循环一起使用，例如：
    ```
    for (i in 1..4) print(i) // 输出 "1234"
    for (i in 4..1) print(i) // 因为"4..1"这个区间为空，所以什么都没有输出
    ```
    Kotlin 1.1以后新增了```Double```和```Float```的range，但是它们只能进行```in```和```!in```操作，不能对它们进行迭代。
- 使用```downTo()```函数可以对range进行倒序迭代，例如
    ```
    for (i in 4 downTo 1) print(i) // 输出 "4321"
    ```
- 使用```step()```函数，可以修改每次迭代增加的值，例如：
    ```
    for (i in 1..4 step 2) print(i)  // 输出 "13"
    for (i in 4 downTo 1 step 2) print(i) // 输出 "42"
    ```
    
# How it works
Kotlin中的for循环和Java中的foreach循环原理是类似，在编译的时候编译器会自动将对for这个关键字的使用转化为对目标的迭代器的使用。   
所以Kotlin中的range、```Array```、```List```和```Map```都是具有返回迭代器的```iterator```方法。   
我们知道```1..20```这个表达式是Int中实现了```rangeTO()```操作符，它等价于```1.rangTo(20)```，返回一个```IntRange(1, 20)```，Kotlin中的源码如下
```
class Int {
    //...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    //...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    //...
}
```
## 原理
下面将以IntRange为例，简单分析Range的实现和原理。

```IntRange```实现了```ClosedRange<T>```接口，该接口有两个作用

1. 定义了```contains```方法，即```in```操作符，用来判断元素是否在Range中，所以Range中的元素必须实现```Comparable<T>```，才能进行比较；

2. 定义了Range的最大值和最小值，即定义了区间；    

```ClosedRange<T>```源码如下：
```
public interface ClosedRange<T: Comparable<T>> {
    /**
     * The minimum value in the range.
     */
    public val start: T

    /**
     * The maximum value in the range (inclusive).
     */
    public val endInclusive: T

    /**
     * Checks whether the specified [value] belongs to the range.
     */
    public operator fun contains(value: T): Boolean = value >= start && value <= endInclusive

    /**
     * Checks whether the range is empty.
     */
    public fun isEmpty(): Boolean = start > endInclusive
}
```

```IntRange```的父类```IntProgression```实现了迭代的功能和步长，
1. ```IntProgression```实现```Iterable<T>```接口，该接口中定义了获取迭代器的操作符```in```。在for循环中迭代时，```in```操作符调用```iterator```返回```IntProgressionIterator```。
2. 在迭代器```IntProgressionIterator```的```next```函数中，根据步长返回下一个值，从而实现具有步长的迭代。
关键代码如下：
    ```
    /**
     * A progression of values of type `Int`.
     */
    public open class IntProgression
        internal constructor
        (
                start: Int,
                endInclusive: Int,
                step: Int
        ) : Iterable<Int> {
        init {
            if (step == 0) throw kotlin.IllegalArgumentException("Step must be non-zero")
        }
        ...
        
        /**
          * The step of the progression.
         */
        public val step: Int = step
        
        override fun iterator(): IntIterator = IntProgressionIterator(first, last, step)
            
        ...
    }
    ```
    ```
    /**
     * An iterator over a progression of values of type `Int`.
     * @property step the number by which the value is incremented on each step.
     */
    internal class IntProgressionIterator(first: Int, last: Int, val step: Int) : IntIterator() {
        private val finalElement = last
        private var hasNext: Boolean = if (step > 0) first <= last else first >= last
        private var next = if (hasNext) first else finalElement
    
        override fun hasNext(): Boolean = hasNext
    
        override fun nextInt(): Int {
            val value = next
            if (value == finalElement) {
                if (!hasNext) throw kotlin.NoSuchElementException()
                hasNext = false
            }
            else {
                next += step
            }
            return value
        }
    }
    ```

# 自定义range
以日期为例自定义一个range，进行迭代，效果如下：
```
val myDate1 = MyDate(2017, 1, 10)
val myDate2 = MyDate(2018, 1, 20)
for (i in myDate1..myDate2){
    print("date: $i") //输出2017-1-10到2018-1-20
}
```
所以要实现自定义的Range，需要Range中的元素实现```Comparable<T>```接口，然后创建一个元素的Range类，实现```ClosedRange<T>```，同时实现```Iterable<T>```接口返回一个迭代器。步骤如下：
1. 创建一个实现了```Comparable<T>```接口的类，这个类就是区间里的元素，相当于前面介绍的Int。重写```compareTo```操作符函数，重写该方法之后，就可以使用```>```、```<```运算符对该类进行运算。这里我们以一个MyDate的日期类为例：
    ```
    data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int) : Comparable<MyDate> {
        override operator fun compareTo(date: MyDate): Int {
            if (year != date.year) {
                return year - date.year
            } else if (month != date.month) {
                return month - date.month
            } else {
                return dayOfMonth - date.dayOfMonth
            }
        }
    }
    ```
    ```
    println(MyDate(2016, 11, 11) > MyDate(2017, 10, 10)) // 输出 false
    ```
2. 创建一个range类，实现```ClosedRange<T>```接口，因为使用Mydate进行比较，这里的T需要传入MyDate。给Mydate添加```rangeTo```操作符函数。
    ```
    class DateRange(override val endInclusive: MyDate, override val start: MyDate) :ClosedRange<MyDate>{
        // ...
    }
    ```
    ```
    operator fun MyDate.rangeTo(other:MyDate) = DateRange(this, other)
    ```
    这里通过扩展函数的方式，为```MyDate```实现```rangeTo```操作符函数。返回一个```DateRange```。调用如下：
    ```
    val first = MyDate(2016, 11, 11)
    val second = MyDate(2017, 10, 10)
    val other = MyDate(2017, 1, 1)
    
    println(other in first..second) // 输出 true
    ```
3. 让```DateRange```实现```Iterable<MyDate>```的接口，重写接口中的```iterator()```方法，返回一个```Iterator<MyDate>```对象，这里我们返回一个实现```Iterator<T>```接口的```DateInterator```对象，该对象真正实现了迭代的```hasNext()```和```next()```方法，实现迭代功能：
    ```
    class DateRange( override val start: MyDate, override val endInclusive: MyDate) : Iterable<MyDate>, ClosedRange<MyDate> {
        override fun iterator(): Iterator<MyDate> = DateIterator(start, endInclusive)
    }
    ```
    ```
    class DateIterator(first: MyDate, val last: MyDate) : Iterator<MyDate> {
        var hasNext = first <= last
        var next = if (hasNext) first else last
        override fun hasNext(): Boolean = hasNext
        override fun next(): MyDate {
            val result = next
            next = next.addOneDay()
            hasNext = next <= last
            return result
        }
    }
    ```
    ```
    fun MyDate.addOneDay():MyDate{
        val c = Calendar.getInstance()
        c.set(this.year, this.month, this.dayOfMonth)
        c.add(Calendar.DAY_OF_MONTH, 1)
        return MyDate(c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DAY_OF_MONTH))
    }
    ```


    
    
