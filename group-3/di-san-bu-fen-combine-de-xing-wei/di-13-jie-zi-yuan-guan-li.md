# 第 13 节：资源管理

### 第 13 节：资源管理

在前面的章节中，你发现有时你希望共享网络请求、图像处理和文件解码等资源，而不是重复你的工作。任何可以避免重复多次的资源密集型都值得研究。换句话说，你应该在多个订阅者之间共享单个资源的结果——发布者产生的值，而不是复制该结果。

Combine 提供了两个操作符来管理资源：share() 操作符和 multicast(\_:) 操作符。

#### share() 操作符

该操作符的目的是让你通过引用而不是通过值来获取发布者。 发布者通常是结构体：当你将发布者传递给函数或将其存储在多个属性中时，Swift 会多次复制它。 当你订阅每个副本时，发布者只能做一件事：开始其工作并交付值。

share() 操作符返回 `Publishers.Share` 类的实例。通常，发布者被实现为结构，但在 share() 的情况下，如前所述，操作符获取对 Share 发布者的引用而不是使用值语义，这允许它共享底层发布者。

这个新发布者“共享”上游发布者。它将与第一个传入订阅者一起订阅一次上游发布者。然后它将从上游发布者接收到的值转发给这个订阅者以及所有在它之后订阅的人。

> 注意：新订阅者只会收到上游发布者在订阅后发出的值。不涉及缓冲或重放。如果订阅者在上游发布者完成后订阅共享发布者，则该新订阅者只会收到完成事件。

要将这个概念付诸实践，假设你正在执行一个网络请求，你希望多个订阅者无需多次请求即可接收结果。你的代码将如下所示：

```
let shared = URLSession.shared  .dataTaskPublisher(for: URL(string: "https://www.raywenderlich.com")!)  .map(\.data)  .print("shared")  .share()​print("subscribing first")​let subscription1 = shared.sink(  receiveCompletion: { _ in },  receiveValue: { print("subscription1 received: '\($0)'") })​print("subscribing second")​let subscription2 = shared.sink(  receiveCompletion: { _ in },  receiveValue: { print("subscription2 received: '\($0)'") })
```

第一个订阅者触发 `share()` 的上游发布者的“工作”（在这种情况下，执行网络请求）。 第二个订阅者将简单地“连接”到它并与第一个订阅者同时接收值。

在 Playground 中运行此代码，你会看到类似于以下内容的输出：

```
subscribing firstshared: receive subscription: (DataTaskPublisher)shared: request unlimitedsubscribing secondshared: receive value: (303425 bytes)subscription1 received: '303425 bytes'subscription2 received: '303425 bytes'shared: receive finished
```

使用 `print(_:to:)` 操作符的输出，你可以看到：

* 第一个订阅触发对 DataTaskPublisher 的订阅。
* 第二次订阅没有任何改变：发布者继续运行。没有第二个请求发出。
* 当请求完成时，发布者将结果数据发送给两个订阅者，然后完成。

要验证请求只发送一次，你可以注释掉 share() 行，输出将类似于以下内容：

```
subscribing firstshared: receive subscription: (DataTaskPublisher)shared: request unlimitedsubscribing secondshared: receive subscription: (DataTaskPublisher)shared: request unlimitedshared: receive value: (303425 bytes)subscription1 received: '303425 bytes'shared: receive finishedshared: receive value: (303425 bytes)subscription2 received: '303425 bytes'shared: receive finished
```

可以清楚的看到，当 DataTaskPublisher 不共享时，它收到了两个订阅！ 在这种情况下，请求会运行两次，每次订阅一次。

但是有一个问题：如果第二个订阅者是在共享请求完成之后来的呢？ 你可以通过延迟第二次订阅来模拟这种情况。

如果你在操场上跟随，请不要忘记取消注释 share()。 然后，将 subscription2 代码替换为以下内容：

```
var subscription2: AnyCancellable? = nil​DispatchQueue.main.asyncAfter(deadline: .now() + 5) {  print("subscribing second")​  subscription2 = shared.sink(    receiveCompletion: { print("subscription2 completion \($0)") },    receiveValue: { print("subscription2 received: '\($0)'") }  )}
```

运行此程序，如果延迟时间长于请求完成所需的时间，你会看到 subscription2 什么也没有收到：

```
subscribing firstshared: receive subscription: (DataTaskPublisher)shared: request unlimitedsubscribing secondshared: receive value: (303425 bytes)subscription1 received: '303425 bytes'shared: receive finishedsubscribing secondsubscription2 completion finished
```

在创建 subscription2 时，请求已经完成并且结果数据已经发出。如何确保两个订阅都收到请求结果？

#### multicast(\_:) 操作符

即使在上游发布者完成后，要与发布者共享单个订阅并将值重播给新订阅者，你需要类似 `shareReplay()` 操作符。不幸的是，这个操作符不是 Combine 的一部分。但是，你将在后文中学习如何创建一个。

在“网络”中，你使用了 `multicast(_:)`。此操作符基于 `share()` 构建，并使用你选择的 Subject 将值发布给订阅者。 `multicast(_:)` 的独特之处在于它返回的发布者是一个 `ConnectablePublisher`。这意味着它不会订阅上游发布者，直到你调用它的 `connect()` 方法。这让你有足够的时间来设置你需要的所有订阅者，然后再让它连接到上游发布者并开始工作。

要调整前面的示例以使用 `multicast(_:)`，你可以编写：

```
// 1let subject = PassthroughSubject<Data, URLError>()​// 2let multicasted = URLSession.shared  .dataTaskPublisher(for: URL(string: "https://www.raywenderlich.com")!)  .map(\.data)  .print("multicast")  .multicast(subject: subject)​// 3let subscription1 = multicasted  .sink(    receiveCompletion: { _ in },    receiveValue: { print("subscription1 received: '\($0)'") }  )​let subscription2 = multicasted  .sink(    receiveCompletion: { _ in },    receiveValue: { print("subscription2 received: '\($0)'") }  )​// 4let cancellable = multicasted.connect()
```

下面是这段代码的作用：

1. 准备一个 subject，它传递上游发布者发出的值和完成事件。
2. 使用上述 subject 准备多播发布者。
3. 订阅共享的——即多播的——发布者，就像本章前面的那样。
4. 指示发布者连接到上游发布者。

这有效地开始了工作，但只有在你有时间设置所有订阅之后。 这样，你可以确保没有订阅者会错过下载的数据。

如果你在 Playground 上运行，结果输出将是：

```
multicast: receive subscription: (DataTaskPublisher)multicast: request unlimitedmulticast: receive value: (303425 bytes)subscription1 received: '303425 bytes'subscription2 received: '303425 bytes'multicast: receive finished
```

注意：一个多播发布者，和所有的 ConnectablePublisher 一样，也提供了一个 `autoconnect()` 方法，这使它像 `share()` 一样工作：第一次订阅它时，它会连接到上游发布者并立即开始工作。 这在上游发布者发出单个值并且你可以使用 `CurrentValueSubject` 与订阅者共享它的情况下很有用。

对于大多数现代应用程序来说，共享订阅工作，特别是资源密集型流程（例如网络）是必须的。不注意这一点不仅会导致内存问题，而且可能会用大量不必要的网络请求轰炸你的服务器。

#### Future

虽然 `share()` 和 `multicast(_:)` 为你提供了成熟的发布者，Combine 还提供了另一种让你共享计算结果的方法：`Future`。

你可以通过将接收 Promise 参数的闭包交给 Future 来创建它。 只要你有可用的结果（成功或失败），你就会进一步履行承诺。 看一个例子来刷新你的记忆：

```
// 1func performSomeWork() throws -> Int {  print("Performing some work and returning a result")  return 5}​// 2let future = Future<Int, Error> { fulfill in  do {    let result = try performSomeWork()    // 3    fulfill(.success(result))  } catch {    // 4    fulfill(.failure(error))  }}​print("Subscribing to future...")​// 5let subscription1 = future  .sink(    receiveCompletion: { _ in print("subscription1 completed") },    receiveValue: { print("subscription1 received: '\($0)'") }  )​// 6let subscription2 = future  .sink(    receiveCompletion: { _ in print("subscription2 completed") },    receiveValue: { print("subscription2 received: '\($0)'") }  )
```

这段代码：

1. 提供一个模拟 Future 执行的工作（可能是异步的）的功能。
2. 创造新的 Future。请注意，工作立即开始，无需等待订阅者。
3. 如果工作成功，则以结果履行 Promise。
4. 如果工作失败，它将错误传递给 Promise。
5. 订阅一次表明我们收到了结果。
6. 第二次订阅表明我们也收到了结果，没有执行两次工作。

从资源的角度来看，有趣的是：

* Future 是一个类，而不是一个结构。
* 创建后，它立即调用你的闭包开始计算结果并尽快履行承诺。

它存储已履行的 Promise 的结果并将其交付给当前和未来的订阅者。

在实践中，这意味着 Future 是一种方便的方式，可以立即开始执行某些工作（无需等待订阅），同时只执行一次工作并将结果交付给任意数量的订阅者。但它执行工作并返回单个结果，而不是结果流，因此用例比成熟的发布者要窄。

当你需要共享网络请求产生的单个结果时，它是一个很好的选择！

> 注意：即使你从未订阅 Future，创建它也会调用你的闭包并执行工作。你不能依赖 Deferred 来推迟闭包执行，直到订阅者进来，因为 Deferred 是一个结构体，每次有新订阅者时都会创建一个新的 Future！

#### 关键点

* 在处理资源密集型流程（例如网络）时，共享订阅工作至关重要。
* 当你只需要与多个订阅者共享发布者时，使用 `share()`。
* 当你需要精细控制上游发布者何时开始工作以及值如何传播给订阅者时，请使用`multicast(_:)`。
* 使用 Future 将单个计算结果共享给多个订阅者。

#### 接下来去哪儿？

恭喜你完成了本节的最后一个理论小章节！你将通过一个动手项目来结束本节，你将在其中构建一个 API 客户端以与 Hacker News API 交互。
