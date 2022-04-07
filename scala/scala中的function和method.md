# scala 中的Functions 和 Methods 

## 简介

Scala中的中的Functions（函数）和Methods（方法）表示类似的概念，但在如何使用它们方面存在显著差异。

## Functions函数

Functions函数是一个可调用的代码块，它可以接受一个参数、一个参数列表或者不接受任何参数。一个函数可以执行一条或多条语句，可以返回一个值、一个值列表或者不返回任何值。
我们可以重用这个代码块，这样就不需要重复定义了。

### Anonymous Function匿名函数

匿名函数是没有名称的函数。当我们需要向另一个函数提供一段代码或操作作为参数时，这可能很有用。比如：

```scala
(number: Int) => number + 1
```

在左边的括号中，我们定义了函数参数, 在=>符号之后，定义一个表达式或语句列表。形参列表可以是empty()，也可以根据需要定义任意多个形参:

```scala
() => scala.util.Random.nextInt
(x: Int, y: Int) => (x + 1, y + 1)
```

要定义多个语句，必须将它们用花括号括起来。代码块中最终表达式的结果是函数的返回值。cala有一个return关键字，但是很少使用:

```scala
(number: Int) => {
    println("We are in a function")
    number + 1
}
```

在上面的例子中，我们定义了一个函数，它接受一个名为number的参数。函数体有两个表达式。最后一个表达式将参数值加1并返回结果。结果的类型是自动推断的-参数number是Int类型，加上1之后，结果也是Int类型。

### Named Function命名函数

在Scala中，一切都是对象，所以我们可以给函数赋值:

```scala
val inc = (number: Int) => number + 1
```

变量inc现在包含一个函数。我们可以在任何需要调用函数中定义的代码单元的地方使用这个值:

```scala
scala> println(inc(10))
11
```

因此，inc被认为是一个命名函数。

Scala在后台做了一些“神奇”的工作，允许将函数赋值给变量。Scala中的所有函数都是Function1、Function2或FunctionN类型的特殊对象，其中N是函数的输入参数个数。编译器会自动将我们提供的函数代码装入适当的对象中。这个对象包含一个方法apply()，可以调用它来执行我们的函数。

此外，括号是用来调用函数的apply()方法的“语法糖”。所以，这两个表述是等价的:

```scala
scala> println(inc(10))
11
scala> println(inc.apply(10))
11
```

### Closure闭包

返回值依赖于上下文外某个状态的函数称为闭包。闭包是一种模拟对象的纯函数机制，因为它允许开发人员在某些上下文中传递闭包函数。为了说明这一点，让我们假设我们有一个绘图仪功能:

```scala
def plot(f: Double => Double) = { // Some implementation }
```

我们的函数plot接受任意函数f(x)并可视化它。下面是一个线性方程的定义:

```scala
val lines: (Double, Double, Double) => Double = (a,b,x) => a*x + b
```

这个方程描述了二维空间中所有可能的直线。为了得到一条特定的线——例如，为了绘图——我们必须提供a和b的系数。只有x是线性方程的变量部分。
我们不能在plot函数中只使用直线函数。我们的plot函数接收另一个函数作为参数，该函数只有一个Double类型的形参和一个Double类型的返回值。闭包可以帮助实现这一点:

```scala
val line: (Double, Double) => Double => Double = (a,b) => x => lines(a,b,x)
```

line函数是一个高阶函数——它返回另一个类型为Double => Double的函数，我们可以在plot函数中使用这个函数。我们只需要提供两个系数，我们就会得到一个封闭在这两个参数上的函数:

```scala
val a45DegreeLine = line(1,0)
```

函数a45DegreeLine()接受X坐标的值并返回相应的Y坐标，它定义了一条以45度穿过这些坐标中心的直线。现在，我们可以使用plot功能来画一条线:

```scala
plot(a45DegreeLine())
```

## Methods方法

方法本质上是函数，是类结构的一部分，可以被重写，并使用不同的语法。Scala不允许我们定义匿名方法。我们必须用关键字def来定义方法:

```scala
def inc(number: Int): Int = number + 1
```

：我们可以在方法名后面的圆括号中定义方法的参数列表。在参数列表之后，我们可以用冒号开头为方法提供一个返回类型,然后在等号后面定义方法体。如果代码中只有一条语句，编译器允许我们跳过花括号。

方法定义允许我们对不带参数的方法完全跳过括号。为了演示这一点，让我们提供一个具有相同功能的方法和函数:

```scala
def randomIntMethod(): Int = scala.util.Random.nextInt
val randomIntFunction = () => scala.util.Random.nextInt
```

每当我们写一个方法名时，编译器就会把它看作是对这个方法的调用。相反，没有括号的函数名只是一个保存函数对象引用的变量:

```scala
scala> println(randomIntMethod)
1811098264
scala> println(randomIntFunction)
$line12.$read$$iw$$iw..$$iw$$iw$$$Lambda$4008/0x00000008015cac40@767ee25d
scala> println(randomIntFunction())
1292055875
```

因此，如果需要将方法转换为函数，可以使用下划线:

```scala
val incFunction = inc _
```

变量incFunction包含一个函数，我们可以将它用作我们定义的其他函数:

```scala
scala> println(incFunction(32))
33
```

没有相应的方法可以将函数function 转换为方法method。

### Nested Methods(嵌套的方法)

在Scala中，我们可以在另一个方法中定义一个方法:

```scala
def outerMethod() = {
    def innerMethod() = {
        // inner method's statements here
    }

    // outer method's statements
    innerMethod()
}
```

当某个方法与另一个方法的上下文紧密耦合时，我们可以使用嵌套特性。当我们使用嵌套时，代码看起来更有组织。我们看看通过递归实现阶乘计算来使用嵌套方法的一个常见用法:

```scala
import scala.annotation.tailrec

def factorial(num: Int): Int = {
    @tailrec
    def fact(num: Int, acc: Int): Int = {
        if (num == 0) 
            acc
        else
            fact(num - 1, acc * num)
    }

    fact(num, 1)
}
```

递归实现需要一个额外的方法参数来积累递归调用之间的临时结果，因此我们可以使用一个带有简单接口的装饰外部方法来隐藏实现。

### Parameterization参数化

在Scala中，我们可以按类型参数化方法。参数化允许我们用可重用代码创建一个泛型方法:

```scala
def pop[T](seq: Seq[T]): T = seq.head
```

正如我们所看到的，在方法声明期间，类型参数是用方括号提供的。现在，我们可以对任何类型的T使用pop方法，并获得参数序列的头节点。
简单地说，我们泛化了pop函数:

```scala
scala> val strings = Seq("a", "b", "c")
scala> val first = pop(strings)
first: String = a

scala> val ints = Seq(10, 3, 11, 22, 10)
scala> val second = pop(ints)
second: Int = 10
```

在这种情况下，编译器将推断出类型，因此相应的值将是适当的类型。第一个值的类型是String，第二个值的类型是Int。

### Extension Method扩展方法

Scala的隐式特性允许我们为任何现有类型提供额外的功能。我们只需要定义一个带有方法的新类型，并提供从新类型到现有类型的隐式转换:

```scala
implicit class IntExtension(val value: Int) extends AnyVal {
    def isOdd = value % 2 == 0
}
```

这里，我们定义了一个值类（value class）。值类只是一个特定类型的任何值的包装器。在我们的例子中，value的类型是Int。

值类是在其主体中只有一个参数和方法的类。我们将它从AnyVal类型扩展而来，这样就避免了不必要的对象分配。

要定义隐式转换，必须将类标记为隐式转换。这允许类内的所有方法成为隐式可访问的。当将这样一个类导入到当前上下文中时，可以使用该类中定义的所有方法，就像它们是在本身类中定义的:

```scala
scala> 10.isOdd
res1: Boolean = true
scala> 11.isOdd
res2: Boolean = false
```

Int值10没有显式的方法isOdd，但IntExtension类有。编译器将搜索从Int到IntExtension 的隐式转换。我们的值类提供了这样的转换。编译器将使用它来解析对函数isOdd的调用，并带有必要的参数。

## By-Name Parameters(按名称参数)

到目前为止，我们一直使用“按值(by-value)”参数。换句话说，函数或方法中的任何形参都要在访问该值之前求值:

```scala
def byValue(num: Int) = {
    println(s"First access: $num. Second access: $num")
}

scala> byValue(scala.util.Random.nextInt)
First access: 1705836657. Second access: 1705836657
```

我们提供了一个函数scala.util.Random.nextInt作为调用byValue函数的参数。正如预期的那样，在println函数中使用该参数之前，会先计算它的值。对参数的两次访问都提供相同的值。scala还支持“by-name”参数。每次在函数中访问它们时，都会计算“By-name”参数。

要定义“by-name”形参，必须在其类型前加上=>(箭头符号):

```scala
def byName(num: => Int) = {
    println(s"First access: $num. Second access: $num")
}

scala> byName(scala.util.Random.nextInt)
First access: 1349966645. Second access: 236297674
```

对函数的调用是相同的，但结果略有不同。每次我们访问“by-name”提供的参数时，我们都会再次计算参数的值。这就是为什么每次访问都会得到不同的结果。

## 结论

在本文中，我们展示了如何在Scala中定义函数和方法，以及它们在使用上的主要区别和共同之处。我们展示了闭包的使用，并学习了如何为现有类型提供扩展方法。

## 参考资料

[Functions and Methods in Scala](https://www.baeldung.com/scala/functions-methods)
