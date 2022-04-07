# 偏函数

## 定义

Partial functions are partial in the sense that they aren’t defined for all possible inputs, only those inputs that match at least one of the specified case clauses.

在Partial functions函数中只能指定case子句，而整个函数必须用大括号括起来。相反，普通函数可以包在括号或花括号中。

如果一个输入不与任何case子句匹配，则会在运行时抛出MatchError 匹配错误。

```scala
/**
  * Created by BrownWong on 2016/9/29.
  */
object Hello {
  def main(args: Arra
y[String]): Unit = {
    val pf: PartialFunction[Any, String] = {case s:String => "Yes"}
    println(pf("HAHA"))
    println(pf(12))
  }
}
```

在Scala中，所有偏函数的类型皆被定义为PartialFunction[-A, +B]类型，PartialFunction[-A, +B]又派生自Function1。由于它仅仅处理输入参数的部分分支，它通过isDefineAt()来判断输入值是否应该由当前偏函数进行处理。

## isDefinedAt()方法

可以使用isDefinedAt方法测试Partial Function 函数是否与输入匹配。此函数避免了抛出MatchError异常的风险。

```scala
/**
  * Created by BrownWong on 2016/9/29.
  */
object Hello {
  def main(args: Array[String]): Unit = {
    val pf: PartialFunction[Any, String] = {case s:String => "Yes"}
    println(pf.isDefinedAt("HAHA"))
    println(pf.isDefinedAt(12))
  }
}
```

## orElse方法

偏函数中最常见的组合方法为orElse、andThen与compose。

orElse相当于一个或运算，如果通过它将多个偏函数组合起来，就相当于形成了多个case合成的模式匹配。倘若所有偏函数满足了输入值的所有分支，组合起来就形成一个函数了。例如写一个求绝对值的运算，就可以利用偏函数：

```scala
val positiveNumber:PartialFunction[Int, Int] = { case x if x > 0 => x }
val zero:PartialFunction[Int, Int] = { case x if x == 0 => 0 }
val negativeNumber:PartialFunction[Int, Int] = { case x if x < 0 => -x }
 
def abs(x: Int): Int = {
    (positiveNumber orElse zero orElse negativeNumber)(x)
}
```

利用orElse组合时，还可以直接组合case语句，例如：

```scala
val pf: PartialFunction[Int, String] = {
  case i if i%2 == 0 => "even"
}
val tf: (Int => String) = pf orElse { case _ => "odd" }
```

可以连接多个Partial Functions函数在一起:pf1 orElse pf2 orElse pf3…如果pf1不匹配，则尝试pf2，然后pf3，等等。MathError只有在它们都不匹配时才会被抛出。

## andThen与compose

orElse被定义在PartialFunction类型中，而andThen与compose却不同，它们实则被定义在Function中，PartialFunction只是重写了这两个方法。这意味着函数之间的组合可以使用andThen与compose，偏函数也可以。这两个方法的功能都是将多个（偏）函数组合起来形成一个新函数，只是组合的顺序不同，andThen是组合第一个，接着是第二个，依次类推；而compose则顺序相反。

利用andThen组合偏函数，设计本质接近Pipe-and-Filter模式，每个偏函数都可以理解为是一个Filter。因为要将这些偏函数组合起来形成一个管道，这就要求被组合的偏函数其输入值与输出值必须支持可串接，即上一个偏函数的输出值会作为下一个偏函数的输入值。

注意看，andThen接收的参数为k: B => C，即函数类型而非偏函数类型。当然，由于偏函数继承自函数，它也可以组合偏函数。如果andThen组合了偏函数，则要求输入参数必须满足所有参与组合的偏函数，否则就会抛出MatchError错误。例如编写一个函数，要求将字符串中的数字替换为对应的英文单词，则可以实现为：

```scala
val p1:PartialFunction[String, String] = { case s if s.contains("1") => s.replace("1", "one") }
val p2:PartialFunction[String, String] = { case s if s.contains("2") => s.replace("2", "two") }
val p = p1 andThen p2
```

如果调用p("123")，返回结果为"onetwo3"，但如果传入p("13")，由于p2偏函数的isDefineAt返回false，就会抛出MatchError错误。

偏函数只能接收一个参数，受限于case语句只能声明一个参数？

case语句会被自动编译成偏函数

```scala
{
    case x =>
    case y =>
    case z =>
}
```
