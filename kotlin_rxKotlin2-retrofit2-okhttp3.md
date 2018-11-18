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
最近刚刚把服务于北京高精尖项目的network module用kotlin重构完成，代码量<b>从3100多行一下降到了1500多行</b>，代码得到大幅精简的同时，还能有效降低bug率。

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

## 定义数据类
- FavoriteStatus.kt：
```
@Parcelize
data class FavoriteStatus(@SerializedName("isIs_favorite") var isFavorite: Boolean = false,
                          @SerializedName("favorite_id") var favoriteId: String="",
                          @SerializedName("favorite_num")  var favoriteNum: Int = 0,
                          @SerializedName("favorite_desc") var favoriteDesc: String?) : Parcelable
```
- FavoriteStatus.java：
```
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
```
interface SlpService {
    @GET("favorites/{favorite_id}")
    fun getFavorite(@Query("favorite_id") favoriteId: String): Flowable<FavoriteStatus>
}
```
API接口定义比较简单，java跟kotlin的区别不大，这里不贴Java代码了。

## 封装okhttp+retrofit
- RequestClient.kt
```
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
从字面很容易理解，const指的是常量，那它跟我们在java中定义的public static final有何区别呢？const修饰的COMPONENT_ID是<b>编译时常量</>，这种类型的 常量 的 值 早在 编译 期间 就 已经 确定， 相当于 这个 常 量值 被 固化 到了 App 安装 包 里面。 无论 App 在哪 部 手机 上 安装、 在 何时 运行， 编译 时 常量 的 值 都是 统一 且 唯一 的， 不会 随 环境 的 变化 产生 任何 变化。
