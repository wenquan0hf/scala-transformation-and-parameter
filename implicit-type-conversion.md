# 隐含类型转换 #
使用隐含转换将变量转换成预期的类型是编译器最先使用 implicit 的地方。这个规则非常简单，当编译器看到类型X而却需要类型Y，它就在当前作用域查找是否定义了从类型X到类型Y的隐式定义。

比如，通常情况下，双精度实数不能直接当整数使用，因为会损失精度：

```
scala> val i:Int = 3.5
<console>:7: error: type mismatch;
 found   : Double(3.5)
 required: Int
       val i:Int = 3.5
                   ^
```

当然你可以直接调用 3.5.toInt。

这里我们定义一个从 Double 到 Int 的隐含类型转换的定义，然后再把 3.5 赋值给整数，就不会报错。

```
scala> implicit def doubleToInt(x:Double) = x toInt
doubleToInt: (x: Double)Int
```

```
scala> val i:Int = 3.5
i: Int = 3
```

此时编译器看到一个浮点数 3.5，而当前赋值语句需要一个整数，此时按照一般情况，编译器会报错，但在报错之前，编译器会搜寻是否定义了从 Double 到 Int 的隐含类型转换，本例，它找到了一个 doubleToInt。 因此编译器将把

```
val i:Int = 3.5
```  

转换成

```
val i:Int = doubleToInt(3.5)
```

这就是一个隐含转换的例子，但是从浮点数自动转换成整数并不是一个好的例子，因为会损失精度。 Scala 在需要时会自动把整数转换成双精度实数，这是因为在 Scala.Predef 对象中定义了一个

```
implicit def int2double(x:Int) :Double = x.toDouble
```

而 Scala.Predef 是自动引入到当前作用域的，因此编译器在需要时会自动把整数转换成 Double 类型。
