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
3. 所有的 “-” 优先级高于 “*”
4. 所有的 “*” 优先级高于 “+”

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
                          @SerializedName("favorite_id") val favoriteId: String="",
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
