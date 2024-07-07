# 第 11 节：计时器

### 第 11 节：计时器

重复和非重复的 Timer 在编码时总是有用的。除了异步执行代码之外，你通常还需要控制任务应该重复的时间和频率。

在 Dispatch 框架可用之前，开发人员依靠 RunLoop 来异步执行任务并实现并发。 你可以使用 Timer 创建重复和非重复 Timer。 随后，Apple 发布了 Dispatch 框架，包括 DispatchSourceTimer。

尽管以上所有方法都能够创建计时器，但在 Combine 中并非所有计时器都相同。

#### 使用 RunLoop

主线程和你创建的任何线程（最好使用 Thread 类）可以拥有自己的 RunLoop。 只需从当前线程调用 RunLoop.current：如果需要，Foundation 会为你创建一个。请注意，除非你了解 RunLoop 是如何运行的——特别是你需要一个 RunLoop ——否则最好只使用运行应用程序主线程的主 RunLoop。

> 注意：Apple 文档中的一个重要说明和红灯警告是 RunLoop 类不是线程安全的。 你应该只为当前线程的 RunLoop 用 RunLoop 方法。

RunLoop 实现了你将后文中了解的调度程序协议。它定义了几种相对较低级别的方法，并且是唯一一种可以让你创建可取消计时器的方法：

```
let runLoop = RunLoop.main​let subscription = runLoop.schedule(  after: runLoop.now,  interval: .seconds(1),  tolerance: .milliseconds(100)) {  print("Timer fired")}
```

此计时器不传递任何值，也不创建发布者。 它从 after: 参数中指定的日期开始，具有指定的间隔和容差，仅此而已。 它与 Combine 相关的唯一用处是它返回的 Cancelable 可让你在一段时间后停止计时器。

这方面的一个例子可能是：

```
runLoop.schedule(after: .init(Date(timeIntervalSinceNow: 3.0))) {  subscription.cancel()}
```

但考虑到所有因素，RunLoop 并不是创建计时器的最佳方式。 使用 Timer 类会更好！

#### 使用 Timer 类

Timer 是原始 Mac OS X 中可用的最古老的计时器，早在 Apple 将其重新命名为“macOS”之前。 由于它的委托模式和与 RunLoop 的紧密关系，它一直很难使用。 Combine 带来了一个现代变体，你可以直接用作发布者，而无需所有设置样板。

你可以通过以下方式创建重复计时器发布者：

```
let publisher = Timer.publish(every: 1.0, on: .main, in: .common)
```

on 和 in 两个参数确定：

* 你的计时器附加到哪个 RunLoop。 这里是主线程的 RunLoop。
* 计时器在哪个 RunLoop 模式下运行。 这里，默认的 RunLoop 模式。

除非你了解运行循环是如何运行的，否则你应该坚持使用这些默认值。 运行循环是 macOS 中异步事件源处理的基本机制，但它们的 API 有点繁琐。 你可以通过调用 RunLoop.current 为你自己创建或从 Foundation 获取的任何线程获取 RunLoop，因此你也可以编写以下代码：

```
let publisher = Timer.publish(every: 1.0, on: .current, in: .common)
```

> 注意：在 DispatchQueue.main 以外的 Dispatch 队列上运行此代码可能会导致不可预知的结果。 Dispatch 框架在不使用 RunLoop 的情况下管理其线程。 由于 RunLoop 需要调用其运行方法之一来处理事件，因此你永远不会看到计时器在除主队列之外的任何队列上触发。 保持安全并为你的计时器定位 RunLoop.main。

计时器返回的发布者是 `ConnectablePublisher`。 它是 Publisher 的一个特殊变体，在你显式调用它的 `connect()` 方法之前，它不会在订阅时开始触发。 你还可以使用 `autoconnect()` 操作符，它会在第一个订阅者订阅时自动连接。

> 注意：你将在后文中了解有关可连接发布者的更多信息。

因此，创建将在订阅时启动计时器的发布者的最佳方法是编写：

```
let publisher = Timer  .publish(every: 1.0, on: .main, in: .common)  .autoconnect()
```

计时器重复发出当前日期，其 Publisher.Output 类型为 Date。 你可以使用 scan 操作符制作一个发出递增值的计时器：

```
let subscription = Timer  .publish(every: 1.0, on: .main, in: .common)  .autoconnect()  .scan(0) { counter, _ in counter + 1 }  .sink { counter in    print("Counter is \(counter)")  }
```

还有一个你在这里没有看到的额外 `Timer.publish()` 参数：容差(Tolerance)。 它以 TimeInterval 形式指定与你要求的持续时间的可接受偏差。但请注意，使用低于 RunLoop 的 minimumTolerance 值的值可能不会产生预期的结果。

#### 使用 DispatchQueue

你可以使用调度队列来生成计时器事件。 虽然 Dispatch 框架有一个 `DispatchTimerSource` 事件源，但令人惊讶的是，Combine 没有为其提供计时器接口。 相反，你将使用另一种方法在队列中生成计时器事件。不过，这可能有点令人费解：

```
let queue = DispatchQueue.main​// 1let source = PassthroughSubject<Int, Never>()​// 2var counter = 0​// 3let cancellable = queue.schedule(  after: queue.now,  interval: .seconds(1)) {  source.send(counter)  counter += 1}​// 4let subscription = source.sink {  print("Timer emitted \($0)")}
```

在前面的代码中：

1. 创建一个 subject，你将向其发送计时器值。
2. 准备一个 counter，每次计时器触发时，你都会增加它。
3. 每秒在所选队列上安排一个重复操作。 动作立即开始。
4. 订阅 subject 获取定时器值。

如你所见，这并不漂亮。 将此代码移动到函数并传递间隔和开始时间会有所帮助。

#### 关键点

* 如果你怀念 Objective-C 代码，请使用旧的 RunLoop 类创建计时器。
* 使用 Timer.publish 获取发布者，该发布者在指定的 RunLoop 上以给定间隔生成值。
* 将 DispatchQueue.schedule 用于在调度队列上发出事件的现代计时器。

#### 接下来去哪儿？

在后文中，你将学习如何编写自己的发布者，并且你将使用 DispatchSourceTimer 创建一个替代的计时器发布者。在此之前有很多东西要学，从下一节的 Key-Value Observing 开始。
