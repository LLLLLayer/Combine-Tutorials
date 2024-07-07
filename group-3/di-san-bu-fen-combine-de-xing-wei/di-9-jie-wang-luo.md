# 第 9 节：网络

### 第 9 节：网络

我们所做的很多事情都是围绕网络展开的。与后端通信、获取数据、推送更新、编码和解码 JSON……这是移动开发人员的日常工作。

Combine 提供了一些精选的 API 来帮助以声明方式执行常见任务。这些 API 围绕现代应用程序的两个关键组件展开：

1. 使用 URLSession 执行网络请求。
2. 使用 Codable 协议对 JSON 数据进行编码和解码。

#### URLSession 扩展

URLSession 是执行网络数据传输任务的标准方式。它提供了具有强大配置选项和完全透明的后台的、支持现代异步的 API。它支持多种操作，例如：

* 用于检索 URL 内容的数据传输任务。
* 下载任务以检索 URL 的内容并将其保存到文件中。
* 上传任务将文件和数据上传到 URL。
* 流式传输任务以在两方之间传输数据。
* 连接到 websocket 的 Websocket 任务。

其中，只有第一个，数据传输任务，公开了一个 Combine 发布者。 Combine 使用具有两个变体的单个 API 处理这些任务，采用 URLRequest 或仅采用 URL。

下面看看如何使用这个 API：

```
guard let url = URL(string: "https://mysite.com/mydata.json") else {   return }​// 1let subscription = URLSession.shared  // 2  .dataTaskPublisher(for: url)  .sink(receiveCompletion: { completion in    // 3    if case .failure(let err) = completion {      print("Retrieving data failed with error \(err)")    }  }, receiveValue: { data, response in    // 4    print("Retrieved data of size \(data.count), response = \(response)")  })
```

下面是这段代码发生的事情：

1. 保留最终的 subscription 至关重要； 否则，它会立即被取消，并且请求永远不会执行。
2. 你正在使用将 URL 作为参数的 `dataTaskPublisher(for:)` 的重载。
3. 确保你总是处理错误！ 网络连接容易出现故障。
4. 结果是包含 `Data` 对象和 `URLResponse` 的元组。

如你所见，Combine 在 URLSession.dataTask 之上提供了一个透明的准系统发布者抽象，仅公开发布者而不是闭包。

#### Codable 支持

Codable 协议是你绝对应该了解的现代、强大且仅限 Swift 的编码和解码机制。

Foundation 支持通过 JSONEncoder 和 JSONDecoder 对 JSON 进行编码和解码。 你也可以使用 PropertyListEncoder 和 PropertyListDecoder，但这些在网络请求的上下文中用处不大。

在前面的示例中，你下载了一些 JSON。 当然，你可以使用 JSONDecoder 对其进行解码：

```
let subscription = URLSession.shared  .dataTaskPublisher(for: url)  .tryMap { data, _ in    try JSONDecoder().decode(MyType.self, from: data)  }  .sink(receiveCompletion: { completion in    if case .failure(let err) = completion {      print("Retrieving data failed with error \(err)")    }  }, receiveValue: { object in    print("Retrieved object \(object)")  })
```

你在 tryMap 中解码 JSON，该方法有效，但 Combine 提供了一个操作符来帮助减少代码：`decode(type:decoder:)`。

在上面的示例中，将 tryMap 操作符替换为以下行：

```
.map(\.data).decode(type: MyType.self, decoder: JSONDecoder())
```

不幸的是，由于 `dataTaskPublisher(for:)` 发出一个元组，你不能直接使用 `decode(type:decoder:)` 而不首先使用只发出数据部分结果的 `map(_:)`。

唯一的优点是你只在设置发布者时实例化 `JSONDecoder` 一次，而不是每次在 `tryMap(_:)` 闭包中创建它。

#### 向多个订阅者发布网络数据

每次订阅发布者时，它都会开始工作。 在网络请求的情况下，这意味着如果多个订阅者需要结果，则多次发送相同的请求。

令人惊讶的是，Combine 没有像其他框架那样容易实现这一点的操作符。 你可以使用 share() 操作符，但这很棘手，因为你需要在结果返回之前订阅所有订阅者。

除了使用缓存机制外，一种解决方案是使用 `multicast()` 操作符，它创建一个 `ConnectablePublisher`，通过 Subject 发布值。 它允许你多次订阅 subject，然后在你准备好时调用发布者的 `connect()` 方法：

```
let url = URL(string: "https://www.raywenderlich.com")!let publisher = URLSession.shared// 1  .dataTaskPublisher(for: url)  .map(\.data)  .multicast { PassthroughSubject<Data, URLError>() }​// 2let subscription1 = publisher  .sink(receiveCompletion: { completion in    if case .failure(let err) = completion {      print("Sink1 Retrieving data failed with error \(err)")    }  }, receiveValue: { object in    print("Sink1 Retrieved object \(object)")  })​// 3let subscription2 = publisher  .sink(receiveCompletion: { completion in    if case .failure(let err) = completion {      print("Sink2 Retrieving data failed with error \(err)")    }  }, receiveValue: { object in    print("Sink2 Retrieved object \(object)")  })​// 4let subscription = publisher.connect()
```

在此代码中：

1. 创建你的 `DataTaskPublisher`，map 到它的 data，然后 multicast。你传递的闭包必须返回适当类型的 subject。 或者，你可以将现有 subject 传递给`multicast(subject:)`。 你将在后文中了解有关多播的更多信息。
2. 首次订阅发布者。 由于它是一个 `ConnectablePublisher`，它不会立即开始工作。
3. 再次订阅。
4. 准备好后连接发布者。 它将开始工作并向所有订阅者推送值。

使用此代码，你可以一次性发送请求并与两个订阅者共享结果。

> 注意：确保存储所有 Cancelable；否则，它们将在离开当前代码范围时被释放和取消，这在这种特定情况下是立即的。

这个过程仍然有点复杂，因为 Combine 不像其他响应式框架那样为这种场景提供操作符。在后文自定义发布者和处理背压”中，你将探索如何设计一个更好的解决方案。

#### 关键点

* Combine 为 `dataTask(with:completionHandler:)` 方法提供了一个基于发布者的抽象，称为 `dataTaskPublisher(for:)`。
* 你可以使用发出 Data 值的发布者上的内置解码操作符来解码符合 Codable 的模型。
* 虽然没有操作符可以与多个订阅者共享订阅的重播，但你可以使用 ConnectablePublisher 和 multicast 操作符重新创建此行为。

#### 然后去哪儿？

如果你想了解有关使用 Codable 的更多信息，可以查看以下资源：

* raywenderlich.com 上的“Swift 中的编码和解码”：[https://www.raywenderlich.com/3418439-encoding-and-decoding-in-swift](https://www.raywenderlich.com/3418439-encoding-and-decoding-in-swift)
* Apple 官方文档上的“编码和解码自定义类型”：[https://developer.apple.com/documentation/foundation/archives\_and\_serialization/encoding\_and\_decoding\_custom\_types](https://developer.apple.com/documentation/foundation/archives\_and\_serialization/encoding\_and\_decoding\_custom\_types)
