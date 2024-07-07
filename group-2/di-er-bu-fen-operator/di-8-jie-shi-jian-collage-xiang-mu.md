# 第 8 节：实践：“Collage”项目

### 第 8 节：实践：“Collage”项目

在过去的几章中，你学到了很多关于在 Swift Playground 中使用发布者、订阅者和各种不同的操作符的知识。但是现在，是时候让这些新技能发挥作用，并使用真正的 iOS 应用程序来动手了。

为了结束本节，你将处理一个包含真实场景的项目，你可以在其中应用新获得的组合知识。

本项目将带你完成：

* 将 Combine 发布者与照片等系统框架结合使用。
* 使用 Combine 处理用户事件。
* 使用各种操作符创建不同的订阅来驱动你的应用程序的逻辑。
* 包装现有的 Cocoa API，以便你可以方便地在你的组合代码中使用它们。

该项目名为 Collage Neue，它是一个 iOS 应用程序，允许用户从他们的照片中创建简单的拼贴画，如下所示：

![image-20221006170607898](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006170607898.png)

在你继续学习更多操作符之前，该项目将为你提供一些使用 Combine 的实践经验，并且是一个很好的突破理论繁重的章节。

你将完成许多松散连接的任务，在这些任务中，你将使用基于你在本书中所涵盖的材料的技术。

此外，你将使用一些稍后将介绍的操作符来帮助你为应用程序的一些高级功能提供支持。

事不宜迟——是时候开始编码了！

#### “Collage Neue”入门

要开始使用 Collage Neue，请打开本章材料提供的启动项目。该应用程序的结构相当简单——有一个用于创建和预览拼贴画的主视图和一个附加视图，用户可以在其中选择照片以添加到他们正在进行的拼贴画中：

![image-20221006171319538](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006171319538.png)

> 注意：在本章中，你将专门练习使用 Combine。你将尝试各种绑定数据的方式，但不会专注于专门使用 Combine 和 SwiftUI；你将在后文中了解如何同时使用这两个框架：Combine & SwiftUI。

目前，该项目没有实现任何逻辑。 但是，它确实包含一些你可以利用的代码，因此你可以只关注组合相关的代码。 让我们首先充实将照片添加到当前拼贴画的用户交互。

打开 `CollageNeueModel.swift` 并在文件顶部导入 Combine 框架：

```
import Combine
```

这将允许你在 model 文件中使用 Combine 类型。 首先，向 `CollageNeueModel` 类添加两个新的私有属性：

```
private var subscriptions = Set<AnyCancellable>()private let images = CurrentValueSubject<[UIImage], Never>([])
```

`subscriptions` 是一个集合，你将在其中存储与主视图或模型本身的生命周期相关的任何订阅。如果模型发布，或者你手动重置订阅，所有正在进行的订阅将被方便地取消。

> 注意：如前文所述，订阅者返回一个 Cancellable token 以允许控制 subscription 的生命周期。AnyCancellable 是一种类型擦除类型，允许将不同类型的 Cancellable 项存储在同一个集合中，就像上面的代码一样。

你将使用图像为当前拼贴发出用户当前选择的照片。将数据绑定到 UI 控件时，最适合使用 `CurrentValueSubject` 而不是 `PassthroughSubject`。前者始终保证在订阅时至少会发送一个值，并且你的 UI 永远不会有未定义的状态。

一般来说，`CurrentValueSubject` 非常适合表示状态，例如照片数组或加载状态，而 `PassthroughSubject` 更适合表示事件，例如用户点击按钮或简单地指示发生了某事。

接下来，要将一些图像添加到拼贴中并测试你的代码，请将以下行附加到 add() 中：

```
images.value.append(UIImage(named: "IMG_1907")!)
```

每当用户点击绑定到 `CollageNeueModel.add()` 的右上角导航项中的 + 按钮时，你将添加 IMG\_1907.jpg 到当前图像数组并通过 subject 发送该值。

你可以在项目的资产目录中找到 IMG\_1907.jpg — 这是几年前我在巴塞罗那附近拍摄的一张漂亮照片。

方便的是，`CurrentValueSubject` 允许你直接改变它的值，而不是使用 `send(_:)` 发出新值。两者是相同的，因此你可以使用感觉更好的语法 - 你可以在下一段中尝试 `send(_:)`。

为了还能够清除当前选定的照片，请移至同一个文件中的 clear()，并在其中添加：

```
images.send([])
```

此行发送一个空数组作为图像的最新值。

最后，你需要将 images subjec t绑定到屏幕上的视图。有不同的方法可以做到这一点，但为了在本实用章节中涵盖更多内容，你将为此使用 @Published 属性。

像这样向你的 model 添加一个新属性：

```
@Published var imagePreview: UIImage?
```

@Published 是一个属性包装器，由于你的模型符合 `ObservableObject`，因此将 `imagePreview` 绑定到屏幕上的视图变得非常简单。

滚动到 `bindMainView()` 并添加此代码以将图像绑定到屏幕上的图像预览。

```
// 1images  // 2  .map { photos in    UIImage.collage(images: photos, size: Self.collageSize)  }  // 3  .assign(to: &$imagePreview)
```

上述代码：

1. 你开始订阅当前的照片集 images。
2. 你可以通过调用 `UIImage.collage(images:size:)` 来使用 `map` 将它们转换为单个拼贴，这是一个在 `UIImage+Collage.swift` 中定义的帮助方法。
3. 你使用 `assign(to:)` 订阅者将生成的拼贴图像绑定到 `imagePreview`，这是中心屏幕图像视图。 使用 `assign(to:)` 订阅者自动管理订阅生命周期。

最后但同样重要的是，你需要在视图中显示 `imagePreview`。 打开 `MainView.swift` 并找到 `Image(uiImage: UIImage())` 行。 将其替换为：

```
Image(uiImage: model.imagePreview ?? UIImage())
```

你使用最新的预览，如果预览不存在，则使用空的 UIImage。

是时候测试新订阅了！ 构建并运行应用程序并单击 + 按钮几次。 你应该会看到拼贴预览，每次单击 + 时都会显示同一张照片的多个副本：

![image-20221006173346341](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006173346341.png)

你将获得照片集，将其转换为 collage 并将其分配给 subscription 中的图像视图！

但是，通常你需要更新的不是一个 UI 控件，而是多个。为每个绑定创建单独的订阅可能是矫枉过正。那么，让我们看看如何将多个更新作为一个批次执行。

`MainView` 中已经包含一个名为 `updateUI(photosCount:)` 的方法，它会执行各种 UI 更新：当当前选择包含奇数张照片时，它会禁用保存按钮，当拼贴正在进行时启用清除按钮和更多。

要在用户每次向拼贴添加照片时调用 `upateUI(photosCount:)`，你将使用 `handleEvents(...)` 操作符。如前所述，这是在你想要执行诸如日志记录或其他方面的副作用时使用的操作符。

通常，建议从 `sink(...)` 或 `assign(to:on:)` 更新 UI，但为了尝试一下，在本节中，你将在 handleEvents 中执行此操作。

回到 CollageNeueModel.swift 并添加一个新属性：

```
let updateUISubject = PassthroughSubject<Int, Never>()
```

要练习使用 subject 在不同类型之间进行通信（例如在这种情况下，你正在使用它以便你的 model 可以“回复”你的视图），你可以添加一个名为 `updateUISubject` 的新 subject。

通过这个新 subject，你将发出当前所选照片的数量，以便视图可以观察计数并相应地更新其状态。

在 `bindMainView()` 中，将此操作符插入到使用 map 的行之前：

```
.handleEvents(receiveOutput: { [weak self] photos in  self?.updateUISubject.send(photos.count)})
```

> 注意：`handleEvents` 操作符使你能够在发布者发出事件时执行副作用。你将在后文了解更多相关信息。

这会将当前选择提供给 `updateUI(photosCount:)` 在它们被转换为 map 操作符内的单个拼贴图像之前。

现在，在 `MainView` 中观察 `updateUISubject`，打开 `MainView.swift` 和 `.onAppear(...)` 正下方的新修饰符：

```
.onReceive(model.updateUISubject, perform: updateUI)
```

此修饰符观察给定的发布者并在视图的生命周期内调用 `updateUI(photosCount:)`。如果你好奇，向下滚动到 `updateUI(photosCount:)` 并进入代码。

构建并运行项目，你会注意到预览下方的两个按钮被禁用，这是正确的初始状态：

![image-20221006174323573](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006174323573.png)

当你向当前拼贴添加更多照片时，这些按钮将不断改变状态。例如，当你选择一张或三张照片时，保存按钮将被禁用，但清除按钮将被启用，如下所示：

![image-20221006174338658](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006174338658.png)

#### 选择相册照片

你看到了通过 subject 传递 UI 数据并将其绑定到屏幕上的某些控件是多么容易。接下来，你将处理另一项常见任务：呈现一个新视图并在用户使用完毕后取回一些数据。

绑定数据的总体思路保持不变。你只需要更多的发布者或 subject 来定义正确的数据流。

打开 `PhotosView`，你会看到它已经包含了从相册加载照片并将它们显示在 collection view 中的代码。

你的下一个任务是将必要的 Combine 代码添加到你的 model 中，以允许用户选择一些相册照片并将它们添加到他们的 collage 中。

在 `CollageNeueModel.swift` 中添加以下 subject：

```
private(set) var selectedPhotosSubject = PassthroughSubject<UIImage, Never>()
```

此代码允许 `CollageNeueModel` 在 subject 完成后将 subject 替换为新 subject，但其他类型只能访问发送或订阅接收事件。让我们将集合视图委托方法连接到该 subject。

向下滚动到 `selectImage(asset:)`。已经提供的代码从设备库中获取给定的照片 asset。照片准备好后，你应该使用 subject 将图像发送给任何订阅者。

将 // Send the selected image 注释替换为：

```
self.selectedPhotosSubject.send(image)
```

嗯，这很容易！但是，由于你将 subject 暴露给其他类型，因此你希望显式发送完成事件，以防视图被关闭以拆除任何外部订阅。

同样，你可以通过几种不同的方式实现这一点，但对于本章，打开 `PhotosView.swift` 并找到 `.onDisappear(...)` 修饰符。

在 `.onDisappear(...)` 中添加：

```
model.selectedPhotosSubject.send(completion: .finished)
```

当你从呈现的视图返回时，此代码将发送一个完成的事件。要结束当前任务，你仍然需要订阅所选照片并将其显示在主视图中。

打开 `CollageNeueModel.swift`，找到 `add()`，并将其正文替换为：

```
let newPhotos = selectedPhotosSubjectnewPhotos  .map { [unowned self] newImage in  // 1    return self.images.value + [newImage]  }  // 2  .assign(to: \.value, on: images)  // 3  .store(in: &subscriptions)
```

在上面的代码中，你：

1. 获取所选图像的当前列表，并将任何新图像附加到其中。
2. 使用 assign 通过图像主题发送更新的图像数组。
3. 你将新 subscription 存储在 subscriptions 中。但是，只要用户关闭呈现的视图控制器，订阅就会结束。

准备好测试新绑定后，最后一步是抬起显示照片选择器视图的标志。

打开 `MainView.swift` 并找到调用 `model.add()` 的 + 按钮操作闭包。在该闭包中再添加一行：

```
isDisplayingPhotoPicker = true
```

`isDisplayingPhotoPicker` 状态属性在设置为 `true` 时，显示 PhotosView，你已准备好进行测试！

运行应用程序并尝试新添加的代码。点击 + 按钮，你将在屏幕上看到系统照片访问对话框。由于这是你自己的应用程序，因此可以安全地点击允许访问所有照片以允许从 Collage Neue 应用程序访问你模拟器上的完整照片库：![image-20221006202742290](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006202742290.png)

这将使用 iOS 模拟器中包含的默认照片或你自己的照片（如果你在设备上进行测试）重新加载集合视图：

![image-20221006202756293](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006202756293.png)

点击其中的几个。 它们会闪烁以表明它们已被添加到拼贴画中。 然后，点击返回主屏幕，你将在其中看到你的新拼贴画：

![image-20221006202838537](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006202838537.png)

在继续之前，有一个未解决的问题需要处理。 如果你在照片选择器和主视图之间导航几次，你会注意到在第一次之后你无法再添加任何照片。

为什么会这样？

问题源于你每次展示照片选择器时如何重用 `selectedPhotosSubject`。 第一次关闭该视图时，你发送完成的完成事件并且 subject 完成。

你仍然可以使用它来创建新订阅，但这些订阅会在你创建后立即完成。

要解决此问题，请在每次展示照片选择器时创建一个新主题。 滚动到 add() 并插入到它的顶部：

```
selectedPhotosSubject = PassthroughSubject<UIImage, Never>()
```

这将在你每次展示照片选择器时创建一个新 subject。 你现在应该可以自由地在视图之间来回导航，同时仍然可以向拼贴添加更多照片。

#### 将回调函数包装为 future

在 Playground 中，你可能会与 subject 和发布者一起玩，并且能够完全按照自己的喜好设计一切，但在实际应用程序中，你将与各种 Cocoa API 进行交互，例如访问相机胶卷、读取设备的传感器或与一些数据库。

在本书后面，你将学习如何创建自己的自定义发布者。但是，在许多情况下，只需将 subject 添加到现有的 Cocoa 类就足以将其功能插入到你的组合工作流程中。

在本章的这一部分中，你将使用一种称为 `PhotoWriter` 的新自定义类型，它允许你将用户的 collage 保存到磁盘。你将使用基于回调的 Photos API 进行保存，并使用 Combine Future 来允许其他类型订阅操作结果。

打开 `Utility/PhotoWriter.swift`，其中包含一个空的 `PhotoWriter` 类，并在其中添加以下静态函数：

```
static func save(_ image: UIImage) -> Future<String, PhotoWriter.Error> {  Future { resolve in  }}
```

此函数将尝试将给定图像异步存储在磁盘上，并返回订阅的 Future 给此 API 的使用者。

你将使用基于闭包的 Future 初始化程序返回一个随时可用的 Future，一旦初始化，它将执行提供的闭包中的代码。

让我们通过在闭包中插入以下代码来充实 Future 的逻辑：

```
do {} catch {  resolve(.failure(.generic(error)))}
```

这是一个很好的开始。 你将在 do 块内执行保存，如果它抛出错误，你将通过失败 resolve future。

由于你不知道保存照片时可能引发的确切错误，因此你只需获取引发的错误并将其包装为 `PhotoWriter.Error.generic` 错误。

现在，对于函数的真正内容：在 do 主体中插入以下内容：

```
try PHPhotoLibrary.shared().performChangesAndWait {  // 1  let request = PHAssetChangeRequest.creationRequestForAsset(from: image)  // 2  guard let savedAssetID =     request.placeholderForCreatedAsset?.localIdentifier else {    // 3    return resolve(.failure(.couldNotSavePhoto))  }  // 4  resolve(.success(savedAssetID))}
```

在这里，你使用 `PHPhotoLibrary.performChangesAndWait(_)` 同步访问照片库。 future 的闭包本身是异步执行的，所以不用担心阻塞主线程。 有了这个，你将在闭包中执行以下更改：

1. 首先，你创建一个存储图像的请求。
2. 然后，你尝试通过 `request.placeholderForCreatedAsset?.localIdentifier` 获取新创建资产的标识符。
3. 如果创建失败并且你没有返回标识符，则使用 `PhotoWriter.Error.couldNotSavePhoto` 错误 resolve future。
4. 最后，如果你取回了一个已保存的 `AssetID`，你就成功地 resolve future。

这就是包装回调函数所需的一切，如果你返回错误则以失败解决，或者如果你有一些结果要返回，则以成功解决！

现在，你可以在用户点击保存时使用 `PhotoWriter.save(_:)` 保存当前拼贴。 打开 `CollageNeueModel.swift` 并在 `save()` 中追加：

```
guard let image = imagePreview else { return }// 1PhotoWriter.save(image)  .sink(    receiveCompletion: { [unowned self] completion in      // 2      if case .failure(let error) = completion {        lastErrorMessage = error.localizedDescription      }      clear()    },    receiveValue: { [unowned self] id in      // 3      lastSavedPhotoID = id    }  )  .store(in: &subscriptions)
```

在此代码中，你：

1. 使用 `sink(receiveCompletion:receiveValue:)` 订阅 `PhotoWriter.save(_:)` 的 future。
2. 如果完成失败，将错误消息保存到 `lastErrorMessage`。
3. 如果你得到一个值——新的资产标识符——你将它存储在 `lastSavedPhotoID` 中。

`lastErrorMessage` 和 `lastSavedPhotoID` 已经连接到 SwiftUI 代码中，以向用户展示各自的消息。

再次运行该应用程序，选择几张照片，然后点击保存。这将调用你闪亮的新发布者，并在保存拼贴画后，将显示如下提示：

#### 关于内存管理的说明

这是一个关于使用 Combine 进行内存管理的快速旁注的好地方。如前所述，Combine 代码必须处理大量异步执行的工作，并且在处理类时管理起来总是有点麻烦。

当你编写自己的自定义组合代码时，你可能主要处理结构，因此你不需要在用于 map、flatMap、filter 等的闭包中显式指定捕获语义。

但是，当你使用 UIKit/AppKit 代码处理 UI 代码时（即你有 UIViewController、UICollectionController 等的子类）或者当你的 SwiftUI 视图有 ObservableObjects 时，你需要注意所有的内存管理。

在编写 Combine 代码时，标准规则适用，因此你应该一如既往地使用相同的 Swift 捕获语义：

1. 如果你正在捕获一个可以从内存中释放的对象，例如前面展示的照片视图控制器，你应该使用 `[weak self]` 或其他变量而不是 self 如果你捕获另一个对象。
2. 如果你正在捕获一个无法释放的对象，例如该 Collage 应用程序中的主视图控制器，你可以安全地使用 `[unowned self]`。例如，你永远不会从导航堆栈中弹出，因此始终存在。

#### 共享 subscription

回顾 `CollageNeueModel.add()` 中的代码，你可以对用户在 PhotosView 中选择的图像做更多的事情。

这提出了一个令人不安的问题：你应该多次订阅同一个 selectedPhotos 发布者，还是做其他事情？

事实证明，订阅同一个发布者可能会产生不必要的副作用。如果你考虑一下，你不知道发布者在订阅时在做什么，它可能正在创建新资源、发出网络请求或其他意外工作。

![image-20221006204958814](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006204958814.png)

为同一个发布者创建多个订阅时，正确的方法是使用 `share()` 操作符共享原始发布者。这将发布者包装在一个类中，因此它可以安全地发送给多个订阅者，而无需再次执行其底层工作。

仍然在 `CollageNeueModel.swift` 中，找到 `let newPhotos = selectedPhotos` 行并将其替换为：

```
let newPhotos = selectedPhotos.share()
```

现在，为 newPhotos 创建多个订阅是安全的，而不必担心发布者会为每个新订阅者多次执行副作用：

![image-20221006205155933](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006205155933.png)

需要记住的一点是 share() 不会从共享订阅中重新发出任何值，因此你只会获得订阅后出现的值。

例如，如果你在 share() 发布者上有两个订阅，并且源发布者在订阅时同步发出，则只有第一个订阅者会获得该值，因为第二个订阅者在实际发出该值时没有订阅。

#### 练习 Operator

现在你已经了解了一些有用的响应式模式，是时候练习你在前几章中介绍的一些操作符并查看它们的实际效果了。

打开`CollageNeueModel.swift` 并将 `selectedPhotos` 订阅的行替换为 `let newPhotos = selectedPhotos.share()` ：

```
let newPhotos = selectedPhotos  .prefix(while: { [unowned self] _ in    self.images.value.count < 6  })  .share()
```

你已经了解了 `prefix(while:)` 作为强大的组合过滤操作符之一，在这里你可以在实践中使用它。 只要所选图像的总数少于六个，上面的代码就会保持对 selectedPhotos 的订阅。 这将有效地允许用户为他们的拼贴选择多达六张照片。

在调用 `share()` 之前添加 `prefix(while:)` 允许你过滤传入的值，不仅在一个订阅上，而且在订阅 `newPhotos` 的所有订阅上。

运行应用程序并尝试添加超过六张照片。 你会看到在前六个之后主视图控制器不再接受更多。

同样，你可以通过组合你已经知道和喜爱的所有操作符（如 filter、dropFirst、map 等）来实现你需要的任何逻辑。

#### 挑战

恭喜你完成了本教程式的章节！如果你想在下一章继续学习更多理论之前完成一个可选任务，请继续阅读下文。

打开 Utility/PHPhotoLibrary+Combine.swift 并阅读从用户那里获取 Collage Neue 应用程序的照片库授权的代码。你肯定会注意到逻辑非常简单，并且基于“标准”回调 API。

这为你提供了一个很好的机会来将 Cocoa API 包装为你自己的未来。对于这个挑战，向 `PHPhotoLibrary` 添加一个名为 `isAuthorized` 的新静态属性，其类型为 `Future<Bool, Never>`，并允许其他类型订阅照片库授权状态。

在本章中你已经做过几次了，现有的 `fetchAuthorizationStatus(callback:)` 函数应该很容易使用。祝你好运！如果你在此过程中遇到任何困难，请不要忘记你可以随时进入本章提供的挑战文件夹并查看示例解决方案。

最后，别忘了在 PhotosView 中使用新的 isAuthorized 发布者！

显示错误消息以防用户在点击关闭时未授予对其照片的访问权限并导航回主视图控制器。

![image-20221006210235997](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006210235997.png)

要使用不同的授权状态并测试你的代码，请在你的模拟器或设备上打开设置应用程序并导航到隐私/照片。

将 Collage 的授权状态更改为“无”或“所有照片”以测试你的代码在这些状态下的行为：

![image-20221006210256384](https://raw.githubusercontent.com/LLLLLayer/picture-bed/main/img/Kodeco/image-20221006210256384.png)

如果到目前为止，你是靠自己成功完成挑战的，那么你真的应该得到掌声！ 无论哪种方式，本章的挑战文件夹中都提供了一种你可以随时参考的可能解决方案。

#### 关键点

* 在你的日常任务中，你很可能必须处理基于回调或委托的 API。 幸运的是，这些很容易通过使用 subject 包装为 future 或 publisher。
* 从委托和回调等各种模式转变为单一的发布者/订阅者模式，使得呈现视图和取回值等日常任务变得轻而易举。
* 为避免在多次订阅发布者时产生不必要的副作用，请通过 share() 操作符使用共享发布者。

#### 接下来去哪？

从下一章开始，你将开始更多地研究 Combine 与现有 Foundation 和 UIKit/AppKit API 集成的方式。
