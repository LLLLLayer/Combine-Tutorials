# 第 18 节：自定义发布者和处理背压

### 第 18 章：自定义发布者和处理背压

在你学习 Combine 的过程中，你可能会觉得框架中缺少很多操作符。如果你有其他反应式框架的经验，这可能尤其如此，这些反应式框架通常提供丰富的操作符生态系统，包括内置的和第三方的。Combine 允许你创建自己的发布者。本节将向你展示如何操作。

你将在本章中学习的第二个相关主题是背压(Backpressure)管理。你将了解什么是背压以及如何创建处理它的发布者。

#### 创建自己的发布者

创建自己的发布者的复杂性从“简单”到“相当复杂”不等。对于你实现的每个操作符，你将寻求最简单的实现形式来实现你的目标。在本节中，你将了解创建自己的发布者的三种不同方法：

* 在 Publisher 命名空间中使用简单的扩展方法。
* 在 Publishers 命名空间中使用产生值的 Subscription 实现一个类型。
* 与上面相同，但订阅会转换来自上游发布者的值。

> 注意：从技术上讲，可以在没有自定义 Subscription 的情况下创建自定义发布者。如果你这样做，你将失去应对订阅者 demand 的能力，这使你的发布者在 Combine 生态系统中是非法的。提前取消也可能成为一个问题。这不是推荐的方法，本节将教你如何以正确的方式编写发布者。

#### 发布者作为扩展方法

你的首要任务是通过重用现有的操作符来实现一个简单的操作符。

为此，你将添加一个新的 unwrap() 操作符，它解开可选值并忽略它们的 nil 值。这将是一个非常简单的练习，因为你可以重用现有的 compactMap(\_:) 操作符，它就是这样做的，尽管它需要你提供一个闭包。

使用新的 unwrap() 操作符将使你的代码更易于阅读，并且会使你正在做的事情变得非常清晰。读者甚至不必查看闭包的内容。

你将在 Publisher 命名空间中添加你的操作员，就像你添加所有其他操作符一样。

打开本章的 Playground，可以在 projects/Starter.playground 中找到，然后从 Project Navigator 打开其 Unwrap 操作页面。

然后，添加以下代码：

```
extension Publisher {  // 1  func unwrap<T>() -> Publishers.CompactMap<Self, T> where Output == Optional<T> {    // 2    compactMap { $0 }  }}
```

1. 将自定义操作符编写为方法，最复杂的部分是签名。
2. 实现很简单：只需在 self 上使用 compactMap(\_:)！

方法签名的制作可能令人难以置信。分解它来看看它是如何工作的：

```
func unwrap<T>()
```

你的第一步是使操作符通用，因为它的输出是上游发布者的可选类型包装的类型。

```
-> Publishers.CompactMap<Self, T>
```

该实现使用单个 compactMap(\_:)，因此返回类型由此派生。如果你查看 Publishers.CompactMap，你会发现它是一个泛型类型：public struct CompactMap\<Upstream, Output>。在实现自定义操作符时，Upstream 是 Self（你正在扩展的发布者），Output 是包装类型。

```
where Output == Optional<T> {
```

最后，你将操作符限制为 Optional 类型。你可以方便地编写它以将包装的类型 T 与你的方法的泛型类型匹配......等等！

> 注意：当开发更复杂的操作符作为方法时，例如使用操作符链时，签名会很快变得非常复杂。一个好的技巧是让你的操作符返回一个 AnyPublisher\<OutputType, FailureType>。在该方法中，你将返回一个以 eraseToAnyPublisher() 结尾的发布者，以对签名进行类型擦除。

**测试你的自定义操作符**

现在你可以测试你的新操作符了。在扩展名下方添加此代码：

```
let values: [Int?] = [1, 2, nil, 3, nil, 4]​values.publisher  .unwrap()  .sink {    print("Received value: \($0)")  }
```

运行 Playground，正如预期的那样，只有非 nil 值会打印到调试控制台：

```
Received value: 1Received value: 2Received value: 3Received value: 4
```

现在你已经了解了如何制作简单的操作符方法，是时候深入研究更丰富、更复杂的发布者了。你可以像这样对发布者进行分组：

* 充当“生产者”并直接自己生产价值的操作符。
* 发布者充当“转换器”，转换上游发布者产生的价值。

在本节，你将学习如何使用这两种方法，但你首先需要了解订阅发布者时发生的事情的详细信息。

#### 订阅机制

Subscription 是 Combine 的无名英雄：虽然你到处都能看到发布者，但它们大多是无生命的实体。当你订阅发布者时，它会实例化一个订阅，该订阅负责接收订阅者的需求并产生事件（例如，值和完成）。

以下是订阅生命周期的详细信息：

![image-20221023203025770](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023203025770.png)

1. 订阅者订阅发布者。
2. 发布者创建一个订阅，然后将其交给订阅者（调用 receive(subscription:)）。
3. 订阅者通过向订阅发送所需数量的值（调用订阅的 request(\_:) 方法）从订阅中请求值。
4. Subscription 开始工作并开始发出值。它将它们一一发送给订阅者（调用订阅者的 receive(\_:) 方法）。
5. 收到值后，订阅者返回一个新的 Subscribers.Demand，它会添加到先前的总需求中。
6. Subscription 会一直发送值，直到发送的值数量达到请求的总数。

如果 Subscription 发送的值与订阅者请求的值一样多，它应该在发送更多值之前等待新的请求请求。你可以绕过此机制并继续发送值，但这会破坏订阅者和订阅之间的合同，并可能导致基于 Apple 定义的发布者树中出现未定义的行为。

最后，如果出现错误或订阅的值源完成，订阅会调用订阅者的 receive(completion:) 方法。

#### 发布者发出值

在第 11 节“定时器”中，你了解了 Timer.publish()，但发现将调度队列用于定时器有点不方便。为什么不基于 Dispatch 的 DispatchSourceTimer 开发自己的计时器呢？

你将这样做，同时检查订阅机制的详细信息。

要开始使用，请打开 Playground 的 DispatchTimer 发布者页面。

你将首先定义一个配置结构，这将使订阅者及其订阅之间共享计时器配置变得容易。将此代码添加到 Playground：

```
struct DispatchTimerConfiguration {  // 1  let queue: DispatchQueue?  // 2  let interval: DispatchTimeInterval  // 3  let leeway: DispatchTimeInterval  // 4  let times: Subscribers.Demand}
```

如果你曾经使用过 DispatchSourceTimer，那么其中一些属性对你来说应该很熟悉：

1. 你希望你的计时器能够在某个队列上触发，但如果你不在乎，使队列成为可选的。在这种情况下，计时器将在其选择的队列上触发。
2. 计时器触发的时间间隔，从订阅时间开始。
3. 系统可以延迟传递计时器事件的截止日期之后的最大时间量。
4. 你想要接收的计时器事件数。由于你正在制作自己的计时器，因此请使其灵活并能够在完成之前交付有限数量的事件！

**添加 DispatchTimer 发布者**

你现在可以开始创建 DispatchTimer 发布者。 这将很简单，因为所有工作都发生在 Subscription 内！

在你的配置下方添加此代码：

```
extension Publishers {  struct DispatchTimer: Publisher {    // 5    typealias Output = DispatchTime    typealias Failure = Never    // 6    let configuration: DispatchTimerConfiguration        init(configuration: DispatchTimerConfiguration) {      self.configuration = configuration    }  }}
```

1. 你的计时器将当前时间作为 DispatchTime 值发出。 当然，它永远不会失败，所以发布者的 Failure 类型是 Never。
2. 保留给定配置的副本。 你现在不使用它，但是当你收到订阅者时会需要它。

> 注意：你将在编写代码时开始看到编译器错误。 请放心，在你完成实施要求时，你将解决这些问题。

现在，通过将此代码添加到 DispatchTimer 定义中，在你的初始化程序下方，实现 Publisher 协议所需的 receive(subscriber:) 方法：

```
// 7func receive<S: Subscriber>(subscriber: S)  where Failure == S.Failure,        Output == S.Input {  // 8  let subscription = DispatchTimerSubscription(    subscriber: subscriber,    configuration: configuration  )  // 9  subscriber.receive(subscription: subscription)}
```

1. 功能是通用的；它需要一个编译时特化来匹配订阅者类型。
2. 大部分动作将发生在你将在短时间内定义的 DispatchTimerSubscription 中。
3. 正如你在第 2 节“发布者和订阅者”中所了解的，订阅者会收到一个订阅，然后它可以向该订阅发送值请求。

这就是发布者的全部内容！真正的工作将在 Subscription 内部进行。

**建立你的 Subscription**

Subscription 的作用是：

* 接受订阅者的初始 Demend。
* 按需生成定时器事件。
* 每次订阅者收到一个值并返回一个需求时，都添加到需求计数。
* 确保它不会提供比配置中要求的更多的值。

这可能听起来像很多代码，但它并不复杂！

开始在 Publishers 的扩展下方定义订阅：

```
private final class DispatchTimerSubscription  <S: Subscriber>: Subscription where S.Input == DispatchTime {}
```

签名本身提供了很多信息：

* 此 Subscription 在外部不可见，只能通过订阅协议，因此你将其设为私有。
* 这是一个类，因为你想通过引用传递它。 然后订阅者可以将它添加到 Cancellable 集合中，但也可以保留它并独立调用 cancel()。
* 它用于输入值类型为 DispatchTime 的订阅者，这是订阅发出的内容。

**将所需属性添加到你的 Subscription**

现在将这些属性添加到订阅类的定义中：

```
// 10let configuration: DispatchTimerConfiguration// 11var times: Subscribers.Demand// 12var requested: Subscribers.Demand = .none// 13var source: DispatchSourceTimer? = nil// 14var subscriber: S?
```

此代码包含：

1. 订阅者的配置。
2. 计时器将触发的最大次数，你从配置中复制。你将使用它作为每次发送值时递减的计数器。
3. 当前 Demend；例如，订阅者请求的值的数量 - 每次发送值时都会减少它。
4. 将生成定时器事件的内部 DispatchSourceTimer。
5. 订阅者。这清楚地表明，只要订阅没有完成、失败或取消，Subscription 就有责任保留订阅者。

> 注意：最后一点对于理解 Combine 中的所有权机制至关重要。 Subscription 是订阅者和发布者之间的链接。它可以让订阅者——例如，一个持有闭包的对象，如 AnySubscriber 或 sink——在必要时保持存在。这就解释了为什么，如果你不保留订阅，你的订阅者似乎永远不会收到值：一旦订阅被解除分配，一切都会停止。内部实现当然可能会根据你正在编码的发布者的具体情况而有所不同。

**初始化和取消你的 Subscription**

现在，在 DispatchTimerSubscription 定义中添加一个初始化器：

```
init(subscriber: S,     configuration: DispatchTimerConfiguration) {  self.configuration = configuration  self.subscriber = subscriber  self.times = configuration.times}
```

这很简单。初始化程序将时间设置为发布者应接收计时器事件的最大次数，如配置指定的那样。每次发布者发出事件时，此计数器都会递减。当它达到零时，计时器以完成的事件结束。

现在，实现 cancel()，Subscription 必须提供的必需方法：

```
func cancel() {  source = nil  subscriber = nil}
```

将 DispatchSourceTimer 设置为 nil 足以阻止它运行。将订阅者属性设置为 nil 会将其从订阅的范围中释放出来。不要忘记在你自己的订阅中执行此操作，以确保你不会在内存中保留不再需要的对象。

你现在可以开始编写 Subscription 的核心代码：request(\_:)。

**让你的 Subscription 请求值**

你还记得你在第 2 节“发布者和订阅者”中学到的内容吗？一旦订阅者通过订阅发布者获得订阅，它必须从订阅中请求值。

这就是所有魔法发生的地方。要实现它，请将此方法添加到类中，在取消方法上方：

```
// 15func request(_ demand: Subscribers.Demand) {  // 16  guard times > .none else {    // 17    subscriber?.receive(completion: .finished)    return  }}
```

1. 这个必需的方法接收来自订阅者的请求。
2. Demand 是累积的：它们加起来形成订阅者请求的值的总数。验证你是否已经向订阅者发送了足够的值，如配置中指定的那样。也就是说，如果你发送了最大数量的预期值，则与发布者收到的要求无关。
3. 如果是这种情况，你可以通知订阅者发布者已完成发送值。

在 guard 语句之后添加以下代码来继续实现此方法：

```
// 18requested += demand​// 19if source == nil, requested > .none {​}
```

1. 通过添加新需求来增加请求值的总数。
2. 检查定时器是否已经存在。如果没有，并且请求的值存在，那么是时候启动它了。

**配置你的计时器**

将此代码添加到最后一个的 if：

```
// 20let source = DispatchSource.makeTimerSource(queue: configuration.queue)// 21source.schedule(deadline: .now() + configuration.interval,                repeating: configuration.interval,                leeway: configuration.leeway)
```

1. 从你配置的队列中创建 DispatchSourceTimer。
2. 安排计时器在每 configuration.interval 秒后触发。

一旦计时器启动，你将永远不会停止它，即使你不使用它向订阅者发出事件。它会一直运行，直到订阅者取消订阅——或者你取消分配订阅。

你现在已准备好编写计时器的核心，它向订阅者发出事件。仍然在 if 正文中，添加以下代码：

```
// 22source.setEventHandler { [weak self] in  // 23  guard let self = self,        self.requested > .none else { return }  // 24  self.requested -= .max(1)  self.times -= .max(1)  // 25  _ = self.subscriber?.receive(.now())  // 26  if self.times == .none {    self.subscriber?.receive(completion: .finished)  }}
```

1. 为你的计时器设置事件处理程序。 这是计时器每次触发时调用的简单闭包。 确保保持对 self 的弱引用，否则订阅将永远不会解除分配。
2. 验证当前是否有请求的值——发布者可以在没有当前需求的情况下暂停，正如你将在本章后面了解背压时看到的那样。
3. 减少两个计数器，因为你要发出一个值。
4. 向订阅者发送一个值。
5. 如果要发送的值的总数达到配置指定的最大值，你可以认为发布者已完成并发出完成事件！

**激活你的计时器**

现在你已经配置了源计时器，存储对它的引用并通过在 setEventHandler 之后添加以下代码来激活它：

```
self.source = sourcesource.activate()
```

那是很多步骤，并且很容易在此过程中不经意地放错一些代码。这段代码应该已经清除了操场上的所有错误。如果没有，你可以通过查看上述步骤或将你的代码与项目/Final.playground 中的 Playground 完成版本进行比较来仔细检查你的工作。

最后一步：在 DispatchTimerSubscription 的整个定义之后添加此扩展，以定义一个操作符，以便轻松链接此发布者：

```
extension Publishers {  static func timer(queue: DispatchQueue? = nil,                    interval: DispatchTimeInterval,                    leeway: DispatchTimeInterval = .nanoseconds(0),                    times: Subscribers.Demand = .unlimited)                    -> Publishers.DispatchTimer {    return Publishers.DispatchTimer(      configuration: .init(queue: queue,                           interval: interval,                           leeway: leeway,                           times: times)                      )  }}
```

**测试你的计时器**

你现在可以测试你的新计时器了！

新计时器操作符的大多数参数，除了间隔，都有一个默认值，以便在常见用例中更容易使用。这些默认值创建了一个永不停止的计时器，具有最小的回旋余地，并且不指定要在哪个队列上发出值。

在扩展之后添加此代码以测试你的计时器：

```
// 27var logger = TimeLogger(sinceOrigin: true)// 28let publisher = Publishers.timer(interval: .seconds(1),                                 times: .max(6))// 29let subscription = publisher.sink { time in  print("Timer emits: \(time)", to: &logger)}
```

1. 这个 Playground 定义了一个类 TimeLogger，它与你在第 10 节“调试”中学习创建的类非常相似。唯一的区别是这个可以显示两个连续值之间的时间差，或者自创建计时器以来经过的时间。在这里，你要显示自开始记录以来的时间。
2. 你的计时器发布者将准确触发六次，每秒一次。
3. 记录你通过 TimeLogger 收到的每个值。

运行 Playground，你会看到这个漂亮的输出——或类似的东西，因为时间会略有不同：

```
+1.02668s: Timer emits: DispatchTime(rawValue: 183177446790083)+2.02508s: Timer emits: DispatchTime(rawValue: 183178445856469)+3.02603s: Timer emits: DispatchTime(rawValue: 183179446800230)+4.02509s: Timer emits: DispatchTime(rawValue: 183180445857620)+5.02613s: Timer emits: DispatchTime(rawValue: 183181446885030)+6.02617s: Timer emits: DispatchTime(rawValue: 183182446908654)
```

设置时会有轻微的偏移——而且 Playgrounds 也可能会增加一些延迟——然后计时器每秒触发一次，六次。

你还可以测试取消计时器，例如，几秒钟后。添加此代码以执行此操作：

```
DispatchQueue.main.asyncAfter(deadline: .now() + 3.5) {  subscription.cancel()}
```

再次运行 Playground。这一次，你只看到三个值。看起来你的计时器工作得很好！

尽管它在 Combine API 中几乎看不到，但正如你刚刚发现的那样，Subscription 完成了大部分工作。

享受你的成功。接下来你将进行另一次深潜！

#### 发布者改变值

你现在可以开发自己的操作符，甚至是相当复杂的操作符。接下来要学习的是如何创建 Subscription 来转换来自上游发布者的值。这是完全控制发布者、Subscription 的关键。

在第 9 节“网络”中，你了解了共享 Subscription 有多么有用。当底层发布者执行重要工作时，例如从网络请求数据，你希望与多个订阅者共享结果。但是，你希望避免多次发出相同的请求来检索相同的数据。

如果你不需要再次执行工作，那么将结果重播给未来的订阅者也是有益的。

为什么不尝试实现 shareReplay()，它可以完全满足你的需求？这将是一项有趣的任务！要编写此操作符，你将创建一个执行以下操作的发布者：

* 在第一个订阅者时订阅上游发布者。
* 将最后 N 个值重播给每个新订阅者。
* 中继完成事件，如果事先发出了一个。

请注意，实施起来绝非易事，但你肯定已经掌握了！你将逐步完成，最后，你将拥有一个 shareReplay()，你可以在未来的 Combine 驱动项目中使用它。

在 Playground 中打开 ShareReplay 操作员页面以开始使用。

**实现 ShareReplay 操作符**

要实现 shareReplay()，你需要：

1. 符合 Subscription 协议的类型。这是每个订阅者将收到的 Subscription。为确保你能够应对每个订阅者的 demand 和取消，每个订阅者都将收到单独的 Subscription。
2. 符合 Publisher 协议的类型。你将把它实现为一个类，因为所有订阅者都希望共享同一个实例。

首先添加此代码以创建你的 Subscription 类：

```
// 1fileprivate final class ShareReplaySubscription<Output, Failure: Error>: Subscription {  // 2  let capacity: Int  // 3  var subscriber: AnySubscriber<Output,Failure>? = nil  // 4  var demand: Subscribers.Demand = .none  // 5  var buffer: [Output]  // 6  var completion: Subscribers.Completion<Failure>? = nil}
```

从一开始：

1. 你使用通用类而不是结构来实现订阅：发布者和订阅者都需要访问和改变订阅。
2. 重播缓冲区的最大容量将是你在初始化期间设置的常数。
3. 在订阅期间保留对订阅者的引用。使用类型擦除的 AnySubscriber 可以使你免于与类型系统作斗争。 :]
4. 跟踪发布者从订阅者那里收到的累积 demand，以便你可以准确地交付请求数量的值。
5. 将挂起的值存储在缓冲区中，直到它们被传递给订阅者或被丢弃。
6. 这样可以保留潜在的完成事件，以便在新订阅者开始请求值时立即将其交付给他们。

> 注意：如果你觉得没有必要保留完成事件，而你将立即交付它，请放心，情况并非如此。订阅者应该首先接收它的订阅，然后在它准备好接受值时立即接收一个完成事件——如果之前发出过一个。它发出的第一个 request(\_:) 发出信号。发布者不知道这个请求什么时候发生，所以它只是将完成交给订阅，以便在正确的时间交付它。

**初始化你的订阅**

接下来，将初始化程序添加到订阅定义中：

```
init<S>(subscriber: S,        replay: [Output],        capacity: Int,        completion: Subscribers.Completion<Failure>?)        where S: Subscriber,              Failure == S.Failure,              Output == S.Input {  // 7  self.subscriber = AnySubscriber(subscriber)  // 8  self.buffer = replay  self.capacity = capacity  self.completion = completion}
```

此初始化程序从上游发布者接收多个值并将它们设置在此订阅实例上。具体来说，它：

1. 存储订阅者的类型擦除版本。
2. 存储上游发布者的当前缓冲区、最大容量和完成事件（如果已发出）。

**向订阅者发送完成事件和未完成的值**

你需要一种将完成事件中继给订阅者的方法。将以下内容添加到订阅类以满足该需求：

```
private func complete(with completion: Subscribers.Completion<Failure>) {  // 9  guard let subscriber = subscriber else { return }  self.subscriber = nil  // 10  self.completion = nil  self.buffer.removeAll()  // 11  subscriber.receive(completion: completion)}
```

此私有方法执行以下操作：

1. 在方法执行期间保留订阅者，但在类中将其设置为 nil。 这种防御措施确保用户在完成时可能错误地发出的任何呼叫都将被忽略。
2. 通过将完成设置为 nil 来确保只发送一次完成，然后清空缓冲区。
3. 将完成事件中继给订阅者。

你还需要一种可以向订阅者发出值的方法。 添加此方法以根据需要发出值：

```
private func emitAsNeeded() {  guard let subscriber = subscriber else { return }  // 12  while self.demand > .none && !buffer.isEmpty {    // 13    self.demand -= .max(1)    // 14    let nextDemand = subscriber.receive(buffer.removeFirst())    // 15    if nextDemand != .none {      self.demand += nextDemand    }  }  // 16  if let completion = completion {    complete(with: completion)  }}
```

首先，此方法确保有订阅者。 如果有，该方法将：

1. 仅当缓冲区中有一些值并且有未完成的 Demand 时才发出值。
2. 将未完成的 Demand 减一。
3. 向订阅者发送第一个值，并收到新的 Demand。
4. 将新 Demand 添加到未完成的总 Demand 中，但前提是它不是 .none。 否则，你将遇到崩溃，因为 Combine 不会将 Subscribers.Demand.none 视为零，并且添加或减去 .none 将触发异常。
5. 如果完成事件未决，请立即发送。

事情正在形成！ 现在，实现 Subscription 最重要的要求：

```
func request(_ demand: Subscribers.Demand) {  if demand != .none {    self.demand += demand  }  emitAsNeeded()}
```

那是一件容易的事。 请记住检查 .none 以避免崩溃 - 并密切关注未来版本的 Combine 修复此问题 - 然后继续发射。

> 注意：即使需求是 .none ，调用 emitAsNeeded() 也能保证你正确地传递已经发生的完成事件。

**取消订阅**

取消订阅更加容易。 添加此代码：

```
func cancel() {  complete(with: .finished)}
```

与订阅者一样，你需要实现接受值的方法和完成事件。 首先添加此方法以接受值：

```
func receive(_ input: Output) {  guard subscriber != nil else { return }  // 17  buffer.append(input)  if buffer.count > capacity {    // 18    buffer.removeFirst()  }  // 19  emitAsNeeded()}
```

确保有订阅者后，此方法将：

1. 将值添加到未完成的缓冲区。你可以针对最常见的情况进行优化，例如无限需求，但现在这将完美地完成工作。
2. 确保缓冲的值不要超过请求的容量。你以先进先出的方式处理此问题——当一个已经满的缓冲区接收每个新值时，当前的第一个值将被删除。
3. 将结果交付给订阅者。

**结束你的订阅**

现在，添加以下方法来接受完成事件，你的订阅类将完成：

```
func receive(completion: Subscribers.Completion<Failure>) {  guard let subscriber = subscriber else { return }  self.subscriber = nil  self.buffer.removeAll()  subscriber.receive(completion: completion)}
```

此方法删除订阅者，清空缓冲区——因为这只是良好的内存管理——并将完成发送到下游。

你已完成订阅！这不是很有趣吗？现在，是时候为发布者编写代码了。

**编码你的发布者**

Publishers 通常是 Publishers 命名空间中的值类型（结构）。有时将发布者实现为类如 Publishers.Multicast 或 Publishers.Share 是有意义的。对于这个发布者，你需要一个类，类似于 share()。不过，这是规则的例外，因为大多数情况下你会使用结构。

首先添加此代码以在订阅后定义你的发布者类：

```
extension Publishers {  // 20  final class ShareReplay<Upstream: Publisher>: Publisher {    // 21    typealias Output = Upstream.Output    typealias Failure = Upstream.Failure  }}
```

你希望多个订阅者能够共享此操作符的单个实例，因此你使用类而不是结构。它也是通用的，上游发布者的最终类型作为参数。

这个新的发布者不会改变上游发布者的输出或失败类型——它只是使用上游的类型。

**添加发布者所需的属性**

现在，将你的发布者需要的属性添加到 ShareReplay 的定义中：

```
// 22private let lock = NSRecursiveLock()// 23private let upstream: Upstream// 24private let capacity: Int// 25private var replay = [Output]()// 26private var subscriptions = [ShareReplaySubscription<Output, Failure>]()// 27private var completion: Subscribers.Completion<Failure>? = nil
```

这段代码的作用：

1. 因为你将同时提供多个订阅者，所以你需要一个锁来保证对可变变量的独占访问。
2. 保留对上游发布者的引用。你将在订阅生命周期的各个阶段需要它。
3. 你可以在初始化期间指定重放缓冲区的最大记录容量。
4. 当然，你还需要存储你记录的值。
5. 你提供多个订阅者，因此你需要将它们留在身边以通知他们事件。每个订阅者都从一个专用的 ShareReplaySubscription 获取其值——你将在短时间内编写此代码。
6. 操作符即使在完成后也可以重播值，因此你需要记住上游发布者是否完成。

看起来，还有一些代码要写！最后，你会发现它并没有那么多，但还有一些工作要做，例如使用适当的锁定，以便你的操作员在所有条件下都能顺利运行。

**初始化值并将其转发给你的发布者**

首先，将必要的初始化程序添加到你的 ShareReplay 发布者：

```
init(upstream: Upstream, capacity: Int) {  self.upstream = upstream  self.capacity = capacity}
```

这里没什么特别的，只是存储上游发布者和容量。接下来，你将添加几个方法来帮助将代码拆分为更小的块。

添加将来自上游的传入值中继到订阅者的方法：

```
private func relay(_ value: Output) {  // 28  lock.lock()  defer { lock.unlock() }  // 29  guard completion == nil else { return }  // 30  replay.append(value)  if replay.count > capacity {    replay.removeFirst()  }  // 31  subscriptions.forEach {    $0.receive(value)  }}
```

此代码执行以下操作：

1. 由于多个订阅者共享此发布者，因此你必须使用锁保护对可变变量的访问。在这里使用 defer 并不是绝对必要的，但这是一种很好的做法，以防你以后修改方法，添加提前返回语句并忘记解锁你的锁。
2. 仅在上游尚未完成时才中继值。
3. 将值添加到滚动缓冲区并仅保留最新的容量值。这些是重播给新订阅者的内容。
4. 将缓冲的值中继到每个连接的订阅者。

**完成后通知你的发布者**

其次，添加这个方法来处理完成事件：

```
private func complete(_ completion: Subscribers.Completion<Failure>) {  lock.lock()  defer { lock.unlock() }  // 32  self.completion = completion  // 33  subscriptions.forEach {    $0.receive(completion: completion)  }}
```

使用此代码：

1. 为未来的订阅者保存完成事件。
2. 将其转发给每个连接的订阅者。

你现在已准备好开始编写每个发布者必须实现的接收方法。此方法将接收订阅者。它的职责是创建一个新订阅，然后将其交给订阅者。

添加此代码以开始定义此方法：

```
func receive<S: Subscriber>(subscriber: S)  where Failure == S.Failure,        Output == S.Input {  lock.lock()  defer { lock.unlock() }}
```

receive(subscriber:) 的这个标准原型指定订阅者，无论它是什么，都必须具有与发布者的输出和失败类型匹配的输入和失败类型。 还记得第 2 章“发布者和订阅者”中的内容吗？

**创建你的订阅**

接下来，将此代码添加到创建订阅并将其交给订阅者的方法中：

```
// 34let subscription = ShareReplaySubscription(  subscriber: subscriber,  replay: replay,  capacity: capacity,  completion: completion)// 35subscriptions.append(subscription)// 36subscriber.receive(subscription: subscription)
```

1. 新 Subscription 引用订阅者并接收当前重放缓冲区、容量和任何未完成的完成事件。
2. 你保留订阅以将未来的事件传递给它。
3. 你将订阅发送给订阅者，订阅者可能（现在或以后）开始请求值。

**订阅上游发布者并处理其输入**

你现在已准备好订阅上游发布者。 你只需要做一次：当你收到第一个订阅者时。

将此代码添加到 receive(subscriber:) - 请注意，你故意不包括结束 } 因为还有更多代码要添加：

```
// 37guard subscriptions.count == 1 else { return }let sink = AnySubscriber(  // 38  receiveSubscription: { subscription in    subscription.request(.unlimited)  },  // 39  receiveValue: { [weak self] (value: Output) -> Subscribers.Demand in    self?.relay(value)    return .none  },  // 40  receiveCompletion: { [weak self] in    self?.complete($0)  })
```

使用此代码你：

1. 只向上游发布者订阅一次。
2. 使用方便的 AnySubscriber 类，它接受闭包，并在订阅时立即请求 .unlimited 值以让发布者运行完成。
3. 将你收到的值转发给下游订阅者。
4. 使用你从上游获得的完成事件来完成你的发布者。

> 注意：你最初可以请求 .max(self.capacity) 并仅接收它，但请记住，Combine 是 Demand 驱动的！如果你请求的值没有发布者能够产生的那么多值，那么你可能永远不会收到完成事件！

为避免保留循环，你只保留对 self 的弱引用。

你快完成了！现在，你需要做的就是将 AnySubscriber 订阅到上游发布者。

通过添加以下代码完成此方法的定义：

```
upstream.subscribe(sink)
```

再一次，Playground 上的所有错误现在都应该清楚了。请记住，你可以通过将其与项目/最终版中的 Playground 完成版本进行比较来仔细检查你的工作。

**添加便利操作符**

你的发布者已完成！当然，你还想要一件事：一个便利的操作符，帮助将这个新发布者与其他发布者联系起来。

将其作为扩展添加到 Playground 末尾的 Publishers 命名空间：

```
extension Publisher {  func shareReplay(capacity: Int = .max)    -> Publishers.ShareReplay<Self> {    return Publishers.ShareReplay(upstream: self,                                  capacity: capacity)  }}
```

你现在拥有一个功能齐全的 shareReplay(capacity:) 操作符。这是很多代码，现在是时候尝试一下了！

**测试你的订阅**

将此代码添加到 Playground 的末尾以测试你的新操作符：

```
// 41var logger = TimeLogger(sinceOrigin: true)// 42let subject = PassthroughSubject<Int,Never>()// 43let publisher = subject.shareReplay(capacity: 2)// 44subject.send(0)
```

下面是这段代码的作用：

1. 使用这个 Playground 中定义的方便的 TimeLogger 对象。
2. 为了模拟在不同时间发送值，你使用了一个 Subject。
3. 分享 Subject 并仅重播最后两个值。
4. 通过 Subject 发送一个初始值。 没有订阅者连接到共享发布者，因此你应该看不到任何输出。

现在，创建你的第一个订阅并发送更多值：

```
let subscription1 = publisher.sink(  receiveCompletion: {    print("subscription1 completed: \($0)", to: &logger)  },  receiveValue: {    print("subscription1 received \($0)", to: &logger)  })subject.send(1)subject.send(2)subject.send(3)
```

接下来，创建第二个订阅并发送更多值和完成事件：

```
let subscription2 = publisher.sink(  receiveCompletion: {    print("subscription2 completed: \($0)", to: &logger)  },  receiveValue: {    print("subscription2 received \($0)", to: &logger)  })​subject.send(4)subject.send(5)subject.send(completion: .finished)
```

这两个订阅显示他们收到的每个事件，以及自开始以来经过的时间。

添加一个稍有延迟的订阅，以确保它在发布者完成后发生：

```
var subscription3: Cancellable? = nil​DispatchQueue.main.asyncAfter(deadline: .now() + 1) {  print("Subscribing to shareReplay after upstream completed")  subscription3 = publisher.sink(    receiveCompletion: {      print("subscription3 completed: \($0)", to: &logger)    },    receiveValue: {      print("subscription3 received \($0)", to: &logger)    }  )}
```

请记住，订阅在解除分配时终止，因此你需要使用一个变量来保留延迟的订阅。 一秒钟的延迟演示了发布者在未来如何重播数据。 你准备好测试了！ 运行 Playground 以在调试控制台中查看以下结果：

```
+0.02967s: subscription1 received 1+0.03092s: subscription1 received 2+0.03189s: subscription1 received 3+0.03309s: subscription2 received 2+0.03317s: subscription2 received 3+0.03371s: subscription1 received 4+0.03401s: subscription2 received 4+0.03515s: subscription1 received 5+0.03548s: subscription2 received 5+0.03716s: subscription1 completed: finished+0.03746s: subscription2 completed: finishedSubscribing to shareReplay after upstream completed+1.12007s: subscription3 received 4+1.12015s: subscription3 received 5+1.12057s: subscription3 completed: finished
```

你的新操作符运行良好：

* 0 值永远不会出现在日志中，因为它是在第一个订阅者订阅共享发布者之前发出的。
* 每个值都会传播给当前和未来的订阅者。
* 你在三个值通过主题后创建了订阅 2，因此它只看到最后两个（值 2 和 3）
* 你在主题完成后创建了订阅 3，但订阅仍收到主题发出的最后两个值。
* 完成事件会正确传播，即使订阅者是在共享发布者完成之后出现的。

**验证你的订阅**

极好的！这完全符合你的要求。或者是吗？如何验证发布者只被订阅一次？当然，通过使用 print(\_:) 操作符！你可以通过在 shareReplay 之前插入它来尝试。

找到这段代码：

```
let publisher = subject.shareReplay(capacity: 2)
```

并将其更改为：

```
let publisher = subject  .print("shareReplay")  .shareReplay(capacity: 2)
```

再次运行 Playground，它将产生以下输出：

```
shareReplay: receive subscription: (PassthroughSubject)shareReplay: request unlimitedshareReplay: receive value: (1)+0.03004s: subscription1 received 1shareReplay: receive value: (2)+0.03146s: subscription1 received 2shareReplay: receive value: (3)+0.03239s: subscription1 received 3+0.03364s: subscription2 received 2+0.03374s: subscription2 received 3shareReplay: receive value: (4)+0.03439s: subscription1 received 4+0.03471s: subscription2 received 4shareReplay: receive value: (5)+0.03577s: subscription1 received 5+0.03609s: subscription2 received 5shareReplay: receive finished+0.03759s: subscription1 received completion: finished+0.03788s: subscription2 received completion: finishedSubscribing to shareReplay after upstream completed+1.11936s: subscription3 received 4+1.11945s: subscription3 received 5+1.11985s: subscription3 received completion: finished
```

所有以 shareReplay 开头的行都是显示原始 Subject 发生情况的日志。 现在你确定它只执行一次工作并与所有当前和未来的订阅者共享结果。 工作做得很好！

本章教你创建自己的发布者的几种技术。 它又长又复杂，因为有很多代码要写。 你现在差不多完成了，但在继续之前，你还需要了解最后一个主题。

#### 处理背压

在流体动力学中，背压是与通过管道的所需流体流动相反的阻力或力。在 Combine 中，它是反对来自发布者的期望值流的阻力。但这个阻力是什么？通常，订阅者需要处理发布者发出的值。一些例子是：

* 处理高频数据，例如来自传感器的输入。
* 执行大文件传输。
* 在数据更新时渲染复杂的 UI。
* 等待用户输入。
* 更一般地说，处理订阅者无法以传入速度跟上的传入数据。

Combine 提供的发布者-订阅者机制是灵活的。这是一种拉式设计，而不是推式设计。这意味着订阅者要求发布者发出值并指定他们想要接收的数量。这种请求机制是自适应的：每次订阅者收到一个新值时，需求都会更新。这允许订阅者在他们不想接收更多数据时通过“关闭水龙头”来处理背压，并在他们准备好接收更多数据时“打开它”。

> 注意：请记住，你只能以累加的方式调整需求。你可以在订阅者每次收到新值时增加需求，方法是返回新的 .max(N) 或 .unlimited。或者你可以返回 .none，表示需求不应增加。然而，订阅者随后“挂机”以接收至少达到新的最大需求的值。例如，如果之前的最大需求是接收三个值，而订阅者只接收到一个，则在订阅者的 receive(\_:) 中返回 .none 不会“关闭水龙头”。当发布者准备好发送它们时，订阅者仍将最多接收两个值。

当更多值可用时会发生什么完全取决于你的设计。你可以：

* 通过管理 Demand 来控制流量，以防止发布者发送超出你处理能力的值。
* 缓冲值，直到你可以处理它们 - 存在耗尽可用内存的风险。
* 删除你无法立即处理的值。
* 以上的一些组合，根据你的要求。

浏览所有可能的组合和实现可能需要几个章节。除了上述之外，处理背压还可以采取以下形式：

* 具有处理拥塞的自定义订阅的发布者。
* 在发布者链末端提供值的订阅者。

在本背压管理简介中，你将专注于实现后者。你将创建一个你已经很熟悉的 sink 函数的可暂停变体。

**使用可暂停的 sink 来处理背压**

首先，切换到 Playground 的 PausableSink 页面。

第一步，创建一个协议，让你从暂停中恢复：

```
protocol Pausable {  var paused: Bool { get }  func resume()}
```

这里不需要 pause() 方法，因为你将在收到每个值时确定是否暂停。当然，更复杂的可暂停订阅者可以有一个 pause() 方法，你可以随时调用！现在，你将让代码尽可能简单明了。

接下来，添加此代码以开始定义可暂停订阅者：

```
// 1final class PausableSubscriber<Input, Failure: Error>:  Subscriber, Pausable, Cancellable {  // 2  let combineIdentifier = CombineIdentifier()}
```

1. 你的可暂停订阅者既可以暂停也可以取消。这是你的 pausableSink 函数将返回的对象。这也是为什么你将它作为一个类而不是一个结构来实现的原因：你不希望一个对象被复制，并且你需要在其生命周期的某些点上具有可变性。
2. 订阅者必须为 Combine 提供唯一标识符以管理和优化其发布者流。

现在添加这些附加属性：

```
// 3let receiveValue: (Input) -> Bool// 4let receiveCompletion: (Subscribers.Completion<Failure>) -> Void// 5private var subscription: Subscription? = nil// 6var paused = false
```

1. receiveValue 闭包返回一个 Bool：true 表示它可能会收到更多的值，false 表示订阅应该暂停。
2. 完成闭包将在收到来自发布者的完成事件时被调用。
3. 保留订阅，以便它可以在暂停后请求更多值。当你不再需要它时，你需要将此属性设置为 nil 以避免保留循环。
4. 根据 Pausable 协议公开 paused 属性。

接下来，将以下代码添加到 PausableSubscriber 以实现初始化程序并符合 Cancelable 协议：

```
// 7init(receiveValue: @escaping (Input) -> Bool,     receiveCompletion: @escaping (Subscribers.Completion<Failure>) -> Void) {  self.receiveValue = receiveValue  self.receiveCompletion = receiveCompletion}// 8func cancel() {  subscription?.cancel()  subscription = nil}
```

1. 初始化器接受两个闭包，订阅者将在收到来自发布者的新值和完成时调用它们。闭包类似于你使用 sink 函数的闭包，但有一个例外：receiveValue 闭包返回一个布尔值以指示接收器是否准备好接受更多值，或者你是否需要暂停订阅。
2. 取消订阅时，不要忘记之后将其设置为 nil 以避免保留循环。

现在添加此代码以满足订阅者的要求：

```
func receive(subscription: Subscription) {  // 9  self.subscription = subscription  // 10  subscription.request(.max(1))}func receive(_ input: Input) -> Subscribers.Demand {  // 11  paused = receiveValue(input) == false  // 12  return paused ? .none : .max(1)}func receive(completion: Subscribers.Completion<Failure>) {  // 13  receiveCompletion(completion)  subscription = nil}
```

1. 收到发布者创建的订阅后，将其存储以备后用，以便你能够从暂停中恢复。
2. 立即请求一个值。你的订阅者可以暂停，你无法预测何时需要暂停。这里的策略是一个一个地请求值。
3. 当接收到新值时，调用 receiveValue 并相应更新暂停状态。
4. 如果订阅者被暂停，返回 .none 表示你现在不想要更多的值——记住，你最初只请求了一个。否则，请再请求一个值以保持循环继续。 13.收到完成事件后，将其转发给receiveCompletion，然后将订阅设置为 nil，因为你不再需要它。

最后，实现 Pausable 的其余部分：

```
func resume() {  guard paused else { return }  paused = false  // 14  subscription?.request(.max(1))}
```

如果发布者“暂停”，则请求一个值以重新开始循环。

就像你对以前的发布者所做的那样，你现在可以在 Publishers 命名空间中公开新的可暂停接收器。

在 Playground 的末尾添加以下代码：

```
extension Publisher {  // 15  func pausableSink(    receiveCompletion: @escaping ((Subscribers.Completion<Failure>) -> Void),    receiveValue: @escaping ((Output) -> Bool))    -> Pausable & Cancellable {    // 16    let pausable = PausableSubscriber(      receiveValue: receiveValue,      receiveCompletion: receiveCompletion)    self.subscribe(pausable)    // 17    return pausable  }}
```

1. 你的 pausableSink 操作符与 sink 操作符非常接近。唯一的区别是receiveValue 闭包的返回类型：Bool。
2. 实例化一个新的 PausableSubscriber 并将其订阅给自己，即发布者。
3. 订阅者是你用来恢复和取消订阅的对象。

**测试你的新 Sink**

你现在可以试试你的新水槽了！为简单起见，请模拟发布者应停止发送值的情况。添加此代码：

```
let subscription = [1, 2, 3, 4, 5, 6]  .publisher  .pausableSink(receiveCompletion: { completion in    print("Pausable subscription completed: \(completion)")  }) { value -> Bool in    print("Receive value: \(value)")    if value % 2 == 1 {      print("Pausing")      return false    }    return true}
```

数组的发布者通常按顺序发出所有值，一个接一个。使用你的可暂停接收器，此发布者将在收到值 1、3 和 5 时暂停。

运行 Playground，你会看到：

```
Receive value: 1Pausing
```

要恢复发布者，你需要异步调用 resume()。使用计时器很容易做到这一点。添加此代码以设置计时器：

```
let timer = Timer.publish(every: 1, on: .main, in: .common)  .autoconnect()  .sink { _ in    guard subscription.paused else { return }    print("Subscription is paused, resuming")    subscription.resume()  }
```

再次运行 Playground，你会看到暂停/恢复机制在起作用：

```
Receive value: 1PausingSubscription is paused, resumingReceive value: 2Receive value: 3PausingSubscription is paused, resumingReceive value: 4Receive value: 5PausingSubscription is paused, resumingReceive value: 6Pausable subscription completed: finished
```

恭喜！你现在有了一个功能性的可暂停接收器，并且你已经了解了如何处理代码中的背压！

> 注意：如果你的发布者无法保存值并等待订阅者请求它们怎么办？在这种情况下，你需要使用 buffer(size:prefetch:whenFull:) 操作符来缓冲值。此操作符可以将值缓冲到你在 size 参数中指定的容量，并在订阅者准备好接收它们时传递它们。其他参数确定缓冲区如何填满 - 订阅时立即填满，保持缓冲区满，或根据订阅者的请求 - 以及缓冲区满时会发生什么 - 即删除它收到的最后一个值，drop 最旧的或因错误而终止。

#### 关键点

哇，这是一个漫长而复杂的章节！ 你学到了很多关于发布者的知识：

* 发布者可以是一种利用其他发布者方便的简单方法。
* 编写自定义发布者通常涉及创建随附的订阅。
* 订阅是订阅者和发布者之间的真正链接。
* 在大多数情况下，订阅是完成所有工作的订阅。
* 订阅者可以通过调整其需求来控制价值的传递。
* 订阅有责任尊重订阅者的需求。 Combine 不会强制执行它，但你绝对应该尊重它，将其视为 Combine 生态系统的好公民。

#### 接下来去哪儿？

你了解了发布者的内部运作方式，以及如何设置机制来编写自己的内容。 当然，你编写的任何代码——尤其是发布者！ — 应彻底测试。 继续下一章，了解有关测试组合代码的所有信息！
