1、作为“通配符”，类似Java中的*。如import scala.math._

2、:_*作为一个整体，告诉编译器你希望将某个参数当作参数序列处理！例如val s = sum(1 to 5:_*)就是将1 to 5当作参数序列处理。

3、指代一个集合中的每个元素。例如我们要在一个Array a中筛出偶数，并乘以2，可以用以下办法：
a.filter(_%2==0).map(2*_)。
又如要对缓冲数组ArrayBuffer b排序，可以这样：
val bSorted = b.sorted(_
4、在元组中，可以用方法_1,_2,  _3访问组员。如a._2。其中句点可以用空格替代。

5、使用模式匹配可以用来获取元组的组员，例如
val (first, second, third) = t
但如果不是所有的部件都需要，那么可以在不需要的部件位置上使用_。 比如上一例中 val ( first, second, _ ) = t

6、还有一点，下划线_代表的是某一类型的默认值。
对于Int来说，它是0。
对于Double来说，它是0.0
对于引用类型，它是null。
