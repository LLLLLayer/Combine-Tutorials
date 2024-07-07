# 第 14 节：实践：“News”项目

### 第 14 节：实践：“News”项目

在过去的几节中，你了解了在 Foundation 类型中组合集成的许多实际应用。你学习了如何使用 URLSession 的数据任务发布者进行网络调用，你看到了如何使用 Combine 观察 KVO 兼容的对象等等。

在本章中，你将把你对操作符的扎实知识与你刚刚发现的一些 Foundation 集成相结合，并将完成一系列任务。这一次，你将致力于构建一个 Hacker News API 客户端。

“Hacker News”，你将在本章中使用它的 API，是一个专注于计算机和创业的社交新闻网站。如果你还没有，你可以在以下网址查看它们：[https://news.ycombinator.com](https://news.ycombinator.com)。

![image-20221010010545460](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010010545460.png)

在本章中，你将在 Playground 上只关注 API 客户端本身。

后续章节你将获取完整的 API，并通过将网络层插入基于 SwiftUI 的用户界面，使用它来构建一个真正的 Hacker News 阅读器应用程序。在此过程中，你将学习 SwiftUI 的基础知识以及如何使你的组合代码与新的声明性 Apple 框架一起工作，以构建令人惊叹的反应式应用程序 UI。

事不宜迟，让我们开始吧！

#### Hacker News API 入门

在 projects/starter 中打开包含的 starter playground API.playground 并查看。你会发现其中包含一些简单的入门代码，可帮助你快速入门，并让你只专注于 Combine 代码：

![image-20221010010652940](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010010652940.png)

在 API 中，你会发现两个嵌套的类型：

* 一个名为 Error 的枚举，其中包含两个自定义错误，API 无法到达服务器或无法解码服务器响应时抛出。
* 第二个枚举名为 EndPoint，其中包含将要连接到的两个 API 端点的 URL。

再往下，你会发现 maxStories 属性。 你将使用它来限制你的 API 客户端将获取的最新 Story 的数量，以帮助减少 Hacker News 服务器上的负载，以及用于解码 JSON 数据的解码器。

此外，playground 的 Sources 文件夹包含一个名为 Story 的简单结构，你可以将故事数据解码为该结构。

Hacker News API 可以免费使用，不需要注册开发者帐户。 这很棒，因为你可以立即开始编写代码，而无需像其他公共 API 一样先完成一些冗长的注册。 Hacker News 团队赢得大量业力积分！

#### 获取单个 Story

你的第一个任务是向 API 添加一个方法，该方法将使用 EndPoint 类型联系服务器以获取正确的端点 URL 并获取有关单个 Story 的数据。 新方法将返回一个发布者，API 消费者将订阅该发布者，并获得有效且已解析的 Story 或失败。

向下滚动 Playground 源代码并找到注释说 // 在此处添加你的 API 代码。 在该行下方，插入一个新方法声明：

```
func story(id: Int) -> AnyPublisher<Story, Error> {  return Empty().eraseToAnyPublisher()}
```

为了避免 Playground 中的编译错误，你返回一个 Empty 发布者，它会立即完成。 当你完成构建方法体时，你将删除表达式并返回你的新订阅。

如前所述，这个发布者的输出是一个 Story，它的失败是自定义 API.Error 类型。如果出现网络错误或其他事故，你需要将它们转换为 API.Error 之一以匹配预期的返回类型。

通过向 Hacker News API 的单层端点创建网络请求来开始对订阅进行建模。在新方法中，在 return 语句上方，插入：

```
URLSession.shared  .dataTaskPublisher(for: EndPoint.story(id).url)
```

你首先向 Endpoint.story(id).url 发出请求。端点的 url 属性包含要请求的完整 HTTP URL。单个 Story URL 如下所示（具有匹配的 ID）：[https://hacker-news.firebaseio.com/v0/item/12345.json](https://hacker-news.firebaseio.com/v0/item/12345.json)（如果你愿意，请访问 [https://bit.ly/2nL2ojS](https://bit.ly/2nL2ojS)预览 API 响应。）

接下来，为了在后台线程上解析 JSON 并保持应用程序的其余部分响应，让我们创建一个新的自定义调度队列。在 story(id:) 方法上方的 API 中添加一个新属性，如下所示：

```
private let apiQueue = DispatchQueue(label: "API",                                     qos: .default,                                     attributes: .concurrent)
```

你将使用此队列来处理 JSON 响应，因此，你需要将你的网络订阅切换到该队列。回到 story(id:)，在 dataTaskPublisher(for:) 下面添加调用：

```
.receive(on: apiQueue)
```

切换到后台队列后，你需要从响应中获取 JSON 数据。 dataTaskPublisher(for:) 发布者将 (Data, URLResponse) 类型的输出作为元组返回，但对于你的订阅，你只需要数据。

在方法中添加另一行，将当前输出仅映射到结果元组中的数据：

```
.map(\.data)
```

此操作符的输出类型是数据，你可以将其提供给解码操作符并尝试将响应转换为 Story。

附加到订阅：

```
.decode(type: Story.self, decoder: decoder)
```

如果它接收到除了有效的故事 JSON 之外的任何内容，decode(...) 将抛出一个错误，并且发布者将以失败告终。

后文将详细了解错误处理。在本章中，你将使用少量操作符并体验几种处理错误的不同方法，但你不会深入了解事物的工作原理。

对于当前的 story(id:) 方法，你将返回一个空的发布者，以防万一出于任何原因出现问题。这很容易通过使用 catch 操作符来完成。添加到订阅：

```
.catch { _ in Empty<Story, Error>() }
```

你忽略抛出的错误并返回 Empty()。正如你希望仍然记得的那样，这是一个立即完成而不发出任何输出值的发布者，如下所示：

![image-20221010011955724](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010011955724.png)

通过 catch(\_) 以这种方式处理上游错误允许你：

* 如果你返回一个 Story，则发出值并完成。
* 在失败的情况下，返回一个成功完成且不发出任何值的空发布者。

接下来，要包装方法代码并返回你精心设计的发布者，你需要在最后替换当前订阅。 添加：

```
.eraseToAnyPublisher()
```

你现在可以删除之前添加的临时 Empty。 找到以下行并将其删除：

```
return Empty().eraseToAnyPublisher()
```

你的代码现在应该可以顺利编译，但为了确保你没有错过任何激动人心的步骤，请查看你目前的进度并确保你完成的代码如下所示：

```
func story(id: Int) -> AnyPublisher<Story, Error> {  URLSession.shared    .dataTaskPublisher(for: EndPoint.story(id).url)    .receive(on: apiQueue)    .map(\.data)    .decode(type: Story.self, decoder: decoder)    .catch { _ in Empty<Story, Error>() }    .eraseToAnyPublisher()}
```

即使你的代码编译，此方法仍然不会产生任何输出。 你接下来要处理这个问题。

现在你可以实例化 API 并尝试调用 Hacker News 服务器。

向下滚动一点，找到注释行 // Call the API here。 这是进行测试 API 调用的好地方。 插入以下代码：

```
let api = API()var subscriptions = [AnyCancellable]()api.story(id: 1000)   .sink(receiveCompletion: { print($0) },         receiveValue: { print($0) })   .store(in: &subscriptions)
```

你可以通过调用 api.story(id: 1000) 创建一个新的发布者，并通过 sink(...) 订阅它，它会打印任何输出值或完成事件。 要在请求完成之前保持订阅有效，请将其存储在订阅中。

一旦 Playground 再次运行，它就会对 hacker-news.firebaseio.com 进行网络调用，并在控制台中打印结果：

![image-20221010012423893](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010012423893.png)

从服务器返回的 JSON 数据是一个相当简单的结构，如下所示：

```
{  "by":"python_kiss",  "descendants":0,  "id":1000,  "score":4,  "time":1172394646,  "title":"How Important is the .com TLD?",  "type":"story",  "url":"http://www.netbusinessblog.com/2007/02/19/how-important-is-the-dot-com/"}
```

Story 的 Codable 一致性解析并存储以下属性的值：by、id、time、title 和 url。

请求成功完成后，你将在控制台中看到以下输出，或者类似的输出（如果你更改了请求中的 1000 值）：

```
How Important is the .com TLD?by python_kisshttp://www.netbusinessblog.com/2007/02/19/how-important-is-the-dot-com/-----finished
```

Story 类型符合 CustomDebugStringConvertible 并且它有一个自定义 debugDescription ，它返回整齐有序的标题、作者姓名和 URL，如上所示。

输出以完成的完成事件结束。 要尝试发生错误时会发生什么，请将 id 1000 替换为 -5 并检查控制台中的输出。 你只会看到完成打印，因为你发现了错误并返回了 Empty()。

干得好！ API 类型的第一个方法已经完成，并且你练习了前面章节中介绍的一些概念，例如调用网络和解码 JSON。 此外，你还简要介绍了基本的调度队列切换和一些简单的错误处理。 你将在以后的章节中更详细地介绍这些内容。

尽管这项任务是一项非常棒的练习，但你可能还渴望更多。 因此，在下一节中，你将深入挖掘并编写一些严肃的代码。

#### 通过合并多个发布者的的多个 Story

从 API 服务器中获取单个 Story 是一项相对简单的任务。接下来，你将通过创建自定义发布者来同时获取多个故事，从而了解你一直在学习的更多概念。

新方法 mergeStories(ids:) 将为每个给定的故事 ID 获取一个故事发布者，并将它们合并在一起。在你之前实现的 story(id:) 方法之后，将此新方法声明添加到 API 类型：

```
func mergeStories(ids storyIDs: [Int]) -> AnyPublisher<Story, Error> {}
```

这个方法本质上会为每个给定的 id 调用 story(id:) ，然后将结果扁平化为单个输出值流。

首先，为了减少开发过程中的网络调用次数，你将只从提供的列表中获取第一个 maxStories id。通过插入以下代码来启动新方法：

```
let storyIDs = Array(storyIDs.prefix(maxStories))
```

首先，创建第一个发布者：

```
precondition(!storyIDs.isEmpty)let initialPublisher = story(id: storyIDs[0])let remainder = Array(storyIDs.dropFirst())
```

通过使用 story(id:)，你创建了initialPublisher 发布者，该发布者使用列表中的第一个 id 获取 Story。

接下来，你将对剩余的故事 ID 使用 Swift 标准库中的 reduce(_:_:) 将每个下一个故事发布者合并到初始发布者中，如下所示：

![image-20221010013006480](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010013006480.png)

要将其余 Story 减少到初始发布者中，请添加：

```
return remainder.reduce(initialPublisher) { combined, id in}
```

reduce(_:_:) 将从初始发布者开始，并将剩余数组中的每个 id 提供给闭包进行处理。插入此代码以在空闭包中为给定故事 id 创建一个新发布者，并将其合并到当前组合结果中：

```
return combined  .merge(with: story(id: id))  .eraseToAnyPublisher()
```

最终结果是发布者发出每个成功获取的 Story 并忽略单个 Story 发布者可能遇到的任何错误。

> 注意：恭喜，你刚刚创建了 MergeMany 发布者的自定义实现。不过，自己处理代码并没有白费。 你了解了操作符组合以及如何在实际用例中应用合并和归约等操作符。

完成新的 API 方法后，向下滚动到此代码并注释或删除它以加快 Playground 的执行，同时测试你的新代码：

```
api.story(id: -5)   .sink(receiveCompletion: { print($0) },         receiveValue: { print($0) })   .store(in: &subscriptions)
```

代替刚刚代码：

```
api.mergedStories(ids: [1000, 1001, 1002])   .sink(receiveCompletion: { print($0) },         receiveValue: { print($0) })   .store(in: &subscriptions)
```

让 Playground 使用你的最新代码再运行一次。 这一次，你应该在控制台中看到以下三个故事摘要：

```
How Important is the .com TLD?by python_kisshttp://www.netbusinessblog.com/2007/02/19/how-important-is-the-dot-com/-----Wireless: India's Hot, China's Notby python_kisshttp://www.redherring.com/Article.aspx?a=21355-----The Battle for Mobile Searchby python_kisshttp://www.businessweek.com/technology/content/feb2007/tc20070220_828216.htm?campaign_id=rss_daily-----finished
```

在你的学习 Combine 道路上再创辉煌！ 在本节中，你编写了一个方法，该方法可以组合任意数量的发布者并将它们缩减为单个发布者。 这是非常有用的代码，因为内置的合并操作符最多只能合并 8 个发布者。 但是，有时你只是不知道你需要多少个发布者！

#### 获取最新 Story

你将创建一个 API 方法来获取最新的 Hacker News Story 列表。

本章遵循了一些模式。首先，你重用了单 Story 方法来获取多个 Story。现在，你将重用 multiple Story 方法来获取最新 Story 列表。

将新的空方法声明添加到 API 类型，如下所示：

```
func stories() -> AnyPublisher<[Story], Error> {  return Empty().eraseToAnyPublisher()}
```

像以前一样，你在构造方法体和发布者时返回一个 Empty 对象以防止任何编译错误。

不过，与以前不同的是，这一次你返回的发布者的输出是一个 Story 列表。你将设计发布者以获取多个故事并将它们累积在一个数组中。

这种行为将允许你在下一章中将这个新发布者直接绑定到一个 List UI 控件，该控件将在故事从服务器进入时自动在屏幕上进行动画处理。

和之前一样，开始向 Hacker News API 发起网络请求。在你的新方法中，在 return 语句上方插入以下内容：

```
URLSession.shared  .dataTaskPublisher(for: EndPoint.stories.url)
```

EndPoint 允许你点击以下 URL 以获取最新的故事 ID：[https://hacker-news.firebaseio.com/v0/newstories.json](https://hacker-news.firebaseio.com/v0/newstories.json)。

同样，你需要获取发出结果的数据组件。因此，通过添加以下内容来映射输出：

```
.map(\.data)
```

你将从服务器获得的 JSON 响应是一个简单的列表，如下所示：

```
[1000、1001、1002、1003]
```

你需要将列表解析为整数数组，如果成功，你可以使用 id 来获取匹配的 Story。

附加到订阅：

```
.decode(type: [Int].self, decoder: decoder)
```

这会将当前订阅输出映射到 \[Int]，你将使用它从服务器中一一获取相应的 Story。

然而，现在是时候回到错误处理的主题了。获取单个故事时，你只需忽略任何错误。但是，在 stories() 中，让我们看看你可以做更多的事情。

API.Error 是你将限制从 stories() 抛出的错误的错误类型。你有两个错误定义为枚举案例：

* invalidResponse：当你无法将服务器响应解码为预期类型时。
* addressUnreachable(URL)：当你无法访问 URL 时。

目前，你在 stories() 中的订阅代码可能会引发两种类型的错误：

* 发生网络问题时，dataTaskPublisher(for:) 可能会引发 URLError 的不同变体。
* 当 JSON 与预期类型不匹配时，decode(type:decoder:) 可能会引发解码错误。

你的下一个任务是以一种将它们映射到单个 API.Error 类型的方式处理这些不同的错误，以匹配返回的发布者的预期失败。

将此代码附加到你当前的订阅中：

```
.mapError { error -> API.Error in  switch error {  case is URLError:    return Error.addressUnreachable(EndPoint.stories.url)  default:    return Error.invalidResponse  }}
```

mapError 处理上游发生的任何错误，并允许你将它们映射为单个错误类型——类似于你使用 map 更改输出类型的方式。

在上面的代码中，你切换任何错误并：

* 如果错误是 URLError 类型，因此在尝试到达故事服务器端点时发生，则返回 .addressUnreachable(\_)。
* 否则，你返回 .invalidResponse 作为可能发生错误的唯一其他地方。成功获取后，网络响应将解码 JSON 数据。

这样，你在 stories() 中匹配了预期的失败类型，并可以将其留给 API 使用者来处理下游错误。你将在下一章中使用 stories()。

到目前为止，当前订阅从 JSON API 获取 id 列表，但除此之外并没有做太多事情。接下来，你将使用一些操作符来过滤不需要的内容并将 id 列表映射到实际 Story。

首先，过滤空结果——以防 API 出错并为其最新故事返回一个空列表。附加：

```
.filter { !$0.isEmpty }
```

这将保证下游操作符收到包含至少一个元素的 Story ID 列表。 这非常方便，因为你记得，mergedStories(ids:) 有一个前提条件，确保其输入参数不为空。

要使用 mergeStories(ids:) 并获取故事详细信息，你将通过附加一个 flatMap 操作符来展平所有 Story 发布者：

```
.flatMap { storyIDs in  return self.mergedStories(ids: storyIDs)}
```

将所有发布者合并到一个下游将产生连续的 Story 值流。 发布者从网络中获取它们后立即向下游发出这些：

![image-20221010014903842](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010014903842.png)

你可以保留当前订阅，但你希望将 API 设计为可轻松绑定到列表 UI 控件。这将允许消费者简单地订阅 stories() 并将结果分配给他们的视图控制器或 SwiftUI 视图中的 \[Story] 属性。

为了实现这一点，你需要聚合发出的 Story 并映射订阅以返回一个不断增长的数组——而不是单个 Story 值。

附加到你当前的订阅：

```
.scan([]) { stories, story -> [Story] in  return stories + [story]}
```

你让 scan(...) 从一个空数组开始发射。每次发出新 Story 时，你都可以通过 Story + \[Story] 将其附加到当前聚合结果中。

订阅代码的这一添加改变了它的行为，这样每次你从你正在处理的批次中收到一个新 Story 时，你都会得到某种缓冲的内容：

![image-20221010015109452](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010015109452.png)

最后，在发出输出之前对 Story 进行排序不会有什么坏处。 Story 符合 Comparable，因此你不需要实现任何自定义排序。 你只需要对结果调用 sorted() 即可。 附加：

```
.map { $0.sorted() }
```

通过擦除返回的发布者的类型来结束当前相当长的订阅。 附加最后一个操作符：

```
.eraseToAnyPublisher()
```

此时可以找到如下临时return语句，将其删除：

```
return Empty().eraseToAnyPublisher()
```

你的游乐场现在应该最终编译没有错误。 但是，它仍然显示上一章部分的测试数据。 查找并注释掉：

```
api.mergedStories(ids: [1000, 1001, 1002])   .sink(receiveCompletion: { print($0) },         receiveValue: { print($0) })   .store(in: &subscriptions)
```

在此处添加：

```
api.stories()   .sink(receiveCompletion: { print($0) },         receiveValue: { print($0) })   .store(in: &subscriptions)
```

此代码订阅 api.stories() 并打印任何返回的输出和完成事件。

一旦你让 Playground 再运行一次，你应该会在控制台中看到最新的 Hacker News Story 。 你迭代地打印列表。 最初，你将看到首先获取的 Story：

```
[More than 70% of America’s packaged food supply is ultra-processedby xbetahttps://news.northwestern.edu/stories/2019/07/us-packaged-food-supply-is-ultra-processed/-----]
```

然后：

```
[More than 70% of America’s packaged food supply is ultra-processedby xbetahttps://news.northwestern.edu/stories/2019/07/us-packaged-food-supply-is-ultra-processed/-----, New AI project expects to map all the word’s reefs by end of next yearby Biba89https://www.independent.co.uk/news/science/coral-bleaching-ai-reef-paul-allen-climate-a9022876.html-----]
```

依此类推：

```
[More than 70% of America’s packaged food supply is ultra-processedby xbetahttps://news.northwestern.edu/stories/2019/07/us-packaged-food-supply-is-ultra-processed/-----, New AI project expects to map all the word’s reefs by end of next yearby Biba89https://www.independent.co.uk/news/science/coral-bleaching-ai-reef-paul-allen-climate-a9022876.html-----, People forged judges’ signatures to trick Google into changing resultsby lnguyenhttps://arstechnica.com/tech-policy/2019/07/people-forged-judges-signatures-to-trick-google-into-changing-results/-----]
```

请注意，由于你是从 Hacker News 网站获取 Story 的实时数据，因此随着每隔几分钟添加越来越多的 Story，你在控制台中看到的内容会有所不同。 要查看你确实在获取实时数据，请等待几分钟并重新运行 Playground。 你应该会看到一些新 Story 出现在你已经看过的 Story旁边。

完成本章稍长部分的工作真是太棒了！ 你已经完成了 Hacker News API 客户端的开发，并准备好进入下一节。 在那里，你将使用 SwiftUI 构建一个合适的 Hacker News 阅读器应用程序。

#### 挑战

API 客户端本身没有什么可添加的，但如果你想在本节的项目中投入更多的工作，你仍然可以尝试一下。

**挑战 1：将 API 客户端与 UIKit 集成**

如前所述，在下一章中，你将了解 SwiftUI 以及如何将其与你的组合代码集成。

在这个挑战中，尝试构建一个 iOS 应用程序，该应用程序使用已完成的 API 客户端在表格视图中显示最新故事。你可以根据需要开发尽可能多的细节并添加一些样式或有趣的功能，但在这个挑战中练习的重点是订阅 API.stories() 并将结果绑定到表格视图 - 就像你在其中所做的绑定一样第 8 章，“实践：‘拼贴’项目。”

如果你对使用 UIKit 不感兴趣 - 不用担心，这个挑战只是一个练习，你也可以跳过并首先进入第 15 节，“实践：Combine & SwiftUI”。

如果你按照描述成功完成挑战，当你在模拟器或设备上启动应用程序时，你应该会看到“涌入”的最新 Story：

![image-20221010015933003](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221010015933003.png)

#### 关键点

* Foundation 包括几个发布者，它们反映了 Swift 标准库中的对应方法，你甚至可以像在本章中使用 reduce 一样互换使用它们。
* 许多预先存在的 API，例如 Decodable，也集成了 Combine 支持。这使你可以在所有代码中使用一种标准方法。
* 通过组合组合操作符链，你可以以简化且易于遵循的方式执行相当复杂的操作 - 尤其是与预组合 API 相比！

#### 接下来去哪儿？

恭喜你完成“Combine 的行为”部分！这是一次多么美妙的旅程。你已经了解了 Combine 的基础所提供的大部分内容，因此现在是时候在专门讨论 Combine 框架中的高级主题的整个部分中拿出大手笔了，从构建一个同时使用 SwiftUI 和结合。
