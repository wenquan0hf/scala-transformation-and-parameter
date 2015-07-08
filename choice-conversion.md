# 当有多个隐含转换可以选择时 

有时在当前作用域可能存在多个符合条件的隐含转换，在大多数情况下，Scala 编译器在此种情况下拒绝自动插入转换代码。隐含转换只有在转换非常明显的情况下工作良好，编译器只要例行公事插入所需转换代码即可。如果当前作用域存在多个可选项，编译器不知道优先选择哪一个使用。

```
scala> def printLength(seq:Seq[Int]) = println (seq.length)
printLength: (seq: Seq[Int])Unit
```

```
scala> implicit def intToRange(i:Int) = 1 to i
intToRange: (i: Int)scala.collection.immutable.Range.Inclusive
```

```
scala> implicit def intToDigits(i:Int) = i.toString.toList.map( _.toInt)
intToDigits: (i: Int)List[Int]
```

```
scala> printLength(12)
<console>:11: error: type mismatch;
 found   : Int(12)
 required: Seq[Int]
Note that implicit conversions are not applicable because they are ambiguous:
 both method intToRange of type (i: Int)scala.collection.immutable.Range.Inclusive
 and method intToDigits of type (i: Int)List[Int]
 are possible conversion functions from Int(12) to Seq[Int]
              printLength(12)
```

这个例子产生的歧义是非常明显的，将一个整数转换成一组数字和转换成一个序列是明显两个不同的变化。此时应该明确指明使用那个变换：

```
scala> intToDigits(12)
res1: List[Int] = List(49, 50)
```

```
scala> printLength(intToDigits(12))
2
```

```
scala> printLength(intToRange(12))
12
```

在 Scala2.7 以前，Scala 编译器碰到多个可选项时都这么处理，从 2.8 版本以后，这个规则不再这么严格，如果当前作用域内有多个可选项， Scala 编译器优先选择类型更加明确的隐含转换。 比如两个隐含变换一个参数类型为 String，而另外一个类型为 Any。两个隐含转换都可以备选项时，Scala 编译器优先选择参数类型为 String 的那个隐含转换。

“更明确”的一个判断规则如下：

- 参数的类型是另外一个类型的子类型
- 如果两个转换都是对象的方法，前对象是派生于另外一个对象。

Scala 做出这个改进的原因是为了更好的实现 Java，Scala 集合类型（也包括字符串）之间的互操作性。
 
 
