# 第 10 节：调试

### &#x20;第 10 节：调试

理解异步代码中的事件流一直是一个挑战。在 Combine 的上下文中尤其如此，因为发布者中的操作符链可能不会立即发出事件。 例如，像 `throttle(for:scheduler:latest:)` 这样的操作符不会发出它们接收到的所有事件，所以你需要了解发生了什么。 Combine 提供了一些操作符来帮助调试你的反应流。 了解它们将帮助你解决令人费解的情况。

#### 打印事件

`print(_:to:)` 操作符是你在不确定是否有任何内容通过你的发布者时应该使用的第一个操作符。它是一个PassthroughPublisher ，可以打印很多关于正在发生的事情的信息。

即使是这样的简单案例：

```
let subscription = (1...3).publisher  .print("publisher")  .sink { _ in }
```

输出非常详细：

```
publisher: receive subscription: (1...3)publisher: request unlimitedpublisher: receive value: (1)publisher: receive value: (2)publisher: receive value: (3)publisher: receive finished
```

在这里，你会看到 `print(_:to:)` 操作符显示了很多信息，因为它：

* 在收到订阅时打印并显示其上游发布者的描述。
* 打印订户的 demand request，以便你查看请求的项目数量。
* 打印上游发布者发出的每个值。
* 最后，打印完成事件。

有一个额外的参数接受一个 TextOutputStream 对象。 你可以使用它来重定向字符串以打印到记录器。 你还可以在日志中添加信息，例如当前日期和时间等。可能性无穷无尽！

例如，你可以创建一个简单的记录器来显示每个字符串之间的时间间隔，以便了解发布者发出值的速度：

```
class TimeLogger: TextOutputStream {  private var previous = Date()  private let formatter = NumberFormatter()​  init() {    formatter.maximumFractionDigits = 5    formatter.minimumFractionDigits = 5  }​  func write(_ string: String) {    let trimmed = string.trimmingCharacters(in: .whitespacesAndNewlines)    guard !trimmed.isEmpty else { return }    let now = Date()    print("+\(formatter.string(for: now.timeIntervalSince(previous))!)s: \(string)")    previous = now  }}
```

在你的代码中使用它非常简单：

```
let subscription = (1...3).publisher  .print("publisher", to: TimeLogger())  .sink { _ in }
```

结果显示每条打印行之间的时间：

```
+0.00111s: publisher: receive subscription: (1...3)+0.03485s: publisher: request unlimited+0.00035s: publisher: receive value: (1)+0.00025s: publisher: receive value: (2)+0.00027s: publisher: receive value: (3)+0.00024s: publisher: receive finished
```

如上所述，这里的可能性是无限的。

> 注意：根据你运行此代码的计算机和 Xcode 版本，上面打印的时间间隔可能会略有不同。

#### 执行副作用

除了打印信息外，对特定事件执行操作通常很有用。 我们将此称为执行副作用，因为你“在一边”采取的操作不会直接影响下游的其他发布者，但会产生类似于修改外部变量的效果。

`handleEvents(receiveSubscription:receiveOutput:receiveCompletion:receiveCancel:receiveRequest:)`让你可以拦截发布者生命周期中的所有事件，然后在每个步骤中采取行动。

想象一下，你正在跟踪一个发布者必须执行网络请求，然后发出一些数据的问题。 当你运行它时，它永远不会收到任何数据。 发生了什么？ 请求真的有效吗？ 你甚至听听什么回来？

考虑这段代码：

```
let request = URLSession.shared  .dataTaskPublisher(for: URL(string: "https://www.raywenderlich.com/")!)​request  .sink(receiveCompletion: { completion in    print("Sink received completion: \(completion)")  }) { (data, _) in    print("Sink received data: \(data)")  }
```

你运行它，从来没有看到任何打印。 看代码能看出问题吗？

如果没有，请使用 handleEvents 来跟踪正在发生的事情。 你可以在 publisher 和 sink 之间插入此操作符：

```
.handleEvents(receiveSubscription: { _ in  print("Network request will start")}, receiveOutput: { _ in  print("Network request data received")}, receiveCancel: {  print("Network request cancelled")})
```

然后，再次运行代码。 这次你会看到一些调试输出：

```
Network request will startNetwork request cancelled
```

你忘记保留可取消项。 因此，订阅开始但立即被取消。 现在，你可以通过保留 Cancellable 来修复你的代码：

```
let subscription = request  .handleEvents...
```

然后，再次运行你的代码，你现在将看到它的行为正确：

```
Network request will startNetwork request data receivedSink received data: 303094 bytesSink received completion: finished
```

#### 使用 Debugger 操作符作为最后的手段

Debugger 操作符是你在万不得已的时候确实需要使用的操作符，因为没有其他方法可以帮助你找出问题所在。

第一个简单的操作符是 `breakpointOnError()`。 顾名思义，当你使用此操作符时，如果任何上游发布者发出错误，Xcode 将在调试器中中断，让你查看堆栈，并希望找到你的发布者错误的原因和位置。

一个更完整的变体是 `breakpoint（receiveSubscription:receiveOutput:receiveCompletion:)`。 它允许你拦截所有事件并根据具体情况决定是否要暂停。

例如，只有当某些值通过发布者时，你才能中断：

```
.breakpoint(receiveOutput: { value in  return value > 10 && value < 15})
```

假设上游发布者发出整数值，但值 11 到 14 永远不会发生，你可以将断点配置为仅在这种情况下中断并让你调查！你还可以有条件地中断订阅和完成，但不能像 handleEvents 操作符那样拦截 Cancel。

#### 关键点

* 与 `print` 操作符一起跟踪发布者的生命周期，
* 创建自己的 `TextOutputStream` 来自定义输出字符串，
* 使用 `handleEvents` 操作符拦截生命周期事件并执行操作，
* 使用 `breakpointOnError` 和 `breakpoint` 操作符来中断特定事件。

#### 然后去哪儿？

你已经了解了如何跟踪你的发布商正在做的事情，现在是时候了解计时器了！ 继续下一节，了解如何使用 Combine 定期触发事件
