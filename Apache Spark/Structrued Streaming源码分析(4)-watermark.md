# watermark简介

Structured Streaming里的watermark是指event time watermark，即时间时间的watermark。这个watermark是一个时间点，在这时间之前，我们假设不再有延迟数据到达。

Spark将在以下几个方面使用此水印:

* 对于一个带时间的window操作，什么时候完成并输出最终结果，即使有延迟数据到来也不再更新；
* 减小为了聚合操作而保存的中间状态的大小；

当前的watermark是通过查询所有分区中的`MAX(eventTime)` 然后减去用户指定的延吃时间`delayThreshold`计算得到的。

由于跨分区计算此值的成本，实际使用的watermark只能保证在实际事件时间之后至少为`delayThreshold`，即小于这个阈值的都保证不丢，但是大于这个阈值的不保证必须丢。

因为计算watermark是需要获取实际数据中的事件时间的，所以，watermark的计算时在物理计划执行的时候。

## 多流情况下watermark的选择策略

一个流查询可以有多个流，可以join，也可以不join。每个输入流都可以有不同的watermark。

在执行查询时，Structured Streaming 分别跟踪每个输入流中看到的最大事件时间，基于相应的延迟计算watermark，然后会选择一个作为全局的watermark。默认情况下，选择min策略作为全局watermark，因为它确保了如果其中一个流落后于其他流，不会有数据因为太迟而意外丢失。有些情况下，为了更快得到结果，可以选择max策略来丢掉一些数据。

比如有多个流在一起join，每个流都定义了watermark，max策略就是说，以所有流中看到的最大事件时间来计算watermark，min策略就是以所有流中取一个最小的时间（但是这个时间在这个流里是最大的时间）来作为watermrk。

## watermark的维护

每一个查询流有有一个WatermarkTracker来维护全局的watermark。

## watermark的创建

第一次启动或者重启的时候，会从checkpoint中的offset文件恢复，保证这次跟上次处理相同的一批数据，并且用相同的watermark配置。也是在MicroBatchExecution#populateStartOffsets中计算的。

offset文件和commit文件中都保存有watermark时间戳，如果offset的batchId等于commot的batchId，那么会在这两个文件中取一个最大值作为watermark。

如果没有checkpoint，那么watermark就默认是0.

## 更新

程序起来以后，在代码runBatch中watermark就会随着没批次处理而更新。

因为watermark是一个transform算子，在spark的物理计划中会有一个节点EventTimeWatermarkExec，里面定义了一个累加器eventTimeStats，它会搜集所有executor端发送过来的watermark时间戳。在计算过程中，这个节点会遍历所有数据， 把事件时间的统计信息保存下来。

```scala
case class EventTimeWatermarkExec(
    eventTime: Attribute,
    delay: CalendarInterval,
    child: SparkPlan) extends UnaryExecNode {

  val eventTimeStats = new EventTimeStatsAccum()
  val delayMs = EventTimeWatermark.getDelayMs(delay)

  override protected def doExecute(): RDD[InternalRow] = {
    child.execute().mapPartitions { iter =>
      val getEventTime = UnsafeProjection.create(eventTime :: Nil, child.output)
      iter.map { row =>
        eventTimeStats.add(getEventTime(row).getLong(0) / 1000)
        row
      }
    }
  }

  ......
}


// eventTimeStats.add就对应着这个函数
def add(eventTime: Long): Unit = {
    this.max = math.max(this.max, eventTime)
    this.min = math.min(this.min, eventTime)
    this.count += 1
    this.avg += (eventTime - avg) / count
}

```

当一批次计算完成后，会调用`watermarkTracker.updateWatermark(lastExecution.executedPlan)`来更新watermark。

```scala
 private def runBatch(sparkSessionToRunBatch: SparkSession): Unit = {
     // 构造dataset
     // 计算并输出
     // 更新偏移量和watermark
    withProgressLocked {
      sinkCommitProgress = batchSinkProgress
      watermarkTracker.updateWatermark(lastExecution.executedPlan)
      commitLog.add(currentBatchId, CommitMetadata(watermarkTracker.currentWatermark))
      committedOffsets ++= availableOffsets
    }
 }
```

其中watermarkTracker.updateWatermark(lastExecution.executedPlan)就会把物理计划中的EventTimeWatermarkExec过滤出来，然后获取新的watermark。这个节点

```scala

  def updateWatermark(executedPlan: SparkPlan): Unit = synchronized {
      // 过滤节点
    val watermarkOperators = executedPlan.collect {
      case e: EventTimeWatermarkExec => e
    }
    // 没有的话直接返回
    if (watermarkOperators.isEmpty) return

    watermarkOperators.zipWithIndex.foreach {
      case (e, index) if e.eventTimeStats.value.count > 0 =>
        logDebug(s"Observed event time stats $index: ${e.eventTimeStats.value}")
        val newWatermarkMs = e.eventTimeStats.value.max - e.delayMs
        val prevWatermarkMs = operatorToWatermarkMap.get(index)
        if (prevWatermarkMs.isEmpty || newWatermarkMs > prevWatermarkMs.get) {
          operatorToWatermarkMap.put(index, newWatermarkMs)
        }

      // Populate 0 if we haven't seen any data yet for this watermark node.
      case (_, index) =>
        if (!operatorToWatermarkMap.isDefinedAt(index)) {
          operatorToWatermarkMap.put(index, 0)
        }
    }


  }

```

上面就是watermark的一些原理介绍。
