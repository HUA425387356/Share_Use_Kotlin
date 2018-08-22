# Use Kotlin
### Google And Kotlin
Google力推的开发语言，从最新的Android Studio上就有很大体现了。
- 在创建新项目的时候会看到默认勾选了 `Include Kotlin support`
![创建新项目](https://github.com/HUA425387356/Share_Use_Kotlin/blob/master/image/uk_create_new_project.png)
- 点击右键就能创建Kotlin文件    
![创建kt文件](https://github.com/HUA425387356/Share_Use_Kotlin/blob/master/image/uk_create_new_kt_file.png)
- copy Java代码进入Kotlin文件时，IDE会提示自动转为Kotlin语法
![复制java代码到kt文件](https://github.com/HUA425387356/Share_Use_Kotlin/blob/master/image/uk_copy_java_code.png)
```Java
public class AppInfoResponse {

    private String code;
    private ArrayList<AppInfoBean> data;
    private String desc;

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public ArrayList<AppInfoBean> getData() {
        return data;
    }

    public void setData(ArrayList<AppInfoBean> data) {
        this.data = data;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String toString() {
        return "\ncode = " + this.code + "\n"
                + "desc = " + this.desc + "\n"
                + "data = " + (ListUtils.isEmpty(data) ?"list is null" : this.data.toString()) + "\n";
    }
}
```
```Kotlin
class AppInfoResponseKt {

    var code: String? = null
    var data: ArrayList<AppInfoBean>? = null
    var desc: String? = null

    override fun toString(): String {
        return ("\ncode = " + this.code + "\n"
                + "desc = " + this.desc + "\n"
                + "data = " + (if (ListUtils.isEmpty(data)) "list is null" else this.data!!.toString()) + "\n")
    }
}
```
目前只发现了这3个特性，其他的待发现。
         
------- 
### How to Use Kotlin
相信很多同学都想试试Kotlin，但是`不知从何下手`或者`怕用了出问题赶不上项目进度`等。   
其实不用怕，用就是了。   
要认识要一点，我们的项目中`Kotlin和Java的混合`将会是常态，而混合的兼容IDE已经帮我们做好了，所以`边用边学`也是常态。   
OK，那么我们来用Kotlin。

-------
### Data of Using Kotlin
在使用之前我们可以大概过一遍基本语法，也可以直接上手
* 优势及基础语法    
[Kotlin优势](https://developer.android.google.cn/kotlin/)   
[Kotlin语法风格](https://android.github.io/kotlin-guides/style.html)    
[我用的中文教程网（菜鸟教程）](http://www.runoob.com/kotlin/kotlin-tutorial.html)    
* 官网相关   
[Kotlin官网](http://kotlinlang.org/)   
[Kotlin官网for Android](http://kotlinlang.org/docs/reference/android-overview.html)   

---------
### to Use Kotlin
语法看了又忘？还是得用才行。单独为学Kotlin去写一个demo啥的？   
不需要，拿我们自己的项目来改就行了，开个分支，挑一个`最简单、最稳定的模块`转成Kotlin，然后慢慢覆盖全模块。   
除非你想写博客，不然单独写demo的意义不大，代码量不大覆盖的知识面自然小，不具实战意义；
而用公司项目最大的好处是，自己学到知识的同时也更新公司项目的陈旧代码，相得益彰。   
* 配置   
[Google开发_添加Kotlin代码](https://developer.android.google.cn/studio/projects/add-kotlin)   
[Getting started with Android and Kotlin（Kotlin官网）](http://kotlinlang.org/docs/tutorials/kotlin-android.html)   
* 将代码一行行地转为Kotlin，语法得落地没有捷径
* 如果够自信，新模块可以直接用Kotlin

--------
### Some Problems
我在转换的时候首先遇到的这些问题：
* 构造函数怎么有那么多种
* 单例模式怎么写
* 匿名内部类怎么写
* equals和==、===的区别
* Java代码是转到Kotlin的实现原理
--------
### Some Tips
我主要是用下面两个方法解决的：
* Kotlin官方文档中搜索关键字，如`constructor`、`singleton`、`abstract`；
如果看不懂的，会找一下[“中文网”](http://www.runoob.com/kotlin/kotlin-tutorial.html)   
* 复制Java代码到Kotlin文件中，让IDE帮忙转化，然后再一行行看，一行行改
--------
### Some Notes
* Kotlin Constructor
```Kotlin
//primary constructor可以写在类名后面
class Person constructor(firstName: String) { ... }

//primary constructor如果没有注解或可见的其他修改，可以省略constructor关键字
class Person(firstName: String) { ... }

//primary constructor不含其他初始化代码，这些代码可以放在initializer blocks
class InitOrderDemo(name: String) {
    //also关键字：调用某对象的also函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为该对象自己。
    //::双冒号操作符 表示把一个方法当做一个参数，传递到另一个方法中进行使用，通俗的来讲就是引用一个方法
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints ${name}")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}

//primary constructor的参数可以声明为var或者val
class Person(val firstName: String, val lastName: String, var age: Int) { ... }

//primary constructor如果要加注解或者可见的指示符，constructor关键字就不能省
//@Inject 是Dagger的注解
class Customer public @Inject constructor(name: String) { ... }

//Secondary Constructors，跟Java的构造函数一样了
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}

//当Secondary Constructors需要调用primary constructor的时候，可以这样写
//这里this(name)先走
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}

//primary constructor和initializer blocks都会比secondary constructor先执行
//primary constructor比initializer blocks先执行【待验证】
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
//未完
```
* Kotlin Singleton与“匿名内部类”
```Kotlin
//单例需要使用object关键字，object的初始化是线程安全的
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }
}

//匿名内部类
fun countClicks(window: JComponent) {
	//kotlin不需要像Java一样定义final常量
	var clickCount = 0
	var enterCount = 0

	window.addMouseListener(object : MouseAdapter() {
		override fun mouseClicked(e: MouseEvent) {                         
			clickCount++
		}

		verride fun mouseEntered(e: MouseEvent) {
			enterCount++
		}
	})           
}

//关于object关键字的中文博客
https://blog.csdn.net/qq_32115439/article/details/73717858
```
* equals和==、===的区别
```Kotlin
equals
1.equals方法不能作用于基本数据类型的变量
2.如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；
3.诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。

==
1.如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等
2.如果作用于引用类型的变量，则比较的是所指向的对象的地址

===
1.对于基本数据类型，如果类型不同，其结果就是不等。如果同类型相比，与“==”一致，直接比较其存储的 “值”是否相等；
2.对于引用类型，与“==”一致，比较的是所指向的对象的地址
```
* Java代码是转到Kotlin的实现原理   
> 这块还在研究，以后可以讲讲。
-------- 
### Last Word
我这篇主要是介绍自己的学习Kotlin的经验，干货没多少。主要是想和大家说，新东西别光看，可以试着用到项目上。
