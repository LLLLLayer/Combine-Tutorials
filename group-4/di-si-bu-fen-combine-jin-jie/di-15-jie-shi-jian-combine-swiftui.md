# 第 15 节：实践：Combine & SwiftUI

### 第 15 节：实践：Combine & SwiftUI

SwiftUI 是 Apple 用于以声明方式构建应用程序 UI 的最新技术。这与旧的 UIKit 和 AppKit 框架有很大的不同。它为构建用户界面提供了一种非常简洁且易于阅读和编写的语法。SwiftUI 语法清楚地代表了你想要构建的视图层次结构：

```
HStack(spacing: 10) {  Text("My photo")  Image("myphoto.png")    .padding(20)    .resizable()}
```

你可以轻松地直观地解析层次结构。 HStack 视图——一个水平堆栈——包含两个子视图：一个文本视图和一个图像视图。

每个视图都可以有一个修饰符列表——它们是你在视图上调用的方法。在上面的示例中，你使用视图修饰符 padding(20) 在图像周围添加 20 个填充点。此外，你还可以使用 resizable() 来启用图像内容的大小调整。

SwiftUI 还统一了构建跨平台 UI 的方法。例如，一个 Picker 控件在你的 iOS 应用程序中显示一个新的模式视图，允许用户从列表中选择一个项目，但在 macOS 上，相同的 Picker 控件显示一个 Dropbox。

数据表单的快速代码示例可能是这样的：

```
VStack {  TextField("Name", text: $name)  TextField("Proffesion", text: $profession)  Picker("Type", selection: $type) {    Text("Freelance")    Text("Hourly")    Text("Employee")  }}
```

此代码将在 iOS 上创建两个单独的视图。类型选择器控件将是一个按钮，将用户带到一个单独的屏幕，其中包含如下选项列表：

![image-20221015154833446](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015154833446.png)

然而，在 macOS 上，SwiftUI 会考虑 mac 上丰富的 UI 屏幕空间，并创建一个带有下拉菜单的单一表单：

![image-20221015154850556](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015154850556.png)

最后，在 SwiftUI 中，屏幕上呈现的用户界面是你的状态的函数。你维护此状态的单个副本，称为“事实来源”，并且 UI 是从该状态动态派生的。幸运的是，Combine 发布者可以轻松地作为数据源插入 SwiftUI 视图。

#### 你好，SwiftUI！

如上一节所述，使用 SwiftUI 时，你以声明方式描述用户界面，并将渲染留给框架。

你为 UI 声明的每个视图（文本标签、图像、形状等）都符合 View 协议。 View 的唯一要求是一个名为 body 的属性。

每当你更改数据模型时，SwiftUI 都会向你的每个视图询问它们当前的主体表示。这可能会根据你最新的数据模型更改而发生变化。然后，该框架通过仅计算受模型更改影响的视图来构建视图层次结构以在屏幕上呈现，从而产生高度优化和有效的绘图机制。

实际上，SwiftUI 使 UI “快照”由数据模型的任何更改触发，如下所示：

![image-20221015155023945](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015155023945.png)

在本章中，你将完成许多任务，这些任务涵盖了 Combine 和 SwiftUI 之间的互操作以及一些 SwiftUI 基础知识。

#### 内存管理

上述 UI 工作的内容很大一部分是内存管理的转变。

#### 无数据重复

让我们看一个例子来说明这意味着什么。在使用 UIKit/AppKit 时，你可以粗略地说，将你的代码在数据模型、某种控制器和视图之间分离：

![image-20221015155348997](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015155348997.png)

这三种类型可以有几个相似的特征。它们包括数据存储、支持可变性、可以是引用类型等等。

假设你想在屏幕上显示当前天气。对于此示例，假设模型类型是一个名为 Weather 的结构，并将当前条件存储在一个名为 conditions 的文本属性中。要向用户显示该信息，你需要创建另一种类型的实例，即 UILabel，并将条件的值复制到标签的文本属性中。

现在，你有两个你使用的值的副本。一个在你的模型类型中，另一个存储在 UILabel 中，只是为了在屏幕上显示它：

![image-20221015155511414](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015155511414.png)

文本和条件之间没有联系或绑定。你只需将字符串值复制到你需要的任何地方。

现在，你已向 UI 添加了依赖项。屏幕上信息的新鲜度取决于 Weather.conditions。每当条件属性更改时，你有责任使用 Weather.conditions 的新副本手动更新标签的文本属性。

SwiftUI 消除了为了在屏幕上显示而复制数据的需要。能够从 UI 中卸载数据存储，可以让你在模型中的一个位置有效地管理数据，并且永远不会让应用程序的用户在屏幕上看到陈旧的信息。

**更少需要“控制”你的视图**

作为额外的奖励，消除在模型和视图之间使用“胶水”代码的需要也可以让你摆脱大部分视图控制器代码！

在本章中，你将学习：

* 简要介绍用于构建声明性 UI 的 SwiftUI 语法基础知识。
* 如何声明各种类型的 UI 输入并将它们连接到它们的“事实来源”。
* 如何使用 Combine 构建数据模型并将数据通过管道传输到 SwiftUI。

> 注意：如果你想了解有关 SwiftUI 的更多信息，请考虑查看 SwiftUI by Tutorials ([https://bit.ly/2L5wLLi](https://bit.ly/2L5wLLi)) 以获得深入的学习体验。

现在，对于我们的功能演示：Combine 和 SwiftUI！

#### “News”入门

本章的入门项目已经包含一些代码，以便你可以专注于编写连接 Combine 和 SwiftUI 的代码。

该项目还包括一些文件夹，你可以在其中找到以下内容：

* App 包含 main app type。
* Network 包括上一章完整的 Hacker News API。
* Model 是你可以找到简单模型类型的地方，例如 Story、FilterKeyword 和 Settings。 此外，这是 ReaderViewModel 所在的位置，这是主新闻阅读器视图使用的模型类型。
* View 包含应用程序视图，在 View/Helpers 中，你会发现一些简单的可重用组件，如 button、badge 等。
* 最后，在 Util 中有一个帮助类型，允许你轻松地从磁盘读取和写入 JSON 文件。

完成的项目将显示 Hacker News Story 列表，并允许用户管理关键字过滤器：

![image-20221015160403861](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015160403861.png)

#### 初体验管理视图状态

构建并运行启动项目，你将在屏幕上看到一个空表和一个标题为“设置”的单条按钮：

![image-20221015160510604](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015160510604.png)

这是你开始的地方。 要了解通过更改数据与 UI 交互的工作原理，你将让设置按钮在点击时显示 SettingsView。

打开 View/ReaderView.swift，其中包含显示主应用程序界面的 ReaderView 视图。

该类型已经包含一个名为 presentingSettingsSheet 的属性，它是一个简单的布尔值。 更改此值将显示或关闭设置视图。 向下滚动源代码并找到注释 Set presentingSettingsSheet to true here，替换为：

```
self.presentingSettingsSheet = true
```

添加此行后，你将看到以下错误：

![image-20221015160706352](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015160706352.png)

确实 self 是不可变的，因为 view 的 body 是动态属性，因此不能改变 ReaderView。

SwiftUI 提供了许多内置的属性包装器来帮助你指示给定属性是你的状态的一部分，并且对这些属性的任何更改都应该触发一个新的 UI“快照”。

让我们看看这在实践中意味着什么。 调整普通的旧 presentingSettingsSheet 属性，使其如下所示：

```
@State var presentingSettingsSheet = false
```

@State 属性包装器：

1. 将属性存储移出视图，因此修改 presentingSettingsSheet 不会改变 self。
2. 将属性标记为本地存储。换句话说，它表示该数据段由视图拥有。
3. 向 ReaderView 添加一个名为 $presentingSettingsSheet 的发布者，有点像 @Published，你可以使用它来订阅该属性或将其绑定到 UI 控件或其他视图。

将 @State 添加到 presentingSettingsSheet 后，错误将清除，因为编译器知道你可以从 non-mutating 上下文中修改此特定属性。

最后，要使用 presentingSettingsSheet，你需要声明新状态如何影响UI。在这种情况下，你将向视图层次结构添加一个 sheet(...) 视图修饰符，并将 $presentingSettingsSheet 绑定到工作表。每当你更改 presentingSettingsSheet 时，SwiftUI 将采用当前值并根据布尔值呈现或关闭你的视图。

找到 // Present the Settings sheet here 将其替换为：

```
.sheet(isPresented: self.$presentingSettingsSheet, content: {  SettingsView()})
```

sheet(isPresented:content:) 修饰符接受一个 Bool 发布者和一个视图，以在演示发布者发出 true 时呈现。

构建并运行项目。点击设置，你的新演示文稿将显示目标视图：

![image-20221015161203408](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015161203408.png)

#### 获取最新 Storiy

接下来，是时候回到一些 Combine 代码了。 在本节中，你将合并现有的 ReaderViewModel 并将其连接到 API 。

打开 Model/ReaderViewModel.swift。 在顶部，插入：

```
import Combine
```

这段代码自然允许你在 ReaderViewModel.swift 中使用 Combine 类型。 现在，向 ReaderViewModel 添加一个新的订阅属性来存储你的所有订阅：

```
private var subscriptions = Set<AnyCancellable>()
```

有了所有扎实的准备工作，现在是时候创建一种新方法并使用网络 API。 将以下空方法添加到 ReaderViewModel：

```
func fetchStories() {}
```

在此方法中，你将订阅 API.stories() 并将服务器响应存储在模型类型中。 你应该从上一章中熟悉此方法。

在 fetchStories() 中添加以下内容：

```
api  .stories()  .receive(on: DispatchQueue.main)
```

你使用 receive(on:) 操作符接收主队列上的任何输出。 可以说，你可以将线程管理留给 API 的使用者。但是，由于在 ReaderViewModel 的情况下肯定是 ReaderView，因此你在此处进行优化并切换到主队列以准备提交对 UI 的更改。

接下来，你将使用 sink(...) 订阅者在模型中存储故事和任何发出的错误。 附加：

```
.sink(receiveCompletion: { completion in  if case .failure(let error) = completion {    self.error = error  }}, receiveValue: { stories in  self.allStories = stories  self.error = nil}).store(in: &subscriptions)
```

首先，你检查 completion 是否失败。 如果是这样，你将关联的错误存储在 self.error 中。 如果你从 Story 发布者那里收到值，则将它们存储在 self.allStories 中。

这就是你要在本节中添加到模型的所有逻辑。 fetchStories() 方法现已完成，你可以在屏幕上显示 ReaderView 后立即“启动”模型。

为此，打开 App/App.swift 并向 ReaderView 添加一个新的 onAppear(...) 视图修饰符，如下所示：

```
ReaderView(model: viewModel)  .onAppear {    viewModel.fetchStories()  }
```

现在，ReaderViewModel 并没有真正连接到 ReaderView，所以你不会在屏幕上看到任何变化。 但是，要快速验证一切是否按预期工作，请执行以下操作：返回 Model/ReaderViewModel.swift 并将 didSet 处理程序添加到 allStories 属性：

```
private var allStories = [Story]() {  didSet {    print(allStories.count)  }}
```

运行应用程序并观察控制台。 你应该会看到一个令人放心的输出，如下所示：

```
1234...
```

你可以删除刚刚添加的 didSet 处理程序，以防你不想在每次运行应用程序时都看到该输出。

#### 将 ObservableObject 用于 model

ObservableObject 是一种使普通旧数据模型可观察的协议，并让观察者 SwiftUI 视图知道数据已更改，因此它能够重建依赖于该数据的任何用户界面。

该协议要求类型实现一个名为 objectWillChange 的发布者，该发布者在类型的状态即将改变时发出。

协议中已经有该发布者的默认实现，因此在大多数情况下，你不必向数据模型添加任何内容。当你将 ObservableObject 一致性添加到你的类型时，默认协议实现将在你的任何 @Published 属性发出时自动发出！

打开 ReaderViewModel.swift 并将 ObservableObject 一致性添加到 ReaderViewModel，所以它看起来像这样：

```
class ReaderViewModel: ObservableObject {
```

接下来，你需要考虑数据模型的哪些属性构成了它的状态。你当前在 sink(...) 订阅者中更新的两个属性是 allStories 和 error。你会认为那些状态改变是值得的。

> 注意：还有第三个属性叫做过滤器。暂时忽略它，稍后你会回到它。

调整 allStories 以包含 @Published 属性包装器，如下所示：

```
@Published private var allStories = [Story]()
```

然后，对错误执行相同的操作：

```
@Published var error: API.Error? = nil
```

本节的最后一步是，由于 ReaderViewModel 现在符合 ObservableObject，因此将数据模型实际绑定到 ReaderView。

打开 View/ReaderView.swift 并将 @ObservedObject 属性包装器添加到行 var model: ReaderViewModel 中，如下所示：

```
@ObservedObject var model: ReaderViewModel
```

你绑定模型，以便在其状态更改时，你的视图将接收最新数据并生成其新的 UI“快照”。

@ObservedObject 包装器执行以下操作：

1. 从视图中删除属性存储，并改为使用与原始模型的绑定。换句话说，它不会复制数据。
2. 将属性标记为外部存储。换句话说，它表示该数据不属于视图。
3. 与@Published 和@State 一样，它向属性添加了一个发布者，以便你可以订阅它和/或在视图层次结构中进一步绑定到它。

通过添加@ObservedObject，你可以使模型动态化。这意味着它会在你的视图模型从 Hacker News 服务器获取故事时获取所有更新。事实上，现在运行应用程序，你将看到视图在模型获取 Story 时自行刷新：

![image-20221015163909480](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015163909480.png)

#### 显示错误

你还将以与显示获取的故事相同的方式显示错误。目前，视图模型将任何错误存储在其错误属性中，你可以将其绑定到屏幕上的 UI 警报。

打开 View/ReaderView.swift 并找到注释 // Display errors here。将此注释替换为以下代码以将模型绑定到提示视图：

```
.alert(item: self.$model.error) { error in  Alert(    title: Text("Network error"),     message: Text(error.localizedDescription),    dismissButton: .cancel()  )}
```

alert(item:) 修饰符控制屏幕上的警报显示。 它接受一个带有可选输出的绑定，称为项目。 每当该绑定源发出非零值时，UI 都会显示警报视图。

模型的错误属性默认为 nil，并且只有在模型从服务器获取故事时遇到错误时才会设置为非 nil 错误值。 这是呈现警报的理想方案，因为它允许你将错误直接绑定为 alert(item:) 输入。

要对此进行测试，请打开 Network/API.swift 并将 baseURL 属性修改为无效 URL，例如 [https://123hacker-news.firebaseio.com/v0/](https://123hacker-news.firebaseio.com/v0/)。

再次运行应用程序，一旦对故事端点的请求失败，你将看到错误警报显示：

![image-20221015165117006](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015165117006.png)

在继续完成下一部分之前，花点时间将你的更改恢复为 baseURL，以便你的应用程序再次成功连接到服务器。

#### 订阅外部发布者

有时你不想走 ObservableObject/ObservedObject 路线，因为你想做的只是订阅一个发布者并在你的 SwiftUI 视图中接收它的值。对于像这种更简单的情况，不需要创建额外的类型——你可以简单地使用 onReceive(\_) 视图修饰符。它允许你直接从你的视图订阅发布者。

如果你现在运行该应用程序，你将看到每个故事都有一个相对时间，并包含在故事作者的姓名旁边：

![image-20221015165231389](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015165231389.png)

那里的相对时间有助于立即将故事的“新鲜度”传达给用户。然而，一旦呈现在屏幕上，信息会在一段时间后变得陈旧。如果用户长时间打开应用程序，“1 分钟前”可能会关闭一段时间。

在本节中，你将使用计时器发布者定期触发 UI 更新，以便每一行都可以重新计算并显示正确的时间。

代码现在的工作方式如下：

* ReaderView 有一个名为 currentDate 的属性，该属性在创建视图时使用当前日期设置一次。
* Story 列表中的每一行都包含一个 PostedBy(time:user:currentDate:) 视图，该视图使用 currentDate 的值编译作者和时间信息。

要定期“刷新”屏幕上的信息，你将添加一个新的计时器发布者。每次它发出时，你都会更新 currentDate。此外，正如你可能已经猜到的那样，你会将 currentDate 添加到视图的状态中，以便在它发生变化时触发新的 UI“快照”。

要与发布者合作，首先在 ReaderView.swift 的顶部添加：

```
import Combine
```

然后，向 ReaderView 添加一个新的发布者属性，该属性创建一个新的计时器发布者，只要有人订阅它就可以立即使用：

```
private let timer = Timer.publish(every: 10, on: .main, in: .common)  .autoconnect()  .eraseToAnyPublisher()
```

正如你在本书前面已经了解到的那样，其返回一个可连接的发布者。这是一种“休眠”发布者，需要订阅者连接到它才能激活它。上面你使用 autoconnect() 来指示发布者在订阅时自动“唤醒”。

现在剩下的是在每次计时器发出时更新 currentDate 。你将使用名为 onReceive(\_) 的 SwiftUI 修饰符，其行为与 sink(receiveValue:) 订阅者非常相似。向下滚动一点，找到 // Add timer here 将其替换为：

```
.onReceive(timer) {  self.currentDate = $0}
```

计时器发出当前日期和时间，因此你只需将该值分配给 currentDate。这样做会产生一个有点熟悉的错误：

自然，发生这种情况是因为你无法从非变异上下文中对属性进行变异。和以前一样，你将通过将 currentDate 添加到视图的本地存储状态来解决这个难题。

像这样向属性添加 @State 属性包装器：

```
@State var currentDate = Date()
```

这样，对 currentDate 的任何更新都会触发一个新的 UI“快照”，并强制每一行重新计算故事的相对时间，并在必要时更新文本。

再次运行该应用程序并使其保持打开状态。 记下头条新闻是多久前发布的，这是我尝试发布的内容：

等待至少一分钟，你将看到可见行将其信息更新为当前时间。 橙色时间徽章仍将显示故事发布的时间，但标题下方的文本将更新为正确的“...分钟前”文本：

![image-20221015165735835](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015165735835.png)

除了让发布者在你的视图上具有属性外，你还可以通过视图的初始化程序或环境将 Combine 模型中的任何发布者注入到视图中。然后，只需以与上述相同的方式使用 onReceive(...) 即可。

#### 初始化应用程序的设置

在本章的这一部分，你将继续使设置视图工作。在处理 UI 本身之前，你需要先完成 Settings 类型的实现。

打开 Model/Settings.swift，你会看到，目前，该类型几乎是简陋的。它包含一个包含 FilterKeyword 值列表的属性。

现在，打开 Model/FilterKeyword.swift。 FilterKeyword 是一种辅助模型类型，它包装了一个关键字，用作主阅读器视图中故事列表的过滤器。它符合 Identifiable 要求，它需要一个唯一标识每个实例的 id 属性，例如当你在 SwiftUI 代码中使用这些类型时。如果你分别仔细阅读 Network/API.swift 和 Model/Story.swift 中的 API.Error 和 Story 定义，你会发现这些类型也符合 Identifiable。

你需要将普通的旧模型设置转换为现代类型，以便与你的组合和 SwiftUI 代码一起使用。

通过在 Model/Settings.swift 顶部添加开始：

```
import Combine
```

然后，通过将 @Published 属性包装器添加到 keywords 来为 keywords 添加发布者，如下所示：

```
@Published var keywords = [FilterKeyword]()
```

现在，其他类型可以订阅设置的当前关键字。你还可以通过管道将关键字列表传递给接受绑定的视图。

最后，为了启用对设置的观察，使类型符合 ObservableObject ，如下所示：

```
final class Settings: ObservableObject {
```

无需添加任何其他内容即可使 ObservableObject 符合性工作。默认实现将在 $keywords 发布者发出的任何时候发出。

这就是你如何通过几个简单的步骤将“设置”变成类固醇的模型类型。现在，你可以将其插入应用程序中的其余反应式代码中。

要绑定应用程序的设置，你将在应用程序中对其进行实例化。打开 App/App.swift 并向 HNReader 添加一个新属性：

```
let userSettings = Settings()
```

像往常一样，你还需要一个可取消的集合来存储你的订阅。 为此向 HNReader 添加一个属性：

```
private var subscriptions = Set<AnyCancellable>()
```

现在，你可以将 Settings.keywords 绑定到 ReaderViewModel.filter 以便主视图不仅会收到初始关键字列表，还会在用户每次编辑关键字列表时收到更新列表。

你将在初始化 HNReader 时创建该绑定。 向该类型添加一个新的初始化程序：

```
init() {  userSettings.$keywords    .map { $0.map { $0.value } }    .assign(to: \.filter, on: viewModel)    .store(in: &subscriptions)}
```

你订阅 userSettings.$keywords，它输出 \[FilterKeyword]，并通过获取每个关键字的 value 属性将其映射到 \[String]。然后，将结果值分配给 viewModel.filter。

现在，无论何时更改 Settings.keywords 的内容，与视图模型的绑定最终都会导致 ReaderView 的新 UI“快照”的生成，因为视图模型是其状态的一部分。

到目前为止，绑定有效。但是，你仍然必须添加 filter 属性才能成为 ReaderViewModel 状态的一部分。你将这样做，以便每次更新关键字列表时，新数据都会转发到视图。

为此，请打开 Model/ReaderViewModel.swift 并添加 @Published 属性包装器以进行过滤，如下所示：

```
@Published var filter = [String]()
```

从设置到视图模型再到视图的完整绑定现已完成！

这非常方便，因为在下一节中，你会将 Settings 视图连接到 Settings 模型，并且用户对关键字列表所做的任何更改都将触发整个绑定和订阅链，最终刷新主应用程序视图 Story 列表，例如：

![image-20221015171432194](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015171432194.png)

#### 编辑关键字列表

在本章的最后一部分，你将了解 SwiftUI 环境。环境是自动注入视图层次结构的发布者共享池。

**系统环境**

环境包含系统注入的发布者，如当前日历、布局方向、语言环境、当前时区等。如你所见，这些都是可能随时间变化的值。因此，如果你声明视图的依赖项，或者将它们包含在你的状态中，则视图将在依赖项更改时自动重新呈现。

要尝试观察其中一项系统设置，请打开 View/ReaderView.swift 并向 ReaderView 添加一个新属性：

```
@Environment(\.colorScheme) var colorScheme: ColorScheme
```

你使用 @Environment 属性包装器，它定义了环境的哪个键应该绑定到 colorScheme 属性。现在，这个属性是视图状态的一部分。每次系统外观模式在明暗之间切换，反之亦然，SwiftUI 都会重新渲染你的视图。

此外，你将可以访问视图主体中的最新配色方案。因此，你可以在明暗模式下以不同方式渲染它。

向下滚动并找到设置 Story 链接颜色的行 .foregroundColor(Color.blue)。将该行替换为：

```
.foregroundColor(self.colorScheme == .light ? .blue : .orange)
```

现在，根据 colorScheme 的当前值，链接将是蓝色或橙色。

通过将系统外观更改为深色来尝试这种新的代码奇迹。在 Xcode 中，打开 Debug ► View Debugging ► Configure Environment Overrides... 或点击 Xcode 底部工具栏上的 Environment Overrides 按钮。然后，打开界面样式旁边的开关。

![image-20221015171722618](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015171722618.png)

**自定义环境对象**

就像通过 @Environment(\_) 观察系统设置一样酷，但这并不是 SwiftUI 环境必须提供的全部。事实上，你也可以环境化你的对象！

这非常方便。尤其是当你有深度嵌套的视图层次结构时。将模型或其他共享资源插入环境中，无需通过大量视图进行依赖注入，直到你到达实际需要数据的深度嵌套视图。

你在视图环境中插入的对象可自动用于该视图的任何子视图及其所有子视图。

这听起来像是与应用程序的所有视图共享用户设置的绝佳机会，以便他们可以使用用户的故事过滤器。

将依赖项注入所有视图的地方是主应用程序文件。这是你之前创建 Settings 的 userSettings 实例并将其 $keywords 绑定到 ReaderViewModel 的位置。现在，你还将将 userSettings 注入到环境中。

打开 App/App.swift 并将 environmentObject 视图修改器添加到 ReaderView，方法是在 ReaderView(model: viewModel) 下方添加：

```
.environmentObject(userSettings)
```

environmentObject 修饰符是一个视图修饰符，它将给定对象插入到视图层次结构中。由于你已经有一个设置实例，你只需将其发送到环境即可。

接下来，你需要将环境依赖项添加到要使用自定义对象的视图中。打开 View/SettingsView.swift 并使用 @EnvironmentObject 包装器添加一个新属性：

```
@EnvironmentObject var settings: Settings
```

设置属性将自动填充环境中的最新用户设置。

对于你自己的对象，你不需要像系统环境那样指定密钥路径。 @EnvironmentObject 会将属性类型（在本例中为设置）与存储在环境中的对象匹配并找到正确的对象。

现在，你可以像使用任何其他视图状态一样使用 settings.keywords。你可以直接获取该值、订阅它或将其绑定到其他视图。

要完成 SettingsView 功能，你将显示关键字列表并启用从列表中添加、编辑和删除关键字。

找到以下行：

```
ForEach([FilterKeyword]()) { keyword in
```

并将其替换为：

```
ForEach(settings.keywords) { keyword in
```

更新后的代码将使用屏幕列表的过滤器关键字。但是，这仍将显示一个空列表，因为用户无法添加新关键字。

初始项目包括一个用于添加关键字的视图。因此，你只需在用户点击 + 按钮时显示它。 + 按钮操作在 SettingsView 中设置为 addKeyword()。

滚动到 addKeyword() 方法并在其中添加：

```
presentingAddKeywordSheet = true
```

presentingAddKeywordSheet 是一个 published 的属性，与本章前面已经使用过的属性非常相似，用于显示提示。你可以在源代码中稍微向上看到演示文稿声明：.sheet(isPresented: $presentingAddKeywordSheet)。

要尝试手动将对象注入给定视图的工作原理，请切换到 View/ReaderView.swift 并找到你提供 SettingsView 的位置——它是你只需创建一个新实例的单行代码，如下所示：SettingsView()。

与将设置注入 ReaderView 的方式相同，你也可以在此处注入它们。向 ReaderView 添加一个新属性：

```
@EnvironmentObject var settings: Settings
```

然后，直接在 SettingsView() 下添加 .environmentObject 修饰符：

```
.environmentObject(self.settings)
```

现在，你声明了对 Settings 的 ReaderView 依赖项，并通过环境将该依赖项传递给 SettingsView。在这种特殊情况下，你也可以将它作为参数传递给 SettingsView 的 init。

在继续之前，请再次运行该应用程序。你应该能够点击设置并看到 SettingsView 弹出。

现在，切换回 View/SettingsView.swift 并按照最初的预期完成列表编辑操作。

在 sheet(isPresented: $presentingAddKeywordSheet) 中，已经为你创建了一个新的 AddKeywordView。这是一个包含在启动项目中的自定义视图，它允许用户输入一个新的关键字并点击一个按钮将其添加到列表中。

AddKeywordView 接受一个回调，当用户点击按钮添加新关键字时，它将调用该回调。在 AddKeywordView 的空完成回调中添加：

```
let new = FilterKeyword(value: newKeyword.lowercased())self.settings.keywords.append(new)self.presentingAddKeywordSheet = false
```

你创建一个 keyword，将其添加到用户设置中，最后关闭显示的工作表。

请记住，在此处将关键字添加到列表中将更新设置模型对象，进而更新阅读器视图模型并刷新阅读器视图。全部按照你的代码中的声明自动进行。

结束 SettingsView，让我们添加删除和移动关键字。查找 // List editing actions 并将其替换为：

```
.onMove(perform: moveKeyword).onDelete(perform: deleteKeyword)
```

此代码将 moveKeyword() 设置为当用户在列表中向上或向下移动其中一个关键字时的处理程序，并在用户向右滑动删除关键字时将 deleteKeyword() 设置为处理程序。

在当前为空的 moveKeyword(from:to:) 方法中，添加：

```
guard let source = source.first,      destination != settings.keywords.endIndex else { return }​settings.keywords  .swapAt(source,          source > destination ? destination : destination - 1)
```

在 deleteKeyword(at:) 中，添加：

```
settings.keywords.remove(at: index.first!)
```

这就是你在列表中启用编辑所需的全部内容！最后一次构建并运行应用程序，你将能够完全管理 Filter 过滤器，包括添加、移动和删除关键字：

![image-20221015174400897](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015174400897.png)

此外，当你导航回故事列表时，你将看到设置与你的订阅和绑定一起在应用程序中传播，并且列表仅显示与你的过滤器匹配的故事。标题还将显示匹配故事的数量：

![image-20221015174540737](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015174540737.png)

#### 挑战

本章包括两个完全可选的 SwiftUI 练习，你可以选择完成这些练习。你也可以将它们搁置一旁，在接下来的章节中继续讨论更令人兴奋的 Combine 主题。

**挑战 1：在阅读器视图中显示过滤器**

在第一个挑战中，你将在 ReaderView 的故事列表标题中插入过滤器关键字列表。目前，标题始终显示“显示所有故事”。更改该文本以显示关键字列表，以防用户添加任何关键字，如下所示：

![image-20221015174620950](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221015174620950.png)

**挑战 2：在应用启动之间保持过滤器**

启动项目包括一个名为 JSONFile 的辅助类型，它提供两种方法：loadValue(named:) 和 save(value:named:)。

使用此类型：

* 每当用户通过将 didSet 处理程序添加到 Settings.keywords 来修改过滤器时，将关键字列表保存在磁盘上。
* 在 Settings.init() 中从磁盘加载关键字。

这样，用户的过滤器将在应用程序启动之间保持不变，就像在真实应用程序中一样。

如果你不确定这些挑战中的任何一个的解决方案，或者需要一些帮助，请随时查看项目/挑战文件夹中的已完成项目。

#### 关键点

* 使用 SwiftUI，你的 UI 是你状态的函数。你可以通过提交对声明为视图状态的数据以及其他视图依赖项的更改来呈现自己的 UI。你学习了在 SwiftUI 中管理状态的各种方法：
* 使用@State 将本地状态添加到视图，并使用@ObservedObject 在你的组合代码中添加对外部 ObservableObject 的依赖。
* 使用 onReceive 视图修饰符直接订阅外部发布者。
* 使用 @Environment 将依赖项添加到系统提供的环境设置之一，并使用 @EnvironmentObject 为你自己的自定义环境对象添加依赖项。

#### 接下来去哪儿？

恭喜你使用 SwiftUI 和 Combine 轻松搞定！我希望你现在意识到两者之间的联系是多么紧密和强大，以及 Combine 如何在 SwiftUI 的响应式功能中发挥关键作用。

即使你应该始终致力于编写无错误的应用程序，但世界很少如此完美。这正是为什么你将在下一章中学习如何在 Combine 中处理错误的原因。
