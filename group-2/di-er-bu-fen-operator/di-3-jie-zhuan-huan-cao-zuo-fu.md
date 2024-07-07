# 第 3 节：转换操作符

#### 第 3 节：转换操作符

完成第一部分后，你已经学到了很多东西。你已经在 Combine 上打下了坚实的基础。在本章中，你将了解 Combine 中操作符的基本类别之一：转换操作符。 你将一直使用转换操作符，将来自发布者的值操作为可供订阅者使用的格式。正如你将看到的，Combine 中的转换操作符和 Swift 标准库中的常规操作符(例如 map 和 flatMap)之间存在相似之处。

#### 入门

打开本章的 playground，它已经导入了 Combine，可以开始编码了。

**Operator 是 Publisher**

Combine 的每个操作符都返回一个发布者。 一般来说，发布者接收上游事件，对其进行操作，然后将操作后的事件向下游发送给消费者。

为了简化这个概念，在本章中，你将专注于使用操作符并处理它们的输出。 除非操作符的目的是处理上游错误，否则它只会在下游重新发布所述错误。

> 注意：本章将重点介绍转换操作符，因此错误处理不会出现在每个操作符示例中。 你将在后续了解有关错误处理的内容。

#### 收集值(Collecting values)

发布者可以发出单个值或值集合。你将经常使用集合，例如当你想要填充列表或网格视图时。 你将在后面学习如何做到这一点。

**collect()**

collect 操作符提供了一种方便的方法来将来自发布者的单值流转换为单个数组。

弹珠图有助于可视化操作符的工作方式。 最上面一行是上游发布者。方框代表操作符，最下方的是订阅者。最下方也可以是另一个操作符，它接收来自上游发布者的输出，执行其操作，并将这些值发送到下游。

![image-20220926014450947](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220926014450947.png)

这个弹珠图图描述了 collect 如何缓冲单值的流，直到上游发布者完成。 然后它向下游发出该数组。

将这个新示例添加到你的 Playground：

```
example(of: "collect") {    ["A", "B", "C", "D", "E"].publisher        .sink(receiveCompletion: { print($0) },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

此代码尚未使用 collect 操作符。 运行 Playground，你会看到每个值都出现在单独的行上，后面跟着一个完成事件：

```
——— Example of: collect ———ABCDEfinished
```

现在在调用 sink 之前使用 collect。 你的代码现在应该如下所示：

```
example(of: "collect") {    ["A", "B", "C", "D", "E"].publisher        .collect()        .sink(receiveCompletion: { print($0) },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

再次运行 Playground，你现在将看到 sink 接收到单个数组值，然后是完成事件：

```
——— Example of: collect ———["A", "B", "C", "D", "E"]finished
```

> 注意：在使用 collect() 和其他不需要指定计数或限制的缓冲操作符时要小心。 它们将使用无限量的内存来存储接收到的值，因为它们不会在上游完成之前发出。

collect 操作符有一些变体。例如你可以指定你只希望接收最多一定数量的值，从而有效地将上游切割成“批次”，将 `.collect()` 替换为：

```
.collect(2)
```

运行 Playground，你将看到以下输出：

```
——— Example of: collect ———["A", "B"]["C", "D"]["E"]finished
```

最后一个值 E 也是一个数组。这是因为上游发布者在 collect 填满其规定的缓冲区之前就完成了，所以它将剩余的所有内容作为数组发送。

#### 映射值(Mapping values)

除了收集值之外，你通常还希望以某种方式转换这些值。 为此，Combine 提供了几个映射操作符。

**`map(_:)`**

首先要了解的是 map，它的工作方式与 Swift 的标准 map 类似，不同之处在于它对发布者发出的值进行操作。 在弹珠图中，map 采用将每个值乘以 2 的闭包。

![image-20220926020723935](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220926020723935.png)

请注意，与 collect 不同，此操作符如何在上游发布值后立即重新发布值。

将这个新示例添加到你的 Playground：

```
example(of: "map") {    // 1    let formatter = NumberFormatter()    formatter.numberStyle = .spellOut        // 2    [123, 4, 56].publisher    // 3        .map {            formatter.string(for: NSNumber(integerLiteral: $0)) ?? ""        }        .sink(receiveValue: { print($0) })        .store(in: &subscriptions)}
```

下面是具体操作：

1. 创建一个数字格式化程序来拼出每个数字。
2. 创建整数发布者。
3. 使用 map，传递一个获取上游值的闭包，并返回使用格式化程序返回数字的拼写字符串的结果。

运行 Playground，你将看到以下输出：

```
——— Example of: map ———one hundred twenty-threefourfifty-six
```

**映射 KeyPath**

map 操作符还包括三个版本，它们可以使用 key path 映射到值的一个、两个或三个属性。 他们的签名如下：

```
map<T>(_:)map<T0, T1>(_:_:)map<T0, T1, T2>(_:_:_:)
```

T 表示在给定 KeyPath 中找到的值的类型。

在下一个示例中，你将使用 `Sources/SupportCode.swift` 中定义的 `Coordinate` 类型和 `quadrantOf(x:y:)` 方法。 坐标有两个属性：x 和 y。 `quadrantOf(x:y:)` 将 x 和 y 值作为参数，并返回一个字符串，指示 x 和 y 值的象限。

如果你有兴趣，请随意查看这些定义，否则只需将 `map(_:_:)` 与以下示例一起用于你的 Playground：

```
example(of: "mapping key paths") {    // 1    let publisher = PassthroughSubject<Coordinate, Never>()        // 2    publisher    // 3        .map(\.x, \.y)        .sink(receiveValue: { x, y in            // 4            print(                "The coordinate at (\(x), \(y)) is in quadrant",                quadrantOf(x: x, y: y)            )        })        .store(in: &subscriptions)        // 5    publisher.send(Coordinate(x: 10, y: -8))    publisher.send(Coordinate(x: 0, y: 5))}
```

在此示例中，你使用 KeyPath 映射到两个属性：

1. 创建一个永远不会发出错误的坐标发布者。
2. 开始订阅发布者。
3. 使用它们的关键路径映射到 Coordinate 的 x 和 y 属性。
4. 打印一条语句，指出提供 x 和 y 值的象限。
5. 通过发布者发送一些坐标。

运行 Playground，此订阅的输出将如下所示：

```
——— Example of: mapping key paths ———The coordinate at (10, -8) is in quadrant 4The coordinate at (0, 5) is in quadrant boundary​
```

**`tryMap(_:)`**

包括 map 在内的几个操作符都有一个带有 try 前缀的对应项，该前缀接受一个抛出的闭包。 如果你抛出错误，操作符将在下游发出该错误。

要尝试 tryMap，请将此示例添加到 Playground：

```
example(of: "tryMap") {    // 1    Just("Directory name that does not exist")    // 2        .tryMap { try FileManager.default.contentsOfDirectory(atPath: $0) }    // 3        .sink(receiveCompletion: { print($0) },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

这是你刚刚做的：

1. 创建表示不存在的目录名称的字符串的发布者。
2. 使用 tryMap 尝试获取该不存在目录的内容。
3. 接收并打印出任何值或完成事件。

请注意，在调用抛出方法时仍需要使用 try 关键字。

运行 Playground 并观察 tryMap 输出失败完成事件：

```
——— Example of: tryMap ———failure(..."The folder “Directory name that does not exist” doesn't exist."...)
```

#### 展平发布者(Flattening publishers)

展平的概念并不太复杂。 通过几个精选示例，你将了解有关它的所有内容。

**`flatMap(maxPublishers:_:)`**

flatMap 操作符将多个上游发布者展平为一个下游发布者。flatMap 返回的发布者与它接收的上游发布者的类型不同，而且通常都是不同的。

在 Combine 中 flatMap 的一个常见用例是，当你想要将一个发布者发出的元素传递给一个本身返回一个发布者的方法，并最终订阅第二个发布者发出的元素时。

添加这个新示例：

```
example(of: "flatMap") {    // 1    func decode(_ codes: [Int]) -> AnyPublisher<String, Never> {        // 2        Just(            codes                .compactMap { code in                    guard (32...255).contains(code) else { return nil }                    return String(UnicodeScalar(code) ?? " ")                }            // 3                .joined()        )        // 4        .eraseToAnyPublisher()    }}
```

1. 定义一个函数，该函数接受一个整数数组，每个整数表示一个 ASCII 码，并返回一个类型擦除的字符串发布者，该发布者从不发出错误。
2. 如果字符代码在 0.255 范围内，则创建一个将其转换为字符串的 Just 发布者，其中包括标准和扩展的可打印 ASCII 字符。
3. 使用 Join 将字符串连接在一起。
4. 擦除发布者类型，匹配函数的返回类型。

> 注意：有关 ASCII 字符代码的更多信息，你可以访问 [www.asciitable.com](https://www.asciitable.com)。

继续将此代码添加到你当前的示例中：

```
// 5[72, 101, 108, 108, 111, 44, 32, 87, 111, 114, 108, 100, 33]    .publisher    .collect()// 6    .flatMap(decode)// 7    .sink(receiveValue: { print($0) })    .store(in: &subscriptions)
```

使用此代码：

1. 将 ASCII 字符代码数组转换为发布者，并将其发出的元素收集到单个数组中。
2. 使用 `flatMap` 将数组元素传递给你的 `decode` 函数。
3. 订阅由 `decode(_:)` 返回的发布者发出的元素并打印出这些值。

运行 Playground，你会看到：

```
——— Example of: flatMap ———Hello, World!
```

回想一下之前的定义：`flatMap` 将所有接收到的发布者的输出展平为一个新发布者。这可能会引起内存问题，因为它会缓冲与你发送它一样多的发布者，以更新它在下游发出的单个发布者。

要了解如何管理它，请看一下 flatMap 的弹珠图：

![image-20220927014008951](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220927014008951.png)

在图中，flatMap 接收三个发布者：P1、P2 和 P3。 这些发布者中的每一个都有一个 value 属性，它也是发布者。 flatMap 从 P1 和 P2 发出值，但忽略 P3，因为 maxPublishers 设置为 2。你将后面的章节中获得更多使用 flatMap 及其 maxPublishers 参数的练习。

你现在掌握了 Combine 中最强大的操作符之一。 但是，flatMap 并不是将输入与不同输出交换的唯一方法。

#### 替换上游输出

在前面的地图示例中，你使用了 Foundation 的 Formatter.string(for:) 方法。 它生成一个可选字符串，并且你使用 nil-coalescing 操作符 (??) 将 nil 值替换为非 nil 值。 Combine 还包括一个操作符，当你希望始终提供价值时可以使用该操作符。

**replaceNil(with:)**

如下图所示，replaceNil 将接收可选值并将 nils 替换为你指定的值：

![image-20220927014630185](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220927014630185.png)

将这个新示例添加到你的 Playground：

```
example(of: "replaceNil") {    // 1    ["A", nil, "C"].publisher        .eraseToAnyPublisher()        .replaceNil(with: "-") // 2        .sink(receiveValue: { print($0) }) // 3        .store(in: &subscriptions)}
```

你刚刚做了：

1. 从可选字符串数组创建发布者。
2. 使用 `replaceNil(with:)` 将来自上游发布者的 nil 值替换为新的非 nil 值。
3. 打印出数值。

> 注意：replaceNil(with:) 有重载，这会使 Swift 为用例类型进行错误的猜测。这导致类型保留为 Optional\<String> 而不是完全展开。 上面的代码使用 eraseToAnyPublisher() 来解决该错误。 你可以在 Swift 论坛中了解有关此问题的更多信息：[https://bit.ly/30M5Qv7](https://bit.ly/30M5Qv7)

运行 Playground，你将看到以下内容：

```
——— Example of: replaceNil ———A-C
```

**replaceEmpty(with:)**

如果发布者完成了但没有发出一个值，你可以使用 `replaceEmpty(with:)` 操作符来替换——或者插入一个值。

在下面的弹珠图中，发布者完成时没有发出任何内容，此时 `replaceEmpty(with:)` 操作符插入一个值并将其发布到下游：

![image-20220927015749133](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220927015749133.png)

添加这个新示例以查看它的实际效果：

```
example(of: "replaceEmpty(with:)") {    // 1    let empty = Empty<Int, Never>()        // 2    empty        .sink(receiveCompletion: { print($0) },              receiveValue: { print($0) })        .store(in: &subscriptions)}
```

你在这里：

1. 创建一个立即发出完成事件的空发布者。
2. 订阅它，并打印收到的事件。

使用 `Empty` 发布者类型创建立即发出 `.finished` 完成事件的发布者。 你还可以通过将 `false` 传递给其 `completeImmediately` 参数来将其配置为从不发出任何内容，默认情况下为 `true`。 此发布者可用于演示或测试目的，或者当你只想向订阅者发送完成某些任务的信号时。 运行 Playground，你会看到它成功完成：

```
——— Example of: replaceEmpty ———finished
```

现在，在调用 sink 之前插入这行代码：

```
.replaceEmpty(with: 1)
```

再次运行 Playground，这次你在完成前得到 1：

```
1finished
```

#### 增量转换输出

你已经看到了 Combine 如何包含诸如 map 之类的操作符，这些操作符与 Swift 标准库中的高阶函数相对应且工作方式相似。 但是，Combine 还有一些技巧可以让你操纵从上游发布者接收到的值。

**`scan(_:_:)`**

转换类别中的一个很好的例子是 scan。 它将上游发布者发出的当前值提供给闭包，以及该闭包返回的最后一个值。

在下面的弹珠图中，扫描从存储起始值 0 开始。当它从发布者接收每个值时，它将其添加到先前存储的值中，然后存储并发出结果：

![image-20220928011847428](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220928011847428.png)

> 注意：和 Playground 相比，如果使用完整的项目来输入和运行此代码，则没有直接的方法来绘制图像。 你可以通过将下面示例中的接收器代码更改为 .sink(receiveValue: { print($0) }) 来打印输出。

请将这个新示例添加到你的 Playground：

```
example(of: "scan") {    // 1    var dailyGainLoss: Int { .random(in: -10...10) }        // 2    let august2019 = (0..<22)        .map { _ in dailyGainLoss }        .publisher        // 3    august2019        .scan(50) { latest, current in            max(0, latest + current)        }        .sink(receiveValue: { _ in })        .store(in: &subscriptions)}
```

这一次，你没有在订阅中打印任何内容。 运行 Playground，然后单击右侧结果边栏中的方形 Show Results 按钮。

![image-20220928012355114](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20220928012355114.png)

还有一个抛出错误的 tryScan 操作符，其工作方式类似。 如果闭包抛出错误，则 tryScan 会因该错误而失败。

#### 挑战

**挑战：使用转换操作符创建电话号码查找**

创建一个可以做两件事的发布者：

1. 接收一串十个数字或字母。
2. 在联系人数据结构中查找该号码。

挑战文件夹中的 Playground 包括一个联系人字典和三个函数。你需要使用转换操作符和这些函数创建对输入发布者的订阅。在将测试你的实现的 forEach 块之前，将你的代码插入到 `Add your code here placeholder` 的正下方。

> 提示：如果函数签名匹配，你可以将函数或闭包作为参数直接传递给操作符。例如，map(convert。

打破这一挑战，你需要：

1. 将输入转换为数字——使用 convert 函数，如果它不能将输入转换为整数，它将返回 nil。
2. 如果前一个操作符返回 nil，则将其替换为 0。
3. 一次采集十个值，对应美国使用的三位数区号和七位数电话号码格式。
4. 格式化收集的字符串值以匹配联系人字典中电话号码的格式——使用提供的格式化函数。
5. 拨打从前一个操作符那里收到的输入。

**解决方案**

首先你需要将字符串一次输入一个字符转换为整数：

```
input    .map(convert)
```

接下来，你需要用 0 替换从 convert 返回的 nil 值：

```
.replaceNil(with: 0)
```

要查找前面操作的结果，你需要收集这些值，然后将它们格式化以匹配联系人字典中使用的电话号码格式：

```
.collect(10).map(format)
```

最后，你需要使用 dial 函数查找格式化的字符串输入，然后订阅：

```
.map(dial).sink(receiveValue: { print($0) })
```

运行 Playground 将产生以下结果：

```
——— Example of: Create a phone number lookup ———Contact not found for 000-123-4567Dialing Marin (408-555-4321)...Dialing Shai (212-555-3434)...
```

#### 关键点

* 你调用对发布者“操作员”的输出执行操作的方法。
* 运营商也是出版商。
* 转换操作符将来自上游发布者的输入转换为适合下游使用的输出。
* 大理石图是可视化每个组合操作员如何工作的好方法。
* 使用任何缓冲值的操作符（如 collect 或 flatMap）时要小心，以避免内存问题。
* 在应用 Swift 标准库中的现有函数知识时要注意。一些名称相似的 Combine 操作符的工作方式相同，而另一些则完全不同。
* 在订阅中将多个操作符链接在一起以对发布者发出的事件创建复杂且复合的转换是很常见的。

#### 接下来去哪？

现在是时候学习如何使用另一个重要的操作符符集合来过滤你从上游发布者那里获得的内容。
