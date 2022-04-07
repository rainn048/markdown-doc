# spark 动态代码生成

## spark动态代码生成的目的

## 原理

为什么叫Whole-stage code-generation？
答：个人猜测，在Whole-stage code-generation之前，spark也支持简单的动态代码生成，但是仅限于很少的算子，而且仅做一些常量折叠等的操作，现在的Whole-stage code-generation是对整个执行计划的代码生成。


## 参考资料

<https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html>
