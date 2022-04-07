Scala的构造函数分为主构造函数和辅助构造函数。

辅助构造函数
辅助构造函数比较容易理解，它们同C++和Java的构造函数十分类似，只有两处不同：
1、辅助构造函数的名称为this，这主要是考虑到在C++和Java中，构造函数名与类名同名，当更改类名时需要同时修改构造函数名，因此使用this为构造函数名使程序可靠性更强；
2、每一个辅助构造函数都必须以一个对先前已定义的其他辅助构造函数或主构造函数的调用开始。

```scala
    class Time {
        private var hr = 0
        private var min = 0
 
        def this (hr : Int) {
            this ()    //调用主构造函数
            this.hr = hr
        }
        def this (hr : Int, min : Int) {
            this (hr)    //调用辅助构造函数
            this.min = min
        }
    }
```

可以以以下三种方式构造对象：
val t1 = new Time
val t2 = new Time(16)
val t3 = new Time(16, 27)

主构造函数
在Scala中，每个类都有主构造函数。主构造函数并不以this方法定义，而是和类的定义交织在一起。
一个Scala类的主构造函数是以下的组合：
1、构造函数参数；
2、在类内部被调的方法；
3、在类内部执行的语句和表达式。
下面的例子说明了构造函数参数、类字段和表达式：

```scala
class Person (var firstName : String, var lastName : String) {    //构造函数参数
　　println("the constructor begins")
　　var age = 0
 
　　override def toString = s"$firstName $lastName is $age years old"
　　def printFullName {print(this)}
　　
 
　　printFullName    //被调的方法
　　println("still in the constructor")
}
```

因为类里边的方法是主构造函数的一部分，当实例化一个Person对象时，会看到从类定义的开始到结尾各个方法的输出：
scala> val big = new Person("Poison", "Pink")

the constructor begins
Poison Pink is o years oldstill in the constructor
如果类名之后没有参数，则该类具备一个无参主构造函数，这样一个构造函数仅仅是简单地执行类体中的所有语句而已。
如果主构造函数让你困惑，也可以不使用它，只需要按照常规方法提供一个或者多个辅助构造函数即可，但是要记得调用this()（第一个辅助构造函数，假设其他辅助构造函数是串联的）。

Scala默认主构造函数
在scala中，如果不指定主构造函数，编译器将创建一个主构造函数的构造函数。 所有类的主体的声明都被视为构造函数的一部分。它也被称为默认构造函数。

Scala主要构造函数
Scala提供了一个类的主构造函数的概念。如果代码只有一个构造函数，则可以不需要定义明确的构造函数。它有助于优化代码，可以创建具有零个或多个参数的主构造函数。

Scala次要(辅助)构造器
可以在类中创建任意数量的辅助构造函数，必须要从辅助构造函数内部调用主构造函数。this关键字用于从其他构造函数调用构造函数。当调用其他构造函数时，要将其放在构造函数中的第一行。
