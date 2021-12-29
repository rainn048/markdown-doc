题目中的sql,不仅只sql语句，还指DataFrame，整体流程如下图所示：
![](images/Catalyst-Optimizer-diagram.png)

1. 接收 sql 语句，初步解析成 logical plan
2. 分析上步生成的 logical plan，生成验证后的 logical plan
3. 对分析过后的 logical plan，进行优化
4. 对优化过后的 logical plan，生成 physical plan
5. 根据 physical plan，生成 rdd 的程序，并且提交运行