# 隐含参数(一) 

编译器可以自动插入 implicit 的最后一个用法是隐含参数。 比如编译器在需要是可以把 someCall(a)修改为 someCall(a)（b)或者 new  someClass(a) 修改为 new SomeClass(a)(b)，也就是说编译器在需要的时候会自动补充缺少的参数来完成方法的调用。其中(b)为一组参数，而不仅仅只最后一个参数。

这里我们给出一个简单的例子：假定你定义了一个类 PreferredPrompt，其中定义了一个用户选择的命令行提示符（比如”$ “或者”> “)。

```
class PreferredPrompt(val preference:String)
```

另外又定义了一个 Greeter 对象，该对象定义了一个 greet 方法，该方法定义了两个参数，第一个参数代表用户姓名，第二个参数类型为 P referredPrompt，代表提示符。

```
object Greeter{
	def greet(name:String)(implicit prompt: PreferredPrompt) {
		println("Welcome, " + name + ". The System is ready.")
		println(prompt.preference)
	}
}
```

第二个参数标记为 implicit，表明允许编译器根据需要自动添加。 我们首先采用一般方法的调用方法，提供所有的参数：

```
scala> val bobsPrompt =new PreferredPrompt("relax> ")
bobsPrompt: PreferredPrompt = PreferredPrompt@7e68a062
```

```
scala> Greeter.greet("Bob")(bobsPrompt)
Welcome, Bob. The System is ready.
relax> 
```

这种用法和我们不给第二个参数添加 implicit 调用时一样的结果。前面我们提过，隐含参数的用法有点类似某些 Dependency Injection 框架。 比如我们在某些地方定义一个 PreferredPrompt 对象，而希望编译器在需要时注入该对象，那么该如果使用呢。

首先，我们定义一个对象，然后在该对象中定义一个 PreferredPrompt 类型的隐含实例：

```
object JamesPrefs{
	implicit val prompt=new PreferredPrompt("Yes, master> ")
}
```

然后我们只提供第二个参数看看什么情况：

```
scala> Greeter.greet("James")
<console>:10: error: could not find implicit value for parameter prompt: PreferredPrompt
              Greeter.greet("James")
                           ^
```

出错了，这是因为编译器在当前作用域找不到 PreferredPrompt 类型的隐含变量，它定义在对象 JamesPrefs 中，因此需要使用 Import 引入：

```
scala> import JamesPrefs._
import JamesPrefs._
```

```
scala> Greeter.greet("James")
Welcome, James. The System is ready.
Yes, master> 
```

可以看到编译器自动插入了第二个参数，要注意的是，implicit 关键字作用到整个参数列表，我们修改一下上面的例子看看：

```
class PreferredPrompt(val preference:String)
class PreferredDrink(val preference:String)
object Greeter{
	def greet(name:String)(implicit prompt: PreferredPrompt, drink:PreferredDrink) {
		println("Welcome, " + name + ". The System is ready.")
		print("But while you work,")
		println("why not enjoy a cup of " + drink.preference + "?")
		println(prompt.preference)
	}
}
object JamesPrefs{
	implicit val prompt=new PreferredPrompt("Yes, master> ")
	implicit val drink=new PreferredDrink("coffee")
}
import JamesPrefs._
Greeter.greet("James")
scala> Greeter.greet("James")
Welcome, James. The System is ready.
But while you work,why not enjoy a cup of coffee?
Yes, master> 
```

这里有一点要注意的是，这里 implicit 参数的类型我们没有直接使用  String 类型，事实我们可以使用 String 类型：

```
object Greeter{
	def greet(name:String)(implicit prompt: String) {
		println("Welcome, " + name + ". The System is ready.")
		println(prompt)
	}
}
implicit val prompt="Yes, master> "
Greeter.greet("James")
```

```
scala> Greeter.greet("James")
Welcome, James. The System is ready.
Yes, master> 
```

当问题是如果有多个参数都使用 implicit 类型，而类型相同，你就无法提供多个参数，因此 implicit 类型的参数一般都是定义特殊的类型。
