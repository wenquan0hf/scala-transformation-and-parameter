# 使用 implicits 的一些规则 

在 Scala 中的 implicit 定义指编译器在需要修复类型匹配时可以用来自动插入的定义。比如说，如果 x+y 类型不匹配，那么编译器可能试着使用 convert(x) + y， 其中 convert 由某个 implicit 定义的，这有点类似一个整数和一个浮点数相加，编译器可以自动把整数转换为浮点数。Scala 的 implicit 定义是对这种情况的一个推广，你可以定义一个类型在需要时，如何自动转换成另外一种类型。

Scala 的 implicit 定义符合下面一些规则：

## 标记规则 

只有哪些使用 implicit 关键字的定义才是可以使用的隐式定义。关键字 implicit 用来标记一个隐式定义。编译器才可以选择它作为隐式变化的候选项。你可以使用 implicit 来标记任意变量，函数或是对象。

例如下面为一个隐式函数定义：

```
implicit def intToString(x:Int) : x.toString
```

编译器只有在 convert 被标记成 implicit 才会将 x + y 改成convert(x) + y 。当然这是在 x + y 类型不匹配时。

## 范围规则 

编译器在选择备选 implicit 定义时，只会选取当前作用域的定义，比如说编译器不会去调用 someVariable.convert。如果你需要使用 someVariable.convert，你必须把 someVarible 引入到当前作用域。也就是说编译器在选择备选 implicit 时，只有当 convert 是当前作用域下单个标志符时才会作为备选 implicit。比如说，对于一个函数库来说，在一个 Preamble 对象中定义一些常用的隐式类型转换非常常见，因此需要使用 Preamble 的代码可以使用 “import Preamble._” 把这些 implicit 定义引入到当前作用域才可以。

这个规则有一个例外，编译器也会在类的伙伴对象定义中查找所需的 implicit 定义。例如下面的定义：

```
object Dollar {
	implicit def dollarToEuro(x:Dollar):Euro = ...
	...
}
class Dollar {
   ...
}
```

如果在 class Dollar 的方法有需要 Euro 类型，但输入数据使用的是 Dollar，编译器会在其伙伴对象 object Dollar 查找所需的隐式类型转换，本例定义一个从 Dollar 到 Euro 的 implicit 定义可以使用。

## 一次规则 

编译器在需要使用 implicit 定义时，只会试图转换一次，也就是编译器永远不会把 x + y 改写成 convert1(convert2(x)) + y。

## 优先规则 

编译器不会在 x+y 已经是合法的情况下去调用 implicit 规则。

## 命名规则 

你可以为 implicit 定义任意的名称。通常情况下你可以任意命名，implicit 的名称只在两种情况下有用：一是你想在一个方法中明确指明，另外一个是想把那一个引入到当前作用域。比如我们定义一个对象，包含两个 implicit定义：

```
object MyConversions {
	implicit def stringWrapper(s:String):IndexedSeq[Char] = ...
	implicit def intToString(x:Int):String = ...
}
```

在你的应用中，你想使用 stringWrapper 变换，而不想把整数自动转换成字符串，你可以只引入 stringWrapper。

```
import  MyConversions.stringWrapper
```

## 编译器使用 implicit 的几种情况 

有三种情况使用 implicit: 一是转换成预期的数据类型，而是转换 selection 的 receiver，三是隐含参数。转换成预期的数据类型比如你有一个方法参数类型是 IndexedSeq[Char]，在你传入 String 时，编译器发现类型不匹配，就检查当前作用域是否有从 String 到 IndexedSeq 隐式转换。

转换 selection 的 receiver 允许你适应某些方法调用，比如 “abc”.exist ，”abc”类型为 String，本身没有定义 exist 方法，这时编辑器就检查当前作用域内 String 的隐式转换后的类型是否有 exist 方法，发现 stringWrapper 转换后成 IndexedSeq 类型后，可以有 exist 方法，这个和 C# 静态扩展方法功能类似。

隐含参数有点类似是缺省参数，如果在调用方法时没有提供某个参数，编译器会查找当前作用域是否有符合条件的 implicit 对象作为参数传入（有点类似 dependency injection)。
