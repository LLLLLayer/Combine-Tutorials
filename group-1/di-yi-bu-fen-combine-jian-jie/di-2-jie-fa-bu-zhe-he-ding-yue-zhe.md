# 第 2 节：发布者和订阅者

### 第二节：发布者和订阅者

现在你已经了解了 Combine 的一些基本概念，是时候开始使用 Combine 的两个核心组件——发布者和订阅者。在本节中，你将尝试各种方法来创建发布者并订阅它们。

> 注意：在本书的每一章中，你都会用到 Playground 和项目的初始版本和最终版本。starter 已准备好让你为每个示例和挑战输入代码。 如果遇到困难，你可以在 final 或查看最终版本或进行比较。

#### 入门

在本章中，你将使用导入了 Combine 的 Xcode Playground。 打开项目文件夹中的 starter/playground，你将看到以下内容：

![image-20220914015722757](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220914015722757.png)

在项目导航器中打开源代码（View ▸ Navigators ▸ Show Project Navigator ▸ Combine playground页面），然后选择 `SupportCode.swift`。 它包含以下辅助函数 `example(of:)`：

```
public func example(of description: String,                    action: () -> Void) {  print("\n——— Example of:", description, "———")  action()}
```

你将使用这个函数来封装你将在本书中使用的每个示例。

但是，在开始使用这些示例之前，你首先需要更详细地了解发布者、订阅者和订阅。 它们构成了 Combine 的基础，使你能够发送和接收数据，通常是异步的。

#### 你好，Publisher

Combine 的核心是 Publisher 协议。 该协议定义了一种类型的要求，以便能够随时间将一系列值传输给一个或多个订阅者。 换句话说，发布者发布或发出可能包含感兴趣值的事件。

订阅发布者的想法类似于订阅来自 NotificationCenter 的特定通知。使用 NotificationCenter，你可以表达对某些事件的兴趣，然后在有新事件发生时异步通知你。

事实上，它们非常相似，以至于 NotificationCenter 有一个名为 `publisher(for:object:)` 的方法，它提供了一个可以发布通知的 Publisher 类型。

要在实践中检查这一点，请返回到 starter playground 并将 `Add your code here` 占位符替换为以下代码：

```
example(of: "Publisher") {    // 1    let myNotification = Notification.Name("MyNotification")        // 2    let publisher = NotificationCenter.default        .publisher(for: myNotification, object: nil)}
```

在此代码中：

1. 创建通知名称。
2. 访问 `NotificationCenter` 的默认实例，调用它的 `publisher(for:object:)` 方法，并将它的返回值赋给一个局部常量。

按住 Option 点击 `publisher(for:object:)`，你会看到它返回一个 Publisher，当默认通知中心广播一个通知时，它会发出一个事件。

那么当通知中心已经能够在没有发布者的情况下广播其通知时，发布通知有什么意义呢？

你可以将这些类型的方法视为从较旧的异步 API 到较新的替代方案的桥梁——如果你愿意，可以将它们 Combine 起来。

发布者发出两种事件：

1. Value，也称为 Element。
2. Completion 事件。

发布者可以发出零个或多个 Value，但只能发出一个 Completion 事件，它可以是正常Completion 事件或 Error。 一旦发布者发出 Completion 事件，它就完成了，不能再发出任何事件。

> 注意：从这个意义上说，发布者有点类似于 Swift iterator。 一个非常有价值的区别是 Publisher 的 Completion 可能是成功的也可能是失败的，你需要主动从迭代器中获取 Value，而 Publisher 则将 Value 推送给它的消费者。

接下来，你将通过使用 NotificationCenter 来观察并发布通知。 当你不再有兴趣接收该通知时，你还将取消注册该观察者。

将以下代码附加到示例闭包中已有的代码中：

```
// 3let center = NotificationCenter.default​// 4let observer = center.addObserver(    forName: myNotification,    object: nil,    queue: nil) { notification in        print("Notification received!")    }​// 5center.post(name: myNotification, object: nil)​// 6center.removeObserver(observer)
```

使用此代码，你可以：

1. 获取 `NotificationCenter.default` 的句柄。
2. 创建一个观察者来监听你之前创建的通知。
3. 使用该名称发布通知。
4. 从通知中心移除观察者。

运行 playground，你将看到此输出打印到控制台：

```
——— Example of: Publisher ———Notification received!
```

该示例的标题有点误导，因为输出实际上并非来自发布者。 为此，你需要订阅者。

#### 你好，Subscriber

Subscriber 是一种协议，它定义了能够从发布者接收输入的类型的要求。你将很快深入了解如何遵守发布者和订阅者协议； 现在，你将专注于基本流程。

向 Playground 添加一个新示例，该示例与上一个类似：

```
example(of: "Subscriber") {    let myNotification = Notification.Name("MyNotification")    let center = NotificationCenter.default        let publisher = center.publisher(for: myNotification, object: nil)    }
```

如果你现在要发布通知，发布者不会发出它，因为还没有订阅来使用通知。

**使用 `sink(_:_:)` 订阅**

继续上一个示例并添加以下代码：

```
// 1let subscription = publisher    .sink { _ in        print("Notification received from a publisher!")    }// 2center.post(name: myNotification, object: nil)// 3subscription.cancel()
```

使用此代码：

1. 通过在发布者上调用 `sink` 创建订阅。
2. 发布通知。
3. 取消订阅。

不要让 `sink` 方法名的晦涩难懂迷惑。 按住 Option 键单击 `sink`，你会看到它提供了一种简单的方法来附加带有闭包的订阅者以处理来自发布者的输出。在此示例中，米只需打印一条消息以指示已收到通知。 你将很快了解有关取消订阅的更多信息。

运行 playground，你将看到以下内容：

```
——— Example of: Publisher ———Notification received from a publisher!
```

`skin` 将持续接收与发布者发出的一样多的值——这被称为无限需求(Unlimited demand)。尽管你在前面的示例中忽略了它们，但 `sink` 操作符实际上提供了两个闭包：一个用于处理接收 completion 事件（成功或失败），另一个用于处理接收的 Value。

要尝试这些，请将这个新示例添加到你的 Playground：

```
example(of: "Just") {    // 1    let just = Just("Hello world!")        // 2    _ = just        .sink(            receiveCompletion: {                print("Received completion", $0)            },            receiveValue: {                print("Received value", $0)            })}
```

在这里：

1. 使用 Just 创建发布者，它允许你从单个值创建发布者。
2. 创建对发布者的订阅并为每个接收到的事件打印一条消息。

运行 playground，你将看到以下内容：

```
——— Example of: Just ———Received value Hello world!Received completion finished
```

按住 Option 键单击 `Just`，它是一个发布者，它向每个订阅者发出一次输出，然后完成。

尝试通过将以下代码添加到示例末尾来添加另一个订阅者：

```
_ = just    .sink(        receiveCompletion: {            print("Received completion (another)", $0)        },        receiveValue: {            print("Received value (another)", $0)        })
```

运行 playground，正如它所说的，Just 很高兴地向每个新订阅者发送它的输出一次，然后完成。

```
Received value (another) Hello world!Received completion (another) finished
```

**使用 `assign(to:on:)` 订阅**

除了`sink` 之外，内置的 `assign(to:on:)` 运算符能够将接收到的值分配给对象的 KVO 兼容属性。添加此示例以查看其工作原理：

```
example(of: "assign(to:on:)") {    // 1    class SomeObject {        var value: String = "" {            didSet {                print(value)            }        }    }        // 2    let object = SomeObject()        // 3    let publisher = ["Hello", "world!"].publisher        // 4    _ = publisher        .assign(to: \.value, on: object)}
```

从一开始：

1. 定义一个具有属性的类，该属性具有打印新值的`didSet` 属性观察器。
2. 创建该类的实例。
3. 从字符串数组创建发布者。
4. 订阅发布者，将收到的每个值分配给对象的 `value` 属性。

运行 playground，你会看到打印出来的：

```
——— Example of: assign(to:on:) ———Helloworld!
```

> 注意：在后面的章节中，你将看到 `assign(to:on:)` 在处理 UIKit 或 AppKit 应用程序时特别有用，因为你可以将值直接分配给 label、 textView、checkbox 和其他 UI 组件。

**使用 `assign(to:)` 重新发布**

`assign` 操作符的一个变体，重新发布发布者可用于通过另一个用 `@Published` 属性包装器标记的属性发出的值。 请将这个新示例添加到 playground：

```
example(of: "assign(to:)") {    // 1    class SomeObject {        @Published var value = 0    }        let object = SomeObject()        // 2    object.$value        .sink {            print($0)        }        // 3    (0..<10).publisher        .assign(to: &object.$value)}
```

使用此代码：

1. 定义并创建一个类的实例，该实例的属性使用`@Published` 属性包装器注解，除了可作为常规属性访问之外，它还为值创建了一个发布者。
2. 使用 `@Published` 属性上的 `$` 前缀来访问其底层发布者，订阅它，并打印出收到的每个值。
3. 创建一个数字发布者并将它发出的每个值分配给 `object` 的值发布者。 请注意使用 `&` 来表示对属性的 inout 引用。

`assign(to:)` 运算符不返回 `AnyCancellable` token，因为它在内部管理生命周期并在 `@Published` 属性取消初始化时取消订阅。

你可能想知道这与简单地使用 `assign(to:on:)` 相比有何用处？ 考虑以下示例（你无需将其添加到 playground 上）：

```
class MyObject {    @Published var word: String = ""    var subscriptions = Set<AnyCancellable>()        init() {        ["A", "B", "C"].publisher            .assign(to: \.word, on: self)            .store(in: &subscriptions)    }}
```

在此示例中，使用 `assign(to: \.word, on: self)` 并存储生成的 `AnyCancellable` 会导致引用循环。 用 `assign(to: &$word)` 替换 `assign(to:on:)` 可以防止这个问题。

你现在将专注于使用 `sink` 残章符，但你将在以及后续章节中了解更多关于使用 `@Published` 属性的内容。

#### 你好，Cancellable

当订阅者完成其工作并且不再希望从发布者接收值时，最好取消订阅以释放资源并停止发生任何相应的活动，例如网络调用。

订阅将 `AnyCancellable` 的实例作为“cancellation token”返回，这使得你可以在完成订阅后取消订阅。 `AnyCancellable` 符合 `Cancellable` 协议，该协议正是为此目的需要 `cancel()` 方法。

在前面的订阅者示例的底部，你添加了代码 `subscription.cancel()`。你可以在订阅上调用 `cancel()`，因为 `Subscription` 协议继承自 `Cancellable`。

如果你没有在订阅上显式调用 `cancel()`，它将一直持续到发布者完成，或者直到正常的内存管理导致存储的订阅取消初始化。届时，它会取消订阅。

> 注意：忽略 Playground 中订阅的返回值也可以（例如，`_ = just.sink...`）。但是，需要注意的是：如果你没有在完整项目中存储订阅，则该订阅将在程序流退出创建它的范围后立即取消！

这些都是很好的例子，但幕后还有很多事情要做。是时候了解更多关于发布者、订阅者和订阅者在 Combine 中的角色了。

#### 了解正在发生的事情

我们先来解释一下发布者和订阅者之间的相互作用：

![image-20220921013402724](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220921013402724.png)

查看这个 UML 图：

1. 订阅者订阅发布者。
2. 发布者创建订阅并将其提供给订阅者。
3. 订阅者请求值。
4. 发布者发送值。
5. 发布者发送完成。

> 注意：上图提供了正在发生的事情的简化概述。 稍后，你将在后文更深入地了解此过程。

看看 `Publisher` 协议及其最重要的扩展之一：

```
public protocol Publisher {    // 1    associatedtype Output        // 2    associatedtype Failure : Error        // 4    func receive<S>(subscriber: S)    where S: Subscriber,          Self.Failure == S.Failure,          Self.Output == S.Input}​extension Publisher {    // 3    public func subscribe<S>(_ subscriber: S)    where S : Subscriber,          Self.Failure == S.Failure,          Self.Output == S.Input}
```

仔细看看：

1. 发布者可以生成的值的类型。
2. 发布者可能产生的错误类型，如果保证发布者不会产生错误，则为 `Never`。
3. 订阅者在发布者上调用 subscribe(\_:) 以附加到它。
4. subscribe(\_:) 的实现将调用 receive(subscriber:) 将订阅者附加到发布者，即创建订阅。

`associatedtype` 是发布者接口，阅者必须匹配才能创建订阅的。现在，看看订阅者协议：

```
public protocol Subscriber: CustomCombineIdentifierConvertible {    // 1    associatedtype Input        // 2    associatedtype Failure: Error        // 3    func receive(subscription: Subscription)        // 4    func receive(_ input: Self.Input) -> Subscribers.Demand        // 5    func receive(completion: Subscribers.Completion<Self.Failure>)}
```

仔细看看：

1. 订阅者可以接收的值的类型。
2. 订阅者可以收到的错误类型； 如果订阅者不会收到错误则为 `Never`。
3. 发布者在订阅者上调用 `receive(subscription:)` 来给它订阅。
4. 发布者在订阅者上调用 `receive(_:)` 以向它发送它刚刚发布的新值。
5. 发布者在订阅者上调用 `receive(completion:)` 来告诉它它已经完成了生成值，无论是正常还是由于错误。

发布者和订阅者之间的连接是 `subscription`。 这是订阅协议：

```
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {    func request(_ demand: Subscribers.Demand)}
```

订阅者调用 `request(_:)` 表示它愿意接收更多的值，最多可达最大数量或无限制。

> 注意：我们将订阅者的概念称为它愿意接收背压管理的值。如果没有它，或者其他一些策略，订阅者可能会被来自发布者的更多值淹没，而不是它可以处理的。

在 Subscriber 中，请注意 `receive(_:)` 返回一个 `Demand`。 即使 `subscription.request(_:)` 设置了订阅者愿意接收的初始最大值数，你也可以在每次收到新值时调整该最大值。

> 注意：在 `Subscriber.receive(_:)` 中调整最大值是累加的，即新的最大值会添加到当前最大值。 最大值必须为正，传递负值将导致致命错误。 这意味着你可以在每次收到新值时增加原始最大值，但不能减少它。

#### 创建自定义订阅者

是时候把你刚刚学到的东西付诸实践了。 将这个新示例添加到你的 playground：

```
example(of: "Custom Subscriber") {    // 1    let publisher = (1...6).publisher        // 2    final class IntSubscriber: Subscriber {        // 3        typealias Input = Int        typealias Failure = Never                // 4        func receive(subscription: Subscription) {            subscription.request(.max(3))        }                // 5        func receive(_ input: Int) -> Subscribers.Demand {            print("Received value", input)            return .none        }                // 6        func receive(completion: Subscribers.Completion<Never>) {            print("Received completion", completion)        }    }}
```

你在这里做的是：

1. 通过 range 的发布者属性创建整数发布者。
2. 定义一个自定义订阅者，`IntSubscriber`。
3. 实现类型别名以指定此订阅者可以接收整数输入并且永远不会收到错误。
4. 实现所需的方法，以 `receive(subscription:)` 开头，由发布者调用；并在该方法中，在 subscription 上调用 `.request(_:)`，指定订阅者愿意在订阅时接收最多三个值。
5. 收到后打印每个值，返回 `.none`，表示订阅者不会调整自己的需求； `.none` 等价于 .max(0)。
6. 打印完成事件。

发布者要发布任何内容，就需要订阅者。 在示例末尾添加以下内容：

```
let subscriber = IntSubscriber()publisher.subscribe(subscriber)
```

在此代码中，你将创建一个与发布者的输出和失败类型相匹配的订阅者。 然后你告诉发布者订阅或附加订阅者。

运行 playground，你将看到以下打印到控制台：

```
——— Example of: Custom Subscriber ———Received value 1Received value 2Received value 3
```

你没有收到完成事件。 这是因为发布者具有有限数量的值，并且你指定了 `.max(3)` 的需求。

在你的自定义订阅者的 `receive(_:)` 中，尝试将 `.none` 更改为 `.unlimited`，因此你的 `receive(_:)` 方法如下所示：

```
func receive(_ input: Int) -> Subscribers.Demand {		print("Received value", input)		return .unlimited}
```

再次运行 playground，这次你将看到输出包含所有值以及完成事件：

```
——— Example of: Custom Subscriber ———Received value 1Received value 2Received value 3Received value 4Received value 5Received value 6Received completion finished
```

尝试将 `.unlimited` 更改为 `.max(1)` 并再次运行 Playground。

你将看到与返回 `.unlimited` 时相同的输出，因为每次收到事件时，你都指定要将最大值增加 1。

将 `.max(1)` 改回 `.none`，并将发布者的定义改为字符串数组。

用：

```
let publisher = ["A", "B", "C", "D", "E", "F"].publisher
```

代替：

```
let publisher = (1...6).publisher
```

运行 playground， 你会收到一个错误，即 `subscribe` 方法要求 `String` 和 `IntSubscriber.Input` 类型（即 `Int`）等价。 你收到此错误是因为发布者的`Input` 和 `Failure` 关联类型必须匹配订阅者的输入和失败类型才能在两者之间创建订阅。记得将发布者定义更改回整数范围以解决错误。

#### 你好，Future

就像你可以使用 `Just` 创建一个向订阅者发出单个值然后完成的发布者一样，`Future` 可以用于异步生成单个结果然后完成。 将这个新示例添加到你的 playground：

```
example(of: "Future") {    func futureIncrement(        integer: Int,        afterDelay delay: TimeInterval) -> Future<Int, Never> {                    }}
```

在这里，你创建了一个返回 `Int` 和 `Never` 类型的 `future` 的工厂函数； 意思是，它将发出一个整数并且永远不会失败。

在示例中，你还添加了一个订阅集，你将在其中存储对 `future` 的订阅。 对于长时间运行的异步操作，不存储订阅将导致在当前代码范围结束后立即取消订阅。 在 Playground 的情况下，那将是立即的。

接下来，填充函数的主体以创建 `future`：

```
Future<Int, Never> { promise in    DispatchQueue.global().asyncAfter(deadline: .now() + delay) {        promise(.success(integer + 1))    }}
```

这段代码定义了 `future`，它创建了一个 `promise`，然后你使用函数调用者指定的值执行该 `promise`，以在延迟后递增整数。

`Future` 是一个发布者，它最终会产生一个单一的值并完成，否则它将失败。它通过在值或错误可用时调用闭包来做到这一点，而闭包实际上就是 `promise`。 Command 单击 `Future` 并选择 Jump to Definition。 你将看到以下内容：

```
final public class Future<Output, Failure> : Publisher  where Failure: Error {  public typealias Promise = (Result<Output, Failure>) -> Void  ...}
```

Promise 是闭包的类型别名，它接收一个 `Result`，其中包含由 `Future` 发布的单个值或错误。

回到 Playground ，在 `futureIncrement` 的定义之后添加以下代码：

```
// 1let future = futureIncrement(integer: 1, afterDelay: 3)// 2future    .sink(receiveCompletion: { print($0) },          receiveValue: { print($0) })    .store(in: &subscriptions)
```

在这里，你：

1. 使用你之前创建的工厂函数创建一个未来，指定在三秒延迟后递增你传递的整数。
2. 订阅并打印接收到的值和完成事件，并将生成的订阅存储在订阅集中。 你将在本章稍后部分了解有关将订阅存储在集合中的更多信息，因此如果你不完全理解示例的这一部分，请不要担心。

运行 playground，你将看到打印的示例标题，然后是延迟三秒后的未来输出：

```
——— Example of: Future ———2finished
```

通过在 Playground 中输入以下代码来添加对未来的第二个订阅：

```
future    .sink(receiveCompletion: { print("Second", $0) },          receiveValue: { print("Second", $0) })    .store(in: &subscriptions)
```

在运行 Playground 之前，在 `futureIncrement` 函数中的 `DispatchQueue` 块之前插入以下打印语句：

```
print("Original")
```

运行 Playground。 在指定的延迟之后，第二个订阅接收相同的值。Future 不会重新执行 `promise`； 相反，它共享或重放其输出。

```
——— Example of: Future ———Original2finishedSecond 2Second finished
```

代码在订阅发生之前立即打印 Original。 发生这种情况是因为 `Future` 是贪婪的，意味着一旦创建就执行。 它不需要像 lazy 的常规发布者那样的订阅者。

在最后几个示例中，你一直在使用具有有限数量要发布的值的发布者，这些值是按顺序和同步发布的。

你开始使用的通知中心示例是发布者的示例，它可以无限期地和异步地继续发布值，前提是：

1. 底层通知发送者发出通知。
2. 有指定通知的订阅者。

是否有一种方法可以让你在自己的代码中做同样的事情？ 有！ 在继续之前，注释掉整个“Future”示例，因此每次运行 Playground 时都不会调用 Future——否则它的延迟输出会在最后一个示例之后打印。

#### 你好，Subject

你已经学会了如何与发布者和订阅者合作，甚至如何创建自己的自定义订阅者。在本书的后面部分，你将学习如何创建自定义发布者。 我们接着来看是一个 `Subject`。

`Subject` 充当中间人，使非 Combine 命令式代码能够向 Combine 订阅者发送值。

将这个新示例添加到你的 Playground：

```
example(of: "PassthroughSubject") {    // 1    enum MyError: Error {        case test    }        // 2    final class StringSubscriber: Subscriber {        typealias Input = String        typealias Failure = MyError                        func receive(subscription: Subscription) {            subscription.request(.max(2))        }                func receive(_ input: String) -> Subscribers.Demand {            print("Received value", input)            // 3            return input == "World" ? .max(1) : .none        }                func receive(completion: Subscribers.Completion<MyError>) {            print("Received completion", completion)        }            }        // 4    let subscriber = StringSubscriber()}
```

使用此代码：

1. 定义自定义错误类型。
2. 定义一个接收字符串和 `MyError` 错误的自定义订阅者。
3. 根据收到的值调整需求。
4. 创建自定义订阅者的实例。

当输入为“World”时，在 `receive(_:)` 中返回 `.max(1)` 会导致新的最大值设置为 3（原始最大值加 1）。

除了定义自定义错误类型并根据接收到的值调整需求之外，这里没有什么新东西。更有趣的部分来了。将此代码添加到示例中：

```
// 5let subject = PassthroughSubject<String, MyError>()​// 6subject.subscribe(subscriber)​// 7let subscription = subject    .sink(        receiveCompletion: { completion in            print("Received completion (sink)", completion)        },        receiveValue: { value in            print("Received value (sink)", value)        }    )
```

这段代码：

1. 创建一个 `String` 类型的 `PassthroughSubject` 实例和你定义的自定义错误类型。
   1. 为订阅者订阅 `subject`。
2. 使用 `sink` 创建另一个订阅。

`PassthroughSubject` 使你能够按需发布新值。 他们将愉快地传递这些值和完成事件。 与任何发布者一样，你必须提前声明它可以发出的值和错误的类型； 订阅者必须将这些类型与其输入和失败类型相匹配，才能订阅该 `PassthroughSubject`。

现在你已经创建了一个可以发送值和订阅以接收它们的传递主题，是时候发送一些值了。 将以下代码添加到你的示例中：

```
subject.send("Hello")subject.send("World")
```

这使用 `subject` 的 `send` 方法发送两个值。

运行 playground，你会看到的：

```
——— Example of: PassthroughSubject ———Received value HelloReceived value (sink) HelloReceived value WorldReceived value (sink) World
```

每个订阅者都会在发布时收到这些值。添加以下代码：

```
// 8subscription.cancel()​// 9subject.send("Still there?")
```

在这里：

1. 取消第二次订阅。
2. 发送另一个值。

运行 playground，如你所料，只有第一个订阅者会收到该值。 发生这种情况是因为你之前取消了第二个订阅者的订阅：

```
——— Example of: PassthroughSubject ———Received value HelloReceived value (sink) HelloReceived value WorldReceived value (sink) WorldReceived value Still there?
```

将此代码添加到示例中：

```
subject.send(completion: .finished)subject.send("How about another one?")
```

运行 playground，第二个订阅者没有收到“How about another one?”值，因为它在 `subject` 发送值之前收到了 completion 事件。第一个订阅者没有收到完成事件或值，因为它的订阅之前被取消了。

```
——— Example of: PassthroughSubject ———Received value HelloReceived value (sink) HelloReceived value WorldReceived value (sink) WorldReceived value Still there?Received completion finished
```

在发送完成事件的行之前添加以下代码。

```
subject.send(completion: .failure(MyError.test))
```

再次运行playground，你将看到以下打印到控制台：

```
——— Example of: PassthroughSubject ———Received value HelloReceived value (sink) HelloReceived value WorldReceived value (sink) WorldReceived value Still there?Received completion failure(...MyError.test)
```

> 注意：为便于阅读，错误类型已缩写。

第一个订阅者收到错误，但没有收到错误后发送的完成事件。 这表明，一旦发布者发送了一个 completion 事件——无论是正常完成还是错误——它就完成了。使用 `PassthroughSubject` 传递值是将命令式代码连接到 `Combine` 的声明性世界的一种方式。 但是，有时你可能还想在命令式代码中查看发布者的当前值——为此，你有一个恰当命名的`subject`：`CurrentValueSubject`。

你可以将多个订阅存储在 `AnyCancellable` 的集合中，而不是将每个 subscription 存储为一个值。 然后，该集合将在集合 deinit 之前自动取消添加到其中的每个 subscription。

将这个新示例添加到你的 playground：

```
example(of: "CurrentValueSubject") {    // 1    var subscriptions = Set<AnyCancellable>()        // 2    let subject = CurrentValueSubject<Int, Never>(0)        // 3    subject        .sink(receiveValue: { print($0) })        .store(in: &subscriptions) // 4}
```

这是正在发生的事情：

1. 创建 subscription set。
2. 创建类型为 `Int` 和 `Never` 的 `CurrentValueSubject`。 这将发布整数并且永远不会发布错误，初始值为 0。
3. 创建 `subject` 的 `subscription` 并打印从 `subject` 接收到的值。
4. 将 `subscription` 存储在 subscription set 中（作为 inout 参数而不是 copy 传递）。

你必须使用初始值初始化当前值 subject；新订阅者立即获得该值或该 subject 发布的最新值。 运行 Playground 以查看实际情况：

```
——— Example of: CurrentValueSubject ———0
```

现在，添加此代码以发送两个新值：

```
subject.send(1)subject.send(2)
```

与 `PassthroughSubject` 不同，你可以随时向当前 subject 询问其 value。 添加以下代码以打印出 `subject` 的当前值：

```
print(subject.value)
```

正如你可能通过 subject 的类型名称推断的那样，你可以通过访问其 value 属性来获取其当前值。 运行 Playground，你会看到 2 第二次打印出来。

```
——— Example of: CurrentValueSubject ———0122
```

在当前值主题上调用 `send(_:)` 是发送新值的一种方法。 另一种方法是为其 value 属性分配一个新值。 添加此代码：

```
subject.value = 3print(subject.value)
```

运行 Playground。 你会看到 2 和 3 分别打印了两次——一次由接收订阅者打印，一次是在将主题的值添加到主题后打印主题的值。

接下来，在本示例的最后，创建一个对当前值 subject 的新订阅：

```
subject    .sink(receiveValue: { print("Second subscription:", $0) })    .store(in: &subscriptions)
```

在这里，你创建订阅并打印接收到的值。 你还将该订阅存储在订阅集中。

你刚才读到 subscription set 会自动取消添加到其中的 subscription ，但是你如何验证这一点？ 你可以使用 `print()` 运算符，它将所有发布事件记录到控制台。

在主题和接收器之间的两个订阅中插入 `print()` 运算符。 每个订阅的开头应如下所示：

```
subject    .print()    .sink...
```

再次运行 Playground，你将看到整个示例的以下输出：

```
——— Example of: CurrentValueSubject ———receive subscription: (CurrentValueSubject)request unlimitedreceive value: (0)0receive value: (1)1receive value: (2)22receive value: (3)33receive subscription: (CurrentValueSubject)request unlimitedreceive value: (3)Second subscription: 3receive cancel
```

代码在打印订阅处理程序中的每个值。 之所以会出现接收取消事件，是因为 subscription set 是在此示例的范围内定义的，因此它会在取消初始化时取消它包含的订阅。

那么，你可能想知道，你是否也可以将完成事件分配给 value 属性？ 通过添加以下代码尝试一下：

```
subject.value = .finished
```

没有！ 这会产生错误。 `CurrentValueSubject` 的 value 属性仅用于：value。 你仍然需要使用 `send(_:)` 发送完成事件。 将错误的代码行更改为以下内容：

```
subject.send(completion: .finished)
```

再次运行操场。 这次你将在底部看到以下输出：

```
receive finishedreceive finished
```

两个订阅都接收完成事件而不是取消事件。 由于它们已经完成，你不再需要取消它们。

#### 动态调整 demand

你之前了解到，在 `Subscriber.receive(_:)` 中调整需求是累加的。 你现在可以在更详细的示例中仔细研究它是如何工作的。 将这个新示例添加到 playground 上：

```
example(of: "Dynamically adjusting Demand") {    final class IntSubscriber: Subscriber {        typealias Input = Int        typealias Failure = Never                func receive(subscription: Subscription) {            subscription.request(.max(2))        }                func receive(_ input: Int) -> Subscribers.Demand {            print("Received value", input)                        switch input {            case 1:                return .max(2) // 1            case 3:                return .max(1) // 2            default:                return .none // 3            }        }                func receive(completion: Subscribers.Completion<Never>) {            print("Received completion", completion)        }    }        let subscriber = IntSubscriber()        let subject = PassthroughSubject<Int, Never>()        subject.subscribe(subscriber)        subject.send(1)    subject.send(2)    subject.send(3)    subject.send(4)    subject.send(5)    sub
```

大部分代码与本章之前的示例类似，因此你将专注于 `receive(_:)` 方法。你不断调整自定义订阅者内部的需求：

1. 新的最大值为 4（原始最大值为 2 + 新最大值为 2）。
2. 新的最大值为 5（之前的 4 + 新的 1）。
3. 最大值仍然是 5（之前的 4 + 新的 0）。

运行 playground，你将看到以下内容：

```
——— Example of: Dynamically adjusting Demand ———Received value 1Received value 2Received value 3Received value 4Received value 5
```

正如预期的那样，代码发出了五个值，但没有打印出第六个值。

在继续之前，你还需要了解一件更重要的事情：向订阅者隐藏有关发布者的详细信息。

#### 类型擦除

有时你希望让订阅者订阅来自发布者的事件，而无法访问有关该发布者的其他详细信息。

最好用一个例子来证明这一点，所以把这个新的添加到你的 Playground 中：

```
example(of: "Type erasure") {    // 1    let subject = PassthroughSubject<Int, Never>()    // 2    let publisher = subject.eraseToAnyPublisher()        // 3    publisher        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)        // 4    subject.send(0)}
```

使用此代码：

1. 创建一个 `PassthroughSubject`。
2. 从该 `Subject` 创建一个类型擦除的发布者。
3. 订阅类型擦除的发布者。
4. 通过 `PassthroughSubject` 新值。

按住 Option 键单击发布者，你会看到它的类型为 `AnyPublisher<Int, Never>`。

`AnyPublisher` 是符合 `Publisher` 协议的类型擦除结构。 类型擦除允许你隐藏你可能不想向订阅者或下游发布者公开的发布者的详细信息，你将在下一节中了解这些信息。

你现在是否有一种似曾相识的感觉？ 如果是这样，那是因为你之前看到了另一种类型擦除的情况。 `AnyCancellable` 是一个符合 `Cancellable` 的类型擦除类，它允许调用者取消订阅，而无需访问底层 subscription 来执行其他操作。

当你想要对发布者使用类型擦除的一个示例是，当你想要使用一对公共和私有属性时，以允许这些属性的所有者在私有发布者上发送值，并让外部调用者只订阅但不能发送值。

`AnyPublisher` 没有 `send(_:)` 运算符，因此你不能直接向该发布者添加新值。

`eraseToAnyPublisher()` 运算符将提供的发布者包装在 `AnyPublisher` 的实例中，隐藏发布者实际上是 `PassthroughSubject` 的事实。 这也是必要的，因为你不能专门化 `Publisher` 协议，例如，你不能将类型定义为 `Publisher<UIImage, Never>`。

要证明发布者是类型擦除的并且你不能使用它来发送新值，请将此代码添加到示例中。

```
publisher.send(1)
```

你收到 `AnyPublisher<Int, Never>` 类型的错误值没有成员 `send`。 在继续之前注释掉那行代码。

#### 桥接 Combine 发布者到 async/await

在 iOS 15 和 macOS 12 中，Swift 5.5 中的 Combine 框架新增了两个很棒的功能，可帮助你轻松地将 Combine 与 Swift 中的新 async/await 语法结合使用。

换句话说——你在本书中学到的所有 publisher、future 和 subject 都可以从你的现代 Swift 代码中使用。

将最后一个示例添加到你的 playground：

```
example(of: "async/await") {    let subject = CurrentValueSubject<Int, Never>(0)    }
```

在此示例中，你将使用 `CurrentValueSubject`，但如前所述，API 可用于 `Future` 和任何符合 `Publisher`的类型。

你将使用 subject 来发出元素并使用 for 循环来迭代这些元素的异步序列。

你将订阅新异步任务中的值。请添加：

```
Task {    for await element in subject.values {        print("Element: \(element)")    }    print("Completed.")}
```

Task 创建一个新的异步任务——闭包代码将与本代码示例中的其余代码异步运行。

此代码块中的关键 API 是 subject 的 values 属性。 values 返回一个异步序列，其中包含subject 或发布者发出的元素。你可以像上面那样在一个简单的 for 循环中迭代该异步序列。

一旦发布者完成，无论是成功还是失败，循环都会结束。

接下来，将此代码添加到当前示例中以发出一些值：

```
subject.send(1)subject.send(2)subject.send(3)subject.send(completion: .finished)
```

这将发出 1、2 和 3，然后完成 subject。

这很好地包装了这个例子——发送完成的事件也会结束你的异步任务中的循环。 再次运行 Playground 代码，你将看到以下输出：

```
——— Example of: async/await ———Element: 0Element: 1Element: 2Element: 3Completed.
```

在 `Future` 发出单个元素（如果有）的情况下，values 属性没有多大意义。 这就是为什么 `Future` 有一个 `value` 属性，你可以使用它来异步获取未来的结果。

你在本章中学到了很多东西，并且你将在本书的其余部分及以后的内容中运用这些新技能。 但没那么快！ 是时候练习你刚刚学到的东西了。

#### 挑战

完成挑战有助于巩固你在本章中学到的知识。练习文件下载中有挑战的初始版本和最终版本。

**挑战：二十一点发牌**

在挑战文件夹中打开 Starter.playground，选择 SupportCode.swift：

查看此挑战的辅助代码，包括

* 一个卡片数组，包含 52 个元组，代表一副标准的卡片组。
* 两种类型别名：Card 是 String 和 Int 的元组，Hand 是 Cards 数组。
* 手头有两个辅助属性：cardString 和 points。
* HandError 错误枚举。

在 playground 主页面中，在注释 `// Add code to update dealtHand here` 下方添加代码，从 hand 的 points 属性返回的结果。如果结果大于 21，则通过 dealtHand subject 发送 HandError.busted。 否则，发送手牌值。

同样在 playground 主页面中，在注释 `// Add subscription to dealtHand here` 之后立即添加代码以订阅 dealtHand 并处理接收值和错误。

对于接收到的值，打印一个字符串，其中包含手牌的 `cardString` 和 `points` 属性的结果。

对于错误，将其打印出来。 提示：你可以在 `receivedCompletion` 块中接收 `.finished` 或 `.failure`，因此你需要区分该完成是否失败。

HandError 符合 `CustomStringConvertible`，打印它会得到用户友好的错误消息。 你可以像这样使用它：

```
if case let .failure(error) = $0 {  	print(error)}
```

对 `deal(_:)` 的调用当前通过了 3，因此每次运行 Playground 时都会发三张牌。

在真正的二十一点游戏中，你最初会收到两张牌，然后你必须决定再拿一张或多张牌，称为命中，直到你命中 21 或破产。 对于这个简单的示例，你只需立即获得三张牌。

**参考方案**

你怎么做的？ 要完成这个挑战，你需要添加两件事。 首先是在 deal 函数中更新 dealtHand 发布者，检查手牌点数，如果超过 21 则发送错误：

```
// Add code to update dealtHand hereif hand.points > 21 {    dealtHand.send(completion: .failure(.busted))} else {    dealtHand.send(hand)}
```

接下来，你需要订阅 dealtHand 并打印接收到的值或如果是错误的完成事件：

```
_ = dealtHand    .sink(receiveCompletion: {        if case let .failure(error) = $0 {            print(error)        }    }, receiveValue: { hand in      print(hand.cardString, "for", hand.points, "points")    })
```

每次运行 Playground 时，你都会得到一个新的手，并输出类似于以下内容：

```
——— Example of: Create a Blackjack card dealer ———🃕🃆🃍 for 21 points
```

#### 关键点

* 发布者随着时间的推移将一系列值同步或异步传输给一个或多个订阅者。
* 订阅者可以订阅发布者以接收值；但是，订阅者的输入和失败类型必须与发布者的输出和失败类型相匹配。
* 你可以使用两个内置运算符来订阅发布者：`sink(_:_:)` 和 `assign(to:on:)`。
* 订阅者每次收到价值时可能会增加对价值的需求，但不能减少需求。
* 要释放资源并防止不必要的副作用，请在完成后取消每个 subscription。
* 你还可以将订阅存储在 `AnyCancellable` 的实例或集合中，以便在取消初始化时接收自动取消。
* 你使用 `Future` 在以后异步接收单个值。
* `Subject` 是发布者，它使外部调用者能够异步向订阅者发送多个值，有或没有起始值。
* 类型擦除可防止调用者访问基础类型的其他详细信息。
* 使用 `print()` 操作符将所有发布事件记录到控制台并查看发生了什么。

#### 然后去哪儿？

恭喜！ 通过完成本章，你向前迈出了一大步。 你学习了如何与发布者一起发送值和 completion 事件，以及如何使用订阅者接收这些值和事件。 接下来，你将学习如何操纵来自发布者的值，以帮助过滤、转换或组合它们。
