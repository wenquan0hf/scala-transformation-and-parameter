# 转换被方法调用的对象 #
隐式变换也可以转换调用方法的对象，比如但编译器看到X .method，而类型 X 没有定义 method（包括基类)方法，那么编译器就查找作用域内定义的从 X 到其它对象的类型转换，比如 Y，而类型Y定义了 method 方法，编译器就首先使用隐含类型转换把 X 转换成 Y，然后调用 Y 的 method。

下面我们看看这种用法的两个典型用法：
## 支持新的类型 ##
这里我们使用前面例子 Scala开发教程(50): Ordered Trait 中定义的 Rational 类型为例：

```
class Rational (n:Int, d:Int) {
	require(d!=0)
	private val g =gcd (n.abs,d.abs)
	val numer =n/g
	val denom =d/g
	override def toString = numer + "/" +denom
	def +(that:Rational)  =
	  new Rational(
		numer * that.denom + that.numer* denom,
		denom * that.denom
	  )
	def +(i:Int) :Rational =
		new Rational(numer +1*denom,denom)
	def * (that:Rational) =
	  new Rational( numer * that.numer, denom * that.denom)
	def this(n:Int) = this(n,1)
	private def gcd(a:Int,b:Int):Int =
	  if(b==0) a else gcd(b, a % b)
}
```

类 Rational 重载了两个+运算，参数类型分别为 Rational 和 Int。因此你可以把 Rational 和 Rational 相加，也可以把 Rational 和整数相加。

```
scala> val oneHalf = new Rational(1,2)
oneHalf: Rational = 1/2
```

```
scala> oneHalf + oneHalf
res0: Rational = 1/1
```

```
scala> oneHalf + 1
res1: Rational = 3/2
```

但是我们如果使用 1+ oneHalf 会出现什么问题呢？

```
scala> 1 + oneHalf
<console>:10: error: overloaded method value + with alternatives:
  (x: Double)Double <and>
  (x: Float)Float <and>
  (x: Long)Long <and>
  (x: Int)Int <and>
  (x: Char)Int <and>
  (x: Short)Int <and>
  (x: Byte)Int <and>
  (x: String)String
 cannot be applied to (Rational)
              1 + oneHalf
                ^
```

整数和其相关类型都没定义和 Rational 类型相加的操作，因此编译器报错，此时编译器在1能够转换成 Rational 类型才可以编译过，因此我们可以定义一个从整数到 Rational 的隐含类型变换：

```
scala> implicit def int2Rational(x:Int) = new Rational(x)
int2Rational: (x: Int)Rational
```

现在再执行 1+oneHalf:

```
scala> 1 + oneHalf
res3: Rational = 3/2
```

在定义了 int2Rational 之后，编译器看到 1+oneHalf，发现 1 没有定义和 Rational 相加的操作，通常需要报错，编译器在报错之前查找当前作用域从 Int 到其他类型的定义，而这个转换定义了支持和 Rational 相加的操作，本例发现 int2Rational，因此编译器将 1+ oneHalf 转换为

```
int2Rational(1)+oneHalf
```

## 模拟新的语法结构 ##
隐式转换可以用来扩展 Scala 语言，定义新的语法结构，比如我们在定义一个 Map 对象时可以使用如下语法：

```
Map(1 -> "One", 2->"Two",3->"Three")
```

你有没有想过->内部是如何实现的，->不是 scala 本身的语法，而是类型 ArrowAssoc 的一个方法。这个类型定义在包 Scala.Predef 对象中。 Scala.Predef 自动引入到当前作用域，在这个对象中，同时定义了一个从类型 Any 到 ArrowAssoc 的隐含转换。因此当使用 1 -> “One”时，编译器自动插入从 1 转换到 ArrowAsso c转换。具体定义可以参考 Scala 源码。

利用这种特性，你可以定义新的语法结构，比如行业特定语言（DSL）。
