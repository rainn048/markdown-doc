## 定义

Self-types是一种声明一个trait 必须混合到另一个trait 中的方法，即使它没有直接extend 它。这使得依赖项的成员无需导入即可使用。

## 继承 vs Self Types
继承是面向对象编程中最耦合的关系之一，如果A extends B，那就意味着A就是B。

比如下面这个例子，我们可以说Dog就是一种Animal 。

```scala
trait Animal {
  def stop(): Unit = println("it stops")
}
class Dog extends Animal {
  def bark: String = "Woof!"
}
val goodboy: Dog = new Dog
goodboy.bark
// Woof!
goodboy.stop()
// it stops
```
假如我们又有一个新的trait
```scala
trait Security {
  this: Animal =>   // <--- here is your self type
  def lookout: Unit = { stop(); println("looking out!") }
}
```
Security 必须与Animal一起使用，这是有依赖性的，但Security不是Animal，这很好理解，因为如果我们断言Security是Animal，语义上是不正确的，它们是不同的概念，但是可以一起使用。

这里有一点很重要，尽管Security 没有stop方法，也不是从Animal继承而来的，但它可以调用stop方法。这是可能的，因为Security 只能在Animal类型的对象中使用，所以我们可以访问Animal变量和方法。

然后我们再创建一个新类，guardDog 是一个Dog，也是一个Animall和Security ，它可以访问三者的方法和变量。
```scala
val guardDog = new Dog with Security
guardDog.lookout
// it stops
// looking out!
```
如果我们修改一下定义，如下，这时候guardDog 仅仅是Dog，所以我们不能调用lookout方法。
```scala
val guardDog:Dog = new Dog with Security
guardDog.lookout // <--- ERROR
```
继承也处理不了循环依赖的问题
```scala
trait A extends B
trait B extends A
 
可以用Self Types实现
trait A {
  this: B =>
}
trait B {
  this: A =>
}
```

## spark中的用法
举例
```scala
class UserRepository {
  def authenticate(user: User): User = {
    println("authenticating user: " + user)
    user
   }
  def create(user: User) = println("creating user: " + user)
  def delete(user: User) = println("deleting user: " + user)
}
 
 
//在这里，可以看到我们正在引用UserRepository的一个实例。这就是我们希望为自己注入的依赖。
class UserService {
  def authenticate(username: String, password: String): User =
    userRepository.authenticate(username, password)
 
  def create(username: String, password: String) =
    userRepository.create(new User(username, password))
 
  def delete(user: User) = All is statically typed.
    userRepository.delete(user)
}
```
让我们首先将UserRepository 存储库封装在一个封闭的trait中，并在那里实例化UserRepository .
```scala
trait UserRepositoryComponent {
  val userRepository = new UserRepository
  class UserRepository {
    def authenticate(user: User): User = {
      println("authenticating user: " + user)
      user
    }
    def create(user: User) = println("creating user: " + user)
    def delete(user: User) = println("deleting user: " + user)
  }
}
 
//使用self-type注释声明该组件需要的依赖项，在我们的例子中是UserRepositoryComponent
//如果有多个依赖，可以写成this: Foo with Bar with Baz =>
trait UserServiceComponent { this: UserRepositoryComponent =>
  val userService = new UserService
  class UserService {
    def authenticate(username: String, password: String): User =
      userRepository.authenticate(username, password)
    def create(username: String, password: String) =
      userRepository.create(new User(username, password))
    def delete(user: User) = userRepository.delete(user)
  }
}
 
//
object ComponentRegistry extends
  UserServiceComponent with
  UserRepositoryComponent
```
上面的形式也不是很好，我们在服务实现和它的创建之间有很强的耦合性，连接配置分散在我们的代码库中;完全不灵活。让我们将其更改为抽象成员字段，而不是在其封装的组件特征中实例化服务。
```scala
trait UserRepositoryComponent {
  val userRepository: UserRepository
 
  class UserRepository {
    ...
  }
}
 
 
 
trait UserServiceComponent {
  this: UserRepositoryComponent =>
 
  val userService: UserService
 
  class UserService {
    ...
  }
}
 
//现在，我们可以将服务的实例化(和配置)移动到ComponentRegistry模块
object ComponentRegistry extends
  UserServiceComponent with
  UserRepositoryComponent
{
  val userRepository = new UserRepository
  val userService = new UserService
}
```
通过进行此切换，我们现在已经将实际的组件实例化以及连接抽象为单个“configuration”对象。在这里，我们可以在服务的不同实现之间切换(如果我们定义了一个trait 和多个实现的话)。但更有趣的是，我们可以创造多个“世界”或“环境”，只要简单地组合这些特征以不同的组合。