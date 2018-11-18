---
title: rxKotlin2+retrofit2+okhttp3
author: conghaonet
date: 2018-11-17 12:00:00
categories:

- Kotlin

tags:

- Kotlin
- rxkotlin2
- retrofit2
- okhttp3
- rxjava2

---

- 正文内容，使用 `markdown` 进行书写。
- 本地文件，使用 `[template.txt](raws/template.txt)` 进行引用，如

[template.txt](raws/template.txt)

- 本地图片，使用 `![](images/template.png)` 进行引用，如

![](images/template.jpg)

文章 markdown 文件 **分级规范** 如下：

1. 根据内容使用 “#，##，###” 依次标识高级别分级
2. 次级别使用 “-，*，+” 三种符合区分

# rxKotlin2 + retrofit2 + okhttp3 实战
　　最近刚刚把服务于北京高精尖项目的network module用kotlin重构完成，代码量**从3100多行一下降到了1500多行**，代码得到大幅精简的同时，还能有效降低bug率。  
　　当然kotlin的优点不仅仅能帮我们减少代码量这么简单，Kotlin对比Java还有很多优势，如：空安全、扩展函数、数据类、lambda表达式（比Java8更好）、代理模式、 内联函数，关于Kotlin语言的特性一时半会也说不完。  
　　下面将主要围绕本次network module重构展开对kotlin讲解。

## 主要依赖环境
- org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.2.21
- io.reactivex.rxjava2:rxjava:2.1.2
- io.reactivex.rxjava2:rxkotlin:2.1.0
- io.reactivex.rxjava2:rxandroid:2.0.1
- com.squareup.okhttp3:okhttp:3.6.0
- com.squareup.retrofit2:adapter-rxjava2:2.2.0
- com.squareup.retrofit2:retrofit:2.2.0
- com.squareup.retrofit2:converter-gson:2.2.0

　　由于rxkotlin:2.1.0依赖的rxjava版本是2.1.0，但在白名单中只有io.reactivex.rxjava2:rxjava:2.1.2，所以还需要额外定义rxjava的依赖，否则打包时会提示与白名单版本不一致。  
　　就在我写这篇博客前不久，kotlin刚刚发布了v1.3.0，最大的变化就是在1.3中协程已经稳定了，不再是体验版。我正在写这边博客时，kotlin又已经发布了v1.3.10，世界变化真快 : (  
　　截至本文发布时，以上版本均在白名单中有定义，同学们可以放心使用。

## 定义数据类
- FavoriteStatus.kt：
```kotlin
@Parcelize
data class FavoriteStatus(@SerializedName("isIs_favorite") var isFavorite: Boolean = false,
                          @SerializedName("favorite_id") var favoriteId: String="",
                          @SerializedName("favorite_num")  var favoriteNum: Int = 0,
                          @SerializedName("favorite_desc") var favoriteDesc: String?) : Parcelable
```
- FavoriteStatus.java：
```java
public class FavoriteStatus implements Serializable {

    @SerializedName("is_favorite")
    private boolean isFavorite;

    @SerializedName("favorite_id")
    private String favoriteId;

    @SerializedName("favoriteNum")
    private int favoriteNum;

    @SerializedName("favorite_desc")
    private int favoriteDesc;


    public boolean isFavorite() {
        return isFavorite;
    }

    public void setFavorite(boolean favorite) {
        isFavorite = favorite;
    }

    public String getFavoriteId() {
        return favoriteId;
    }

    public void setFavoriteId(String favoriteId) {
        this.favoriteId = favoriteId;
    }

    public int getFavoriteNum() {
        return favoriteNum;
    }

    public void setFavoriteNum(int favoriteNum) {
        this.favoriteNum = favoriteNum;
    }

    public int getFavoriteDesc() {
        return favoriteDesc;
    }

    public void setFavoriteDesc(int favoriteDesc) {
        this.favoriteDesc = favoriteDesc;
    }
}
```
　　一个简单数据类就精简了几十行代码，当然java也可以借助第三方框架实现类似kotlin的数据类写法，但毕竟kotlin原生就支持，而且kotlin还帮我们实现了Parcelable，Parcelable比Serializable效率高很多大家都清楚，之所以很多人还在用Serializable，其中一点就是因为Parcelable实现起来更麻烦。

## 定义API接口
- SlpService.kt
```kotlin
interface SlpService {
    @GET("favorites/{favorite_id}")
    fun getFavorite(@Query("favorite_id") favoriteId: String): Flowable<FavoriteStatus>
}
```
API接口定义比较简单，java跟kotlin的区别不大，这里不贴Java代码了。

## 封装okhttp+retrofit
- RequestClient.kt
```kotlin
const val COMPONENT_ID = "com.nd.sdp.component.demo"
/**
 * base_url在sdp编辑器上的对应值：http://demo.debug.api.sdp.nd/v1/
 */
const val BASE_URL = "base_url"
class RequestClient private constructor() {
    companion object {
        private val DEFAULT_CLIENT by lazy {
            OkHttpClient.Builder()
                    .connectTimeout(10, TimeUnit.SECONDS)
                    .readTimeout(30, TimeUnit.SECONDS)
                    .addInterceptor(HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.HEADERS))
                    //自定义okhttp拦截器
                    .addInterceptor(SlpRequestInterceptor())
                    //OkHttpClient.Builder的扩展函数，为版本小于等于Build.VERSION_CODES.KITKAT时提供访问https能力
                    .enableTls12OnKitkat()
                    .build()
        }

        private fun retrofit(baseUrl: String): Retrofit {
            return Retrofit.Builder()
                    .baseUrl(baseUrl)
                    //自定义Converter.Factory
                    .addConverterFactory(SlpGsonConverterFactory.create())
                    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                    .client(DEFAULT_CLIENT)
                    .build()
        }

        /**
         * @param baseUrl 使用自定义的baseUrl进行构建对应的retrofit实例
         */
        @JvmStatic
        fun <T> buildService(baseUrl: String, serviceClass: Class<T>): T {
            return retrofit(baseUrl).create(serviceClass)
        }
        
        /**
         * 使用业务组件的环境配置中预定义的base_url
         */
        @JvmStatic
        fun <T> buildService(serviceClass: Class<T>): T {
            val configManager = AppFactory.instance().configManager
            val configBean = configManager.getServiceBean(COMPONENT_ID)
            val baseUrl = configBean.getProperty(BASE_URL, null)
            return buildService(baseUrl, serviceClass)
        }
    }
}
```
- const 修饰符：  
　　代码中的COMPONENT_ID、BASE_URL都使用了const修饰符，从字面很容易理解const 指的是常量，那它跟我们在java中定义的 public static final 有何区别呢？  
　　被const修饰的是**编译时常量**，这种类型的常量的值早在编译期间就已经确定，相当于这个常量值被固化到了App安装包里面。无论App在哪部手机上安装、在何时运行，编译时常量的值都是统一且唯一的，不会随环境的变化产生任何变化。而java中定义的是**运行时常量**，这种类型的常量其实不是严格意义上的常量，更确切地说，应该是一个仅能赋值一次的只读属性（这里不对编译时常量做更多讨论）。  
　　**编译时常量才是真正意义上的常量。**  
　　*_注意：const只能修饰 val，不能修饰 var。_
- companion object(伴生对象)
　　Kotlin取消了关键字static，也就无法直接声明静态成员。为了弥补这方面的功能缺陷，Kotlin引入了伴生对象的概念，简单说companion object {... ...}代码中的所有成员都可以在java中已static方式访问。本例中RequestClient还有一种更为简便的定义方式：**对象声明（Object Declaration）**
  ```kotlin
  object RequestClient {

      ... ...

      @JvmStatic
      fun <T> buildService(baseUrl: String, serviceClass: Class<T>): T {
          return retrofit(baseUrl).create(serviceClass)
      }

      @JvmStatic
      fun <T> buildService(serviceClass: Class<T>): T {
          val configManager = AppFactory.instance().configManager
          val configBean = configManager.getServiceBean(COMPONENT_ID)
          val baseUrl = configBean.getProperty(BASE_URL, null)
          return buildService(baseUrl, serviceClass)
      }
  }

  ```
  
# Flowable的扩展函数
NetworkExtFun.kt
```kotlin
@JvmOverloads
fun<T> Flowable<T>.schedule(subscribeOn : Scheduler = Schedulers.io(),
                            observeOn : Scheduler = AndroidSchedulers.mainThread()): Flowable<T> {
    return subscribeOn(subscribeOn).observeOn(observeOn)
}
```
Kotlin提供了一种方法——可以在既不需要继承父类，也不需要使用类似装饰器设计模式的情况下，对类进行扩展。简直是黑科技! Flowable<T>.schedule()就是我给Flowable定义的扩展函数，后面会讲到如何使用。

- 默认参数  
    我们给subscribeOn、observeOn都分别定义了默认的参数值，如果我们在调用扩展函数schedule()时，刚好是要在io线程执行并在主线程观察，那么在调用schedule()时就可以不需要传递任何参数了。如果默认值跟你的实际调用场景不一致也没关系，你完全可以按照你的需要去手工设置其中的某个参数的赋值（后面会讲到）。
    
- @JvmOverloads 注解  
    由于我们设置了参数默认值，为了兼容java，我们给函数加上了@JvmOverloads注解，以下是Java在调用schedule()时的代码片段：
    ![](https://upload-images.jianshu.io/upload_images/15007862-2e09823475317fec.jpg)
    看，编译器直接为我们新增了两个我们没有定义的函数，其中的$receiver就是我们代码片段中定义的flowable。如果我们在kotlin中定义了默认参数，且为了兼容java，基本上都要用到JvmOverloads注解，尤其是我们定义了带默认参数的构造方法时。（这里就不再对JvmOverloads展开讨论了）
