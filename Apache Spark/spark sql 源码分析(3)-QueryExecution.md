说到Spark SQL ，我们不得不提到它的优化器（Catalyst），Catalyst是Spark sql的核心，它是针对于Spark SQL语句执行过程中的查询优化框架,最终生成物理计划。下面一张图就涵盖了Catalyst的整个过程。
![](images/Catalyst-Optimizer-diagram.png)

SQL语句首先通过Parser模块被解析成parsed Logical Plan AST语法树，而这棵树是Unresolved Logical Plan（未解析的逻辑计划）的，parsed Logical Plan 通过   Analyzer（分析器）模块借助于Catalog中的表信息解析成为 Analyzer Logical  Plan AST语法树，这棵树是resolved Logical Plan（解析的逻辑计划）；此时，Optimizer（优化器）在通过各种基于规则的优化策略进行深入优化，得到Optimized Logical Plan；优化后的逻辑执行计划依然是逻辑的，是不能够被spark系统理解，此时需要使用SparkPlan将此逻辑执行计划转换为Physical Plan，最后我们使prepareForExecution（）将Physical Plan转换成为系统可以执行的物理计划。

总结：

1. sqlText 经过 SqlParser 解析成 parsed Logical Plan AST语法树 ，而这颗语法树是（Unresolved Logical Plan）;
2. analyzer 模块结合catalog进行绑定,生成 nalyzer Logical  Plan AST语法树，这颗树是（resolved Logical Plan）;
3. optimizer 模块对 resolved LogicalPlan 进行优化,生成 optimized LogicalPlan;
4. SparkPlan 将 LogicalPlan 转换成PhysicalPlan;
5. prepareForExecution()将 PhysicalPlan 转换成可执行物理计划;
6. 使用 execute()执行可执行物理计划;


下面分模块介绍一下Catalyst。

## Parser 模块

```scala
    val tracker = new QueryPlanningTracker
    val plan = tracker.measurePhase(QueryPlanningTracker.PARSING) {
      sessionState.sqlParser.parsePlan(sqlText)
    }
    Dataset.ofRows(self, plan, tracker)
```

 首先调用sqlParser的ParsePlan方法，将sql字符串解析成Unresolved逻辑执行计划。parsePlan的具体是现在AbstractSqlParser类中。  
 ```scala
   /** Creates LogicalPlan for a given SQL string. */
  override def parsePlan(sqlText: String): LogicalPlan = parse(sqlText) { parser =>
    astBuilder.visitSingleStatement(parser.singleStatement()) match {
      case plan: LogicalPlan => plan
      case _ =>
        val position = Origin(None, None)
        throw new ParseException(Option(sqlText), "Unsupported SQL statement", position, position)
    }
  }
 ```
 上段代码可以看出，调用的主函数是parse ，解析sql语句形成AST， 然后调用astBuilder.visitSingleStatement的访问者模式遍历整个AST语法树，把所有节点都替换成了LogicalPlan或者Tableldentifier。

 此时生成的逻辑计划称为unresolved logical Plan。只是将sql字符串解析成类似语法树结构的执行计划，系统并不知道每个词所表示的意思，还不能真正的执行！

 ## Analyzer模块
 在上面的步骤中我们总结了，将parser生成逻辑计划以后，使用analyzer将逻辑执行进行分析。    
 ```scala
  def ofRows(sparkSession: SparkSession, logicalPlan: LogicalPlan, tracker: QueryPlanningTracker)
    : DataFrame = {
    // 这里生成物理计划
    val qe = new QueryExecution(sparkSession, logicalPlan, tracker)
    qe.assertAnalyzed()

    // 这里构造dataset
    new Dataset[Row](sparkSession, qe, RowEncoder(qe.analyzed.schema))
  }
}
```
这里面首先创建了QueryExecution类对象，QueryExecution中定义了sql执行过程中的关键步骤，是sql执行的关键类。QueryExecution类中的成员都是lazy（懒惰）的，被调用时才会执行。只有等到程序中出现action算子时，才会调用queryExecution类中的executed Plan成员，原来生成的逻辑执行计划才会被优化器优化，并转化成物理执行计划真正的 被系统调用执行。

那么QueryExecution中的主要成员有哪些呢？？？主要定义了解析器analyzer、优化器optimizer以及生成物理执行计划的sparkPlan。   

```scala
 lazy val analyzed: LogicalPlan = tracker.measurePhase(QueryPlanningTracker.ANALYSIS) {
    SparkSession.setActiveSession(sparkSession)
    sparkSession.sessionState.analyzer.executeAndCheck(logical, tracker)
  }

lazy val optimizedPlan: LogicalPlan = tracker.measurePhase(QueryPlanningTracker.OPTIMIZATION) {
    sparkSession.sessionState.optimizer.executeAndTrack(withCachedData, tracker)
  }

lazy val sparkPlan: SparkPlan = tracker.measurePhase(QueryPlanningTracker.PLANNING) {
    SparkSession.setActiveSession(sparkSession)
    // TODO: We use next(), i.e. take the first plan returned by the planner, here for now,
    //       but we will implement to choose the best plan.
    planner.plan(ReturnAnswer(optimizedPlan)).next()
  }

lazy val executedPlan: SparkPlan = tracker.measurePhase(QueryPlanningTracker.PLANNING) {
    prepareForExecution(sparkPlan)
}
```
其中的sparkSession.sessionState.analyzer和sparkSession.sessionState.optimizer都继承自RuleExecutor，都定义了自己的一些策略，都是针对LogicalPlan进行处理，返回的结果也还是LogicalPlan。

## SparkPlanner模块
此时，已经得到了优化后的逻辑计划，但时还是没办法直接运行，比如join只是一个抽象概念，代表两个表根据相同的id进行合并，然而具体怎么实现这个合并，逻辑执行计划并没有说明。这就需要将逻辑计划转换为physical plan物理执行计划，将逻辑上可行的执行计划变为spark可以真正执行的计划。

