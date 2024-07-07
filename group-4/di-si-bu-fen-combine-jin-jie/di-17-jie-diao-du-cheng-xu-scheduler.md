# 第 17 节：调度程序 Scheduler

### 第 17 节：调度程序 Scheduler

在阅读本书的过程中，你已经阅读了有关将 Scheduler 作为参数的操作符。大多数情况下，你会简单地使用 DispatchQueue.main，因为它方便、易于理解并带来令人放心的安全感。

作为开发人员，你至少对 DispatchQueue 是什么有一个大致的了解。除了 DispatchQueue.main，你肯定已经使用了全局并发队列之一，或者创建了一个串行调度队列来串行运行操作。如果你不记得或不记得详细信息，请不要担心。你将在本章中重新评估有关调度队列的一些重要信息。

但是，为什么 Combine 需要一个新的类似概念呢？现在是你深入了解 Combine Scheduler 的真正性质、意义和目的的时候了！

在本章中，你将了解为什么会出现 Scheduler 的概念。你将探索 Combine 如何使异步事件和操作易于使用，当然，你将尝试使用 Combine 提供的所有 Scheduler。

#### 调度器简介

根据 Apple 的文档，Scheduler 是一种定义何时以及如何执行闭包的协议。尽管定义是正确的，但这只是一部分。

调度程序提供上下文以尽快或在将来的某个日期执行未来的操作。该操作是协议本身中定义的闭包。但是术语闭包也可以隐藏发布者在特定调度程序上执行的某些值的传递。

你是否注意到此定义有意避免对线程的任何引用？这是因为具体的实现是定义调度程序协议提供的“上下文”在哪里执行的！

因此，你的代码将在哪个线程上执行的确切细节取决于你选择的 Scheduler。

记住这个重要的概念：Scheduler 不等于线程。你将在本章后面详细了解这对每个 Scheduler 意味着什么。

让我们从事件流的角度来看 Scheduler 的概念：

![image-20221023150356813](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023150356813.png)

你在上图中看到的内容：

* 在主 (UI) 线程上发生用户操作（按钮按下）。
* 它会触发一些工作在后台调度程序上进行处理。
* 要显示的最终数据在主线程上传递给订阅者，因此订阅者可以更新应用程序的 UI。

你可以看到 Scheduler 的概念如何深深植根于前台/后台执行的概念。此外，根据你选择的实现，工作可以串行化或并行化。

因此，要全面了 Scheduler，需要查看哪些类符合 Scheduler 协议。

但首先，你需要了解与Scheduler 相关的两个重要操作符！

> 注意：在下一节中，你将主要使用符合 Combine 的 Scheduler 协议的 DispatchQueue。

#### Scheduler 操作符

Combine 框架提供了两个基本的操作符来使用调度器：

* subscribe(on:) 和 subscribe(on:options:) 在指定的 Scheduler 上创建订阅（开始工作）。
* receive(on:) 和 receive(on:options:) 在指定的 Scheduler 上传递值。

此外，以下操作符将调度程序和调度程序选项作为参数。你在第 6 节了它们：

* debounce(for:scheduler:options:)
* delay(for:tolerance:scheduler:options:)
* measureInterval(using:options:)
* throttle(for:scheduler:latest:)
* timeout(\_:scheduler:options:customError:)

如果你需要刷新对这些操作符的记忆，请立即回顾第 6 节。

**介绍 subscribe(on:)**

请记住——在你订阅它之前，发布者是一个无生命的实体。但是当你订阅发布者时会发生什么？有几个步骤：

![image-20221023150832513](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023150832513.png)

1. 发布者 receive 订阅者并创建 Subscription。
2. 订阅者 receive Subscription 并从发布者请求值（虚线）。
3. 发布者开始工作（通过 Subscription）。
4. 发布者发出值（通过 Subscription）。
5. 操作符转换值。
6. 订阅者收到最终值。

当你的代码订阅发布者时，步骤一、二和三通常发生在当前线程上。 但是当你使用 subscribe(on:) 操作符时，所有这些操作都在你指定的 Scheduler 上运行。

> 注意：在查看 receive(on:) 操作符时，你将回到此图。 然后，你将了解底部的两个框，其中标有步骤五和六。

你可能希望发布者在后台执行一些昂贵的计算以避免阻塞主线程。 执行此操作的简单方法是使用 subscribe(on:)。

是时候看一个例子了！

打开项目文件夹中的 Starter.playground 并选择 subscribeOn-receiveOn 页面。确保显示 Debug 区域，然后添加以下代码：

```
// 1let computationPublisher = Publishers.ExpensiveComputation(duration: 3)​// 2let queue = DispatchQueue(label: "serial queue")​// 3let currentThread = Thread.current.numberprint("Start computation publisher on thread \(currentThread)")
```

以下是上述代码的细分：

1. 这个 Playground 在 Sources/Computation.swift 中定义了一个名为 ExpensiveComputation 的特殊发布者，它模拟一个长时间运行的计算，在指定的持续时间后发出一个字符串。
2. 一个串行队列，你将使用它来触发特定调度程序上的计算。正如你在上面了解到的，DispatchQueue 符合调度程序协议。
3. 你获取当前执行线程号。在 Playground 中，主线程（线程编号 1）是你的代码运行的默认线程。Thread 类的编号扩展在 Sources/Thread.swift 中定义。

注意：ExpensiveComputation 发布者如何实现的细节目前并不重要。你将在下一节中了解有关创建自己的发布者的更多信息。

回到 subscribeOn-receiveOn Playground 页面，你需要订阅 computePublisher 并显示它发出的值：

```
let subscription = computationPublisher  .sink { value in    let thread = Thread.current.number    print("Received computation result on thread \(thread): '\(value)'")  }
```

执行 Playground 并查看输出：

```
Start computation publisher on thread 1ExpensiveComputation subscriber received on thread 1Beginning expensive computation on thread 1Completed expensive computation on thread 1Received computation result on thread 1 'Computation complete'
```

让我们深入研究各个步骤以了解会发生什么：

* 你的代码在主线程上运行。 从那里，它订阅计算发布者。
* ExpensiveComputation 发布者接收订阅者。
* 它创建一个订阅，然后开始工作。
* 工作完成后，发布者通过订阅传递结果并完成。

你可以看到所有这些都发生在主线程线程 1 上。

现在，更改发布者订阅以插入 subscribe(on:) 调用：

```
let subscription = computationPublisher  .subscribe(on: queue)  .sink { value in...
```

再次执行 Playground 可以看到类似如下的输出：

```
Start computation publisher on thread 1ExpensiveComputation subscriber received on thread 5Beginning expensive computation from thread 5Completed expensive computation on thread 5Received computation result on thread 5 'Computation complete'
```

啊!这是不同的！现在你可以看到你仍在从主线程订阅，但是将委托 Combine 到你提供的队列以有效地执行订阅。队列在其线程之一上运行代码。由于计算在线程 5 上开始并完成，然后从该线程发出结果值，因此你的接收器也接收该线程上的值。

> 注意：由于 DispatchQueue 的动态线程管理特性，你可能会在此日志中看到不同的线程号，并在本章中看到更多日志。重要的是一致性：相同的线程号应该在相同的步骤中显示。

但是，如果你想更新一些屏幕信息怎么办？你需要在接收器闭包中执行类似 DispatchQueue.main.async { ... } 的操作，以确保你正在从主线程执行 UI 更新。

有一种更有效的方法可以使用 Combine！

**介绍 receive(on:)**

你想知道的第二个重要的操作符是 receive(on:)。它允许你指定应该使用哪个调度程序向订阅者传递值。但是，这是什么意思？

在订阅中的接收器之前插入一个调用 receive(on:) ：

```
let subscription = computationPublisher  .subscribe(on: queue)  .receive(on: DispatchQueue.main)  .sink { value in
```

然后，再次执行 Playground。现在你看到这个输出：

```
Start computation publisher on thread 1ExpensiveComputation subscriber received on thread 4Beginning expensive computation from thread 4Completed expensive computation on thread 4Received computation result on thread 1 'Computation complete'
```

> 注意：你可能会在与接下来的两个步骤不同的线程上看到第二条消息。由于 Combine 中的内部管道，这一步和下一步可能在同一个队列上异步执行。由于 Dispatch 动态管理自己的线程池，因此你可能会看到这一行和下一行的线程号不同，但不会看到线程 1。

成功！即使计算工作正常并从后台线程发出结果，你现在也可以保证始终在主队列上接收值。这是你安全地执行 UI 更新所需要的。

在本调度操作符介绍中，你使用了 DispatchQueue。 Combine 扩展了它来实现调度器协议，但它不是唯一的！是时候深入了解 Schedule 了！

#### Scheduler 实现

Apple 提供了几种调度器协议的具体实现：

* ImmediateScheduler：一个简单的 Scheduler，它立即在当前线程上执行代码，这是默认的执行上下文，除非使用 subscribe(on:)、receive(on:) 或任何其他将调度程序作为参数的操作符进行修改。
* RunLoop：绑定到 Foundation 的 Thread 对象。
* DispatchQueue：可以是串行的或并发的。
* OperationQueue：规范工作项执行的队列。

在本章的其余部分，你将了解所有这些及其具体细节。

> 注意：这里一个明显的遗漏是缺少 TestScheduler，它是任何响应式编程框架的测试部分不可或缺的一部分。如果没有这样一个虚拟的、模拟的 Scheduler，彻底测试你的 Conbine 代码是一项挑战。你将在第 19 节“测试”中探索有关这种特殊调度程序的更多细节。

**ImmediateScheduler**

调度程序类别中最简单的条目也是 Combine 框架提供的最简单的：ImmediateScheduler。这个名字已经破坏了细节，所以看看它的作用！

打开 Playground 的 ImmediateScheduler 页面。你不需要此调试区域，但请确保你使实时视图可见。你将使用这个 Playground 中内置的一些精美的新工具来跨调度程序跟踪你的发布者值！

开始创建一个简单的计时器，就像你在前几章中所做的那样：

```
let source = Timer  .publish(every: 1.0, on: .main, in: .common)  .autoconnect()  .scan(0) { counter, _ in counter + 1 }
```

接下来，准备一个创建发布者的闭包。你将使用 Sources/Record.swift 中定义的自定义操作符：recordThread(using:)。该算子记录了算子看到一个值通过时当前的线程，并且可以多次记录从发布者源到最终接收器。

> 注意：此 recordThread(using:) 操作符仅用于测试目的，因为该操作符将数据类型更改为内部值类型。它的实现细节超出了本章的范围，但喜欢冒险的读者可能会发现它很有趣。

添加此代码：

```
// 1let setupPublisher = { recorder in  source    // 2    .recordThread(using: recorder)    // 3    .receive(on: ImmediateScheduler.shared)    // 4    .recordThread(using: recorder)    // 5    .eraseToAnyPublisher()}// 6let view = ThreadRecorderView(title: "Using ImmediateScheduler", setup: setupPublisher)PlaygroundPage.current.liveView = UIHostingController(rootView: view)
```

在上面的代码中：

1. 准备一个返回发布者的闭包，使用给定的记录器对象通过 recordThread(using:) 设置当前线程记录。
2. 在这个阶段，定时器发出一个值，所以你记录当前线程。你已经猜到是哪一个了吗？
3. 确保发布者在共享的 ImmediateScheduler 上传递值。
4. 记录你现在所在的线程。
5. 闭包必须返回一个 AnyPublisher 类型。这主要是为了方便内部实现。
6. 准备并实例化一个 ThreadRecorderView，它显示发布的值在各个记录点的线程之间的迁移。

执行 Playground 页面，几秒钟后查看输出：

![image-20221023153712833](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023153712833.png)

此表示显示源发布者（计时器）发出的每个值。在每一行上，你都可以看到值正在经历的线程。每次添加 recordThread(using:) 操作符时，都会在该行上看到一个额外的线程号。

在这里你可以看到在你添加的两个记录点处，当前线程是主线程。这是因为 ImmediateScheduler 立即在当前线程上“调度”。

此表示显示源发布者（计时器）发出的每个值。在每一行上，你都可以看到值正在经历的线程。每次添加 recordThread(using:) 操作符时，都会在该行上看到一个额外的线程号。

在这里你可以看到在你添加的两个记录点处，当前线程是主线程。这是因为 ImmediateScheduler 立即在当前线程上“调度”。

为了验证这一点，你可以做一个小实验！回到你的 setupPublisher 闭包定义，就在第一条 recordThread 行之前，插入以下内容：

```
.receive(on: DispatchQueue.global())
```

这请求源发出的值在全局并发队列上进一步可用。这会产生有趣的结果吗？执行 Playground：

![image-20221023153915913](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023153915913.png)

这是完全不同的！你能猜到为什么线程一直在变化吗？你将在本章的 DispatchQueue 介绍中了解更多信息！

**ImmediateScheduler 选项**

由于大多数操作符在其参数中接受调度程序，你还可以找到一个接受 SchedulerOptions 值的选项参数。在 ImmediateScheduler 的情况下，此类型被定义为 Never，因此在使用 ImmediateScheduler 时，你永远不应该为操作符的 options 参数传递值。

**ImmediateScheduler 的陷阱**

关于 ImmediateScheduler 的一件事是它是即时的。你将无法使用 Scheduler 协议的任何 schedule(after:) 变体，因为你需要指定延迟的 SchedulerTimeType 没有公共初始化程序，并且对于立即调度毫无意义。

你将在本节中学习的第二种调度程序存在类似但不同的陷阱：RunLoop。

**RunLoop scheduler**

早期 iOS 和 macOS 开发人员熟悉 RunLoop。早于 DispatchQueue，它是一种在线程级别管理输入源的方法，包括在主 (UI) 线程中。你的应用程序的主线程仍然有一个关联的 RunLoop。你还可以通过从当前线程调用 RunLoop.current 为任何基础线程获取一个。

> 注意：现在 RunLoop 是一个不太有用的类，因为 DispatchQueue 在大多数情况下是一个明智的选择。这就是说，仍有一些特定情况下 RunLoop 很有用。例如，Timer 将自己安排在 RunLoop 上。 UIKit 和 AppKit 依赖 RunLoop 及其执行模式来处理各种用户输入情况。描述有关 RunLoop 的所有内容超出了本书的范围。

要查看 RunLoop，请在 Playground 中打开 RunLoop 页面。你之前使用的 Timer 源是相同的，所以已经为你编写好了。在其后添加此代码：

```
let setupPublisher = { recorder in  source    // 1    .receive(on: DispatchQueue.global())    .recordThread(using: recorder)    // 2    .receive(on: RunLoop.current)    .recordThread(using: recorder)    .eraseToAnyPublisher()}let view = ThreadRecorderView(title: "Using RunLoop", setup: setupPublisher)PlaygroundPage.current.liveView = UIHostingController(rootView: view)
```

1. 和之前一样，首先让值通过全局并发队列。为什么？因为很好玩！
2. 然后，你要求在 RunLoop.current 上接收值。

但是什么是 RunLoop.current？它是与调用时当前的线程相关联的 RunLoop。闭包由 ThreadRecorderView 调用，从主线程设置发布者和订阅者。因此，RunLoop.current 就是主线程的 RunLoop。

执行 Playground 看看会发生什么：

![image-20221023154756596](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023154756596.png)

正如你所要求的，第一个 recordThread 显示每个值都通过一个全局并发队列的线程，然后在主线程上继续。

**一点理解挑战**

如果你第一次使用 subscribe(on: DispatchQueue.global()) 而不是 receive(on:) 会发生什么？试试看！

你会看到所有内容都记录在线程一上。一开始可能并不明显，但完全合乎逻辑。是的，发布者已在并发队列上订阅。但是请记住，你正在使用一个 Timer，它在主 RunLoop 上发出其值！因此，无论你选择在哪个 Scheduler 上订阅此发布者，值都将始终在主线程上开始它们的旅程。

**使用 RunLoop 调度代码执行**

Scheduler 允许你安排尽快执行的代码，或在未来某个日期之后执行。虽然不可能将后一种形式与 ImmediateScheduler 一起使用，但 RunLoop 完全能够延迟执行。

每个 Scheduler 实现都定义了自己的 SchedulerTimeType。在你弄清楚要使用的数据类型之前，这会让事情变得有点复杂。在 RunLoop 的情况下，SchedulerTimeType 值是一个 Date。

你将安排一个操作，在几秒钟后取消 ThreadRecorderView 的订阅。事实证明 ThreadRecorder 类有一个可选的 Cancellable 可用于停止其对发布者的订阅。

首先，你需要一个变量来保存对 ThreadRecorder 的引用。在页面的开头，添加以下行：

```
var threadRecorder: ThreadRecorder? = nil
```

现在你需要捕获线程记录器实例。执行此操作的最佳位置是 setupPublisher 闭包。但是怎么做？你可以：

* 向闭包添加显式类型，分配 threadRecorder 变量并返回发布者。你需要添加显式类型，因为糟糕的 Swift 编译器会抱怨“可能无法推断出复杂的闭包返回类型。
* 在订阅时使用一些操作符来捕获记录器。

去狂野做后者！

在你的 setupPublisher 闭包中添加此行，然后在 eraseToAnyPublisher() 之前：

```
.handleEvents(receiveSubscription: { _ in threadRecorder = recorder })
```

捕获记录器的有趣选择！

> 注意：你已经在第 10 节“调试”中了解了 handleEvents。它有一个很长的签名，让你可以在发布者生命周期的不同点执行代码（在反应式编程术语中，这称为注入副作用），而无需实际与其发出的值进行交互。在这种情况下，你正在拦截记录器订阅发布者的时刻，以便在你的全局变量中捕获记录器。不漂亮，但它以一种有趣的方式完成了这项工作！

现在你已准备就绪，可以在几秒钟后安排一些操作。在页面末尾添加此代码：

```
RunLoop.current.schedule(  after: .init(Date(timeIntervalSinceNow: 4.5)),  tolerance: .milliseconds(500)) {    threadRecorder?.subscription?.cancel()  }
```

这个 schedule(after:tolerance:) 让你可以安排提供的闭包何时执行以及可容忍的漂移，以防系统无法在所选时间精确执行代码。你将当前日期添加 4.5 秒，以允许在执行之前发送四个值。

运行 Playground。你可以看到列表在第四项之后停止更新。这是你的取消机制有效！

> 注意：如果你只获得三个值，这可能意味着你的 Mac 运行速度有点慢并且无法容纳半秒的容差，因此你可以尝试更多地调整日期，例如，将 timeIntervaleSinceNow 设置为 5.0，将容差设置为 1.0。

**RunLoop 选项**

与 ImmediateScheduler 一样，RunLoop 不为采用 SchedulerOptions 参数的调用提供任何合适的选项。

**RunLoop 陷阱**

RunLoop 的使用应仅限于主线程的 RunLoop，以及你在需要时控制的 Foundation 线程中可用的 RunLoop。也就是说，任何你自己使用 Thread 对象开始的东西。

要避免的一个特殊陷阱是在 DispatchQueue 上执行的代码中使用 RunLoop.current。这是因为 DispatchQueue 线程可能是短暂的，这使得它们几乎不可能依赖 RunLoop。

你现在已经准备好学习最通用和最有用的调度程序：DispatchQueue！

**DispatchQueue Scheduler**

在本章和之前的章节中，你一直在各种情况下使用 DispatchQueue。毫不奇怪，DispatchQueue 符合 Scheduler 协议，并且完全可用于所有将 Scheduler 作为参数的操作符。

但首先，快速回顾一下调度队列。 Dispatch 框架是 Foundation 的一个强大组件，它允许你通过向系统管理的调度队列提交工作来在多核硬件上同时执行代码。

DispatchQueue 可以是串行的（默认）或并发的。串行队列按顺序执行你提供给它的所有工作项。并发队列将并行启动多个工作项，以最大限度地提高 CPU 使用率。两种队列类型都有不同的用法：

* 串行队列通常用于保证某些操作不重叠。因此，如果所有操作都发生在同一个队列中，他们可以使用共享资源而无需锁定。
* 并发队列将同时执行尽可能多的操作。因此，它更适合纯计算。

**队列和线程**

你一直使用的最熟悉的队列是 DispatchQueue.main。它直接映射到主（UI）线程，在这个队列上执行的所有操作都可以自由地更新用户界面。 UI 更新只允许从主线程进行。

所有其他队列，无论是串行的还是并发的，都在系统管理的线程池中执行它们的代码。这意味着你永远不应该对队列中运行的代码中的当前线程做出任何假设。特别是，你不应使用 RunLoop.current 来安排工作，因为 DispatchQueue 管理其线程的方式。

所有调度队列共享同一个线程池。你提供工作执行的串行队列将使用该池中的任何可用线程。一个直接的结果是，来自同一队列的两个连续工作项可能使用不同的线程，同时仍按顺序执行。

这是一个重要的区别：当使用 subscribe(on:)、receive(on:) 或任何其他采用 Scheduler 参数的操作符时，你永远不应假设支持调度程序的线程每次都是相同的。

**使用 DispatchQueue 作为 Scheduler**

是你试验的时候了！像往常一样，你将使用计时器来发出值并观察它们在调度程序之间的迁移。但是这一次，你将使用 Dispatch Queue 计时器创建计时器。

打开名为 DispatchQueue 的游乐场页面。首先，你将创建几个队列以供使用。将此代码添加到你的 Playground：

```
let serialQueue = DispatchQueue(label: "Serial queue")let sourceQueue = DispatchQueue.main
```

你将使用 sourceQueue 发布来自计时器的值，然后使用 serialQueue 来试验切换调度程序。

现在添加以下代码：

```
// 1let source = PassthroughSubject<Void, Never>()// 2let subscription = sourceQueue.schedule(after: sourceQueue.now,                                        interval: .seconds(1)) {  source.send()}
```

1. 当计时器触发时，你将使用 Subject 发出一个值。你不关心实际的输出类型，因此你只需使用 Void。
2. 正如你在第 11 节“定时器”中所了解的，队列完全能够生成定时器，但没有用于队列定时器的 Publisher API。 你必须使用调度程序协议中的 schedule() 方法的重复变体。它立即开始并返回一个 Cancellable。每次计时器触发时，你都会通过 source 发送一个 Void 值。

> 注意：你是否注意到你是如何使用 now 属性来指定计时器的开始时间的？这是调度程序协议的一部分，并返回使用调度程序的 SchedulerTimeType 表示的当前时间。每个实现调度器协议的类都为此定义了自己的类型。

现在，你可以开始练习 scheduler 了。通过添加以下代码来设置你的发布者：

```
let setupPublisher = { recorder in  source    .recordThread(using: recorder)    .receive(on: serialQueue)    .recordThread(using: recorder)    .eraseToAnyPublisher()}
```

这里没有什么新东西，你在本章中已经多次编写了类似的模式。

然后，与前面的示例一样，设置显示：

```
let view = ThreadRecorderView(title: "Using DispatchQueue",                              setup: setupPublisher)PlaygroundPage.current.liveView = UIHostingController(rootView: view)
```

执行 Playground。很容易，你会看到它的意图：

![image-20221023161035542](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023161035542.png)

1. 计时器在主队列上触发并通过 Subject 发送 Void 值。
2. 发布者在你的串行队列上接收值。

你有没有注意到第二个 recordThread(using:) 在 receive(on:) 操作符之后是如何记录当前线程中的变化的？这是 DispatchQueue 如何不保证每个工作项在哪个线程上执行的一个完美示例。在 receive(on:) 的情况下，工作项是从当前调度程序跳到另一个调度程序的值。

现在，如果你从串行队列中发出值并保持相同的 receive(on:) 操作符会发生什么？值还会在旅途中改变线程吗？

试试看！回到代码的开头，将 sourceQueue 定义更改为：

```
let sourceQueue = serialQueue
```

现在，再次执行 Playground：

![image-20221023161244842](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023161244842.png)

有趣的！你再次看到 DispatchQueue 的无线程保证效果，但你也看到 receive(on:) 操作符从不切换线程！看起来内部正在进行一些优化以避免额外的切换。你将在本章的挑战中探索这一点！

**DispatchQueue 选项**

DispatchQueue 是唯一提供一组选项的 Scheduler，当操作符采用 SchedulerOptions 参数时，你可以传递这些选项。这些选项主要围绕指定 QoS（服务质量）值独立于 DispatchQueue 上已设置的值。工作项还有一些额外的标志，但在绝大多数情况下你都不需要它们。

不过，要查看如何指定 QoS，请将 setupPublisher 中的 receive(on:options:) 修改为以下内容：

```
.receive(  on: serialQueue,  options: DispatchQueue.SchedulerOptions(qos: .userInteractive))
```

你将 DispatchQueue.SchedulerOptions 的实例传递给指定最高服务质量的选项：.userInteractive。它指示操作系统尽最大努力将价值交付优先于不太重要的任务。当你想尽快更新用户界面时，可以使用此功能。相反，如果快速交付的压力较小，你可以使用 .background 服务质量。在此示例的上下文中，你不会看到真正的区别，因为它是唯一正在运行的任务。

在实际应用程序中使用这些选项有助于操作系统决定在你同时有许多队列忙碌的情况下首先安排哪个任务。它确实可以微调你的应用程序性能！

你几乎完成了调度程序！再坚持一点。你需要了解最后一个 Scheduler。

**OperationQueue**

你将在本章中学习的最后一个调度程序是 OperationQueue。文档将其描述为一个规范操作执行的队列。它是一种丰富的监管机制，可让你创建具有依赖关系的高级操作。但是在Combine 的上下文中，你将不会使用这些机制。

由于 OperationQueue 在后台使用 Dispatch，因此在使用另一种时表面上几乎没有区别。或者有吗？

举一个简单的例子。打开 OperationQueue Playground 页面并开始编码：

```
let queue = OperationQueue()let subscription = (1...10).publisher  .receive(on: queue)  .sink { value in    print("Received \(value)")  }
```

你正在创建一个简单的发布者发出 1 到 10 之间的数字，确保值到达你创建的 OperationQueue。然后在接收器中打印该值。

你能猜到会发生什么吗？展开 Debug 区域并执行 Playground：

```
Received 4Received 3Received 2Received 7Received 5Received 10Received 6Received 9Received 1Received 8
```

这令人费解！按顺序发出但无序到达！怎么会这样？要找出答案，你可以更改打印行以显示当前线程号：

```
print("Received \(value) on thread \(Thread.current.number)")
```

再次执行 Playground：

```
Received 1 on thread 5Received 2 on thread 4Received 4 on thread 7Received 7 on thread 8Received 6 on thread 9Received 10 on thread 10Received 5 on thread 11Received 9 on thread 12Received 3 on thread 13Received 8 on thread 14
```

啊哈！如你所见，每个值都是在不同的线程上接收的！如果你查看有关 OperationQueue 的文档，有一条关于线程的说明，其中说 OperationQueue 使用 Dispatch 框架（因此是 DispatchQueue）来执行操作。这意味着它不保证它会为每个交付的值使用相同的底层线程。

此外，每个 OperationQueue 中都有一个参数可以解释一切：它是 maxConcurrentOperationCount。它默认为系统定义的数字，允许操作队列同时执行大量操作。由于你的发布者几乎在同一时间发出所有项目，它们被 Dispatch 的并发队列分派到多个线程！

对你的代码进行一些修改。定义队列后，添加以下行：

```
queue.maxConcurrentOperationCount = 1
```

然后运行页面，查看调试区域：

```
Received 1 on thread 3Received 2 on thread 3Received 3 on thread 3Received 4 on thread 3Received 5 on thread 4Received 6 on thread 3Received 7 on thread 3Received 8 on thread 3Received 9 on thread 3Received 10 on thread 3
```

这一次，你将获得真正的顺序执行——将 maxConcurrentOperationCount 设置为 1 相当于使用串行队列——并且你的值按顺序到达。

**OperationQueue 选项**

OperationQueue 没有可用的 SchedulerOptions。它实际上是别名为 RunLoop.SchedulerOptions 的类型，它本身没有提供任何选项。

**OperationQueue 陷阱**

你刚刚看到 OperationQueue 默认并发执行操作。你需要非常清楚这一点，因为它可能会给你带来麻烦：默认情况下，OperationQueue 的行为类似于并发 DispatchQueue。

但是，当你每次发布者发出值时都有大量工作要执行时，它可能是一个很好的工具。你可以通过调整 maxConcurrentOperationCount 参数来控制负载。

#### 挑战

**挑战 1：停止计时器**

在本章关于 DispatchQueue 的部分中，你创建了一个可取消的计时器来为你的源发布者提供值。

设计两种不同的方法在 4 秒后停止计时器。提示：你需要使用 DispatchQueue.SchedulerTimeType.advanced(by:)。

找到解决方案了吗？将它们与 projects/challenge/challenge1/ final playground 中的进行比较：

1. 使用串行队列的调度程序协议 schedule(after:\_:) 方法来安排取消订阅的闭包的执行。
2. 使用 serialQueue 的普通 asyncAfter(_:_:) 方法（pre-Combine）来做同样的事情。

**挑战 2：发现优化**

在本章前面，你读到了一个有趣的问题：当你在连续的 receive(on:) 调用中使用相同的调度程序时，Combine 是在优化，还是 Dispatch 框架优化？

要找出答案，你需要转到挑战 2。你的挑战是设计一种方法来回答这个问题。这不是很复杂，但也不是微不足道的。

在 Dispatch 框架中，DispatchQueue 的初始化程序采用可选的目标参数。它使你可以指定要在其上执行代码的队列。换句话说，你创建的队列只是一个影子，而执行代码的真正队列是目标队列。

因此，尝试猜测是 Combine 还是 Dispatch 正在执行优化的想法是使用两个不同的队列，其中一个以另一个为目标。所以在 Dispatch 框架级别，代码都在同一个队列上执行，但是（希望）Combine 不会注意到。

因此，如果你执行此操作并看到在同一线程上接收到所有值，则很可能 Dispatch 正在为你执行优化。你为解决方案编写代码所采取的步骤是：

1. 创建第二个串行队列，以第一个为目标。
2. 为第二个串行队列添加 .receive(on:) 以及 .recordThread 步骤。

完整的解决方案可在 projects/challenge/challenge2 final playground 中找到。

#### 关键点

* Scheduler 定义了一项工作的执行上下文。
* Apple 的操作系统提供了丰富多样的工具来帮助你安排代码执行。
* 将这些 Scheduler 与 Scheduler 协议相结合，以帮助你在任何给定情况下选择最适合工作的 Scheduler。
* 每次使用 receive(on:) 时，发布者中的其他操作符都会在指定的调度程序上执行。也就是说，除非他们自己采用 Scheduler 参数！

#### 接下来去哪儿？

你学到了很多，你的大脑一定会被所有这些信息融化！下一章会涉及更多内容，因为它会教你创建自己的发布者和处理背压。确保你现在安排一个当之无愧的休息时间，并为下一章恢复精神！
