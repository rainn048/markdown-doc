# Trigger机制

Structured Streaming定义了几种触发方式，包括：

* unspecified (default)
* Fixed interval micro-batches
* One-time micro-batch
* Continuous with fixed checkpoint interval

这里大概介绍一下它们的实现。

很简单，先看一下代码，这四种方式从同一个基类继承而来：

```scala
trait TriggerExecutor {

  /**
   * Execute batches using `batchRunner`. If `batchRunner` runs `false`, terminate the execution.
   */
  def execute(batchRunner: () => Boolean): Unit
}
```

execute就是驱动程序，根据不同的逻辑实现4种触发方式，其中参数batchRunner就是业务处理逻辑，即读数据处理数据写数据都在batchRunner中进行，如果batchRunner返回false，则说明Structured Streaming出错了，则终止程序。

## One-time micro-batch

从源码可以看到，One-time micro-batch就是直接调用了业务处理逻辑的batchRunner，计算完就退出。

```scala
case class OneTimeExecutor() extends TriggerExecutor {

  /**
   * Execute a single batch using `batchRunner`.
   */
  override def execute(batchRunner: () => Boolean): Unit = batchRunner()
}
```

## 其他三种方式

```scala
/**
 * A trigger executor that runs a batch every `intervalMs` milliseconds.
 */
case class ProcessingTimeExecutor(processingTime: ProcessingTime, clock: Clock = new SystemClock())
  extends TriggerExecutor with Logging {

  private val intervalMs = processingTime.intervalMs
  require(intervalMs >= 0)

  override def execute(triggerHandler: () => Boolean): Unit = {
    while (true) {
      val triggerTimeMs = clock.getTimeMillis
      val nextTriggerTimeMs = nextBatchTime(triggerTimeMs)
      // triggerHandler就是业务处理逻辑
      val terminated = !triggerHandler()
      if (intervalMs > 0) {
        val batchElapsedTimeMs = clock.getTimeMillis - triggerTimeMs
        if (batchElapsedTimeMs > intervalMs) {
          notifyBatchFallingBehind(batchElapsedTimeMs)
        }
        if (terminated) {
          return
        }
        // 函数里也是循环等待。
        clock.waitTillTime(nextTriggerTimeMs)
      } else {
        if (terminated) {
          return
        }
      }
    }
  }
```
