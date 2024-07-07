# 第 6 节：时间操纵操作符

### 第 6 节：时间操纵操作符

反应式编程背后的核心思想是随着时间的推移对异步事件流进行建模。在这方面，Combine 框架提供了一系列允许你处理时间的操作符。特别是序列如何随着时间的推移对值做出反应和转换。

管理值序列的时间维度简单直接，这是使用 Combine 这样的框架的一大好处。

#### 入门

要了解时间操作操作符，你将使用动画 Xcode Playground 进行练习，该 Playground 可视化数据如何随时间流动。本章附带一个 Playground，你可以在项目文件夹中找到。

Playground 分为几个页面。你将使用每个页面来练习一个或多个相关操作符。它还包括一些现成的类、函数和示例数据，可用于构建示例。

如果你将 Playground 设置为显示渲染标记，则在每个页面的底部都会有一个 Next 链接，你可以单击该链接转到下一页。

> 注意：要打开和关闭显示渲染标记，请从菜单中选择 Editor ▸ Show Rendered/Raw Markup。

你还可以从左侧边栏中的项目导航器甚至页面顶部的跳转栏中选择所需的页面。

查看 Xcode，你会在窗口的左上角看到侧边栏控件：

1. 确保左侧边栏按钮已切换，以便你可以看到 Playground 页面列表：

![image-20221004153833275](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004153833275.png)

1. 接下来，查看窗口的右上角。 你将看到视图控件：

![image-20221004153927958](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004153927958.png)

使用 Live View 显示 editor。 这将显示你在代码中构建的序列的实时视图。

1. 显示调试区域对于本章中的大多数示例都很重要。 使用窗口右下角的以下图标或使用键盘快捷键 Command-Shift-Y 切换调试区域：

![image-20221004154059399](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004154059399.png)

**Playground 不工作？**

有时 Xcode 可能会无法正常运行你的 Playground。 如果你遇到这种情况，请在 Xcode 中打开 Preferences 对话框并选择 Locations 选项卡。 单击 Derived Data 位置旁边的箭头，在下面的屏幕截图中用红色圆圈 1 表示。 它在 Finder 中显示 DerivedData 文件夹。

![image-20221004154205292](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004154205292.png)

退出 Xcode，将 DerivedData 文件夹移至垃圾箱，然后再次启动 Xcode。你的 Playground 现在应该可以正常工作了！

#### 移动时间(Shifting time)

最基本的时间操纵操作符会延迟来自发布者的值，以便你看到它们比它们实际出现的时间晚。

`delay(for:tolerance:scheduler:options)` 操作符对整个值序列进行时移：每次上游发布者发出一个值时，延迟都会将其保留一段时间，然后在你要求的延迟后在你的调度程序上发出它指定的。

打开 delay Playground 页面。你将看到的第一件事是，你不仅导入了 Combine 框架，还导入了 SwiftUI！这个动画游乐场是使用 SwiftUI 和 Combine 构建的。当你想探索时，最好仔细阅读 Sources 文件夹中的代码。

但首先要做的事情。首先定义几个常量，稍后你可以对其进行调整：

```
let valuesPerSecond = 1.0let delayInSeconds = 1.5
```

你将创建一个每秒发出一个值的发布者，然后将其延迟 1.5 秒并同时显示两个时间线以进行比较。 完成此页面上的代码后，你将能够调整常量并在时间轴中查看结果。

接下来，创建你需要的发布者：

```
// 1let sourcePublisher = PassthroughSubject<Date, Never>()​// 2let delayedPublisher = sourcePublisher.delay(for: .seconds(delayInSeconds), scheduler: DispatchQueue.main)​// 3let subscription = Timer    .publish(every: 1.0 / valuesPerSecond, on: .main, in: .common)    .autoconnect()    .subscribe(sourcePublisher)
```

分解这段代码：

1. sourcePublisher 是一个 PassthroughSubject，你将提供 Timer 发出的 Date。 值的类型在这里并不重要。 你只关心发布者发出值时以及延迟后显示值时的映像。
2. delayPublisher 将延迟来自 sourcePublisher 的值并将它们发送到主调度程序。你将后文了解所有关于调度器的知识。 现在，指定值必须在主队列中结束，准备好显示以使用它们。
3. 创建一个在主线程上每秒传递一个值的计时器。 立即使用 `autoconnect()` 启动它，并通过 sourcePublisher 提供它发出的值。

> 注意：这个 Timer 是 Foundation Timer类的组合扩展。 它需要一个 RunLoop 和 RunLoop.Mode，而不是你可能期望的 DispatchQueue。 你将在后文了解有 Timer 的信息。 此外，Timer 是可连接的发布者类的一部分。 这意味着它们需要在开始发出值之前被连接。你使用 autoconnect() 在第一次订阅时立即连接。

你将创建两个视图，这两个视图将让你可视化事件。 将此代码添加到你的 Playground：

```
// 4let sourceTimeline = TimelineView(title: "Emitted values (\(valuesPerSecond) per sec.):")// 5let delayedTimeline = TimelineView(title: "Delayed values (with a \(delayInSeconds)s delay):")// 6let view = VStack(spacing: 50) {    sourceTimeline    delayedTimeline}// 7PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))
```

在此代码中：

1. 创建一个将显示来自 Timer 的值的 TimelineView。 TimelineView 是一个 SwiftUI 视图，它的代码可以在 Sources/Views.swift 找到。
2. 创建另一个 TimelineView 来显示延迟值。
3. 创建一个简单的 SwiftUI 垂直堆栈，以将两个时间线一个接一个地显示。
4. 设置此 Playground 页面的实时视图。 `frame(widht:height:)` 修饰符只是用来帮助为 Xcode 的预览设置一个固定的框架。

在这个阶段，你会在屏幕上看到两条空的时间线。 你现在需要向他们提供来自每个发布者的值！ 将此最终代码添加到操场：

```
sourcePublisher.displayEvents(in: sourceTimeline)delayedPublisher.displayEvents(in: delayedTimeline)
```

在最后一段代码中，你将源发布者和延迟发布者连接到各自的时间线以显示事件。

保存这些源代码更改后，Xcode 将重新编译 Playground，查看实时视图窗格。

![image-20221004160336680](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004160336680.png)

你会看到两个时间线。 顶部时间线显示 Timer 发出的值。 底部时间线显示相同的延迟的值。 圆圈内的数字反映了发出的值的数量，而不是它们的实际内容。

#### 收集值(Collecting values)

在某些情况下，你可能需要以指定的时间间隔从发布者那里收集值。这是一种很有用的缓冲形式。 例如，当你想要在短时间内平均一组值并输出平均值时。

通过单击底部的 Next 链接或在 Project navigator 或跳转栏中选择它来切换到 Collect 页面。

与前面的示例一样，你将从一些常量开始：

```
let valuesPerSecond = 1.0let collectTimeStride = 4
```

当然，阅读这些常量可以让你了解这一切的发展方向。 立即创建你的发布者：

```
// 1let sourcePublisher = PassthroughSubject<Date, Never>()// 2let collectedPublisher = sourcePublisher    .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))
```

与前面的示例一样：

1. 设置源发布者——一个将传递 Timer 发出的值的 `PassthroughSubject`。
2. 创建一个 `collectedPublisher`，它使用 `collect` 操作符收集它在 `collectTimeStride` 跨步期间接收到的值。 操作符将这些值组作为数组发送到指定的调度程序：`DispatchQueue.main`。

> 注意：你可能还记得在第 3 节“转换操作符”中学习了 collect 操作符，在那里你使用了一个简单的数字来定义如何将值组合在一起。 collect 的这种重载接受了对值进行分组的策略； 在这种情况下，按时间。

你将再次使用 Timer 定期发出值，就像你对延迟操作符所做的那样：

```
let subscription = Timer    .publish(every: 1.0 / valuesPerSecond, on: .main, in: .common)    .autoconnect()    .subscribe(sourcePublisher)
```

接下来，像前面的示例一样创建时间线视图。 然后将 Playground 的实时视图设置为垂直堆栈，显示源时间轴和收集值的时间轴：

```
let sourceTimeline = TimelineView(title: "Emitted values:")let collectedTimeline = TimelineView(title: "Collected values (every \(collectTimeStride)s):")let view = VStack(spacing: 40) {    sourceTimeline    collectedTimeline}PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))
```

最后，将来自两个发布者的事件提供给时间线：

```
sourcePublisher.displayEvents(in: sourceTimeline)collectedPublisher.displayEvents(in:collectedTimeline)
```

现在看一下实时视图：

![image-20221004162942164](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004162942164.png)

你会看到值定期出现在 Emitted values 时间轴上。 在它下方，你会看到收集的值时间线每四秒显示一个值。 但它是什么？

你可能已经猜到，该值是最近四秒内收到的一组值。 你可以改进显示以查看其中的实际内容！ 返回到创建 `collectedPublisher` 对象的行。 在其下方添加对 flatMap 操作符的使用，如下所示：

```
let collectedPublisher = sourcePublisher    .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))    .flatMap { dates in dates.publisher }
```

你还记得你在第 3 节“转换操作符”中学到的 flatMap 吗？ 你在这里很好地利用了它：每次 collect 发出一组它收集的值时，flatMap 再次将其分解为单个值，但会立即一个接一个地发出。 它使用 Collection 的 publisher extension，将一系列值转换为发布者，立即将序列中的所有值作为单独的值发出。

现在，看看它对时间轴的影响：

![image-20221004163101662](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004163101662.png)

你现在可以看到，每四秒 collect 会发出一个在最后一个时间间隔内收集的值数组。

`collect(_:options:)` 操作符提供的第二个选项允许你定期收集值。它还允许你限制收集的值的数量。

留在同一个收集页面上，并在顶部的 collectTimeStride 正下方添加一个新常量：

```
let collectMaxCount = 2
```

接下来，在collectedPublisher之后创建一个新的发布者：

```
let collectedPublisher2 = sourcePublisher    .collect(.byTimeOrCount(DispatchQueue.main,                            .seconds(collectTimeStride),                            collectMaxCount))    .flatMap { dates in dates.publisher }
```

这一次，你使用 `.byTimeOrCount(Context, Context.SchedulerTimeType.Stride, Int)` 变体一次最多收集 collectMaxCount 值。

在collectedTimeline 之间为第二个collect 发布者添加一个新的 TimelineView 和 VStack：

```
let collectedTimeline2 = TimelineView(title: "Collected values (at most \(collectMaxCount) every \(collectTimeStride)s):")​let view = VStack(spacing: 40) {    sourceTimeline    collectedTimeline    collectedTimeline2}
```

最后，通过在 Playground 末尾添加以下内容，确保它在时间轴中显示它发出的事件：

```
collectedPublisher2.displayEvents(in:collectedTimeline2)
```

现在，让这个时间线运行一段时间，这样你就可以看到不同之处：

![image-20221004164508977](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221004164508977.png)

你可以在此处看到第二个时间线将其集合一次限制为两个值，这是 collectMaxCount 常量所要求的。

#### 推迟事件

在编码用户界面时，你经常处理文本字段。使用组合将文本字段内容连接到操作是一项常见任务。 例如，你可能希望发送一个搜索 URL 请求，该请求返回与在文本字段中键入的内容匹配的项目列表。

但是，当然，你不希望每次用户输入一个字母时都发送请求！ 你需要某种机制来帮助仅在用户完成一段时间的输入时才能够识别输入的文本。

Combine 提供了两个可以在这里为你提供帮助的操作符：防抖(Debounce)和节流(Throttle)。

**防抖(Debounce)**

切换到名为 Debounce 的 Playground 页面。 首先创建几个发布者：

```
// 1let subject = PassthroughSubject<String, Never>()// 2let debounced = subject    .debounce(for: .seconds(1.0), scheduler: DispatchQueue.main)// 3    .share()
```

在此代码中：

1. 创建一个发出字符串的发布者。
2. 使用 debounce 等待对象发射一秒钟。 然后，它将发送在该一秒间隔内发送的最后一个值（如果有）。 这具有允许每秒最多发送一个值的效果。
3. 你将多次订阅 debounced。 为了保证结果的一致性，你使用 share() 创建单个订阅点，这将同时向所有订阅者显示相同的结果。

> 注意：深入探讨 share() 操作符超出了本章的范围。请记住，当需要对发布者的单个订阅来向多个订阅者提供相同的结果时，它会很有帮助。 你将在后文“资源管理”中了解有关 share() 的更多信息。

对于接下来的几个示例，你将使用一组数据来模拟用户在文本字段中键入文本。 不要输入这个——它已经在 Sources/Data.swift 中为你实现了：

```
public let typingHelloWorld: [(TimeInterval, String)] = [    (0.0, "H"),    (0.1, "He"),    (0.2, "Hel"),    (0.3, "Hell"),    (0.5, "Hello"),    (0.6, "Hello "),    (2.0, "Hello W"),    (2.1, "Hello Wo"),    (2.2, "Hello Wor"),    (2.4, "Hello Worl"),    (2.5, "Hello World")]
```

模拟用户在 0.0 秒开始打字，在 0.6 秒后暂停，在 2.0 秒恢复打字。

> 注意：你将在 Debug 区域中看到的时间值可能会偏移 0.1 或 0.2 秒。 由于你将使用 DispatchQueue.asyncAfter() 在主队列上发出值，因此可以保证值之间的最小时间间隔，但可能不完全符合你的要求。

在 Playground 的 Debounce 页面中，创建两个时间线来可视化事件，并将它们连接到两个发布者：

```
let subjectTimeline = TimelineView(title: "Emitted values")let debouncedTimeline = TimelineView(title: "Debounced values")let view = VStack(spacing: 100) {    subjectTimeline    debouncedTimeline}PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))subject.displayEvents(in: subjectTimeline)debounced.displayEvents(in: debouncedTimeline)
```

你现在已经熟悉了这个 Playground 结构，你可以在屏幕上堆叠时间线并将它们连接到发布者以进行事件显示。

这一次，你将做更多的事情：打印每个发布者发出的值，以及它们出现的时间（自开始以来）。 这将帮助你弄清楚发生了什么。

添加此代码：

```
let subscription1 = subject    .sink { string in        print("+\(deltaTime)s: Subject emitted: \(string)")    }let subscription2 = debounced    .sink { string in        print("+\(deltaTime)s: Debounced emitted: \(string)")    }
```

每个订阅都会打印它收到的值以及自开始以来的时间。 deltaTime 是在 Sources/DeltaTime.swift 中定义的动态全局变量，它格式化自 Playground 开始运行以来的时间差。

现在你需要为你的主题提供数据。 这一次，你将使用模拟用户键入文本的预制数据源。 全部在 Sources/Data.swift 中定义，你可以随意修改。 看一看，你会发现它是一个模拟用户输入“Hello World”这个词。

将此代码添加到 Playground 页面的末尾：

```
subject.feed(with: typingHelloWorld)
```

feed(with:) 方法获取一个数据集并以预定义的时间间隔将数据发送到给定的主题。 用于模拟和模拟数据输入的便捷工具！ 当你为代码编写测试时，你可能希望保留这一点，因为你将编写测试，不是吗？

现在看看结果：

![image-20221006012419994](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006012419994.png)

你会在顶部看到发出的值，总共有 11 个字符串被推送到 sourcePublisher。 你可以看到用户在这两个词之间停顿了一下。 这是 debounce 发出捕获的内容的时间。

你可以通过查看显示打印的调试区域来确认这一点：

```
+0.0s: Subject emitted: H+0.1s: Subject emitted: He+0.2s: Subject emitted: Hel+0.3s: Subject emitted: Hell+0.5s: Subject emitted: Hello+0.6s: Subject emitted: Hello +1.6s: Debounced emitted: Hello +2.1s: Subject emitted: Hello W+2.1s: Subject emitted: Hello Wo+2.4s: Subject emitted: Hello Wor+2.4s: Subject emitted: Hello Worl+2.7s: Subject emitted: Hello World+3.7s: Debounced emitted: Hello World
```

用户在 0.6 秒时暂停并仅在 2.1 秒时恢复输入。它在 1.6 秒时，发出最新接收到的值。在 2.7 秒打字结束，在一秒后的 3.7 秒结束时发出最新接收到的值。

> 注意：要注意的一件事是发布者的完成事件。如果你的发布者在发出最后一个值之后立即完成，且在为防抖配置的时间之前，你将永远不会看到去抖动发布者中的最后一个值！

**节流(Throttle)**

debounce 允许的延迟模式非常有用，Combine 提供了一个相关的操作：`throttle(for:scheduler:latest:)`。 它非常接近防抖。

切换到 Playground 中的 Throttle 页面并开始编码。 首先，你需要一个常量，像往常一样：

```
let throttleDelay = 1.0​// 1let subject = PassthroughSubject<String, Never>()​// 2let throttled = subject    .throttle(for: .seconds(throttleDelay), scheduler: DispatchQueue.main, latest: false)// 3    .share()
```

分解这段代码：

1. 源发布者将发出字符串。
2. 由于你将 latest 设置为 false，因此你的 throttled subject 现在将仅在每个一秒间隔内发出从 subject 接收到的第一个值。
3. 和之前的操作符 debounce 一样，在此处添加 share() 操作符可以保证所有订阅者同时从节流主题看到相同的输出。

创建时间线以可视化事件，并将它们连接到两个发布者：

```
let subjectTimeline = TimelineView(title: "Emitted values")let throttledTimeline = TimelineView(title: "Throttled values")let view = VStack(spacing: 100) {    subjectTimeline    throttledTimeline}PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))subject.displayEvents(in: subjectTimeline)throttled.displayEvents(in: throttledTimeline)
```

现在你还想打印每个发布者发出的值，以更好地了解发生了什么。 添加此代码：

```
let subscription1 = subject    .sink { string in        print("+\(deltaTime)s: Subject emitted: \(string)")    }let subscription2 = throttled    .sink { string in        print("+\(deltaTime)s: Throttled emitted: \(string)")    }
```

同样，你将向源发布者提供模拟的“Hello World”用户输入。 将最后一行添加到你的 Playground 页面：

```
subject.feed(with: typingHelloWorld)
```

你现在可以在实时视图中看到正在发生的事情：

![image-20221006014413453](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006014413453.png)

它看起来与之前的 debounce 输出没有太大区别。首先，仔细观察两者。可以看到，throttle 发出的值在时间上略有不同。其次，为了更好地了解正在发生的事情，请查看调试控制台：

```
+0.0s: Subject emitted: H+0.0s: Throttled emitted: H+0.1s: Subject emitted: He+0.2s: Subject emitted: Hel+0.3s: Subject emitted: Hell+0.5s: Subject emitted: Hello+0.6s: Subject emitted: Hello +1.0s: Throttled emitted: He+2.2s: Subject emitted: Hello W+2.2s: Subject emitted: Hello Wo+2.2s: Subject emitted: Hello Wor+2.4s: Subject emitted: Hello Worl+2.7s: Subject emitted: Hello World+3.0s: Throttled emitted: Hello W
```

这明显不一样！你可以在这里看到一些有趣的事情：

* 当对象发出它的第一个值时，Throttle 立即转发它。然后，它开始限制输出。
* 在 1.0 秒时，Throttle 发出“He”。请记住，你要求它在一秒钟后向你发送第一个值（自最后一个值）。
* 在 2.2 秒时，键入恢复。可以看到此时，throttle 没有发出任何东西。这是因为没有从源发布者那里收到新值。
* 在 3.0 秒，输入完成后，Throttle 再次启动并再次输出第一个值，即 2.2 秒的值。

在那里你有防抖动和油门节流的根本区别：

* 防抖等待它接收到的值暂停，然后在指定的时间间隔后发出最新的值。
* 节流等待指定的时间间隔，然后发出它在该时间间隔内收到的第一个或最新的值。它不关心暂停。

要查看将 latest 更改为 true 时会发生什么，请将受限制的发布者的设置更改为以下内容：

```
let throttled = subject    .throttle(for: .seconds(throttleDelay), scheduler: DispatchQueue.main, latest: true)
```

现在，在调试区域观察结果输出：

```
+0.0s: Subject emitted: H+0.0s: Throttled emitted: H+0.1s: Subject emitted: He+0.2s: Subject emitted: Hel+0.3s: Subject emitted: Hell+0.5s: Subject emitted: Hello+0.6s: Subject emitted: Hello +1.0s: Throttled emitted: Hello+2.0s: Subject emitted: Hello W+2.3s: Subject emitted: Hello Wo+2.3s: Subject emitted: Hello Wor+2.6s: Subject emitted: Hello Worl+2.6s: Subject emitted: Hello World+3.0s: Throttled emitted: Hello World
```

节流输出恰好发生在 1.0 秒和 3.0 秒，时间窗口中的最新值而不是最早的值。 将此与前面示例中 debounce 的输出进行比较： ... +1.6s: Debounced emitted: Hello ... +3.7s: Debounced emitted: Hello World 输出是一样的，但是 debounce 是在暂停后延迟的。

#### 超时(Timing out)

在本次时间操纵操作符综述中，下一个是一个特殊的操作符：超时。其主要目的是在语义上区分实际计时和超时条件。 因此，当超时操作符触发时，它要么完成发布者，要么发出你指定的错误。 在这两种情况下，发布者都会终止。

切换到 Timeout Playground 页面。 首先添加以下代码：

```
let subject = PassthroughSubject<Void, Never>()​// 1let timedOutSubject = subject.timeout(.seconds(5), scheduler: DispatchQueue.main)
```

timedOutSubject 发布者将在 5 秒后超时，上游发布者不会发出任何值。 这种形式的超时强制发布者完成而没有任何失败。

你现在需要添加你的时间线，以及一个让你触发事件的按钮：

```
let timeline = TimelineView(title: "Button taps")​let view = VStack(spacing: 100) {    // 1    Button(action: { subject.send() }) {        Text("Press me within 5 seconds")    }    timeline}​PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))​timedOutSubject.displayEvents(in: timeline)
```

你在时间线上方添加一个按钮，按下该按钮会通过源主题发送一个新值。 每次按下按钮时，action 闭包都会执行。

> 注意：你是否注意到你正在使用发出 Void 值的 subject？ 是的，这是完全合法的！ 它表明发生了什么事。 但是，没有特别的价值可以携带。 因此，你只需使用 Void 作为值类型。 这是很常见的情况，Subject 有一个带有 send() 函数的扩展，如果输出类型为 Void，则该函数不接受任何参数。 这使你免于编写笨拙的 subject.send(()) 语句！

你的 Playground 页面现已完成。 看着它运行，什么都不做：超时将在五秒后触发并完成发布者。

现在，再次运行它。 这一次，以少于五秒的间隔持续按下按钮。 发布者永远不会完成，因为没有超时。

![image-20221006015755244](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006015755244.png)

当然，在很多情况下，简单的完成一个发布者并不是你想要的。 相反，你需要超时发布者发送失败消息，以便在这种情况下准确地采取措施。

转到 Playground 页面的顶部并定义你想要的错误类型：

```
enum TimeoutError: Error {    case timedOut}
```

接下来，修改 subject 的定义，将错误类型从 Never 更改为 TimeoutError。你的代码应如下所示：

```
let subject = PassthroughSubject<Void, TimeoutError>()
```

现在你需要将调用修改为 `.timeout`。此操作符的完整签名是 `timeout(_:scheduler:options:customError:)`。这是你提供自定义错误类型的机会！

将创建 timedOutSubject 的行修改为：

```
let timedOutSubject = subject.timeout(.seconds(5),                                      scheduler: DispatchQueue.main,                                      customError: { .timedOut })
```

现在，当你运行 Playground 并且五秒钟内没有按下按钮时，你可以看到 timedOutSubject 发出了失败。

![image-20221006020121560](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006020121560.png)

#### 测量时间(Measuring time)

为了完成时间操纵操作符的汇总，你将查看一个不操纵时间而只是测量时间的特定操作符。当你需要找出发布者发出的两个连续值之间经过的时间时，`measureInterval(using:)` 操作符是你的工具。

切换到 MeasureInterval Playground 页面。首先创建一对发布者：

```
let subject = PassthroughSubject<String, Never>()// 1let measureSubject = subject.measureInterval(using: DispatchQueue.main)
```

measureSubject 将在你指定的调度程序上发出测量值。

现在像往常一样，添加几个时间线：

```
let subjectTimeline = TimelineView(title: "Emitted values")let measureTimeline = TimelineView(title: "Measured values")let view = VStack(spacing: 100) {  subjectTimeline  measureTimeline}PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))subject.displayEvents(in: subjectTimeline)measureSubject.displayEvents(in: measureTimeline)
```

最后，有趣的部分来了。 打印出两个发布者发出的值：

```
let subscription1 = subject.sink {    print("+\(deltaTime)s: Subject emitted: \($0)")}let subscription2 = measureSubject.sink {    print("+\(deltaTime)s: Measure emitted: \($0)")}subject.feed(with: typingHelloWorld)
```

运行你的 Playground 并查看调试区域！ 在这里你可以看到 `measureInterval(using:)` 发出的度量值：

```
+0.0s: Subject emitted: H+0.0s: Measure emitted: Stride(magnitude: 16818353)+0.1s: Subject emitted: He+0.1s: Measure emitted: Stride(magnitude: 87377323)+0.2s: Subject emitted: Hel+0.2s: Measure emitted: Stride(magnitude: 111515697)+0.3s: Subject emitted: Hell+0.3s: Measure emitted: Stride(magnitude: 105128640)+0.5s: Subject emitted: Hello+0.5s: Measure emitted: Stride(magnitude: 228804831)+0.6s: Subject emitted: Hello +0.6s: Measure emitted: Stride(magnitude: 104349343)+2.2s: Subject emitted: Hello W+2.2s: Measure emitted: Stride(magnitude: 1533804859)+2.2s: Subject emitted: Hello Wo+2.2s: Measure emitted: Stride(magnitude: 154602)+2.4s: Subject emitted: Hello Wor+2.4s: Measure emitted: Stride(magnitude: 228888306)+2.4s: Subject emitted: Hello Worl+2.4s: Measure emitted: Stride(magnitude: 138241)+2.7s: Subject emitted: Hello World+2.7s: Measure emitted: Stride(magnitude: 333195273)
```

这些值有点令人费解。根据文档，measureInterval 发出的值的类型是“提供的调度程序的时间间隔”。 在 DispatchQueue 的情况下，TimeInterval 定义为“使用此类型的值创建的 DispatchTimeInterval，以纳秒为单位。”。

你在这里看到的是从源主题接收到的每个连续值之间的计数（以纳秒为单位）。 你现在可以修复显示以显示更具可读性的值。 修改从 measureSubject 打印值的代码，如下所示：

```
let subscription2 = measureSubject.sink {  print("+\(deltaTime)s: Measure emitted: \(Double($0.magnitude) / 1_000_000_000.0)")}
```

现在，你将在几秒钟内看到值。

但是如果你使用不同的调度器会发生什么呢？ 你可以使用 RunLoop 而不是 DispatchQueue 进行尝试！

> 注意：你将在后文深入探索 RunLoop 和 DispatchQueue 调度程序。

回到文件顶部，创建第二个使用 RunLoop 的主题：

```
let measureSubject2 = subject.measureInterval(using: RunLoop.main)
```

你无需费心连接新的时间线视图，因为有趣的是调试输出。将第三个订阅添加到你的代码中：

```
let subscription3 = measureSubject2.sink {    print("+\(deltaTime)s: Measure2 emitted: \($0)")}
```

现在，你还将看到 RunLoop 调度程序的输出，其幅度直接以秒表示：

```
+0.0s: Subject emitted: H+0.0s: Measure emitted: 0.016503769+0.0s: Measure2 emitted: Stride(magnitude: 0.015684008598327637)+0.1s: Subject emitted: He+0.1s: Measure emitted: 0.087991755+0.1s: Measure2 emitted: Stride(magnitude: 0.08793699741363525)+0.2s: Subject emitted: Hel+0.2s: Measure emitted: 0.115842671+0.2s: Measure2 emitted: Stride(magnitude: 0.11583995819091797)...
```

你用于测量的调度程序实际上取决于你的个人喜好。对所有事情都坚持使用 DispatchQueue 通常是一个好主意。 但这是你个人的选择！

#### 挑战

**挑战：Data**

如果时间允许，你可能想尝试一点挑战来充分利用这些新知识！

打开项目/挑战文件夹中的 Playground。你会看到一些代码在等着你：

* 发出整数的 subject。
* 向 subject 提供神秘数据的函数调用。

在这些部分之间，你的挑战是：

* 以 0.5 秒为单位对数据进行分组。
* 将分组数据转换为字符串。
* 如果提暂停时间超过 0.9 秒，请打印 👏 表情符号。提示：为此步骤创建第二个发布者并将其与订阅中的第一个发布者合并。
* 打印它。

> 注意：要将 Int 转换为 Character，你可以执行 Character(Unicode.Scalar(value)!) 之类的操作。

如果你正确地编码了这个挑战，你会在 Debug 区域看到一个打印的句子。它是什么？

#### 解决方案

你将在 challenge/Final.playground中找到该挑战的解决方案。

这是解决方案代码：

```
// 1let strings = subject// 2    .collect(.byTime(DispatchQueue.main, .seconds(0.5)))// 3    .map { array in        String(array.map { Character(Unicode.Scalar($0)!) })    }​// 4let spaces = subject.measureInterval(using: DispatchQueue.main)    .map { interval in        // 5        interval > 0.9 ? "👏" : ""    }​// 6let subscription = strings    .merge(with: spaces)// 7    .filter { !$0.isEmpty }    .sink {        // 8        print($0)    }
```

上述代码：

1. 创建从发出字符串的 subject 派生的第一个发布者。
2. 使用 collect() 使用 .byTime 策略以 0.5 秒的批次对数据进行分组。
3. 将每个整数值映射到 Unicode Scalar，然后映射到字符，然后使用 map 将整个整数值转换为字符串。
4. 创建从 subject 派生的第二个发布者，它测量每个字符之间的间隔。
5. 如果间隔大于0.9秒，将值映射到👏表情符号。否则，将其映射到一个空字符串。
6. 最终发布者是字符串和👏表情符号的合并。
7. 过滤掉空字符串以获得更好的显示。
8. 打印结果！

你的解决方案可能略有不同，这没关系。只要满足要求即可！

使用此解决方案运行 Playground 会将以下输出打印到控制台：

```
结合👏是👏凉爽的！
```

#### 关键点

在本章中，你从不同的角度看待时间。特别是，你了解到：

* Combine 对异步事件的处理扩展到操纵时间本身。
* 该框架具有允许你在很长一段时间内抽象工作的操作符，而不仅仅是处理离散事件。
* 可以使用 delay 操作符来移动时间。
* 你可以使用 collect 分块收集它们。
* 使用防抖和节流选择单个值。
* 不让时间用完是 timeout 的工作。
* 时间可以用 measureInterval 来测量。

#### 接下来去哪？

这需要学习很多东西。要将事件按正确的顺序排列，请转到下一章并了解序列操作符！
