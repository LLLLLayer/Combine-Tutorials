# 第 19 节：测试 Combine 代码

### 第 19 节：测试 Combine 代码

研究表明，开发人员跳过编写测试有两个原因：

1. 他们编写无错误的代码。
2. 你还在读这个吗？

如果你不能直截了当地说你总是写出没有错误的代码——并且假设你对第二个回答是肯定的——那么本章就是为你准备的。 感谢你的陪伴！

编写测试是确保应用程序中预期功能的好方法，因为你正在开发新功能，尤其是在事后，以确保你的最新工作不会在以前运行良好的代码中引入回归。

本章将向你介绍针对你的 Combine 代码编写单元测试，你将在此过程中获得一些乐趣。 你将针对这个方便的应用程序编写测试：

ColorCalc 是使用 Combine 和 SwiftUI 开发的。 不过也有一些问题。 如果它只有一些体面的单元测试来帮助发现和解决这些问题。 还好你在这里！

#### 入门

在 projects/starter 文件夹中打开本章的入门项目。这旨在为你输入的十六进制颜色代码提供红色、绿色、蓝色和不透明度（也称为 alpha）值。如果可能，它还将调整背景颜色以匹配当前十六进制，并在可用时给出颜色名称.如果无法从当前输入的十六进制值导出颜色，则背景将设置为白色。这就是它的设计目的。

你有一个全面的 QA 团队，他们会花时间查找和记录问题。你的工作是简化开发-QA 流程，不仅要修复这些问题，还要编写一些测试来验证修复后的正确功能。运行应用程序并确认你的 QA 团队报告的以下问题：

Issue 1

* 行动：启动应用程序。
* 预期：名称标签应显示为 aqua。
* 实际：名称标签显示 Optional(ColorCalc.ColorNam....

Issue 2

* 操作：点击 ← 按钮。
* 预期：在十六进制显示中删除最后一个字符。
* 实际：删除最后两个字符。

Issue 3

* 操作：点击 ← 按钮。
* 预期：背景变为白色。
* 实际：背景变为红色。

Issue 4

* 行动：点击⊗按钮。
* 预期：十六进制值显示清除为#。
* 实际：十六进制值显示不变。

Issue 5

* 行动：输入十六进制值 006636。
* 预期：红绿蓝不透明度显示显示 0、102、54、255。
* 实际：红-绿-蓝-不透明度显示显示 0、62、32、155。

你很快就会着手编写测试并修复这些问题，但首先，你将通过测试 Combine 的实际代码来学习测试 Combine 代码——等等！具体来说，你将测试一些操作符。

> 注意：本章假设你对 iOS 中的单元测试有一定的了解。如果没有，你仍然可以继续进行，一切都会正常进行。但是，本章不会深入研究测试驱动开发（也称为 TDD）的细节。如果你想更深入地了解该主题，请查看 raywenderlich.com 中的 iOS 测试驱动开发教程。

#### 测试 Combine 操作符

在本章中，你将使用 Given-When-Then 模式来组织你的测试逻辑：

* 给定一个条件。
* 执行操作时。
* 然后出现预期的结果。

仍然在 ColorCalc 项目中，打开 ColorCalcTests/CombineOperatorsTests.swift。

首先，添加一个订阅属性来存储订阅，并在 tearDown() 中将其设置为一个空数组。 你的代码应如下所示：

```
var subscriptions = Set<AnyCancellable>()override func tearDown() {  subscriptions = []}
```

**测试 collect()**

你的第一个测试将针对 collect。 回想一下，这个操作符将缓冲上游发布者发出的值，等待它完成，然后在下游发出一个包含这些值的数组。

使用 Given-When-Then 模式，通过在 tearDown() 下方添加以下代码来开始新的测试方法：

```
func test_collect() {  // Given  let values = [0, 1, 2]  let publisher = values.publisher}
```

使用此代码，你可以创建一个整数数组，然后从该数组创建一个发布者。

现在，将此代码添加到测试中：

```
// Whenpublisher  .collect()  .sink(receiveValue: {    // Then    XCTAssert(     $0 == values,     "Result was expected to be \(values) but was \($0)"    )  })  .store(in: &subscriptions)
```

在这里，你使用 collect 操作符，然后订阅其输出，断言输出等于 calues - 并存储订阅。

你可以通过多种方式在 Xcode 中运行单元测试：

1. 要运行单个测试，请单击方法定义旁边的菱形。
2. 要在单个测试类中运行所有测试，请单击类定义旁边的菱形。
3. 要在项目的所有测试目标中运行所有测试，请按 Command-U。请记住，每个测试目标可能包含多个测试类，每个测试类都可能包含多个测试。
4. 你还可以使用 Product ▸ Perform Action ▸ Run “TestClassName”——它也有自己的键盘快捷键：Command-Control-Option-U。

通过单击 test\_collect() 旁边的菱形来运行此测试。 该项目将在执行测试时在模拟器中短暂构建和运行，然后报告它是成功还是失败。

正如预期的那样，测试将通过，你将看到以下内容：![image-20221026014631569](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221026014631569.png)

测试定义旁边的菱形也会变成绿色并包含一个复选标记。

你还可以通过 View ▸ Debug Area ▸ Activate Console 菜单项或按 Command-Shift-Y 来显示控制台以查看有关测试结果的详细信息（此处截断的结果）：

```
Test Suite 'Selected tests' passed at 2021-08-25 00:44:59.629.     Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.007) seconds
```

要验证此测试是否正常工作，请将断言代码更改为：

```
XCTAssert(  $0 == values + [1],  "Result was expected to be \(values + [1]) but was \($0)")
```

你将 1 添加到与 collect() 发出的数组以及消息中的内插值进行比较的 values 数组。

重新运行测试，你会看到它失败了，并显示消息 Result 应该是 \[0, 1, 2, 1] 但是是 \[0, 1, 2]。你可能需要单击错误以展开并查看完整消息或显示控制台，完整消息也将打印在那里。

在继续之前撤消最后一组更改，然后重新运行测试以确保它通过。

> 注意：出于时间和空间的考虑，本章将侧重于编写测试符合条件的测试。但是，如果你有兴趣，我们鼓励你通过其他测试进行实验。请记住在继续之前将测试恢复到原始的通过状态。

这是一个相当简单的测试。下一个示例将测试一个更复杂的操作符。

**测试 flatMap(maxPublishers:)**

正如你在第 3 节“转换操作符”中所了解的，flatMap 操作符可用于将多个上游发布者扁平化为单个发布者，你可以选择指定它将接收和扁平化的最大发布者数量。

通过添加以下代码为 flatMap 添加新的测试方法：

```
func test_flatMapWithMax2Publishers() {  // Given  // 1  let intSubject1 = PassthroughSubject<Int, Never>()  let intSubject2 = PassthroughSubject<Int, Never>()  let intSubject3 = PassthroughSubject<Int, Never>()  // 2  let publisher = CurrentValueSubject<PassthroughSubject<Int, Never>, Never>(intSubject1)  // 3  let expected = [1, 2, 4]  var results = [Int]()  // 4  publisher    .flatMap(maxPublishers: .max(2)) { $0 }    .sink(receiveValue: {      results.append($0)    })    .store(in: &subscriptions)}
```

你可以通过创建以下内容开始此测试：

1. 三个需要整数值的 PassthroughSubject 实例。
2. 一个本身接受和发布整数 PassthroughSubject 的 CurrentValueSubject，用第一个整数Subject 初始化。
3. 预期结果和一个数组来保存收到的实际结果。
4. 订阅发布者，使用最多两个发布者的 flatMap。在处理程序中，你将收到的每个值附加到结果数组中。

这照顾了Given。现在将此代码添加到你的测试中以创建操作：

```
// When// 5intSubject1.send(1)// 6publisher.send(intSubject2)intSubject2.send(2)// 7publisher.send(intSubject3)intSubject3.send(3)intSubject2.send(4)// 8publisher.send(completion: .finished)
```

因为发布者是 CurrentValueSubject，所以它将当前值重播给新订阅者。因此，使用上面的代码，你可以继续该发布者的工作，并且：

1. 向第一个整数发布者发送一个新值。
2. 通过 CurrentValueSubject 发送第二个整数 Subject，然后向该 Subject 发送一个新值。
3. 对第三个整数 Subject 重复上一步，但这次传递两个值。
4. 通过当前值主题发送完成事件。

完成此测试剩下的就是断言这些操作将产生预期的结果。添加此代码以创建此断言：

```
// ThenXCTAssert(  results == expected,  "Results expected to be \(expected) but were \(results)")
```

通过单击其定义旁边的菱形来运行测试，你将看到它以绚丽的色彩通过！

如果你以前有响应式编程的经验，你可能熟悉使用测试调度程序，它是一种虚拟时间调度程序，可让你对基于时间的测试操作进行精细控制。

在撰写本文时，Combine 不包括正式的测试调度程序。 不过，一个名为 Entwine ([https://github.com/tcldr/Entwine](https://github.com/tcldr/Entwine)) 的开源测试调度程序已经可用，如果你需要一个正式的测试调度程序，那么值得一看。

但是，鉴于本书重点使用苹果原生的 Combine 框架，当你想测试 Combine 代码时，绝对可以使用 XCTest 的内置功能。 这将在你的下一个测试中展示。

**测试 publish(every**_**:on**_**:in:)**

在下一个示例中，被测系统将是 Timer 发布者。

你可能还记得第 11 节“定时器”，这个发布器可用于创建重复定时器，而无需大量样板设置代码。为了测试这一点，你将使用 XCTest 的期望 API 来等待异步操作完成。

通过添加以下代码开始一个新的 test：

```
func test_timerPublish() {  // Given  // 1  func normalized(_ ti: TimeInterval) -> TimeInterval {    return Double(round(ti * 10) / 10)  }  // 2  let now = Date().timeIntervalSinceReferenceDate  // 3  let expectation = self.expectation(description: #function)  // 4  let expected = [0.5, 1, 1.5]  var results = [TimeInterval]()  // 5  let publisher = Timer    .publish(every: 0.5, on: .main, in: .common)    .autoconnect()    .prefix(3)}
```

在此设置代码中：

1. 定义一个辅助函数，通过四舍五入到小数点后标准化时间间隔。
2. 存储当前时间间隔。
3. 创建一个期望，用于等待异步操作完成。
4. 定义预期结果和一个数组来存储实际结果。
5. 创建一个自动连接的计时器发布者，并且只获取它发出的前三个值。请参阅第 11 章，“定时器”以重新了解此操作符的详细信息。

接下来，添加此代码以测试此发布者：

```
// Whenpublisher  .sink(    receiveCompletion: { _ in expectation.fulfill() },    receiveValue: {      results.append(        normalized($0.timeIntervalSinceReferenceDate - now)      )    }  )  .store(in: &subscriptions)
```

在上面的订阅处理程序中，你使用辅助函数来获取每个发出日期的时间间隔的规范化版本，然后将其附加到结果数组中。

完成后，就该等待发布者完成其工作并完成，然后进行验证。

添加此代码以执行此操作：

```
// Then// 6waitForExpectations(timeout: 2, handler: nil)// 7XCTAssert(  results == expected,  "Results expected to be \(expected) but were \(results)")
```

你在这里：

1. 最多等待 2 秒。
2. 断言实际结果等于预期结果。

运行测试，你将再次通过测试——Apple 的联合团队 +1，这里的一切都像宣传的那样工作！

说到这，到目前为止，你已经测试了 Combine 内置的操作符。 为什么不测试一个自定义操作符，比如你在第 18 节“自定义发布者和处理背压”中创建的那个？

**测试 shareReplay(capacity:)**

该操作符提供了一个常用的功能：与多个订阅者共享发布者的输出，同时还将最后 N 个值的缓冲区重播给新订阅者。此操作符采用指定滚动缓冲区大小的容量参数。再次，请参阅第 18 节，“自定义发布者和处理背压”以获取有关此操作符的更多详细信息。

你将在下一个测试中测试此操作符的共享和重播组件。添加此代码以开始使用：

```
func test_shareReplay() {  // Given  // 1  let subject = PassthroughSubject<Int, Never>()  // 2  let publisher = subject.shareReplay(capacity: 2)  // 3  let expected = [0, 1, 2, 1, 2, 3, 3]  var results = [Int]()}
```

与之前的测试类似：

1. 创建一个 Subject 以向其发送新的整数值。
2. 使用容量为两个的 shareReplay 从该主题创建发布者。
3. 定义预期结果并创建一个数组来存储实际输出。

接下来，添加此代码以触发应产生预期输出的操作：

```
// When// 4publisher  .sink(receiveValue: { results.append($0) })  .store(in: &subscriptions)// 5subject.send(0)subject.send(1)subject.send(2)// 6publisher  .sink(receiveValue: { results.append($0) })  .store(in: &subscriptions)// 7subject.send(3)
```

从顶部：

1. 创建对发布者的订阅并存储任何发出的值。
2. 通过发布者分享重播的 Subject 发送一些值。
3. 创建另一个订阅并存储任何发出的值。
4. 通过 Subject 再发送一个值。

完成后，剩下的就是确保这个操作符是最新的，只需要创建一个断言。 添加此代码以结束此测试：

```
XCTAssert(  results == expected,  "Results expected to be \(expected) but were \(results)")
```

这是与前两个测试相同的断言代码。

运行这个测试，瞧，你有一个真正的成员值得在你的联合驱动项目中使用！

通过学习如何测试这种小范围的 Combine 操作符，你已经掌握了测试几乎任何 Combine 操作符所需要的技能。 在下一部分中，你将通过测试之前看到的 ColorCalc 应用程序来练习这些技能。

#### 测试生产代码

在本章的开头，你发现了 ColorCalc 应用程序的几个问题。 现在是时候做点什么了。

该项目使用 MVVM 模式组织，你需要测试和修复的所有逻辑都包含在应用程序的唯一视图模型中：CalculatorViewModel。

> 注意：应用程序在 SwiftUI 视图文件等其他方面可能存在问题，但是 UI 测试不是本章的重点。 如果你发现自己需要针对你的 UI 代码编写单元测试，这可能表明你的代码应该重新组织以分离职责。 为此，MVVM 是一种有用的架构设计模式。 如果你想了解更多关于 MVVM 和 Combine 的信息，请查看[ MVVM 和 Combine 教程](https://www.kodeco.com/4161005-mvvm-with-combine-tutorial-for-ios)。

打开 ColorCalcTests/ColorCalcTests.swift，并在 ColorCalcTests 类定义的顶部添加以下两个属性：

```
var viewModel: CalculatorViewModel!var subscriptions = Set<AnyCancellable>()
```

你将在每个测试之前重置两个属性的值，即在每个测试之前的 viewModel 和在每个测试之后的订阅。将 setUp() 和 tearDown() 方法更改为如下所示：

```
override func setUp() {  viewModel = CalculatorViewModel()}override func tearDown() {  subscriptions = []}
```

**问题 1：显示的名称不正确**

有了该设置代码，你现在可以针对视图模型编写第一个测试。添加此代码：

```
func test_correctNameReceived() {  // Given  // 1  let expected = "rwGreen 66%"  var result = ""  // 2  viewModel.$name    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  // 3  viewModel.hexText = "006636AA"  // Then  // 4  XCTAssert(    result == expected,    "Name expected to be \(expected) but was \(result)"  )}
```

这是你所做的：

1. 存储此测试的预期名称标签文本。
2. 订阅视图模型的 $name 发布者并保存接收到的值。
3. 执行应触发预期结果的操作。
4. 断言实际结果等于预期结果。

运行此测试，它将失败并显示以下消息：

```
Name expected to be rwGreen 66% but was Optional(ColorCalc.ColorName.rwGreen)66%. 
```

打开 View Models/CalculatorViewModel.swift。在类定义的底部是一个名为 configure() 的方法。这个方法在初始化器中被调用，它是所有视图模型订阅设置的地方。首先，创建一个 hexTextShared 发布者来共享 hexText 发布者。

设置 name 的订阅：

```
hexTextShared  .map {    let name = ColorName(hex: $0)        if name != nil {      return String(describing: name) +        String(describing: Color.opacityString(forHex: $0))    } else {      return "------------"    }  }  .assign(to: &$name)
```

现在返回 ColorCalcTests/ColorCalcTests.swift 并重新运行 test\_correctNameReceived()。它通过了！

查看该代码。 你知道有什么问题吗？ 与其只检查 ColorName 的本地名称实例是否为 nil，不如使用可选绑定来解包非 nil 值。

将整 个map 代码块更改为以下内容：

```
.map {  if let name = ColorName(hex: $0) {    return "\(name) \(Color.opacityString(forHex: $0))"  } else {    return "------------"  }}
```

无需修复并重新运行项目一次来验证修复，你现在有一个测试，可以在每次运行测试时验证代码是否按预期工作。

**问题 2：点击退格键会删除两个字符**

仍然在 ColorCalcTests.swift 中，添加这个新测试：

```
func test_processBackspaceDeletesLastCharacter() {  // Given  // 1  let expected = "#0080F"  var result = ""  // 2  viewModel.$hexText    .dropFirst()    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  // 3  viewModel.process(CalculatorViewModel.Constant.backspace)  // Then  // 4  XCTAssert(    result == expected,    "Hex was expected to be \(expected) but was \(result)"  )}
```

与之前的测试类似：

1. 设置你期望的结果并创建一个变量来存储实际结果。
2. 订阅 viewModel.$hexText 并保存删除第一个重放值后获得的值。
3. 调用 viewModel.process(\_:) 传递一个表示 ← 字符的常量字符串。
4. 断言实际结果和预期结果相等。

运行测试，如你所料，它失败了。这次的消息是 Hex 应该是#0080F 但是#0080。

回到 CalculatorViewModel 并找到 process(\_:) 方法：

```
case Constant.backspace:  if hexText.count > 1 {    hexText.removeLast(2)  }
```

这一定是在开发过程中被一些手动测试留下的。修复再简单不过了：删除 2 以便 removeLast() 只删除最后一个字符。

返回 ColorCalcTests，重新运行 test\_processBackspaceDeletesLastCharacter()，它通过了！

**问题 3：背景颜色不正确**

编写单元测试在很大程度上可能是一种反复进行的活动。下一个测试遵循与前两个相同的方法。将此新测试添加到 ColorCalcTests：

编写单元测试在很大程度上可能是一种反复进行的活动。下一个测试遵循与前两个相同的方法。将此新测试添加到 ColorCalcTests：

```
func test_correctColorReceived() {  // Given  let expected = Color(hex: ColorName.rwGreen.rawValue)!  var result: Color = .clear  viewModel.$color    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  viewModel.hexText = ColorName.rwGreen.rawValue  // Then  XCTAssert(    result == expected,    "Color expected to be \(expected) but was \(result)"  )}
```

这次你正在测试视图模型的 $color 发布者，当 viewModel.hexText 设置为 rwGreen 时，期望颜色的十六进制值是 rwGreen。起初这似乎没有做任何事情，但请记住，这是测试 $color 发布者是否为输入的十六进制值输出正确的值。

运行测试，它通过了！你做错什么了吗？绝对不！编写测试意味着尽可能主动，如果不是更被动的话。你现在有一个测试，可以验证输入的十六进制颜色是否正确。因此，请务必保持该测试以警惕未来可能出现的回归。

不过，回到这个问题的绘图板上。想想看。是什么导致了这个问题？是你输入的十六进制值，还是……等一下，又是那个←按钮！

添加此测试，以验证点击 ← 按钮时接收到正确的颜色：

```
func test_processBackspaceReceivesCorrectColor() {  // Given  // 1  let expected = Color.white  var result = Color.clear  viewModel.$color    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  // 2  viewModel.process(CalculatorViewModel.Constant.backspace)  // Then  // 3  XCTAssert(    result == expected,    "Hex was expected to be \(expected) but was \(result)"  )}
```

从顶部：

1. 为预期和实际结果创建本地值，并订阅 viewModel.$color，与之前的测试相同。
2. 这次处理退格输入——而不是像之前的测试那样显式设置十六进制文本。
3. 验证结果是否符合预期。

运行此测试并失败并显示以下消息：

```
Hex was expected to be white but was red. The last word here is the most important one: red. 
```

你可能需要打开控制台才能查看整个消息。

跳回 CalculatorViewModel 并查看在 configure() 中设置颜色的订阅：

```
colorValuesShared  .map { $0 != nil ? Color(values: $0!) : .red }  .assign(to: &$color)
```

也许将背景设置为红色是另一个从未被预期值替换的快速开发时间测试？当无法从当前的十六进制值导出颜色时，该设计要求背景为白色。通过将地图实现更改为：

```
.map { $0 != nil ? Color(values: $0!) : .white }
```

返回 ColorCalcTests，运行 test\_processBackspaceReceivesCorrectColor()，它通过了。

到目前为止，你的测试主要集中在测试正确条件上。接下来，你将对否定条件进行测试。

**测试错误输入**

此应用程序的 UI 将阻止用户为十六进制值输入错误数据。

但是，事情可能会发生变化。例如，也许有一天你将十六进制 Text 更改为 TextField，以允许粘贴值。因此，现在添加一个测试来验证当为十六进制值输入错误数据时的预期结果是一个好主意。

将此测试添加到 ColorCalcTests：

```
func test_whiteColorReceivedForBadData() {  // Given  let expected = Color.white  var result = Color.clear  viewModel.$color    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  viewModel.hexText = "abc"  // Then  XCTAssert(    result == expected,    "Color expected to be \(expected) but was \(result)"  )}
```

本次测试与上一次几乎相同。唯一的区别是，这一次，你将错误数据传递给 hexText。

运行这个测试，它就会通过。但是，如果添加或更改了逻辑，从而可能为十六进制值输入错误数据，你的测试将在此问题交到用户手中之前发现它。

还有两个问题需要测试和修复。但是，你已经掌握了技能。因此，你将在下面的挑战部分解决剩余的问题。

在此之前，继续使用 Product ▸ Test 菜单运行所有现有测试，或者按 Command-U 通过测试。

#### 挑战

**挑战 1：解决问题 4：点击清除不会清除十六进制显示**

目前，点击⊗无效。它应该将十六进制显示清除为#。编写一个由于十六进制显示未正确更新而失败的测试，识别并修复有问题的代码，然后重新运行测试并确保它通过。

> 提示：常量 CalculatorViewModel.Constant.clear 可用于 ⊗ 字符。

**解决方案**

这个挑战的解决方案看起来几乎与你之前编写的 test\_processBackspaceDeletesLastCharacter() 测试相同。唯一的区别是预期的结果只是#，并且动作是通过⊗而不是←。这个测试应该是这样的：

```
func test_processClearSetsHexToHashtag() {  // Given  let expected = "#"  var result = ""  viewModel.$hexText    .dropFirst()    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  viewModel.process(CalculatorViewModel.Constant.clear)  // Then  XCTAssert(    result == expected,    "Hex was expected to be \(expected) but was \"\(result)\""  )}
```

按照你在本章中已经多次完成的相同分步过程，你将：

* 创建本地值以存储预期和实际结果。
* 订阅 $hexText 发布者。
* 执行应产生预期结果的操作。
* 断言预期等于实际。

按原样在项目上运行此测试将失败，并显示 Hex 应为 # 但为 "" 的消息。

研究视图模型中的相关代码，你会发现在 process(\_:) 中处理 Constant.clear 输入的案例只有一个中断。

解决方法是将 break 更改为 hexText = "#"。

**挑战 2：解决问题 5：输入的十六进制的红绿蓝不透明度显示不正确**

目前，将应用启动时显示的初始十六进制更改为其他内容后，红绿蓝不透明度 (RGBO) 显示不正确。这可能是那种从开发中得到“无法重现”响应的问题，因为它“在我的设备上运行良好”。幸运的是，你的 QA 团队在输入 006636 等值后提供了显示不正确的明确说明，这将导致 RGBO 显示设置为 0、102、54、170。

因此，你将创建的最初会失败的测试如下所示：

```
func test_correctRGBOTextReceived() {  // Given  let expected = "0, 102, 54, 170"  var result = ""  viewModel.$rgboText    .sink(receiveValue: { result = $0 })    .store(in: &subscriptions)  // When  viewModel.hexText = "#006636AA"  // Then  XCTAssert(    result == expected,    "RGBO text expected to be \(expected) but was \(result)"  )}
```

缩小到此问题的原因，你会在 CalculatorViewModel.configure() 中找到设置 RGBO 显示的订阅代码：

```
colorValuesShared  .map { values -> String in    if let values = values {      return [values.0, values.1, values.2, values.3]        .map { String(describing: Int($0 * 155)) }        .joined(separator: ", ")    } else {      return "---, ---, ---, ---"    }  }  .assign(to: &$rgboText)
```

此代码当前使用不正确的值来乘以发出的元组中返回的每个值。它应该是 255，而不是 155，因为每个红色、绿色、蓝色和不透明度字符串应该代表从 0 到 255 的基础值。

将 155 更改为 255 即可解决问题，随后测试将通过。

#### 关键点

* 单元测试有助于确保你的代码在初始开发期间按预期工作，并且不会在以后引入回归。
* 你应该组织你的代码，将你将进行单元测试的业务逻辑与你将进行 UI 测试的表示逻辑分开。 MVVM 是非常适合此目的的模式。
* 它有助于使用 Given-When-Then 等模式来组织你的测试代码。
* 你可以使用期望来测试基于时间的异步组合代码。
* 测试正确和错误条件都很重要。

#### 接下来去哪儿？

很棒的工作！你已经测试了几个不同的 Combine 操作符，并为以前未经测试和不守规矩的代码库带来了法律和秩序。

在你越过终点线之前还有一节。你将完成一个完整的 iOS 应用程序，该应用程序利用你在整本书中学到的知识，包括本节。
