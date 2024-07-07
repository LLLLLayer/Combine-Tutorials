# 第 4 节：过滤操作符

### 第 4 节：过滤操作符

正如你现在可能已经意识到的那样，操作符基本上是你用来操作 Combine 发布者的词汇。你知道的“单词”越多，你对数据的控制就越好。

在上一章中，你学习了如何使用值并将它们转换为其他值——这是日常工作中最有用的操作符类别之一。

但是，当你想要限制发布者发出的值或事件，并且只使用其中的一部分时，将如何操作？本章都是关于如何使用一组特殊的操作符来做到这一点：过滤操作符！

幸运的是，这些操作符中有许多与 Swift 标准库中的同名操作符有相似之处。

#### 入门

你可以在项目文件夹中找到本章的 Playground。随着本章的深入，你将在 Playground 中编写代码，然后运行 Playground。这将帮助你了解不同的操作符如何操纵发布者发出的事件。

> 注意：本章中的大多数操作符都与 try 前缀有相似之处，例如 filter 与 tryFilter。 它们之间的唯一区别是后者提供了一个抛出的闭包。 你从闭包中抛出的任何错误都会以抛出的错误终止发布者。 为简洁起见，本章将只介绍非抛出的变化，它们实际上是相同的。

#### 基础值(Filtering values)

第一部分将了解过滤的基础——消费发布者的值，并有条件地决定将其中的哪些传递给消费者。

最简单的方法是使用恰当命名的操作符 — filter，它采用返回 Bool 的闭包，只会传递与提供的条件匹配的值：

![image-20220929013110870](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929013110870.png)

将这个新示例添加到 Playground：

```
example(of: "filter") {    // 1    let numbers = (1...10).publisher        // 2    numbers        .filter { $0.isMultiple(of: 3) }        .sink(receiveValue: { n in            print("\(n) is a multiple of 3!")        })        .store(in: &subscriptions)}
```

在上面的示例中：

1. 创建一个新的发布者，它将发出有限数量的值——从 1 到 10，然后使用序列类型的发布者属性完成。
2. 使用过滤器操作符，传入一个谓词，在该谓词中，你只允许通过是三的倍数的数字。

运行 Playground，你应该在控制台中看到以下内容：

```
——— Example of: filter ———3 is a multiple of 3!6 is a multiple of 3!9 is a multiple of 3!
```

在你的应用程序的生命周期中，很多时候你可能会遇到发布者在一行中发出相同的值，你可能希望忽略这些值。 例如，如果用户连续输入“a”五次，然后输入“b”，你可能希望忽略过多的“a”。

Combine 为该任务提供了完美的操作符：removeDuplicates：

![image-20220929013433303](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929013433303.png)

请注意，你不必为此操作符提供任何参数。 removeDuplicates 自动适用于任何符合 Equatable 的值，包括 String。

将以下 removeDuplicates() 示例添加到你的 Playground:

```
example(of: "removeDuplicates") {    // 1    let words = "hey hey there! want to listen to mister mister ?"        .components(separatedBy: " ")        .publisher    // 2    words        .removeDuplicates()        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

此代码与上一个代码没有太大区别：

1. 将一个句子分成一个单词数组，然后创建一个新的发布者来发出这些单词。
2. 将 removeDuplicates() 应用于你的发布者。

运行你的 Playground 并查看调试控制台：

```
——— Example of: removeDuplicates ———heythere!wanttolistentomister?
```

如你所见，你跳过了第二个“hey”和第二个“mister”。

> 注意：不符合 Equatable 的值怎么办？ removeDuplicates 有另一个重载，它接受一个带有两个值的闭包，你将从中返回一个 Bool 来指示值是否相等。

#### 移去 nil 值(Compacting values)和忽略(Ignoring values)

很多时候，你会发现自己与发布可选值的发布者打交道。 或者更常见的是，你会想要对你的值执行一些可能返回 nil 的操作，但是谁想要处理所有这些 nil 呢？

Swift 标准库中的一个非常著名的 Sequence 方法，叫做 compactMap——还有一个同名的操作符！

![image-20220929014234146](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929014234146.png)

将以下内容添加到你的 Playground：

```
example(of: "compactMap") {    // 1    let strings = ["a", "1.24", "3",                   "def", "45", "0.23"].publisher        // 2    strings        .compactMap { Float($0) }        .sink(receiveValue: {            // 3            print($0)        })        .store(in: &subscriptions)}
```

正如图表概述的那样：

1. 创建一个发布有限字符串列表的发布者。
2. 使用 compactMap 尝试从每个单独的字符串初始化一个 Float。 如果失败，则返回 nil。 这些 nil 值会被 compactMap 操作符自动过滤掉。
3. 只打印成功转换为浮点数的字符串。

在你的 Playground 中运行上面的示例，你应该会看到类似于上图的输出：

```
——— Example of: compactMap ———1.243.045.00.23
```

有时，你只想知道发布者完成值的发送，而忽略实际值。当这种情况发生时，你可以使用 ignoreOutput 操作符：

![image-20220929014620869](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929014620869.png)

如上图所示，发出哪些值或发出多少个值并不重要，因为它们都被忽略了； 你只需将完成事件推送给消费者。

通过将以下代码添加到你的 Playground 来试验此示例：

```
example(of: "ignoreOutput") {    // 1    let numbers = (1...10_000).publisher        // 2    numbers        .ignoreOutput()        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

在上面的示例中，你：

1. 创建一个发布者，从 1 到 10,000 发出 10,000 个值。
2. 添加 ignoreOutput 操作符，它会忽略所有值，只向消费者发出完成事件。

运行你的 Playground 并查看调试控制台：

```
——— Example of: ignoreOutput ———Completed with: finished
```

#### 寻找值(Finding values)

在本节中，你将了解两个同样起源于 Swift 标准库的操作符：first(where:) 和 last(where:)。 正如它们的名字所暗示的那样，你可以使用它们分别仅查找和发出与提供的描述匹配的第一个或最后一个值。

是时候看看几个例子了，从 first(where:) 开始：

![image-20220929015607245](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929015607245.png)

这个操作符很有趣，因为它是 lazy 的，意思是：它只取所需的值，直到找到与你提供的谓词匹配的值。 一旦找到匹配项，它就会取消订阅并完成。

将以下代码添加到你的 Playground 以查看其工作原理：

```
example(of: "first(where:)") {    // 1    let numbers = (1...9).publisher        // 2    numbers        .first(where: { $0 % 2 == 0 })        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

这是你刚刚添加的代码的作用：

1. 创建一个新的发布者，发布从 1 到 9 的数字。
2. 使用 first(where:) 操作符查找第一个发出的偶数值。

在你的 Playground 中运行此示例并查看控制台输出：

```
——— Example of: first(where:) ———2Completed with: finished
```

它的工作方式与你可能猜到的完全一样。 但是等等，对上游的订阅，也就是数字发布者呢？ 即使找到匹配的偶数，它是否仍会继续发出其值？ 通过找到以下行来测试这个理论：

在 `numbers` 之后立即添加 print("numbers") 操作符，如下所示：

```
numbers  	.print("numbers")
```

> 注意：你可以在操作符链中的任何位置使用 print 操作符，以准确查看在该点发生的事件。

再次运行你的 Playground，并查看控制台。 你的输出应类似于以下内容：

```
——— Example of: first(where:) ———numbers: receive subscription: (1...9)numbers: request unlimitednumbers: receive value: (1)numbers: receive value: (2)numbers: receive cancel2Completed with: finished
```

如你所见，一旦 first(where:) 找到匹配的值，它就会通过订阅发送取消，上游停止发出值。

转到此操作符的反面 — last(where:)，其目的是找到与提供的描述匹配的最后一个值。

![image-20220929020154306](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220929020154306.png)

与 first(where:) 不同，此操作符是贪婪的，因为它必须等待发布者完成发出值才能知道是否找到了匹配值。 因此，上游必须是有限的。

将以下代码添加到你的 Playground：

```
example(of: "last(where:)") {    // 1    let numbers = (1...9).publisher        // 2    numbers        .last(where: { $0 % 2 == 0 })        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

与前面的代码示例非常相似：

1. 创建一个发出 1 到 9 之间数字的发布者。
2. 使用 last(where:) 操作符查找最后发出的偶数值。

运行你的 Playground并找出：

```
——— Example of: last(where:) ———8Completed with: finished
```

因为操作符无法知道发布者是否会发出与下一行的条件匹配的值，因此操作符必须知道发布者的全部范围，然后才能确定与条件匹配的最后一个项目。

要查看此操作，请将整个示例替换为以下内容：

```
example(of: "last(where:)") {    let numbers = PassthroughSubject<Int, Never>()        numbers        .last(where: { $0 % 2 == 0 })        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)        numbers.send(1)    numbers.send(2)    numbers.send(3)    numbers.send(4)    numbers.send(5)}
```

在此示例中，你使用 PassthroughSubject ，通过它手动发送事件。

再次运行你的 Playground：

```
——— Example of: last(where:) ———
```

正如预期的那样，由于发布者永远不会完成，因此无法确定匹配条件的最后一个值。

要解决此问题，请将以下内容添加为示例的最后一行，PassthroughSubject 发送完成：

```
numbers.send(completion: .finished)
```

再次运行你的 Playground，现在一切都应该按预期工作：

```
——— Example of: last(where:) ———4Completed with: finished
```

> 注意：你还可以使用 first() 和 last() 操作符来简单地获取发布者发出的第一个或最后一个值。 这些人也分别是 lazy 和贪婪的。

#### 删除值(Dropping values)

删除值是你在与发布者合作时经常需要的功能。例如，当你想忽略来自一个发布者的值直到第二个发布者开始发布时，或者如果你想在流开始时忽略特定数量的值。

三个操作符属于这一类，你将首先了解最简单的一个 -`dropFirst`。

`dropFirst` 操作符采用 `count` 参数（如果省略，则默认为 1）并忽略发布者发出的第一个计数值。 只有在跳过计数值之后，发布者才会开始传递值。

![image-20220930012620549](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930012620549.png)

将以下代码添加到 Playground 的末尾：

```
example(of: "dropFirst") {    // 1    let numbers = (1...10).publisher        // 2    numbers        .dropFirst(8)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

如上图所示：

1. 创建一个发布者，它发出 10 个介于 1 和 10 之间的数字。
2. 使用 `dropFirst(8)` 删除前八个值，只打印 9 和 10。

运行你的 Playground，你应该会看到以下输出：

```
——— Example of: dropFirst ———910
```

继续讨论值下下一个操作符 - drop(while:)。 这是另一个非常有用的变体，它采用条件闭包并忽略发布者发出的任何值，直到第一次满足该条件。 一旦满足条件，值就开始流经操作符：

![image-20220930012919504](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930012919504.png)

将以下示例添加到你的 Playground 以查看其实际效果：

```
example(of: "drop(while:)") {    // 1    let numbers = (1...10).publisher        // 2    numbers        .drop(while: { $0 % 5 != 0 })        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

在代码中：

1. 创建一个发出 1 到 10 之间数字的发布者。
2. 使用 drop(while:) 等待可以被 5 整除的第一个值。 一旦满足条件，值将开始流经操作符并且不会再被删除。

运行你的 Playground 并查看调试控制台：

```
——— Example of: drop(while:) ———5678910
```

如你所见，你已经删除了前四个值。 一旦 5 到来，问题是“这能被 5 整除吗？” 最后是真的，所以它现在发出 5 和所有未来值。

你可能会问自己——这个操作符和过滤器有什么不同？ 它们都采用一个闭包，该闭包根据该闭包的结果控制发出哪些值。

第一个区别是，如果你在闭包中返回 true，filter 会让值通过，而只要你从闭包中返回 true，drop(while:) 就会跳过值。

第二个也是更重要的区别是过滤器永远不会停止评估上游发布者发布的所有值的条件。 即使在 filter 的条件评估为 true 之后，仍然会“质疑”下一个值，并且你的闭包必须回答这个问题：“你想让这个值通过吗？”。

相反，drop(while:) 的谓词闭包在满足条件后将不再执行。要确认这一点，请替换以下行：

```
.drop(while: { $0 % 5 != 0 })
```

为：

```
.drop(while: {  	print("x")  	return $0 % 5 != 0})
```

每次调用闭包时，你添加了一条打印语句以将 x 打印到调试控制台。 运行 Playground，你应该会看到以下输出：

```
——— Example of: drop(while:) ———xxxxx5678910
```

你可能已经注意到，x 正好打印了五次。 一旦满足条件（发出 5 时），就不再评估闭包。

过滤类别的最后一个也是最精细的操作符是 drop(untilOutputFrom:)。

想象一个场景，你有一个用户点击一个按钮，但你想忽略所有的点击，直到你的 isReady 发布者发出一些结果。 该操作符非常适合这种情况。

它会跳过发布者发出的任何值，直到第二个发布者开始发出值，从而在它们之间创建关系：

![image-20220930013512951](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930013512951.png)

第一行表示 isReady 流，第二行表示用户通过 drop(untilOutputFrom:) 进行的点击，它以 isReady 作为参数。

在你的 Playground 末尾，添加以下代码：

```
example(of: "drop(untilOutputFrom:)") {    // 1    let isReady = PassthroughSubject<Void, Never>()    let taps = PassthroughSubject<Int, Never>()        // 2    taps        .drop(untilOutputFrom: isReady)        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)        // 3    (1...5).forEach { n in        taps.send(n)                if n == 3 {            isReady.send()        }    }}
```

在此代码中，你：

1. 创建两个你可以手动发送值的 PassthroughSubjects。 第一个是 isReady 而第二个代表用户的点击。
2. 使用 drop(untilOutputFrom: isReady) 忽略用户的任何点击，直到 isReady 发出至少一个值。
3. 通过 subject 发送五个“点击”，就像上图一样。 第三次点击后，你发送 isReady 一个值。

运行你的 Playground，然后看看你的调试控制台。 你将看到以下输出：

```
——— Example of: drop(untilOutputFrom:) ———45
```

此输出与上图相同：

* 用户点击五次。 前三个被忽略。
* 第三次点击后，isReady 发出一个值。
* 用户未来的所有点击都会通过。

#### 限制值(Limiting values)

在上一节中，你已经学习了如何删除（或跳过）值，直到满足特定条件。该条件可能匹配某个静态值、谓词闭包或对不同发布者的依赖。

本节解决相反的需求：接收值直到满足某些条件，然后强制发布者完成。例如，考虑一个可能发出未知数量的值的请求，但你只想要一次发出而不关心其余的值。

结合使用前缀系列操作符解决了这组问题。 尽管名称并不完全直观，但这些操作符提供的功能对于许多现实生活中的情况都很有用。

前缀族类操作符似于 drop，提供 prefix(\_:)、 prefix(while:) 和 prefix(untilOutputFrom:)。 但是，前缀操作符不会在满足某些条件之前删除值，而是在满足该条件之前获取值。

现在，是时候深入研究本章的最后一组操作符了，从 prefix(\_:) 开始。

与 dropFirst 相反，prefix(\_:) 将只取所提供的数量的值，然后完成：

![image-20220930014054540](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930014054540.png)

将以下代码添加到你的 Playground 以演示这一点：

```
example(of: "prefix") {    // 1    let numbers = (1...10).publisher        // 2    numbers        .prefix(2)        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

此代码与你在上一节中使用的放置代码非常相似：

1. 创建一个发布者，它发出从 1 到 10 的数字。
2. 使用 prefix(2) 只允许发射前两个值。 一旦发出两个值，发布者就完成了。

运行你的 Playground，你会看到以下输出：

```
——— Example of: prefix ———12Completed with: finished
```

就像 first(where:) 一样，这个操作符是 lazy 的，这意味着它只占用它需要的值，然后终止，这也可以防止数字产生超出 1 和 2 的其他值，因为它完成了。

接下来是 prefix(while:)，它接受一个条件闭包，只要该闭包的结果为真，就让来自上游发布者的值通过。 一旦结果为假，发布者将完成：

![image-20220930014328619](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930014328619.png)

将以下示例添加到你的 Playground 以尝试此操作：

```
example(of: "prefix(while:)") {    // 1    let numbers = (1...10).publisher        // 2    numbers        .prefix(while: { $0 < 3 })        .sink(receiveCompletion: { print("Completed with: \($0)") },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

除了使用闭包来评估前缀条件之外，此示例与前一个示例基本相同。 你：

1. 创建一个发出 1 到 10 之间值的发布者。
2. 使用 prefix(while:) 让小于 3 的值通过。一旦发出等于或大于 3 的值，发布者就完成了。

运行 Playground 并检查调试控制台； 输出应该与前一个操作符的输出相同：

```
——— Example of: prefix(while:) ———12Completed with: finished
```

前面两个前缀操作符已经过去，是时候使用最复杂的一个了：prefix(untilOutputFrom:)。 再一次，与 drop(untilOutputFrom:) 跳过值直到第二个发布者发出，prefix(untilOutputFrom:) 取值直到第二个发布者发出。

想象一个场景，你有一个用户只能点击两次的按钮。 一旦发生两次点击，按钮上的进一步点击事件应该被省略：

![image-20220930014557757](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220930014557757.png)

将本章的最后一个示例添加到 Playground 的末尾：

```
example(of: "prefix(untilOutputFrom:)") {  // 1  let isReady = PassthroughSubject<Void, Never>()  let taps = PassthroughSubject<Int, Never>()  // 2  taps    .prefix(untilOutputFrom: isReady)    .sink(receiveCompletion: { print("Completed with: \($0)") },          receiveValue: { print($0) })    .store(in: &subscriptions)  // 3  (1...5).forEach { n in    taps.send(n)        if n == 2 {      isReady.send()    }  }}
```

如果你回想一下 drop(untilOutputFrom:) 示例，你应该会发现这很容易理解：

1. 创建两个你可以手动发送值的 PassthroughSubjects。 第一个是 isReady 而第二个代表用户的点击。
2. 使用 prefix(untilOutputFrom: isReady) 让点击事件通过，直到 isReady 发出至少一个值。
3. 通过 subject 发送五个“点击”，与上图完全相同。 第二次点击后，你发送 isReady 一个值。

运行 Playground，查看控制台，你应该看到以下内容：

```
——— Example of: prefix(untilOutputFrom:) ———12Completed with: finished
```

#### 挑战

**挑战：过滤所有东西**

创建一个发布从 1 到 100 的数字集合的示例，并使用过滤操作符：

1. 跳过上游发布者发出的前 50 个值。
2. 在前 50 个值之后取接下来的 20 个值。
3. 只取偶数。

你的示例的输出应产生以下数字，每行一个：

```
52 54 56 58 60 62 64 66 68 70
```

> 注意：在这个挑战中，你需要将多个操作符链接在一起以产生所需的值。

你可以在 projects/challenge/Final.playground 中找到该挑战的解决方案。

#### 关键点

在本章中，你了解到：

* 过滤操作符让你可以控制上游发布者发出的哪些值被发送到下游、另一个操作符或消费者。
* 当你不关心值本身，只想要一个完成事件时，ignoreOutput 是你的朋友。
* 查找值是另一种过滤，你可以分别使用 first(where:) 和 last(where:) 找到第一个或最后一个值以匹配提供的调整。
* First 类型的操作符是 lazy 的；它们只取所需数量的值，然后发送完成。 Last 类型的操作符是贪婪的，在决定哪个值是最后一个满足条件之前，必须知道值的全部范围。
* 你可以使用 drop 系列操作符控制上游发布者发出的值在向下游发送值之前被忽略的数量。
* 同样，你可以使用前缀系列操作符控制上游发布者在完成之前可以发出多少值。

#### 接下来去哪？

了解了转换和过滤操作符的知识后，你就可以进入下一章并学习另一组非常有用的操作符：组合操作符。
