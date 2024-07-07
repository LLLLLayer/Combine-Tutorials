# 第 16 节：错误处理

### 第 16 节：错误处理

你已经了解了很多关于如何编写 Combine 代码以随着时间的推移发出值的知识。不过，你可能已经注意到一件事：到目前为止，在你编写的大部分代码中，你根本没有处理错误，而主要处理的是“happy path”。

正如你在第 1 节“你好，Combine！”中所了解的，Combine 发布者声明了两个通用约束：Output，定义发布者发出的值的类型，以及 Failure，定义发布者可以完成的失败类型。

到目前为止，你已经将精力集中在发布者的输出类型上，但未能深入了解失败在发布者中的作用。这一节会改变这一点！

#### 入门

在 projects/Starter.playground 中打开本章的 Playground。你将使用此 Playground 及其各个页面来试验 Combine 让你处理和操作错误的多种方式。

你现在已准备好深入研究 Combine 中的错误，但首先，请花点时间思考一下。错误是一个广泛的话题，你会从哪里开始呢？从没有错误开始。

#### Never

失败类型为 Never 的发布者表示发布者永远不会失败。

虽然这乍一看可能有点奇怪，但它为这些发布者提供了一些极其强大的保证。具有永不失败类型的发布者可让你专注于使用发布者的值，同时绝对确保发布者永远不会失败，只有成功完成。

![image-20221019011943657](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221019011943657.png)

通过按 Command-1 在初学者 Playground 中打开 Project Navigator，然后选择 Never Playground 页面。将以下示例添加到其中：

```
example(of: "Never sink") {  Just("Hello")}
```

你创建了一个带有 Hello 字符串值的 Just。 Just 是从不失败的。 要确认这一点，请按住 Command 并单击 Just 初始化程序并选择 Jump to Definition，查看定义，你可以看到 Just 失败的类型别名： 公共类型别名失败 = 从不 Combine 对 Never 的无故障保证不仅仅是理论上的，而是深深植根于框架及其各种 API 中。

Combine 提供了几个操作符，这些操作符仅在保证发布者永远不会失败时才可用。 第一个是 sink 的变体，只处理值。

返回 Never Playground 页面并更新上面的示例，使其看起来像这样：

```
example(of: "Never sink") {  Just("Hello")    .sink(receiveValue: { print($0) })    .store(in: &subscriptions)}
```

运行你的 Playground，你会看到 Just 的值被打印出来：

```
——— Example of: Never sink ———Hello
```

在上面的示例中，你使用 `sink(receiveValue:)`。 这种特定的 sink 重载使你可以忽略发布者的完成事件，而只处理其发出的值。

此重载仅适用于可靠的发布者。在错误处理方面，Combine 是智能且安全的，如果可能抛出错误，它会强制你处理完成事件——即对于非失败的发布者。

要看到这一点，你需要将永不失败的发布者变成可能失败的发布者。 有几种方法可以做到这一点，你将从最流行的一种开始 `setFailureType` 操作符。

**setFailureType**

将可靠的发布者转变为可靠的发布者的第一种方法是使用 `setFailureType`。这是另一个仅适用于失败类型为 Never 的发布者的操作符。

将以下代码和示例添加到你的 Playground 页面：

```
enum MyError: Error {  case ohNo}​example(of: "setFailureType") {  Just("Hello")}
```

首先定义示例范围之外的 `MyError` 错误类型。 稍后你将重用此错误类型。 然后，你通过创建一个与你之前使用的类似的 Just 来开始该示例。

现在，你可以使用 `setFailureType` 将发布者的失败类型更改为 `MyError`。在 Just 之后立即添加以下行：

```
.setFailureType(to: MyError.self)
```

要确认这实际上改变了发布者的失败类型，请开始输入 .eraseToAnyPublisher()，自动完成将显示已擦除的发布者类型：

![image-20221019013127326](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221019013127326.png)

在继续之前删除你开始输入的 .erase... 行。

现在是时候使用 sink 来消费发布者了。 在最后一次调用 setFailureType 后立即添加以下代码：

```
// 1.sink(  receiveCompletion: { completion in    switch completion {    // 2    case .failure(.ohNo):      print("Finished with Oh No!")    case .finished:      print("Finished successfully!")    }  },  receiveValue: { value in    print("Got value: \(value)")  }).store(in: &subscriptions)
```

你可能已经注意到关于上述代码的两个有趣的事实：

1. 它正在使用 `sink(receiveCompletion:receiveValue:)`。 `sink(receiveValue:)` 重载不再可用，因为此发布者可能会以失败事件完成。 结合迫使你处理此类发布者的完成事件。
2. 失败类型被严格键入为 `MyError`，这使你可以针对`.failure(.ohNo)` 情况而无需进行不必要的强制转换来处理该特定错误。

运行你的 Playground，你会看到以下输出：

```
——— Example of: setFailureType ———Got value: HelloFinished successfully!
```

当然，setFailureType 的作用只是类型系统定义。 由于原始发布者是 Just，因此实际上不会引发任何错误。

在本章后面，你将了解更多关于如何从你自己的发布者那里实际产生错误的信息。 但首先，还有一些专门针对永不失败的发布者的操作符。

**`assign(to:on:)`**

**你在第 2 节“发布者和订阅者”中学到的 assign 操作符仅适用于不会失败的发布者，与 `setFailureType` 相同。 如果你仔细想想，这完全有道理。 向提供的 key path 发送错误会导致未处理的错误或未定义的行为。**

**添加以下示例进行测试：**

```
example(of: "assign(to:on:)") {  // 1  class Person {    let id = UUID()    var name = "Unknown"  }  // 2  let person = Person()  print("1", person.name)  Just("Shai")    .handleEvents( // 3      receiveCompletion: { _ in print("2", person.name) }    )    .assign(to: \.name, on: person) // 4    .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **定义一个具有 id 和 name 属性的 Person 类。**
2. **创建一个 Person 实例并立即打印其名称。**
3. **一旦发布者发送完成事件，使用你之前了解的handleEvents 再次打印此人的 name。**
4. **最后，使用assign 将人名设置为发布者发出的任何内容。**

**运行你的 Playground 并查看调试控制台：**

```
——— Example of: assign(to:on:) ———1 Unknown2 Shai
```

**正如预期的那样，只要 Just 发出它的值，assign 就会更新这个人的名字，这是有效的，因为 Just 不会失败。 相反，如果发布者有一个非从不失败的类型，你认为会发生什么？**

**在 Just("Shai") 正下方添加以下行：**

```
.setFailureType(to: Error.self)
```

**在此代码中，你已将失败类型设置为标准 Swift 错误。 这意味着它不再是 `Publisher<String, Never>`，而是现在的 `Publisher<String, Error>`。**

**尝试运行你的 Playground 。 对于手头的问题，Combine 非常冗长：**

```
referencing instance method 'assign(to:on:)' on 'Publisher' requires the types 'Error' and 'Never' be equivalent
```

**删除你刚刚添加的对 setFailureType 的调用，并确保你的 Playground 运行时没有编译错误。**

**`assign(to:)`**

**`assign(to:on:)` 有一个棘手的部分——它会 strong 地捕获提供给 on 参数的对象。**

**让我们探讨一下为什么这是有问题的。**

**在上一个示例之后立即添加以下代码：**

```
example(of: "assign(to:)") {  class MyViewModel: ObservableObject {    // 1    @Published var currentDate = Date()    init() {      Timer.publish(every: 1, on: .main, in: .common) // 2        .autoconnect()         .prefix(3) // 3        .assign(to: \.currentDate, on: self) // 4        .store(in: &subscriptions)    }  }  // 5  let vm = MyViewModel()  vm.$currentDate    .sink(receiveValue: { print($0) })    .store(in: &subscriptions)}
```

**这段代码有点长，让我们分解一下：**

1. **在视图模型对象中定义一个@Published 属性。 它的初始值为当前日期。**
2. **创建一个计时器发布者，它每秒发出当前日期。**
3. **使用前缀操作符只接受 3 个日期更新。**
4. **应用 `assign(to:on:)` 操作符将每个日期更新分配给你的 @Published 属性。**
5. **实例化你的视图模型，sink 已发布的发布者，并打印出每个值。**

**如果你运行 Playground，你将看到类似于以下内容的输出：**

```
——— Example of: assign(to:on:) strong capture ———2021-08-21 12:43:32 +00002021-08-21 12:43:33 +00002021-08-21 12:43:34 +00002021-08-21 12:43:35 +0000
```

**正如预期的那样，上面的代码打印分配给发布属性的初始日期，然后连续更新 3 次（受前缀操作符限制）。**

**看起来，一切都很好，那么这里到底出了什么问题呢？**

**对`assign(to:on:)` 的调用创建了一个 strongly retains self 的订阅。 本质上——self 挂在订阅上，而订阅挂在 self 上，创建了一个导致内存泄漏的保留周期。**

![image-20221019015251780](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221019015251780.png)

**幸运的是，Apple 的好人意识到这是有问题的，并引入了该操作符的另一个重载 - `assign(to:)`。**

**该操作符专门处理通过提供对其投影发布者的 inout 引用来将发布的值重新分配给 @Published 属性。**

**回到示例代码，找到以下两行：**

```
.assign(to: \.currentDate, on: self) // 3.store(in: &subscriptions)
```

**并将它们替换为以下行：**

```
.assign(to: &$currentDate)
```

**使用 `assign(to:)` 操作符并将 inout 引用传递给预计的发布者会打破保留周期，让你轻松处理上述问题。**

**此外，它会在内部自动处理订阅的内存管理，这样你就可以省略 `store(in: &subscriptions)` 行。**

> **注意：在继续之前，建议将前面的示例注释掉，这样打印出来的计时器事件就不会给控制台输出添加不必要的误导。**

**在这一点上，你几乎完成了可靠的出版商。但是在开始处理错误之前，你应该知道与可靠发布者相关的最后一个操作符：`assertNoFailure`。**

**`assertNoFailure`**

**当你想在开发过程中保护自己并确认发布者以成失败事件完成时，`assertNoFailure` 操作符非常有用。它不会阻止上游发出失败事件。但是，如果它检测到错误，它会因致命错误而崩溃，这给了你在开发中修复它的良好动力。**

**将以下示例添加到你的 playground：**

```
example(of: "assertNoFailure") {  // 1  Just("Hello")    .setFailureType(to: MyError.self)    .assertNoFailure() // 2    .sink(receiveValue: { print("Got value: \($0) ")}) // 3    .store(in: &subscriptions)}
```

**在前面的代码中：**

1. **使用 Just 创建一个可靠的发布者并将其失败类型设置为 MyError。**
2. **如果发布者以失败事件完成，则使用 assertNoFailure 以致命错误崩溃。 这会将发布者的失败类型转回 Never。**
3. **使用 sink 打印出任何接收到的值。 请注意，由于 assertNoFailure 将失败类型设置回 Never，因此 sink(receiveValue:) 重载再次由你使用。**

**运行你的 Playground，正如预期的那样，它应该可以正常工作：**

```
——— Example of: assertNoFailure ———Got value: Hello 
```

**现在，在 setFailureType 之后，添加以下行：**

```
.tryMap { _ in throw MyError.ohNo }
```

**一旦 Hello 被推送到下游，你刚刚使用 tryMap 引发错误。 你将在本章后面了解更多关于以 try 为前缀的操作符。**

**再次运行你的 Playground 并查看控制台。 你将看到类似于以下内容的输出：**

```
Playground execution failed:​error: Execution was interrupted, reason: EXC_BAD_INSTRUCTION (code=EXC_I386_INVOP, subcode=0x0).​...​frame #0: 0x00007fff232fbbf2 Combine`Combine.Publishers.AssertNoFailure...
```

**由于发布者 failure ，playground 崩溃。 在某种程度上，你可以将 `assertFailure()` 视为代码的保护机制。 虽然你不应该在生产中使用它，但在开发过程中“早早崩溃并严重崩溃”非常有用。**

**在继续下一部分之前注释掉对 tryMap 的调用。**

#### **处理失败**

**哇，到目前为止，你已经在错误处理一章中学到了很多关于如何处理根本不会失败的发布者的知识！ :] 虽然有点讽刺，但我希望你现在能够理解彻底了解可靠发布者的特征和保证是多么重要。**

**考虑到这一点，是时候让你了解一下 Combine 提供的一些技术和工具，以应对实际失败的发布者。这包括内置发布者和你自己的发布者！**

**但首先，你实际上是如何产生失败事件的？如上一节所述，有几种方法可以做到这一点。你刚刚使用了 tryMap，那么为什么不进一步了解这些 try 操作符的工作原理呢？**

**try\* 操作符**

**在“操作符”中，你了解了大部分 Combine 的操作符以及如何使用它们来操纵发布者发出的值和事件。你还学习了如何组合多个操作符的逻辑链来产生所需的输出。**

**在这些章节中，你了解到大多数操作符都有以 try 为前缀的并行操作符，我们现在将了解他们。**

**Combine 提供了一个有趣的区分可能引发错误和可能不会引发错误的操作符。**

> **注意：Combine 中所有以 try 为前缀的操作符在遇到错误时的行为方式相同。你将只在本章中尝试使用 tryMap 操作符。**

**首先，从 Project navigator 中选择 try operator\* playground 页面。向其中添加以下代码：**

```
example(of: "tryMap") {  // 1  enum NameError: Error {    case tooShort(String)    case unknown  }  // 2  let names = ["Marin", "Shai", "Florent"].publisher  names    // 3    .map { value in      return value.count    }    .sink(      receiveCompletion: { print("Completed with \($0)") },      receiveValue: { print("Got value: \($0)") }    )}
```

**在上面的示例中：**

1. **定义一个 NameError 错误枚举，你将立即使用它。**
2. **创建发布三个不同字符串的发布者。**
3. **将每个字符串映射到它的长度。**

**运行示例并查看控制台输出：**

```
——— Example of: tryMap ———Got value: 5Got value: 4Got value: 7Completed with finished
```

**正如预期的那样，所有名称都映射没有问题。 但是随后你收到了一个新的产品要求：如果你的代码接受的名称少于 5 个字符，则它应该引发错误。**

**将上面示例中的 map 替换为以下内容：**

```
.map { value -> Int in  // 1  let length = value.count  // 2  guard length >= 5 else {    throw NameError.tooShort(value)  }  // 3  return value.count}
```

**在上面的映射中，你检查字符串的长度是否大于或等于 5。否则，你会尝试抛出适当的错误。**

**但是，只要你添加上述代码或尝试运行它，你就会看到编译器会产生错误：**

```
Invalid conversion from throwing function of type '(_) throws -> _' to non-throwing function type '(String) -> _'
```

**由于 map 是一个非抛出操作符，因此你不能从其中抛出错误。 幸运的是，try\* 操作符就是为此目的而设计的。**

**用 tryMap 替换 map 并再次运行你的 Playground。 它现在将编译并产生以下输出（截断）：**

```
——— Example of: tryMap ———Got value: 5Got value: 5Completed with failure(...NameError.tooShort("Shai"))
```

**映射错误**

**map 和 tryMap 之间的区别不仅仅是后者允许抛出错误。 虽然 map 继承了现有的失败类型并且只操作发布者的值，但 tryMap 没有——它实际上将错误类型擦除为普通的 Swift 错误。 与带有 try 前缀的对应物相比，所有操作符都是如此。**

**切换到 Mapping errors playground 页面并在其中添加以下代码：**

```
example(of: "map vs tryMap") {  // 1  enum NameError: Error {    case tooShort(String)    case unknown  }  // 2  Just("Hello")    .setFailureType(to: NameError.self) // 3    .map { $0 + " World!" } // 4    .sink(      receiveCompletion: { completion in        // 5        switch completion {        case .finished:          print("Done!")        case .failure(.tooShort(let name)):          print("\(name) is too short!")        case .failure(.unknown):          print("An unknown name error occurred")        }      },      receiveValue: { print("Got value \($0)") }    )    .store(in: &subscriptions)}
```

**在上面的示例中：**

1. **定义一个用于此示例的 NameError。**
2. **创建一个只发出字符串 Hello 的 Just。**
3. **使用 setFailureType 设置失败类型为 NameError。**
4. **使用 map 将另一个字符串附加到已发布的字符串。**
5. **最后，使用 sink 的 receiveCompletion 为 NameError 的每个失败情况打印出适当的消息。**

**运行 Playground，你将看到以下输出：**

```
——— Example of: map vs tryMap ———Got value Hello World!Done!
```

**接下来，找到 switch completion { 行，Option-click 点击 completion：**

![image-20221023004505006](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023004505006.png)

**请注意，Completion 的失败类型是 NameError，这正是你想要的。 setFailureType 操作符允许你专门针对 NameError 故障，例如 failure(.tooShort(let name))。**

**接下来，将 map 更改为 tryMap。你会立即注意到操场不再编译。 Option-click 再次点击 completion：**

![image-20221023004624718](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023004624718.png)

**很有意思！ tryMap 删除了你的严格类型错误并将其替换为通用 Swift.Error 类型。即使你实际上并没有从 tryMap 中抛出错误，也会发生这种情况——你只是使用了它！这是为什么？**

**仔细想想，原因很简单：Swift 还不支持类型化 throws，尽管自 2015 年以来 Swift Evolution 中一直在讨论这个主题。这意味着当你使用带有 try 前缀的操作符时，你的错误类型将总是被抹去到最常见的祖先：Swift.Error。**

**所以你能对它做点啥？对于发布者来说，严格类型的失败的全部意义在于让你处理——在这个例子中——特别是 NameError，而不是任何其他类型的错误。**

**一种天真的方法是将通用错误手动转换为特定的错误类型，但这不是最理想的。它打破了严格类型错误的整个目的。幸运的是，Combine 为这个问题提供了一个很好的解决方案，称为 mapError。**

**在调用 tryMap 之后，立即添加以下行：**

```
.mapError { $0 as? NameError ?? .unknown }
```

**mapError 接收上游发布者抛出的任何错误，并让你将其映射到你想要的任何错误。在这种情况下，你可以利用它将错误转换回 NameError 或回退到 NameError.unknown 错误。在这种情况下，你必须提供一个回退错误，因为从理论上讲，强制转换可能会失败——即使它不会在这里——并且你必须从此操作符返回 NameError。**

**这会将 Failure 恢复为其原始类型，并将你的发布者转回 Publisher\<String, NameError>。**

**构建并运行 Playground。它最终应该可以按预期编译和工作：**

```
——— Example of: map vs tryMap ———Got value Hello World!Done!
```

**最后，将对 tryMap 的整个调用替换为：**

```
.tryMap { throw NameError.tooShort($0) }
```

**此调用将立即从 tryMap 中引发错误。再次检查控制台输出，并确保你得到正确输入的 NameError：**

```
——— Example of: map vs tryMap ———Hello is too short!
```

**设计自己的可能会发生错误的 API**

**在构建你自己的基于 Combine 的代码和 API 时，你通常会使用来自其他来源的 API，这些 API 会返回因各种类型而失败的发布者。在创建自己的 API 时，你通常还希望围绕该 API 提供自己的错误。尝试这个比仅仅理论化更容易，所以你将继续深入研究一个例子！**

**在本节中，你将构建一个快速 API，让你可以从** [**https://icanhazdadjoke.com/api**](https://icanhazdadjoke.com/api) **上的 icanhazdadjoke API 获取一些有趣的笑话。**

**首先切换到 Designing your fallible APIs playground 页面，并向其中添加以下代码，这构成了下一个示例的第一部分：**

```
example(of: "Joke API") {  class DadJokes {    // 1    struct Joke: Codable {      let id: String      let joke: String    }​    // 2    func getJoke(id: String) -> AnyPublisher<Joke, Error> {      let url = URL(string: "https://icanhazdadjoke.com/j/\(id)")!      var request = URLRequest(url: url)      request.allHTTPHeaderFields = ["Accept": "application/json"]            // 3      return URLSession.shared        .dataTaskPublisher(for: request)        .map(\.data)        .decode(type: Joke.self, decoder: JSONDecoder())        .eraseToAnyPublisher()    }​  }}
```

**在上面的代码中，你通过以下方式创建了新 DadJokes 类实例：**

1. **定义一个 Joke 结构。 API 响应将被解码为 Joke 的一个实例。**
2. **提供一个 getJoke(id:) 方法，该方法当前返回一个发布者，该发布者发出一个 Joke，并且可能会因标准 Swift.Error 而失败。**
3. **使用 URLSession.dataTaskPublisher(for:) 调用 icanhazdadjoke API 并使用 JSONDecoder 和 decode 操作符将结果数据解码为 Joke。**

**最后，你需要实际使用你的新 API。 在 DadJokes 类下面直接添加以下内容，但仍在示例范围内：**

```
// 4let api = DadJokes()let jokeID = "9prWnjyImyd"let badJokeID = "123456"​// 5api  .getJoke(id: jokeID)  .sink(receiveCompletion: { print($0) },        receiveValue: { print("Got joke: \($0)") })  .store(in: &subscriptions)
```

**在此代码中：**

1. **创建一个 DadJokes 的实例，并定义具有有效和无效笑话 ID 的两个常量。**
2. **使用有效的笑话 ID 调用 DadJokes.getJoke(id:) 并打印任何完成事件或解码的笑话本身。**

**运行你的 Playground 并查看控制台：**

```
——— Example of: Joke API ———Got joke: Joke(id: "9prWnjyImyd", joke: "Why do bears have hairy coats? Fur protection.")finished
```

**所以你的 API 目前完美地处理了正常路径，但这是一个错误处理章节。 在包装其他发布者时，你需要问自己：“这个特定发布者会导致哪些错误？”**

**在这种情况下：**

* **调用 dataTaskPublisher 可能会因各种原因失败并返回 URLError，例如连接错误或请求无效。**
* **提供的笑话 ID 可能不存在。**
* **如果 API 响应更改或其结构不正确，解码 JSON 响应可能会失败。**
* **任何其他未知错误！ 错误很多而且是随机的，因此不可能考虑每个边缘情况。 出于这个原因，你总是希望有一个案例来涵盖未知或未处理的错误。**

**记住这个列表，在 DadJokes 类中添加以下代码，紧邻 Joke 结构体下方：**

```
enum Error: Swift.Error, CustomStringConvertible { // 1  case network  case jokeDoesntExist(id: String)  case parsing  case unknown  // 2  var description: String {    switch self {    case .network:      return "Request to API Server failed"    case .parsing:      return "Failed parsing response from server"    case .jokeDoesntExist(let id):      return "Joke with ID \(id) doesn't exist"    case .unknown:      return "An unknown error occurred"    }  }}
```

**此错误定义：**

1. **概述 DadJokes API 中可能出现的所有错误。**
2. **符合 CustomStringConvertible，让你可以为每个错误情况提供友好的描述。**

**添加上述错误类型后，你的 Playground 将不再编译。 这是因为 getJoke(id:) 返回一个 AnyPublisher\<Joke, Error>。 之前，Error 指的是 Swift.Error，但现在它指的是 DadJokes.Error——在这种情况下，这实际上是你想要的。**

**那么，你怎么能把各种可能的和不同类型的错误都映射到你的 DadJoke.Error 中呢？ 如果你一直在关注本章，你可能已经猜到了答案：mapError 是你的朋友。**

**在对 decode 和 eraseToAnyPublisher() 的调用之间将以下内容添加到 getJoke(id:) 中：**

```
.mapError { error -> DadJokes.Error in  switch error {  case is URLError:    return .network  case is DecodingError:    return .parsing  default:    return .unknown  }}
```

**这个简单的 mapError 使用 switch 语句将发布者可能抛出的任何类型的错误替换为 DadJokes.Error。 你可能会问自己：“我为什么要包装这些错误？” 这个问题的答案有两个：**

1. **现在保证你的发布者只会因 DadJokes.Error 而失败，这在使用 API 和处理可能出现的错误时很有用。 你确切地知道你将从类型系统中得到什么。**
2. **你不会泄露 API 的实现细节。 想一想，如果你使用 URLSession 执行网络请求并使用 JSONDecoder 解码响应，那么你的 API 的使用者是否关心？ 明显不是！ 消费者只关心你的 API 本身定义的错误——而不关心它的内部依赖。**

**还有一个你没有处理的错误：一个不存在的笑话 ID。 尝试替换以下行：**

```
.getJoke(id: jokeID)
```

**为：**

```
.getJoke(id: badJokeID)
```

**再次运行 Playground。 这一次，你将收到以下错误：**

```
failure(Failed parsing response from server)
```

**有趣的是，当你发送一个不存在的 ID 时，icanhazdadjoke 的 API 并不会因 HTTP 代码 404（未找到）而失败——正如大多数 API 所期望的那样。 相反，它会发回一个不同但有效的 JSON 响应：**

```
{    message = "Joke with id \"123456\" not found";    status = 404;}
```

**处理这种情况需要一些技巧，但绝对不是你不能处理的！**

**回到 getJoke(id:)，用以下代码替换对 map(.data) 的调用：**

```
.tryMap { data, _ -> Data in  // 6  guard let obj = try? JSONSerialization.jsonObject(with: data),        let dict = obj as? [String: Any],        dict["status"] as? Int == 404 else {    return data  }  // 7  throw DadJokes.Error.jokeDoesntExist(id: id)}
```

**在上面的代码中，你使用 tryMap 在将原始数据传递给解码操作符之前执行额外的验证：**

1. **你使用 JSONSerialization 来尝试检查状态字段是否存在且值为 404 — 即，笑话不存在。 如果不是这种情况，你只需返回数据，以便将其推送到下游的解码操作员。**
2. **如果你确实找到了 404 状态码，你会抛出一个 .jokeDoesntExist(id:) 错误。**

**再次运行你的 Playground，你会发现另一个需要解决的小问题：**

```
——— Example of: Joke API ———failure(An unknown error occurred)
```

**失败实际上被视为未知错误，而不是 DadJokes.Error，因为你没有在 mapError 中处理该类型。在你的 mapError 中，找到以下行：**

```
return .unknown
```

**替换为：**

```
return error as? DadJokes.Error ?? .unknown
```

**如果没有其他错误类型匹配，你尝试将其强制转换为 DadJokes.Error，然后放弃并退回到未知错误。**

**再次打开你的操场并查看控制台：**

```
——— Example of: Joke API ———failure(Joke with ID 123456 doesn't exist)
```

**这一次，你收到正确的错误，类型正确！ 惊人的。 :]**

**在结束本示例之前，你可以在 getJoke(id:) 中进行最后一项优化。**

**你可能已经注意到，笑话 ID 由字母和数字组成。 在我们的“Bad ID”的情况下，你只发送了数字。 无需执行网络请求，你可以抢先验证你的 ID 并在不浪费资源的情况下失败。**

**在 getJoke(id:) 的开头添加以下最后一段代码：**

```
guard id.rangeOfCharacter(from: .letters) != nil else {  return Fail<Joke, Error>(    error: .jokeDoesntExist(id: id)  )  .eraseToAnyPublisher()}
```

**在此代码中，你首先要确保 id 至少包含一个字母。如果不是这种情况，你会立即返回 Fail。**

**Fail 是一种特殊的发布者，它可以让你立即且强制地失败并显示提供的错误。它非常适合你希望根据某些条件提前失败的情况。最后，你使用 eraseToAnyPublisher 获得预期的 AnyPublisher\<Joke, DadJokes.Error> 类型。**

**使用无效的 ID 再次运行你的示例，你将收到相同的错误消息。但是，它会立即发布并且不会执行网络请求。巨大的成功！**

**在继续之前，将你的调用恢复为 getJoke(id:) 以使用 jokeID 而不是 badJokeId。**

**此时，你可以通过手动“破坏”你的代码来验证你的错误逻辑。执行以下每个操作后，撤消你的更改，以便你可以尝试下一个操作：**

1. **当你在上面创建 URL 时，在其中添加一个随机字母以破坏 URL。运行 Playground，你会看到：失败（对 API 服务器的请求失败）。**
2. **注释掉以 request.allHttpHeaderFields 开头的行并运行 Playground。由于服务器响应将不再是 JSON，而只是纯文本，因此你将看到输出：失败（来自服务器的解析响应失败）。**

**像以前一样，向 getJoke(id:) 发送一个随机 ID。运行 Playground，你会得到：**

```
failure(Joke with ID {your ID} doesn't exist).
```

**就是这样！你刚刚构建了自己的基于 Combine 的生产级 API 层，其中包含自己的错误。**

**捕获并重试**

**你学到了很多关于 Combine 代码错误处理的知识，但我们将最好的知识留到了最后，还有两个最终主题：捕获错误和重试失败的发布者。**

**Publisher 是一种表示工作的统一方式的好处在于，你拥有许多操作符，可以让你用很少的代码行完成大量工作。**

**继续并直接进入示例。**

**首先切换到 Project navigator 中的 Catching and retrying 页面。 展开 Playground 的 Sources 文件夹并打开 PhotoService.swift。**

**它包括一个带有 fetchPhoto(quality:failingTimes:) 方法的 PhotoService，你将在本节中使用该方法。 PhotoService 使用自定义发布者获取高质量或低质量的照片。 对于这个例子，要求一个高质量的图像总是会失败——所以你可以**

**尝试各种技术以重试并在发生故障时捕获故障。**

**返回到 Catching and retrying playground 页面，并将这个简单的示例添加到你的 Playground：**

```
let photoService = PhotoService()​example(of: "Catching and retrying") {  photoService    .fetchPhoto(quality: .low)    .sink(      receiveCompletion: { print("\($0)") },      receiveValue: { image in        image        print("Got image: \(image)")      }    )    .store(in: &subscriptions)}
```

**上面的代码现在应该很熟悉了。 你实例化一个 PhotoService 并以 .low 质量调用 fetchPhoto。 然后使用 sink 打印出任何完成事件或获取的图像。**

**请注意， photoService 的实例化超出了示例的范围，因此它不会立即被释放。**

**运行你的 Playground 并等待它完成。 你应该看到以下输出：**

```
——— Example of: Catching and retrying ———Got image: <UIImage:0x600000790750 named(lq.jpg) {300, 300}>finished
```

**点击receiveValue 中第一行旁边的显示结果按钮，你会看到一张漂亮的低质量图片......好吧，一个组合。**

![image-20221023015007259](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023015007259.png)

**接下来，将质量从 .low 更改为 .high 并再次运行 Playground。 你将看到以下输出：**

```
——— Example of: Catching and retrying ———failure(Failed fetching image with high quality)
```

**如前所述，要求高质量图像将失败。 这是你的起点！ 你可以在这里改进一些事情。 你将从重试失败开始。**

**很多时候，当你请求资源或执行某些计算时，失败可能是由于网络连接不良或其他资源不可用而导致的一次性事件。**

**在这些情况下，你通常会编写一个大型机制来重试不同的工作，同时跟踪尝试次数并决定如果所有尝试都失败了该怎么办。幸运的是，Combine 让这一切变得非常简单。**

**就像Combine 中的所有好东西一样，有一个操作符！**

**重试操作符接受一个数字。如果发布者失败，它将重新订阅上游并重试至你指定的次数。如果所有重试都失败，它只是将错误推送到下游，就像没有重试操作符一样。**

**是时候让你试试这个了。在 fetchPhoto(quality: .high) 行下方，添加以下行：**

```
.retry(3)
```

**等等，是这样吗？！是的。**

**对于包装在发布者中的每件工作，你都会获得一个免费的重试机制，就像调用这个简单的重试操作符一样简单。**

**在运行你的 Playground 之前，在 fetchPhoto 调用之间添加此代码并重试：**

```
.handleEvents(  receiveSubscription: { _ in print("Trying ...") },  receiveCompletion: {    guard case .failure(let error) = $0 else { return }    print("Got error: \(error)")  })
```

**此代码将帮助你查看何时发生重试 - 它打印出 fetchPhoto 中发生的订阅和失败。**

**现在你准备好了！ 运行你的 Playground 并等待它完成。 你将看到以下输出：**

```
——— Example of: Catching and retrying ———Trying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityfailure(Failed fetching image with high quality)
```

**如你所见，有四次尝试。 初始尝试，加上由重试操作符触发的三次重试。 由于获取高质量照片不断失败，因此操作员会耗尽所有重试尝试并将错误推送到 sink。**

**将以下调用替换为 fetchPhoto：**

```
.fetchPhoto(quality: .high)
```

**为：**

```
.fetchPhoto(quality: .high, failingTimes: 2)
```

**faliingTimes 参数将限制获取高质量图像失败的次数。 在这种情况下，它会在你调用它的前两次失败，然后成功。**

**再次运行你的 Playground，看看输出：**

```
——— Example of: Catching and retrying ———Trying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got image: <UIImage:0x600001268360 named(hq.jpg) {1835, 2446}>finished
```

**如你所见，这次有 3 次尝试，最初的 1 次加两次重试。该方法前两次尝试失败，然后成功并返回这张华丽的、高质量的田间联合收割机照片：**

![image-20221023015716082](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023015716082.png)

**惊人的！但是，在此服务调用中，你还需要改进最后一项功能。如果获取高质量图像失败，你的产品人员要求你回退到低质量图像。如果获取低质量图像也失败，你应该回退到硬编码图像。**

**你将从两项任务中的后者开始。 Combine 包含一个名为 replaceError(with:) 的便捷操作符，如果发生错误，你可以使用该操作符回退到发布者类型的默认值。这也会将发布者的失败类型更改为从不，因为你将所有可能的失败都替换为后备值。**

**首先，从 fetchPhoto 中删除 failedTimes 参数，因此它会像以前一样不断失败。**

**然后在调用重试之后立即添加以下行：**

```
.replaceError(with: UIImage(named: "na.jpg")!)
```

**再次运行你的 Playground，看看这次的图像结果。在四次尝试之后——即最初加上三次重试——你回到磁盘上的硬编码图像：**

![image-20221023015846417](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221023015846417.png)

**此外，查看控制台输出会显示你所期望的：有四次失败的尝试，然后是硬编码的后备图像：**

```
——— Example of: Catching and retrying ———Trying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityGot image: <UIImage:0x6000020e9200 named(na.jpg) {200, 200}>finished
```

**现在，对于本章的第二个任务和最后一部分：如果高质量图像失败，则回退到低质量图像。 Combine 为这项任务提供了完美的操作符，称为 catch。 它使你可以从发布者那里捕获故障并通过不同的发布者从中恢复。**

**要查看实际情况，请在重试之后、replaceError(with:) 之前添加以下代码：**

```
.catch { error -> PhotoService.Publisher in  print("Failed fetching high quality, falling back to low quality")  return photoService.fetchPhoto(quality: .low)}
```

**最后一次运行你的 Playground 并查看控制台：**

```
——— Example of: Catching and retrying ———Trying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityTrying ...Got error: Failed fetching image with high qualityFailed fetching high quality, falling back to low qualityGot image: <UIImage:0x60000205c480 named(lq.jpg) {300, 300}>finished
```

**和以前一样，初始尝试加上三次重试以获取高质量图像都失败了。 一旦操作员用尽了所有重试，catch 就会发挥作用并订阅 photoService.fetchPhoto，请求低质量的图像。 这导致从失败的高质量请求回退到成功的低质量请求。**

#### **关键点**

* **失败类型为 Never 的发布者保证不会发出失败完成事件。**
* **许多操作符只与可靠的发布者合作。例如：sink(receiveValue:)、setFailureType、assertNoFailure 和 assign(to**_**:on**_**:)。**
* **以 try 为前缀的操作符允许你从其中抛出错误，而非 try 操作符则不能。**
* **由于 Swift 不支持类型化的 throws，调用以 try 为前缀的操作符会将发布者的失败删除为普通的 Swift 错误。**
* **使用 mapError 映射发布者的失败类型，并将发布者中的所有失败类型统一为单一类型。**
* **当基于其他发布者使用自己的失败类型创建自己的 API 时，将所有可能的错误包装到自己的错误类型中以统一它们并隐藏 API 的实现细节。**
* **你可以使用重试操作符重新订阅失败的发布者多次。**
* **当你想为你的发布者提供一个默认的后备值时，replaceError(with:) 很有用，以防万一失败。**
* **最后，你可以使用 catch 将失败的发布者替换为不同的后备发布者。**

#### **接下来去哪儿？**

**恭喜你读完本章。你基本上已经掌握了有关 Combine 中的错误处理的所有知识。**

**你只在本章的 try\* 操作符部分中试验了 tryMap 操作符。你可以在** [**https://apple.co/3233VRB**](https://apple.co/3233VRB) **上的 Apple 官方文档中找到带有 try 前缀的操作符的完整列表。**

**随着你对错误处理的掌握，是时候了解 Combine 中较低级别但最重要的主题之一：调度程序。继续下一章，了解什么是调度程序以及如何使用它们。**
