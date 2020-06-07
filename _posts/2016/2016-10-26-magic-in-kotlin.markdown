---
author: demon
comments: true
date: 2016-10-26 22:05:00
layout: post
slug: Kotlin Magic Java Swift
title: Kotlin的黑魔法
categories:
- iOS
- android
- kotlin
tags:
- kotlin
- android
- swift
- java
---

本人目前主要做 iOS 相关的开发工作，但在做 iOS 之前曾经用 Java 写过一些 Android 上的应用，所以基本上算是个半吊子的 Android 开发工程师。在经历 Swift 的洗礼之后，看到 Kotlin 就感觉看到了久违的朋友一般。在这里我会结合自己的一些感受，以及之前在线下开发者分享会的时候整理的资料，说一下自己的一些心得体会。希望可以给想了解这门语言的同学带来一些帮助和新的启发。

这篇文章不会对 Kotlin 的语法细节进行讨论，我会通过一些代码片段对比Kotlin、Swift、和Java，之后重点会对 Kotlin 中的Delegated properties(属性委托，Kotlin中许多重要的特性都是基于该特性)进行说明，最后再进行简单的总结。

# Java、Kotlin、Swift，都拉出来溜溜

我们先来看一段Java代码

## Java

```java
public class Student { 
	public Student(String name) {
		this.name = name;
	} 
	public Student(String name, int gender) {
		this(name);
		this.gender = gender;
	} 
	private String name;
	private int gender;
	private String email;
	private int age;
	private float bodyHeight;

	@Override
	public String toString() {
		HashMap<String, String> map = new HashMap<>();
		map.put("name", name);
		map.put("email", email);
		map.put("age", String.valueOf(age));
		map.put("bodyHeight", String.valueOf(bodyHeight));
		return map.toString();
	} 
	public float getBodyHeight() {
		return bodyHeight;
	}
	public void setBodyHeight(float bodyHeight) {
		this.bodyHeight = bodyHeight;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	} 
	public int getGender() {
		return gender;
	} 
	public void setGender(int gender) {
		this.gender = gender;
	} 
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public int getAge() {
		return age; 
	}
	public void setAge(int age) {
		this.age = age;
	}
 }

```

上面这段代码非常简单，我们声明了一个`Student`的类，它有两个构造方法，分别是单参和多参，为了控制访问，我们需要对所有的成员变量设置`getter`和`setter`，这种做法在Java代码里应该已经司空见惯了。为了方便调试，我们重写了`toString`方法，如果调用`toString`则会把当前对象的一些基本信息作为字符串返回。

接下来我们可以看一下在`Kotlin`下的实现：

## Kotlin

```kotlin
//kotlin特有的构造方法声明语法
class Student(var name: String) {
	var gender: Int = 0 //I’m property, not a member or field
	var email: String? = null //支持optional类型
	var age: Int = 0
	lateinit var firstName: String 
	val isBoy by lazy { gender == 1 }
	//如果需要声明构造方法，需要使用constructor关键字
	constructor(name: String, gender: Int): this(name){
		 this.email = email
		 this.gender = gender
		 firstName = name
	}
	var bodyHeight = 0f
		set(value) { field = value }
		get() { return field }    

	override fun toString() = mapOf(
		"name" to name, 
		"email" to email,
		"age" to age, 
		"bodyHeight" to bodyHeight).toString()
	}
}
```

不熟悉的同学可能会疑惑，在`Kotlin`下`getter`和`setter`都去了哪里？不用担心，`Kotlin`并没有帮我们把访问限制去掉。只是我们在编码的时候不用在代码显式地写出来(你说这些都可以用快捷键自动生成啊？好吧我承认这位同学你的代码量比我的又多了不少，我认输。。。)。如果需要添加自己的访问控制的时候才需要进行声明。

还有一个需要提到很重要的特性的是，`Kotlin`支持 `懒加载`。也就是说当某些成员被声明为`lazy`的时候，他们只在第一次被访问的时候才进行初始化。设想一下，在Java中我们需要写自己的懒加载逻辑，而在`Kotlin`里在语法中就原生支持了这个现代语言特性。

除了上文中提到的这些，作为一个现代的语言，`Kotlin`强大的类型推导虽然算不上什么新鲜玩意儿，但在使用的时候也是功不可没。比如我们在初始化变量的时候如果直接进行赋值，`var email = ”hello@alipay.com“`，那么`email`在被编译的时候会自动声明为`String`类型。在成功安装`Kotlin`的插件之后，在`Android Studio`下通过`Tool->Kotlin->Show Kotlin Bytecode` ：

![](https://github.com/demonnico/demonnico.github.com/raw/master/_images/b8c54f019fcf86ce426bb11979ba79b5.png)

我们可以轻松看到`Kotlin`其实在编译成`class`的时候和`Java`并无差异，但是由于我们在编码的时候因为使用了`Kotlin`，生产力大大提升。

最后出场的是Swift

## Swift

```swift
class Student {
	var name: String
	var email = ""
	var age = 0
	var gender = 0
	lazy var isBoy: Bool = { return self.gender==1 }()
	var bodyHeight: Float{
		set {
			_bodyHeight = newValue
		}
		get {
			return _bodyHeight
		}
	}
	init(name: String) {
		self.name = name
	}
	convenience init(name: String, gender: Int) {
		self.init(name: name)
		self.gender = gender
	}
	
	var description: String {
		return name + email + String(age) + String(gender) + String(bodyHeight)
	}
}
```

从语法特征来看，`Swift`和`Kotlin`简直神似。但如果正真使用过这两种语言编码之后，我发现`Kotlin`在类型推导上做得更好。在声明一些闭包类型的时候，我就遇到过在`Kotlin`语义下是可以被识别的而在`Swift`下无法编译通过的例子。这也是我在用这两种语言做业务开发过程中遇到过最为头疼的事。

接下就让我们一起来看看`Kotlin`里有哪些黑魔法吧。

# 黑魔法篇

## Function extension
在很多地方，我们都可以看到`Kotlin`原生支持了类似`Objective-c`下的`Category`特性，在`Kotlin`下我们把这个特性叫做`Function extension`。也就是说我们可以给任何已存在的类添加方法，包括在Java中的一些基本类型，而不是只能通过继承他们在子类中声明我们的方法。这个特性对我来说简直是`killing feature`一般的存在。这里我举一个具体的例子来说明`Function extension`到底有多重要。

在`Java`的世界里，我们经常会写各种Util工具类，声明很多静态方法用于类型或者字符串转换。例如我们想对一个基本类型(比如`Float`类型)转换为字符串，并进行一定的控制，比如下图中的这几种情况：

![Screen_Shot_2016_10_09_at_2_35_44_PM](https://github.com/demonnico/demonnico.github.com/raw/master/_images/53b6f3f65ed12f9705afb80b4157cfe3.png)

![Screen_Shot_2016_10_09_at_2_35_35_PM](https://github.com/demonnico/demonnico.github.com/raw/master/_images/98415eb7a6aaa2c90db8ad3e861d1768.png)

![Screen_Shot_2016_10_09_at_2_35_30_PM](https://github.com/demonnico/demonnico.github.com/raw/master/_images/ee34c5c28c2311a5b77db9725b79b6c8.png)

在`Kotlin`的世界里，使用`Function extension`来实现的话，我们可以这么做：

```kotlin
fun Double.Yuan(): String{
	val formatter = DecimalFormat("0.00")
	formatter.roundingMode = RoundingMode.DOWN
	return "￥"+formatter.format(this)
}
```
这种链式写法在使用的时候非常方便，尤其是需要进行多次类型转换的时候，我们再也不用写类似`AUtil.convert(BUtil.convert(value))`一层层嵌套的代码了。相比之下链式调用也更符合OO语言习惯。

或许有人会问了，那既然可以给已存在的类添加方法，那是否可以添加属性(成员)呢？答案是肯定的。但`Kotlin` 官网的文档里并没有提及，我们在下面的篇幅里会详细介绍`Delegated properties`这个黑魔法，进而引申出`Property extension`的一种实现。

## Delegated properties
[Delegated properties(委托属性)](!https://kotlinlang.org/docs/reference/delegated-properties.html)在`Kotlin`里是一种非常重要的实现机制，许多重要的特性可以基于`Delegated properties`来实现的。除了上文中提及的`Function Extension`之外，像`Lazy`和`lateinit`特性也都可以视作基于`Delegated properties`的一种特性实现，并且我们还可以基于它来做一些属性观察(实时监听property的变化，类似`Objective-C`中的`KVO`)的事情。`Delegated properties`使用起来非常简单，在申明变量的时候，跟上`by`关键字，后面声明的就是我们的代理类。废话不多说，上码

```kotlin
class Example {
	var member: String by PropertyAttachedDelegation()
}
```

这里的`PropertyAttachedDelegation`就是我们声明的委托类。`PropertyAttachedDelegation`的具体实现如下：

```kotlin
class PropertyAttachedDelegation{
	operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
		return ""
	} 
	operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String){
	
	}
}

```
这里简单说明一下，在外部访问上文中`Example`对象的`member`属性时，会触发`PropertyAttachedDelegation `的`getValue`方法，相应地如果在对`member`赋值时，会触发`setValue`方法。其中`thisRef`代表的是被访问对象的`this`引用，`property`参数是`KProperty`类型，会体现当前`PropertyAttachedDelegation`所修饰`member`成员对象的一些运行时特性(类似Java中的反射机制)，例如成员变量名、类型等。最后的`value`则是通过set方法需要传入的值。

那我们在实际开发过程中`Delegated properties`有什么用处呢？

## Property extension
`Kotlin`官网中并没有提到`Property extension`，这只是我基于`Delegated properties`和`Delegated properties`特性的一个延伸。简单说，我们可以在`Kotlin`里轻松实现给已有的类添加新的成员变量这个特性(在`Objective-C`里我们可以基于`Category`和运行时库的`objc_associate`特性实现)。Takling is cheap, show me the code，上码：

```kotlin
data class User(val name: String? = null, val age: Int = 0, val gender:Int =0)
```
我们新建了一个叫做`User`的[`data class`](!https://kotlinlang.org/docs/reference/data-classes.html)，现在只有`name`、`age`、`gender`三个成员属性。如果我想在不修改`User`类的情况下，添加一个叫做`email`的成员，可以这么做：

```kotlin
var User.email: String by PropertyAttachedDelegation()
```
接下去我们对上文中提到的`PropertyAttachedDelegation`，进行进一步完善：

```kotlin
class PropertyAttachedDelegation<T>{
	//variablesMap是一个字典类型，类似HashMap
	operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return thisRef?.variablesMap?.get(property.toString()) as T
	}  
	operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String){
		thisRef?.variablesMap?.set(property.toString(), value)
	}
}
```
我们将PropertyAttachedDelegation声明为一个泛型类，这样保证了`getValue`方法的返回值与委托类修饰的变量类型保持一致。在`setValue`和`getValue`里通过`variablesMap`对变量进行动态读取与存储，`property`保证读取与存储键值的一致性。说到这里，`variablesMap `又是从哪儿冒出来的呢？接着看下面的代码：

```kotlin
private val Any.variablesMap by lazy {
	val mutableMap: MutableMap <String, Any> = mutableMapOf()
	mutableMap//在Lazy表达式里，会将最后一个表达式的值隐式地作为返回值，不用显式声明retrun
}
```

在`Kotlin`里，所有类型都集成自[`Any`](!https://kotlinlang.org/docs/reference/classes.html)(`Any`不是`java.lang.Object`，当`Kotlin`里引入`Java`的类型时，所有的`java.lang.Object`类型都会[动态地转换为`Any`](!https://kotlinlang.org/docs/reference/java-interop.html#object-methods))，我们给`Any`动态声明一个`variablesMap`的`lazy`属性，这样能保证`variablesMap`在所有类型里都能被访问到，`private`保证了数据安全，无法被外部成员访问或者修改。因为是`lazy`的，因此`variablesMap`的`MutableMap`仅在第一次被访问时才创建，避免了内存上的开销。做完这些事情，我们之后就可以快速地给任何对象添加任何类型的成员属性(property)了。比如我还想给上文中的`User`类型添加一个`tel`字段，可以这样：

```kotlin
var User.tel: String by PropertyAttachedDelegation()
```
是不是很轻松？一劳永逸。自我感觉比`Objective-C`下的做法还优雅不少。

# 总结
除了上文中提到的一些实战技巧，附件有我之前在线下分享时做的一个 [Keynote](https://www.icloud.com/keynote/000SDjKQAvs2PkV0W03CDPk5w#Programming_Android_In_Kotlin)，有兴趣的同学也可以下载下来看看，内容比这篇短文也要丰富些。总的来说`Kotlin`非常容易上手，对有`swift`经验的`iOS`开发者来说尤其友好。如果你有过一些`Android`开发经验或者说你正想学习`Android`开发又不想写`Java`的话，在我看来`Kotlin`是一个非常好的选择。

参考文章：

[Kotlin Reference](https://kotlinlang.org/docs/reference/)



