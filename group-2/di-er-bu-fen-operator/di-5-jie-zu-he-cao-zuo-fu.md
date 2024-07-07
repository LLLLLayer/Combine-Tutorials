# 第 5 节：组合操作符

### 第 5 节：组合操作符

在本章中，你将了解一种更复杂但有用的操作类别：组合操作符。这组操作符允许你组合不同发布者发出的事件，并在你的组合代码中创建有意义的数据组合。

考虑一个包含多个用户输入的表单——一个用户名、一个密码和一个复选框。你需要将这些不同的数据组合起来，组成一个包含你需要的所有信息的发布者。

随着你更多地了解每个操作符的运作方式以及如何根据你的需求选择合适的操作符，你的代码将变得更加强大，你的技能将使你能够解锁新的发布者组合。

#### 入门

你可以在项目 /Starter.playground 文件夹中找到本章的 Playground，你将向 Playground 添加代码并运行它，以了解各种操作符如何创建发布者及其事件的不同组合。

#### 前置(Prepending)

你将在这里开始使用一组操作符，这些操作符都是关于在发布者开头添加值的。换句话说，你将使用它们在原始发布者发出任何值之前添加值。

在本节中，你将了解 `prepend(Output...)`、`prepend(Sequence)` 和 `prepend(Publisher)`。

**`prepend(Output...)`**

**使用 `...` 语法采用可变参数列表。 这意味着它可以采用任意数量的值，只要它们与原始发布者的输出类型相同。**

![image-20221002201211503](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002201211503.png)

**将以下代码添加到你的 Playground 以试验上述示例：**

```
example(of: "prepend(Output...)") {  // 1  let publisher = [3, 4].publisher​  // 2  publisher    .prepend(1, 2)    .sink(receiveValue: { print($0) })    .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **创建一个发布数字 3 4 的发布者。**
2. **使用 prepend 在发布者自己的值之前添加数字 1 和 2。**

**运行 Playground，你应该在调试控制台中看到以下内容：**

```
——— Example of: prepend(Output...) ———1234
```

**你还记得操作符是如何可链接的吗？ 这意味着你可以轻松添加多个前置。**

**在以下行下方：**

```
.prepend(1, 2)
```

**添加：**

```
.prepend(-1, 0)
```

**再次运行你的 Playground。你应该看到以下输出：**

```
——— Example of: prepend(Output...) ———-101234
```

**请注意，此处的操作顺序至关重要 -1 和 0 被前置到最前方，然后是 1 和 2，最后是原始发布者的值。**

**`prepend(Sequence)`**

**prepend 的这种变体与前一种类似，不同之处在于它将任何符合序列的对象作为输入。 例如，它可以采用 Array 或 Set。**

**将以下代码添加到你的 Playground：**

```
example(of: "prepend(Sequence)") {    // 1    let publisher = [5, 6, 7].publisher        // 2    publisher        .prepend([3, 4])        .prepend(Set(1...2))        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建一个发布数字 5、6 和 7 的发布者。**
2. **链接 `prepend(Sequence`) 两次到原始发布者。 一次从数组中添加值，第二次从集合中添加值。**

**运行 Playground，你的输出应类似于以下内容：**

```
——— Example of: prepend(Sequence) ———1234567
```

> **注意：与数组相反，要记住关于 Set 的一个重要事实是它们是无序的，因此不能保证项目发出的顺序。 这意味着上例中的前两个值可以是 1 和 2，也可以是 2 和 1。**

**许多类型都符合 Swift 中的 Sequence，它可以让你做一些有趣的事情。**

**在第二个 prepend 后：**

```
.prepend(Set(1...2))
```

**添加：**

```
.prepend(stride(from: 6, to: 11, by: 2))
```

**在这行代码中，你创建了一个 Strideable，它允许你以 2 为步长在 6 和 11 之间跨步。由于 Strideable 符合 Sequence，你可以在 prepend(Sequence) 中使用它。**

**再次运行你的 Playground 并查看调试控制台：**

```
——— Example of: prepend(Sequence) ———68101234567
```

**如你所见，现在在前一个输出之前向发布者添加了三个新值——6、8 和 10，这是以 2 为步长在 6 和 11 之间移动的结果。**

**`prepend(Publisher)`**

**前两个操作符将值列表添加到现有发布者。 但是，如果你有两个不同的发布者，并且你想将他们的值观合在一起怎么办？ 你可以使用 prepend(Publisher) 在原始发布者的值之前添加第二个发布者发出的值。**

![image-20221002202512577](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002202512577.png)

**通过将以下内容添加到你的 Playground 来尝试上面的示例：**

```
example(of: "prepend(Publisher)") {    // 1    let publisher1 = [3, 4].publisher    let publisher2 = [1, 2].publisher        // 2    publisher1        .prepend(publisher2)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建两个发布者。 一个发出数字 3 和 4，第二个发出 1 和 2。**
2. **将 publisher2 添加到 publisher1 的开头。 只有在 publisher2 发送 .finished 完成事件后，publisher1 才会开始执行其工作并发出事件。**

**如果你运行 Playground，你的调试控制台应显示以下输出：**

```
——— Example of: prepend(Publisher) ———1234
```

**正如预期的那样，值 1 和 2 首先从 publisher2 发出； 只有这样 3 和 4 才由发布者 1 发出。**

**你应该了解有关此操作符的更多细节，将以下内容添加到 Playground 的末尾：**

```
example(of: "prepend(Publisher) #2") {    // 1    let publisher1 = [3, 4].publisher    let publisher2 = PassthroughSubject<Int, Never>()        // 2    publisher1        .prepend(publisher2)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)        // 3    publisher2.send(1)    publisher2.send(2)}
```

**此示例与上一个示例类似，不同之处在于 publisher2 现在是一个 PassthroughSubject，你可以手动将值推送到它。**

**在示例中：**

1. **创建两个发布者。 第一个发出值 3 和 4，而第二个是可以动态接受值的 PassthroughSubject。**
2. **在 publisher1 之前添加主题。**
3. **通过主题 publisher2 发送值 1 和 2。**

**花点时间在你的脑海中运行这段代码。 你期望输出是什么？**

**现在，再次运行 Playground 并查看调试控制台。 你应该看到以下内容：**

```
——— Example of: prepend(Publisher) #2 ———12
```

**为什么 publisher2 在这里只发出两个数字？ 想想看——Combine 怎么知道前置发布者 publisher2 完成值的发送？ 它没有，因为它发出了值，但没有完成事件。 因此，前置发布者必须完成，以便 Combine 知道是时候切换到主发布者了。**

**在以下行之后：**

```
publisher2.send(2)
```

**添加：**

```
publisher2.send(completion: .finished)
```

**Combine 现在知道它可以处理来自 publisher1 的值，因为 publisher2 已经完成了它的工作。**

**再次运行你的 Playground，这次你应该会看到预期的输出：**

```
——— Example of: prepend(Publisher) #2 ———1234
```

#### **追加(Appending)**

**下一组操作符处理将发布者发出的事件与其他值连接起来。 但在这种情况下，你将使用 `append(Output...)`、`append(Sequence)` 和 `append(Publisher)` 处理追加而不是前置。 这些操作符的工作方式与它们的前置操作符类似。**

**`append(Output...)`**

**`append(Output...)` 的工作方式与它的 prepend 对应的类似：它也接受一个 Output 类型的可变参数列表，然后在原始发布者完成 `.finished` 事件后，附加在 `.finished` 前。**

![image-20221002203609517](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002203609517.png)

**将以下代码添加到 Playground：**

```
example(of: "append(Output...)") {    // 1    let publisher = [1].publisher        // 2    publisher        .append(2, 3)        .append(4)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **创建一个只发出一个值的发布者：1。**
2. **使用 append 两次，第一次追加 2 和 3，然后追加 4。**

**运行 Playground 并检查输出：**

```
——— Example of: append(Output...) ———1234
```

**追加的工作方式与你期望的完全一样，每个追加都等待上游完成，然后再添加自己的工作。**

**这意味着上游必须完成，否则追加永远不会发生，因为 Combine 无法知道先前的发布者已经完成了其所有值的发送。**

**要验证此行为，请添加以下示例：**

```
example(of: "append(Output...) #2") {    // 1    let publisher = PassthroughSubject<Int, Never>()        publisher        .append(3, 4)        .append(5)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)        // 2    publisher.send(1)    publisher.send(2)}
```

**此示例与上一个示例相同，但有两个不同之处：**

1. **publisher 现在是一个 PassthroughSubject，它允许你手动向它发送值。**
2. **你将 1 和 2 发送到 PassthroughSubject。**

**再次运行你的 Playground，你会看到只有发送给发布者的值被发出：**

```
——— Example of: append(Output...) #2 ———12
```

**两个追加操作符都无效，因为它们可能在发布者完成之前无法工作。 在示例的最后添加以下行：**

```
publisher.send(completion: .finished)
```

**再次运行你的 Playground，你应该会看到所有值，如预期的那样：**

```
——— Example of: append(Output...) #2 ———12345
```

**对于整个追加操作符系列，此行为是相同的； 除非前一个发布者发送 `.finished` 完成事件，否则不会发生追加。**

**`append(Publisher)`**

**附加操作符组的最后一个变体，它接受一个发布者并将其发出的任何值附加到原始发布者的末尾。**

**要尝试此示例，请将以下内容添加到你的 Playground：**

```
example(of: "append(Publisher)") {    // 1    let publisher1 = [1, 2].publisher    let publisher2 = [3, 4].publisher        // 2    publisher1        .append(publisher2)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

**在此代码中，你：**

1. **创建两个发布者，第一个发出 1 和 2，第二个发出 3 和 4。**
2. **将 publisher2 附加到 publisher1，因此 publisher2 中的所有值一旦完成就会附加在publisher1 的末尾。**

**运行 Playground，你应该会看到以下输出：**

```
——— Example of: append(Publisher) ———1234
```

#### **高级组合**

**我们将深入探讨组合不同发布者相关的一些更复杂的操作符。 尽管它们相对复杂，但它们也是发布者组合最有用的一些操作符。花时间熟悉它们的工作方式是值得的。**

**`switchToLatest`**

**switchToLatest 很复杂，但非常有用。它使你可以在取消挂起的发布者订阅的同时即时切换整个发布者订阅，从而切换到最新的发布者订阅。**

**你只能在自己发出发布者的发布者上使用它。**

![image-20221002205111420](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002205111420.png)

**将以下代码添加到你的 Playground 以试验你在上图中看到的示例：**

```
example(of: "switchToLatest") {    // 1    let publisher1 = PassthroughSubject<Int, Never>()    let publisher2 = PassthroughSubject<Int, Never>()    let publisher3 = PassthroughSubject<Int, Never>()        // 2    let publishers = PassthroughSubject<PassthroughSubject<Int, Never>, Never>()        // 3    publishers        .switchToLatest()        .sink(            receiveCompletion: { _ in print("Completed!") },            receiveValue: { print($0) }        )        .store(in: &subscriptions)        // 4    publishers.send(publisher1)    publisher1.send(1)    publisher1.send(2)        // 5    publishers.send(publisher2)    publisher1.send(3)    publisher2.send(4)    publisher2.send(5)        // 6    publishers.send(publisher3)    publisher2.send(6)    publisher3.send(7)    publisher3.send(8)    publisher3.send(9)        // 7    publisher3.send(completion: .finished)    publishers.send(completion: .finished)}
```

**在上面代码中：**

1. **创建三个接受整数且没有错误的 PassthroughSubject。**
2. **再创建一个 PassthroughSubject 接受其他 PassthroughSubject。你可以通过它发送 publisher1、publisher2 或 publisher3。**
3. **在你的发布者上使用 switchToLatest。现在，每次你通过发布者主题发送不同的发布者时，你都会切换到新的发布者并取消之前的订阅。**
4. **将 publisher1 发送给 publishers，然后将 1 和 2 发送给 publisher1。**
5. **发送 publisher2，取消对 publisher1的订阅。然后，你将 3 发送到 publisher1，但它被忽略了，然后将 4 和 5 发送到 publisher2，因为存在对 publisher2 的活动订阅，所以它们被推送。**
6. **发送 publisher3，取消对 publisher2 的订阅。和之前一样，你发送 6 到 publisher2 并被忽略，然后发送 7、8 和 9，它们通过订阅推送到 publisher3。**
7. **最后，你将完成事件发送给当前发布者，publisher3，并将另一个完成事件发送给发布者。这样就完成了所有活动订阅。**

**如果你按照上图进行操作，你可能已经猜到了这个示例的输出。**

**运行 Playground 并查看调试控制台：**

```
——— Example of: switchToLatest ———1245789Completed!
```

**如果你不确定为什么这在实际应用中很有用，请考虑以下场景：你的用户点击触发网络请求的按钮。 紧接着，用户再次点击按钮，触发第二个网络请求。 但是你如何摆脱挂起的请求，只使用最新的请求呢？ switchToLatest ！将以下代码添加到你的 Playground：**

```
example(of: "switchToLatest - Network Request") {    let url = URL(string: "https://source.unsplash.com/random")!        // 1    func getImage() -> AnyPublisher<UIImage?, Never> {        URLSession.shared            .dataTaskPublisher(for: url)            .map { data, _ in UIImage(data: data) }            .print("image")            .replaceError(with: nil)            .eraseToAnyPublisher()    }        // 2    let taps = PassthroughSubject<Void, Never>()        taps        .map { _ in getImage() } // 3        .switchToLatest() // 4        .sink(receiveValue: { _ in })        .store(in: &subscriptions)        // 5    taps.send()        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {        taps.send()    }        DispatchQueue.main.asyncAfter(deadline: .now() + 3.1) {        taps.send()    }}
```

**在此代码中：**

1. **定义一个函数 `getImage()`，它执行网络请求获取随机图像。这使用 `URLSession.dataTaskPublisher`，它是 Foundation 的众多 Combine 扩展之一。**
2. **创建一个 `PassthroughSubject` 来模拟用户对按钮的点击。**
3. **在点击按钮后，通过调用 `getImage()` 将点击映射到随机图像的新网络请求。这实质上将 `Publisher<Void, Never>` 转换为 `Publisher<Publisher<UIImage?, Never>, Never>` — 发布者的发布者。**
4. **使用 `switchToLatest()` 就像在前面的例子中一样，因为你有一个发布者的发布者。这保证只有一个发布者会发出值，并会自动取消任何剩余的订阅。**
5. **使用 `DispatchQueue` 模拟三个延迟的按钮点击。第一次点击是立即的，第二次点击是在三秒后出现的，最后一次点击是在第二次点击之后的十分之一秒后出现的。**

**运行 Playground 并查看以下输出：**

```
——— Example of: switchToLatest - Network Request ———image: receive subscription: (DataTaskPublisher)image: request unlimitedimage: receive value: (Optional(<UIImage:0x600000364120 anonymous {1080, 720}>))image: receive finishedimage: receive subscription: (DataTaskPublisher)image: request unlimitedimage: receive cancelimage: receive subscription: (DataTaskPublisher)image: request unlimitedimage: receive value: (Optional(<UIImage:0x600000378d80 anonymous {1080, 1620}>))image: receive finished
```

**在转到下一个操作符之前，请务必注释掉整个示例，以避免每次运行 Playground 时都运行异步网络请求。**

**`merge(with:)`**

**你将了解三个专注于组合不同发布者的值的操作符。 你将从 merge(with:) 开始。**

**此操作符将来自同一类型的不同发布者的排放交错，如下所示：**

![image-20221002210553794](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002210553794.png)

**要试用此示例，请将以下代码添加到你的 Playground：**

```
example(of: "merge(with:)") {    // 1    let publisher1 = PassthroughSubject<Int, Never>()    let publisher2 = PassthroughSubject<Int, Never>()        // 2    publisher1        .merge(with: publisher2)        .sink(            receiveCompletion: { _ in print("Completed") },            receiveValue: { print($0) }        )        .store(in: &subscriptions)        // 3    publisher1.send(1)    publisher1.send(2)        publisher2.send(3)        publisher1.send(4)        publisher2.send(5)        // 4    publisher1.send(completion: .finished)    publisher2.send(completion: .finished)}
```

**在与上图相关的这段代码中，你：**

1. **创建两个 PassthroughSubjects 接受和发出整数值并且不会发出错误。**
2. **将 publisher1 与 publisher2 合并，将两者发出的值交错。 组合提供重载，可让你合并多达八个不同的发布者。**
3. **将 1 和 2 添加到 publisher1，然后将 3 添加到 publisher2，然后再次将 4 添加到 publisher1，最后将 5 添加到 publisher2。**
4. **你向发布者 1 和发布者 2 发送完成事件。**

**运行你的 Playground，你应该会看到以下输出，如预期的那样：**

```
——— Example of: merge(with:) ———12345Completed
```

**`combineLatest`**

**combineLatest 是另一个允许你组合不同发布者的操作符。 它还允许你组合不同价值类型的发布者，这非常有用。 但是，它不是交错所有发布者的排放，而是在所有发布者发出值时发出一个包含所有发布者的最新值的元组。**

**但是有一个问题：原始发布者和传递给 combineLatest 的每个发布者必须至少发出一个值，然后 combineLatest 本身才会发出任何内容。**

**将以下代码添加到你的 Playground 以试用此操作符：**

```
example(of: "combineLatest") {    // 1    let publisher1 = PassthroughSubject<Int, Never>()    let publisher2 = PassthroughSubject<String, Never>()        // 2    publisher1        .combineLatest(publisher2)        .sink(            receiveCompletion: { _ in print("Completed") },            receiveValue: { print("P1: \($0), P2: \($1)") }        )        .store(in: &subscriptions)        // 3    publisher1.send(1)    publisher1.send(2)        publisher2.send("a")    publisher2.send("b")        publisher1.send(3)        publisher2.send("c")        // 4    publisher1.send(completion: .finished)    publisher2.send(completion: .finished)}
```

**在上述代码中：**

1. **创建两个 PassthroughSubjects。 第一个接受没有错误的整数，而第二个接受没有错误的字符串。**
2. **将 publisher2 的最新排放量与 publisher1 结合起来。 你可以使用不同的 combineLatest 重载组合最多四个不同的发布者。**
3. **将 1 和 2 发送到 publisher1，然后将“a”和“b”发送到 publisher2，然后将 3 发送到 publisher1，最后将“c”发送到 publisher2。**

**向 publisher1 和 publisher2 发送完成事件。**

**运行 Playground 并查看控制台中的输出：**

```
——— Example of: combineLatest ———P1: 2, P2: aP1: 2, P2: bP1: 3, P2: bP1: 3, P2: cCompleted
```

**你可能会注意到从 publisher1 发出的 1 永远不会通过 combineLatest 推送。 这是因为 combineLatest 只有在每个发布者发出至少一个值时才开始发出组合。在这里，这个条件只有在 "a" 发射后才成立，此时来自 publisher1 的最新发射值是 2。这就是为什么第一个发射是 (2, "a")。**

**`zip`**

**你将使用本章的最后一个操作符：zip。 你可能会从 Swift 标准库方法中识别出这一方法，在序列类型上具有相同的名称。**

**该操作符的工作方式类似，在相同索引中发出成对值的元组。 它等待每个发布者发出一个项目，然后在所有发布者在当前索引处发出一个值后发出一个项目元组。**

**这意味着如果你压缩两个发布者，每次两个发布者发出一个新值时，你都会得到一个元组。**

![image-20221002211423703](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221002211423703.png)

**将以下代码添加到你的 Playground 以尝试此示例：**

```
example(of: "zip") {    // 1    let publisher1 = PassthroughSubject<Int, Never>()    let publisher2 = PassthroughSubject<String, Never>()        // 2    publisher1        .zip(publisher2)        .sink(            receiveCompletion: { _ in print("Completed") },            receiveValue: { print("P1: \($0), P2: \($1)") }        )        .store(in: &subscriptions)        // 3    publisher1.send(1)    publisher1.send(2)    publisher2.send("a")    publisher2.send("b")    publisher1.send(3)    publisher2.send("c")    publisher2.send("d")        // 4    publisher1.send(completion: .finished)    publisher2.send(completion: .finished)}
```

**在最后一个示例中：**

1. **创建两个 PassthroughSubject，第一个接受整数，第二个接受字符串。 两者都不能发出错误。**
2. **将 publisher1 与 publisher2 压缩，一旦它们各自发出一个新值，就将它们的发射配对。**
3. **将 1 和 2 发送到 publisher1，然后将“a”和“b”发送到 publisher2，然后将 3 再次发送到 publisher1，最后将“c”和“d”发送到 publisher2。**
4. **完成 publisher1 和 publisher2。**

**最后一次运行你的 Playground 并查看调试控制台：**

```
——— Example of: zip ———P1: 1, P2: aP1: 2, P2: bP1: 3, P2: cCompleted
```

#### **关键点**

**在本章中，你学习了如何使用不同的发布者并从中创建有意义的组合。**

* **你可以使用前置和附加操作符系列在不同的发布者之前或之后添加来自一个发布者的值。**
* **虽然 switchToLatest 相对复杂，但它非常有用。它需要一个发布者发出发布者，切换到最新发布者并取消对先前发布者的订阅。**
* **merge(with:) 允许你交错来自多个发布者的值。**
* **一旦所有组合发布者都发出至少一个值，只要它们中的任何一个发出值，combineLatest 发出所有组合发布者的最新值。**
* **来自不同发布者的 zip 对，在所有发布者都发出一个值之后发出一个对的元组。**
* **你可以混合组合操作符以在发布者及其值之间创建有趣且复杂的关系。**

#### **接下来去哪？**

**这是一个相当长的章节，但它包含了一些最有用和最复杂的操作符。**

**在接下来的两章中，你还有两组操作符要学习：“时间操纵操作符”和“序列操作符”。**
