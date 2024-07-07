# 第 12 节：KVO

### 第 12 节: KVO

应对变化是 Combine 的核心。 发布者让你订阅它们以处理异步事件。在前面的章节中，你了解了 `assign(to:on:)`，它使你能够在每次发布者发出新值时更新对象属性的值。

但是，观察单个变量变化的机制呢？

* 它为符合 KVO（Key-Value Observing）的对象的任何属性提供发布者。
* ObservableObject 协议处理多个变量可能发生变化的情况。

#### 介绍 publisher(for:options:)

KVO 一直是 Objective-C 的重要组成部分。 Foundation、UIKit 和 AppKit 类的大量属性都符合 KVO。 因此，你可以使用 KVO 观察它们的变化。

很容易观察到符合 KVO 的属性。 下面是一个使用 OperationQueue（来自 Foundation 的类）的示例：

```
let queue = OperationQueue()​let subscription = queue.publisher(for: \.operationCount)  .sink {    print("Outstanding operations in queue: \($0)")  }
```

每次向队列添加新操作时，它的 operationCount 都会增加，并且你的接收器会收到新的计数。当队列消耗了一个 operation 时，计数会减少，并且你的接收器会再次收到更新的计数。

还有许多其他框架类公开了符合 KVO 的属性。 只需将 `publisher(for:)` 与 KVO 兼容属性的关键路径一起使用，你将获得一个能够发出值变化的发布者。 你将在本章稍后部分了解有关此功能和可用选项的更多信息。

> 注意：Apple 没有在其整个框架中提供符合 KVO 的属性的中央列表。 每个类的文档通常会指出哪些属性是 KVO 兼容的。 但有时文档可能很少，你只能在文档中找到一些属性的快速注释，甚至在系统标题本身中。

#### 准备和订阅自己的 KVO 兼容属性

你还可以在自己的代码中使用 Key-Value Observing，前提是：

1. 你的对象是类（不是结构）并且符合 NSObject，
2. 使用 @objc 动态属性标记属性以使其可观察。

完成此操作后，你标记的对象和属性将与 KVO 兼容，并且可以使用 Combine！

> 注意：虽然 Swift 语言不直接支持 KVO，但将属性标记为 @objc dynamic 会强制编译器生成触发 KVO 机制的隐藏方法。 描述这种机器超出了本书的范围。 可以说该机制严重依赖 NSObject 协议中的特定方法，这解释了为什么你的对象需要遵守它。

在 Playground 上尝试一个例子：

```
// 1class TestObject: NSObject {  // 2  @objc dynamic var integerProperty: Int = 0}​let obj = TestObject()​// 3let subscription = obj.publisher(for: \.integerProperty)  .sink {    print("integerProperty changes to \($0)")  }​// 4obj.integerProperty = 100obj.integerProperty = 200
```

在上面的代码中：

1. 创建一个符合 NSObject 协议的类。这是 KVO 所必需的。
2. 将你想要使其可观察的任何属性标记为 @objc 动态。
3. 创建并订阅观察 obj 的 integerProperty 属性的发布者。
4. 更新属性几次。

在 Playground 中运行此代码时，你能猜出调试控制台显示的内容吗？

你可能会感到惊讶，但这是你获得的显示：

```
integerProperty changes to 0integerProperty changes to 100integerProperty changes to 200
```

你首先获得 integerProperty 的初始值，即 0，然后你会收到两个更改。如果你对它不感兴趣，你可以避免这个初始值——继续阅读以了解如何！

你是否注意到在 TestObject 中你使用的是普通的 Swift 类型 (Int)，而作为 Objective-C 特性的 KVO 仍然有效？ KVO 可以与任何 Objective-C 类型以及任何桥接到 Objective-C 的 Swift 类型一起正常工作。这包括所有原生 Swift 类型以及数组和字典，只要它们的值都可以桥接到 Objective-C。

试试看！向 TestObject 添加更多属性：

```
@objc dynamic var stringProperty: String = ""@objc dynamic var arrayProperty: [Float] = []
```

以及订阅其发布者：

```
let subscription2 = obj.publisher(for: \.stringProperty)  .sink {    print("stringProperty changes to \($0)")  }​let subscription3 = obj.publisher(for: \.arrayProperty)  .sink {    print("arrayProperty changes to \($0)")  }
```

最后，一些属性变化：

```
obj.stringProperty = "Hello"obj.arrayProperty = [1.0]obj.stringProperty = "World"obj.arrayProperty = [1.0, 2.0]
```

你将在调试区域中看到初始值和更改。 好的！

如果你曾经使用过没有桥接到 Objective-C 的纯 Swift 类型，你就会开始遇到麻烦：

```
struct PureSwift {  let a: (Int, Bool)}
```

然后，向 TestObject 添加一个属性：

```
@objc dynamic var structProperty: PureSwift = .init(a: (0,false))
```

你会立即在 Xcode 中看到一个错误，指出“属性不能被标记为 @objc，因为它的类型不能在 Objective-C 中表示。” 在这里，你达到了 Key-Value Observing 的局限。

> 注意：观察系统框架对象的变化时要小心。 确保文档提到该属性是可观察的，因为你无法仅通过查看系统对象的属性列表来获得线索。 Foundation、UIKit、AppKit 等都是如此。从历史上看，必须使属性“感知 KVO”才能被观察。

#### Observation options

你调用以观察更改的方法的完整签名是 `publisher(for:options:)`。 `options` 参数是一个具有四个值的选项集：`.initial`、`.prior`、`.old` 和 `.new`。 默认值为 `[.initial]`，这就是为什么你会看到发布者在发出任何更改之前发出初始值。 以下是选项的细分：

* `.initial` 发出初始值。
* `.prior` 在发生更改时发出先前的值和新的值。
* `.old` 和 `.new` 在此发布者中未使用，它们都什么都不做（只是让新值通过）。

如果你不想要初始值，你可以简单地写：

```
obj.publisher(for: \.stringProperty, options: [])
```

如果你指定 `.prior`，则每次发生更改时都会获得两个单独的值。 修改 integerProperty 示例：

```
let subscription = obj.publisher(for: \.integerProperty, options: [.prior])
```

你现在将在 integerProperty 订阅的调试控制台中看到以下内容：

```
integerProperty changes to 0integerProperty changes to 100integerProperty changes to 100integerProperty changes to 200
```

该属性首先从 0 更改为 100，因此你获得两个值：0 和 100。然后，它从 100 更改为 200，因此你再次获得两个值：100 和 200。

#### ObservableObject

Combine 的 ObservableObject 协议适用于 Swift 对象，而不仅仅适用于派生自 NSObject 的对象。 它与 @Published 属性包装器合作，帮助你使用编译器生成的 objectWillChange 发布者创建类。

它使你免于编写大量样板文件，并允许创建可以自我监控自己的属性并在它们中的任何一个发生更改时通知的对象。

这是一个例子：

```
class MonitorObject: ObservableObject {  @Published var someProperty = false  @Published var someOtherProperty = ""}​let object = MonitorObject()let subscription = object.objectWillChange.sink {  print("object will change")}
```

`ObservableObject` 协议一致性使编译器自动生成 `objectWillChange` 属性。 它是一个 `ObservableObjectPublisher`，它发出 Void 项目并且永不失败。

每次对象的 @Published 变量之一发生更改时，都会触发 objectWillChange。 不幸的是，你无法知道实际更改了哪个属性。 这旨在与 SwiftUI 很好地配合使用，它可以合并事件以简化屏幕更新。

#### 关键点

* Key-Value Observing 主要依赖于 Objective-C 运行时和 NSObject 协议的方法。
* Apple 框架中的许多 Objective-C 类提供了一些符合 KVO 的属性。
* 你可以让你自己的属性可观察，只要它们是符合 NSObject 的类，并用 @objc 动态属性标记。
* 你还可以遵守 ObservableObject 并为你的属性使用 @Published。 编译器生成的 objectWillChange 发布者在每次 @Published 属性之一更改时触发（但不会告诉你更改了哪一个）。

#### 接下来去哪？

继续阅读以了解 Combine 中的资源，以及如何通过共享它们来保存它们！
