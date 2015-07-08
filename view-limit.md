# View 限定 

上篇的例子

```
def maxListImpParam[T](element:List[T])
                    (implicit orderer:T => Ordered[T]):T =
    element match {
      case List() =>
        throw new IllegalArgumentException("empty list!")
      case List(x) => x
      case x::rest =>
        val maxRest=maxListImpParam(rest)(orderer)
        if(orderer(x) > maxRest) x
        else maxRest
    }
```

其中函数体部分有机会使用 implicit 却没有使用。要注意的是当年在参数中使用 implicit 类型时，编译器不仅仅在需要时补充隐含参数，而且编译器也会把这个隐含参数作为一个当前作用域内可以使用的隐含变量使用，因此在使用隐含参数的函数体内可以省略掉 implicit 的调用而由编译器自动补上。

因此代码可以简化为：

```
def maxList[T](element:List[T])
                    (implicit orderer:T => Ordered[T]):T =
    element match {
      case List() =>
        throw new IllegalArgumentException("empty list!")
      case List(x) => x
      case x::rest =>
        val maxRest=maxList(rest)
        if(x > maxRest) x
        else maxRest
    }
```

编译在看到 x > maxRest 发现类型不匹配，编译器不会马上停止编译，相反，它会检查是否有合适的隐含转换来修补代码，在本例中，它发现 orderer 可用。因此编译器自动改写为 orderer(x)> maxRest。同理我们在递归调用 maxList 省掉了第二个隐含参数，编译器也会自动补上。
 同时我们发现，maxList 代码定义了隐含参数 orderer，而在函数体中没有地方直接引用到该参数，因此你可以任意改名 orderer，比如下面几个函数定义是等价的：

```
def maxList[T](element:List[T])
                    (implicit orderer:T => Ordered[T]):T =
    ...
```

```
def maxList[T](element:List[T])
                    (implicit iceCream:T => Ordered[T]):T =
    ...
```

由于在 Scala 这种用法非常普遍，Scala 中专门定义了一种简化的写法– View 限定。如下：

```
def maxList[T <% Ordered[T]](element:List[T]) :T =
    element match {
      case List() =>
        throw new IllegalArgumentException("empty list!")
      case List(x) => x
      case x::rest =>
        val maxRest=maxList(rest)
        if(x > maxRest) x
        else maxRest
    }
```

其中 <% 为 View 限定，也就是说，我可以使用任意类型的 T，只要它可以看成类型 Ordered[T]。这和 T 是 Orderer[T]的子类不同，它不需要 T 和 Orderer[T]之间存在继承关系。 而如果类型 T 正好也是一个 Ordered[T]类型，你也可以直接把 List[T]传给 maxList，此时编译器使用一个恒等隐含变换：

```
implicit def identity[A](x:A): A =x 
```

在这种情况下，该变换不做任何处理，直接返回传入的对象。
