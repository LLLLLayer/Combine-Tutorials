# 第 7 节：序列操作符

### 第 7 节：序列操作符

至此，你已经了解了 Combine 提供的大多数操作符！不过，还有一个类别可供你深入研究：序列操作符。

当你意识到发布者本身就是序列时，序列操作符最容易理解。序列操作符与发布者的值一起使用，就像数组或集合一样——当然，它们只是有限序列！

考虑到这一点，序列操作符主要将发布者作为一个整体来处理，而不是像其他操作符类别那样处理单个值。

此类中的许多操作符的名称和行为与 Swift 标准库中的对应操作符几乎相同。

#### 入门

你可以在 projects/Starter.playground 中找到本章的 Playground。在本章中，你将向 Playground 添加代码并运行它，以查看这些不同的序列操作符如何操纵你的发布者。你将使用 print 操作符记录所有发布事件。

#### 寻找值

本节的第一部分，由根据不同的标准定位发布者发出的特定值的操作符组成。这些类似于 Swift 标准库中的 collection 方法。

**`min`**

**min 操作符可让你找到发布者发出的最小值。 它是贪婪的，这意味着它必须等待发布者发送一个 .finished 完成事件。 一旦发布者完成，操作者只会发出最小值：**

![image-20221006143832523](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006143832523.png)

**将以下示例添加到你的 Playground 以尝试 min：**

```
example(of: "min") {    // 1    let publisher = [1, -50, 246, 0].publisher        // 2    publisher        .print("publisher")        .min()        .sink(receiveValue: { print("Lowest value is \($0)") })        .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建一个发布四个不同数字的发布者。**
2. **使用 min 操作符查找发布者发出的最小数量并打印该值。**

**运行你的 Playground，你将在控制台中看到以下输出：**

```
——— Example of: min ———publisher: receive subscription: ([1, -50, 246, 0])publisher: request unlimitedpublisher: receive value: (1)publisher: receive value: (-50)publisher: receive value: (246)publisher: receive value: (0)publisher: receive finishedLowest value is -50
```

**如你所见，发布者发出其所有值并完成，然后 min 找到最小值并将其发送到下游接收器以将其打印出来。**

**但是等等，Combine 怎么知道这些数字中的哪一个是最小值？ 嗯，这要归功于数值符合 Comparable 协议。你可以在发出 Comparable-conforming 类型的发布者上直接使用 min() ，无需任何参数。**

**但是，如果你的值不符合 Comparable 会发生什么？ 幸运的是，你可以使用 min(by:) 操作符提供自己的比较器闭包。**

**考虑以下示例，你的发布者发出许多数据，而你希望找到最小的数据。**

**将以下代码添加到你的 Playground：**

```
example(of: "min non-Comparable") {    // 1    let publisher = ["12345",                     "ab",                     "hello world"]        .map { Data($0.utf8) } // [Data]        .publisher // Publisher<Data, Never>        // 2    publisher        .print("publisher")        .min(by: { $0.count < $1.count })        .sink(receiveValue: { data in            // 3            let string = String(data: data, encoding: .utf8)!            print("Smallest data is \(string), \(data.count) bytes")        })        .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **你创建一个发布者，该发布者发出三个从各种字符串创建的 Data 对象。**
2. **由于 Data 不符合 Comparable，因此使用 min(by:) 操作符查找字节数最少的 Data 对象。**
3. **将最小的 Data 对象转换回字符串并打印出来。**

**运行你的 Playground，你将在控制台中看到以下内容：**

```
——— Example of: min non-Comparable ———publisher: receive subscription: ([5 bytes, 2 bytes, 11 bytes])publisher: request unlimitedpublisher: receive value: (5 bytes)publisher: receive value: (2 bytes)publisher: receive value: (11 bytes)publisher: receive finishedSmallest data is ab, 2 bytes
```

**与前面的示例一样，发布者发出其所有 Data 对象并完成，然后 min(by:) 找到并发出具有最小字节大小的数据，并且接收器将其打印出来。**

**`max`**

**正如你所猜测的，max 的工作方式与 min 完全相同，只是它找到了发布者发出的最大值：**

![image-20221006144557401](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006144557401.png)

**将以下代码添加到你的 Playground 以尝试此示例：**

```
example(of: "max") {    // 1    let publisher = ["A", "F", "Z", "E"].publisher        // 2    publisher        .print("publisher")        .max()        .sink(receiveValue: { print("Highest value is \($0)") })        .store(in: &subscriptions)}
```

**在以下代码中，你：**

1. **创建一个发出四个不同字母的发布者。**
2. **使用 max 操作符找出数值最高的字母并打印出来。**

**运行 Playground。你将在 Playground 中看到以下输出：**

```
——— Example of: max ———publisher: receive subscription: (["A", "F", "Z", "E"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive value: (F)publisher: receive value: (Z)publisher: receive value: (E)publisher: receive finishedHighest value is Z
```

**与 min 完全一样，max 是贪婪的，必须等待上游发布者完成其值的发送，然后才能确定最大值。 在这种情况下，该值为 Z。**

> **注意：与 min 完全一样，max 也有一个伴随的 max(by:) 操作符，它接受一个谓词来确定在 non-Comparable 值中发出的最大值。**

**`first`**

**虽然 min 和 max 操作符处理在某个未知索引处查找发布的值，但本节中的其余操作符处理在特定位置查找发出的值，从 first 操作符符开始。**

**第一个操作符类似于 Swift 的 collection 的 first 属性，它让第一个发出的值通过然后完成。它是惰性的，这意味着它不会等待上游发布者完成，而是会在接收到发出的第一个值时取消订阅。**

![image-20221006145546068](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006145546068.png)

**将上面的示例添加到你的 Playground：**

```
example(of: "first") {    // 1    let publisher = ["A", "B", "C"].publisher        // 2    publisher        .print("publisher")        .first()        .sink(receiveValue: { print("First value is \($0)") })        .store(in: &subscriptions)}
```

**在上面的代码中，你：**

1. **创建一个发出三个字母的发布者。**
2. **使用 first() 只让第一个发出的值通过并打印出来。**

**运行你的 Playground 并查看控制台：**

```
——— Example of: first ———publisher: receive subscription: (["A", "B", "C"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive cancelFirst value is A
```

**一旦 first() 获得发布者的第一个值，它就会取消对上游发布者的订阅。**

**如果你正在寻找更精细的控制，你也可以使用 first(where:)。 就像它在 Swift 标准库中的对应物一样，它将发出与提供的条件匹配的第一个值——如果有的话。**

**将以下示例添加到你的 Playground：**

```
example(of: "first(where:)") {  // 1  let publisher = ["J", "O", "H", "N"].publisher  // 2  publisher    .print("publisher")    .first(where: { "Hello World".contains($0) })    .sink(receiveValue: { print("First match is \($0)") })    .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建一个发出四个字母的发布者。**
2. **使用 first(where:) 操作符查找 Hello World 中包含的第一个字母，然后将其打印出来。**

**运行 Playground，你将看到以下输出：**

```
——— Example of: first(where:) ———publisher: receive subscription: (["J", "O", "H", "N"])publisher: request unlimitedpublisher: receive value: (J)publisher: receive value: (O)publisher: receive value: (H)publisher: receive cancelFirst match is H
```

**在上面的示例中，操作符检查 Hello World 是否包含发出的字母，直到找到第一个匹配项：H。找到那么多后，它会取消订阅并发出字母让 sink 打印出来。**

**`last`**

**正如 min 有一个对立面，max，first 也有一个对应的操作符：last！**

**last 的工作方式与 first 完全相同，只是它发出发布者发出的最后一个值。 这意味着它也是贪婪的，必须等待上游发布者完成：**

**将此示例添加到你的 Playground：**

```
example(of: "last") {    // 1    let publisher = ["A", "B", "C"].publisher        // 2    publisher        .print("publisher")        .last()        .sink(receiveValue: { print("Last value is \($0)") })        .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建一个将发出三个字母并完成的发布者。**
2. **使用 last 操作符只发出最后发布的值并打印出来。**

**运行 Playground，你将看到以下输出：**

```
——— Example of: last ———publisher: receive subscription: (["A", "B", "C"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive value: (B)publisher: receive value: (C)publisher: receive finishedLast value is C
```

**last 等待上游发布者发送一个 `.finished` 完成事件，此时它向下游发送最后一个发出的值，以便在接收器中打印出来。**

> **注意：与 first 完全一样，last 也有一个 `last(where:)` 重载，它发出匹配指定条件的发布者发出的最后一个值。**

**`output(at:)`**

**本节中的最后两个操作符在 Swift 标准库中没有对应的操作符。 output 操作符将查找上游发布者在指定索引处发出的值。**

**你将从 output(at:) 开始，它只发出在指定索引处发出的值：**

![image-20221006150441447](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006150441447.png)

**将以下代码添加到你的 Playground 以尝试此示例：**

```
example(of: "output(at:)") {    // 1    let publisher = ["A", "B", "C"].publisher        // 2    publisher        .print("publisher")        .output(at: 1)        .sink(receiveValue: { print("Value at index 1 is \($0)") })        .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **创建一个发出三个字母的发布者。**
2. **使用 output(at:) 只允许在索引 1 处发出的值——即第二个值。**

**在你的 Playground 中运行该示例并查看你的控制台：**

```
——— Example of: output(at:) ———publisher: receive subscription: (["A", "B", "C"])publisher: request unlimitedpublisher: receive value: (A)publisher: request max: (1) (synchronous)publisher: receive value: (B)Value at index 1 is Bpublisher: receive cancel
```

**在这里，输出表明索引 1 处的值是 B。但是，你可能已经注意到另一个有趣的事实：操作符在每个发出的值之后都需要一个值，因为它知道它只是在寻找单个项目。虽然这是特定操作符的实现细节，但它提供了有关 Apple 如何设计一些自己的内置 Combine 操作符以利用背压的有趣见解。**

**`output(in:)`**

**你将使用 output 操作符的第二个重载：`output(in:)`。当 output(at:) 发出在指定索引处发出的单个值时， output(in:) 发出其索引在给定范围内的值：**

**要尝试这一点，请将以下示例添加到你的 Playground：**

```
example(of: "output(in:)") {    // 1    let publisher = ["A", "B", "C", "D", "E"].publisher        // 2    publisher        .output(in: 1...3)        .sink(receiveCompletion: { print($0) },              receiveValue: { print("Value in range: \($0)") })        .store(in: &subscriptions)}
```

**在前面的代码中：**

1. **创建一个发出五个不同字母的发布者。**
2. **使用 `output(in:)` 操作符只允许在索引 1 到 3 中发出的值通过，然后打印出这些值。**

**运行你的 Playground：**

```
——— Example of: output(in:) ———Value in range: BValue in range: CValue in range: Dfinished
```

**操作符发出索引范围内的单个值，而不是它们的集合。 操作符打印值 B、C 和 D，因为它们分别位于索引 1、2 和 3 中。 然后，由于该范围内的所有项目都已发出，因此它会在收到所提供范围内的所有值后立即取消订阅。**

#### **查询发布者**

**以下操作符还处理发布者发出的整个值集，但它们不会产生它发出的任何特定值。 相反，这些操作符发出一个不同的值，表示对整个发布者的某些查询。 count 操作符就是一个很好的例子。**

**`count`**

**count 操作符将发出单个值 - 一旦发布者发送 .finished 完成事件，上游发布者发出的值的数量：**

![image-20221006152622502](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006152622502.png)

**添加以下代码以尝试此示例：**

```
example(of: "count") {    // 1    let publisher = ["A", "B", "C"].publisher        // 2    publisher        .print("publisher")        .count()        .sink(receiveValue: { print("I have \($0) items") })        .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **创建一个发出三个字母的发布者。**
2. **使用 count() 发出单个值，指示上游发布者发出的值的数量。**

**运行你的 Playground 并检查你的控制台。 你将看到以下输出：**

```
——— Example of: count ———publisher: receive subscription: (["A", "B", "C"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive value: (B)publisher: receive value: (C)publisher: receive finishedI have 3 items
```

**正如预期的那样，只有在上游发布者发送 .finished 完成事件后才会打印出值 3。**

**`contains`**

**另一个有用的操作符是 contains。 你可能不止一次地在 Swift 标准库中使用过它。**

**如果上游发布者发出指定的值，则 contains 操作符将发出 true 并取消订阅，如果发出的值都不等于指定的值，则返回 false：**

**将以下内容添加到你的 Playground 以尝试包含：**

```
example(of: "contains") {    // 1    let publisher = ["A", "B", "C", "D", "E"].publisher    let letter = "C"        // 2    publisher        .print("publisher")        .contains(letter)        .sink(receiveValue: { contains in            // 3            print(contains ? "Publisher emitted \(letter)!"                  : "Publisher never emitted \(letter)!")        })        .store(in: &subscriptions)}
```

**在前面的代码中：**

1. **创建一个发出五个不同字母（A 到 E）的发布者，并创建一个与 contains 一起使用的字母值。**
2. **使用 contains 检查上游发布者是否发出了 letter: C 的值。**
3. **根据是否发出值打印适当的消息。**

**运行你的 Playground 并检查控制台：**

```
——— Example of: contains ———publisher: receive subscription: (["A", "B", "C", "D", "E"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive value: (B)publisher: receive value: (C)publisher: receive cancelPublisher emitted C!
```

**你收到一条消息，表明 C 是由发布者发出的。你可能还注意到 contains 是惰性的，因为它只消耗执行其工作所需的上游值。一旦找到 C，它就会取消订阅并且不会产生任何进一步的值。**

**你为什么不尝试另一种变化？替换以下行：**

```
let letter = "C"
```

**为：**

```
let letter = "F"
```

**接下来，再次运行你的游乐场。你将看到以下输出：**

```
——— Example of: contains ———publisher: receive subscription: (["A", "B", "C", "D", "E"])publisher: request unlimitedpublisher: receive value: (A)publisher: receive value: (B)publisher: receive value: (C)publisher: receive value: (D)publisher: receive value: (E)publisher: receive finishedPublisher never emitted F!
```

**在这种情况下， contains 等待发布者发出 F。但是，发布者在没有发出 F 的情况下完成，因此 contains 发出 false 并且你会看到打印出的相应消息。**

**最后，有时你想为你提供的条件查找匹配项，或检查是否存在不符合 Comparable 的发出值。对于这些特定情况，你有 contains(where:)。**

**将以下示例添加到你的 Playground：**

```
example(of: "contains(where:)") {    // 1    struct Person {        let id: Int        let name: String    }        // 2    let people = [        (123, "Shai Mishali"),        (777, "Marin Todorov"),        (214, "Florent Pillet")    ]        .map(Person.init)        .publisher        // 3    people        .contains(where: { $0.id == 800 })        .sink(receiveValue: { contains in            // 4            print(contains ? "Criteria matches!"                  : "Couldn't find a match for the criteria")        })        .store(in: &subscriptions)}
```

**前面的代码有点复杂，但不是很多：**

1. **定义一个带有 id 和 name 的 Person 结构体。**
2. **创建发布人的三个不同实例的发布者。 3.使用contains查看其中任何一个的 id 是否为 800。**
3. **根据发出的结果打印适当的消息。**

**运行你的 Playground，你会看到以下输出：**

```
——— Example of: contains(where:) ———Couldn't find a match for the criteria
```

**正如预期的那样，它没有找到任何匹配项，因为没有一个发出的人的 id 为 800。**

**接下来，更改 contains(where:) 的实现：**

```
.contains(where: { $0.id == 800 })
```

**为：**

```
.contains(where: { $0.id == 800 || $0.name == "Marin Todorov" })
```

**再次运行 Playground 并查看控制台：**

```
——— Example of: contains(where:) ———Criteria matches!
```

**这次它找到了与条件匹配的值，因为 Marin 确实是你列表中的人之一。**

**`allSatisfy`**

**allSatisfy 接受一个闭包条件并发出一个布尔值，指示上游发布者发出的所有值是否与该谓词匹配。 它是贪婪的，因此会等到上游发布者发出 .finished 完成事件：**

**将以下示例添加到你的 Playground 以尝试此操作：**

```
example(of: "allSatisfy") {    // 1    let publisher = stride(from: 0, to: 5, by: 2).publisher        // 2    publisher        .print("publisher")        .allSatisfy { $0 % 2 == 0 }        .sink(receiveValue: { allEven in            print(allEven ? "All numbers are even"                  : "Something is odd...")        })        .store(in: &subscriptions)}
```

**在上面的代码中：**

1. **创建一个以 2 为步长（即 0、2 和 4）发出 0 到 5 之间的数字的发布者。**
2. **使用 allSatisfy 检查是否所有发出的值都是偶数，然后根据发出的结果打印适当的消息。**

**运行代码并检查控制台输出：**

```
——— Example of: allSatisfy ———publisher: receive subscription: (Sequence)publisher: request unlimitedpublisher: receive value: (0)publisher: receive value: (2)publisher: receive value: (4)publisher: receive finishedAll numbers are even
```

**由于所有值确实是偶数，因此在上游发布者发送 .finished 完成后，操作符会发出 true ，并打印出适当的消息。**

**但是，即使单个值没有通过谓词条件，操作符也会立即发出 false 并取消订阅。**

**替换以下行：**

```
let publisher = stride(from: 0, to: 5, by: 2).publisher
```

**为：**

```
let publisher = stride(from: 0, to: 5, by: 1).publisher
```

**你只需将步幅更改为 1，而不是 2，在 0 和 5 之间步进。再次运行 Playground 并查看控制台：**

```
——— Example of: allSatisfy ———publisher: receive subscription: (Sequence)publisher: request unlimitedpublisher: receive value: (0)publisher: receive value: (1)publisher: receive cancel
```

**在这种情况下，一旦发出 1，条件就不再通过，所以 allSatisfy 发出 false 并取消订阅。**

**`reduce`**

**reduce 操作符与本章介绍的其他操作符有些不同。它不查找特定值或查询整个发布者。 相反，它允许你根据上游发布者的排放迭代地累积一个新值。**

**起初这听起来可能令人困惑，但你马上就会明白。 最简单的方法是使用图表：**

![image-20221006154952644](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006154952644.png)

**Combine 的 reduce 操作符与 Swift 标准库中的对应操作符类似：`reduce(_:_)` 和 `reduce(into:_:)`。**

**它允许你提供种子值和累加器闭包。 该闭包接收累积值（从种子值开始）和当前值。 从该闭包中，你返回一个新的累积值。 一旦操作员收到 .finished 完成事件，它就会发出最终的累积值。**

**在上图的情况下，你可以这样想：**

```
Seed value is 0Receives 1, 0 + 1 = 1Receives 3, 1 + 3 = 4Receives 7, 4 + 7 = 11Emits 11
```

**是时候尝试一个简单的例子来更好地理解这个操作符了。将以下内容添加到你的 Playground：**

```
example(of: "reduce") {    // 1    let publisher = ["Hel", "lo", " ", "Wor", "ld", "!"].publisher        publisher        .print("publisher")        .reduce("") { accumulator, value in            // 2            accumulator + value        }        .sink(receiveValue: { print("Reduced into: \($0)") })        .store(in: &subscriptions)}
```

**在此代码中：**

1. **创建一个发出六个字符串的发布者。**
2. **将 reduce 与空字符串一起使用，将发出的值附加到它以创建最终的字符串结果。**

**运行 Playground 并查看控制台输出：**

```
——— Example of: reduce ———publisher: receive subscription: (["Hel", "lo", " ", "Wor", "ld", "!"])publisher: request unlimitedpublisher: receive value: (Hel)publisher: receive value: (lo)publisher: receive value: ( )publisher: receive value: (Wor)publisher: receive value: (ld)publisher: receive value: (!)publisher: receive finishedReduced into: Hello World!
```

**注意累积的结果——Hello World！ — 仅在上游发布者发送 .finished 完成事件后打印。**

**reduce 的第二个参数是一个闭包，它接受两个某种类型的值并返回一个相同类型的值。 在 Swift 中，+ 也是一个匹配该签名的函数。**

**因此，作为最后一个巧妙的技巧，你可以减少上面的语法。 替换以下代码：**

```
.reduce("") { accumulator, value in    // 3    return accumulator + value}
```

**为：**

```
.reduce("", +)
```

**如果你再次运行你的 Playground，它的工作方式会和以前完全一样，只是语法有点花哨。 ;]**

> **备注：这个操作符是不是感觉有点眼熟？ 嗯，这可能是因为你在第 3 节“转换操作符”中了解了扫描。 scan 和 reduce 具有相同的功能，主要区别在于 scan 为每个发出的值发出累积值，而 reduce 在上游发布者发送 .finished 完成事件后发出单个累积值。 在上面的示例中随意将 reduce 更改为 scan 并自己尝试一下。**

#### **关键点**

* **发布者实际上是序列，因为它们产生的值很像集合和序列。**
* **你可以使用 min 和 max 分别发出发布者发出的最小值或最大值。**
* **当你想要查找在特定索引处发出的值时，first、last 和 output(at:) 很有用。使用 output(in:) 查找在索引范围内发出的值。**
* **first(where:) 和 last(where:) 都采用一个条件来确定它应该让哪些值通过。**
* **count、contains 和 allSatisfy 等操作符不会发出发布者发出的值。相反，它们会根据发出的值发出不同的值。**
* **contains(where:) 采用谓词来确定发布者是否包含给定值。**
* **使用 reduce 将发出的值累积为单个值。**

#### **接下来去哪？**

**恭喜你完成了本书的最后一章操作符你将通过处理你的第一个实际项目来结束本节，在该项目中，你将使用 Combine 和你学习的许多操作符构建一个 Collage 应用程序。深吸一口气，喝杯咖啡，然后继续下一节。**
