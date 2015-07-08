# 隐含参数(二) 

隐含参数的另外一个用法是给前面明确定义的参数补充说明一些信息。 我们先给出一个没有使用隐含参数的例子：

```
def maxListUpBound[T <:Ordered[T]](element:List[T]):T =
    element match {
      case List() =>
        throw new IllegalArgumentException("empty list!")
      case List(x) => x
      case x::rest =>
        val maxRest=maxListUpBound(rest)
        if(x > maxRest) x
        else maxRest
    }
```

这个函数是求取一个顺序列表的最大值。但这个函数有个局限，它要求类型 T 是 Ordered[T]的一个子类，因此这个函数无法求一个整数列表的最大值。

下面我们使用隐含参数来解决这个问题。 我们可以再定义一个隐含参数，其类型为一函数类型，可以把一个类型T转换成 Ordered[T]。

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

在这个函数中，隐含参数使用中两个地方，一个递归调用时传入，第二个是检查列表的表头是否大于列表其余部分的最大值。这个例子的隐含参数给前面定义的类型T补充了一些信息，也就是如果比较两个类型 T 对象。

这种用法非常普遍以至于 Scala 的库缺省定义很多类型隐含的到 Ordered 类型的变换。例如我们调用这个函数：

```
scala> maxListImpParam(List(1,5,10,34,23))
res2: Int = 34
```

```
scala> maxListImpParam(List(3.4,5.6,23,1.2))
res3: Double = 23.0
```

```
scala> maxListImpParam(List("one","two","three"))
res4: String = two
```

在这几个调用中，编译器自动为函数添加了对应的 orderer 参数。
