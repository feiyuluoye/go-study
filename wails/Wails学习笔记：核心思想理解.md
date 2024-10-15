### Wails 学习笔记：Wails核心思想理解



[TOC]

Wails 应用程序是一个带有一个 webkit 前端的标准的 Go 应用程序。 应用程序的 Go 部分由应用程序代码和一个运行时库组成， 该库提供了许多有用的操作，例如控制应用程序窗口。 前端是一个 webkit 窗口，将显示前端资源。 前端还可以使用运行时库的 JavaScript 版本。 最后，可以将 Go 方法绑定到前端，这些将显示为可以调用的 JavaScript 方法，就像它们是原生 JavaScript 方法一样。

---

### 1. Wails 的核心思想

Wails 的核心目标是允许开发者使用现代前端技术开发桌面应用，同时利用 Go 的强大后端性能。它将前端 UI（如 HTML、CSS、JavaScript）嵌入到一个原生的桌面应用窗口中，并通过 Go 来处理后端逻辑和系统调用。

Wails 提供了前端和后端的桥梁，使得前端代码能够调用 Go 后端的功能，反之亦然。这使得开发者可以用现代的前端框架来创建高效、跨平台的桌面应用程序。

---

### 2. 工作流程



![img](https://wails.io/zh-Hans/assets/images/architecture-23c8df42202276ecee3e5cb7a0c6c51a.webp)

Wails 的工作流程包括以下几个步骤：

#### 2.1 前端渲染
Wails 使用现代前端框架（如 Vue、React、Svelte 等）来构建用户界面。前端资源（HTML、CSS、JavaScript）会被打包并嵌入到 Go 的应用程序中。Wails 会创建一个窗口，用来显示这些前端内容。

- **前端框架**：Wails 支持主流的前端框架如 Vue、React 等，开发者可以通过这些框架构建响应式的用户界面。
- **WebView**：前端的界面通过操作系统的原生 WebView 显示，这意味着应用程序的 UI 实际上是在浏览器引擎中渲染的。

#### 2.2 后端逻辑
后端由 Go 编写，负责处理业务逻辑和与系统的交互。开发者可以使用 Go 实现一切与桌面环境相关的功能，如文件系统访问、网络请求、数据处理等。

- **Go 后端**：Wails 的后端是基于 Go 的。Go 提供了强大的并发能力和系统级 API 访问权限，适合处理文件、数据库、网络等复杂的逻辑。

#### 2.3 前后端通信
Wails 提供了一个桥梁，让前端和后端能够轻松通信。前端可以调用 Go 后端的函数，而 Go 后端也可以通知前端进行 UI 更新。这种通信机制通过以下方式实现：
- **Wails Bindings**：后端的 Go 方法可以通过 `wails.Bind()` 暴露给前端。前端可以通过 JavaScript 调用这些绑定的 Go 方法，并获取其返回结果。
- **事件系统**：Wails 内置了一个事件系统，允许前后端以发布/订阅的方式相互通信。例如，当后端处理完某个任务后，可以发布一个事件通知前端进行更新。

#### 2.4 应用打包与分发
Wails 可以将前端资源和后端逻辑一起打包为一个可执行文件，适用于 Windows、macOS 和 Linux。由于它不依赖像 Electron 那样的重型运行时，所以生成的应用程序体积更小，性能更高。

---

### 3. Wails 主要组件

Wails 的整体架构可以分为以下几个核心组件：

#### 3.1 WebView
Wails 使用操作系统的原生 WebView 来展示前端内容：
- **Windows**：使用 Edge (Chromium) WebView2。
- **macOS**：使用 WebKit（Safari）。
- **Linux**：默认使用 WebKitGTK。

与 Electron 不同，Wails 并不嵌入整个 Chromium 引擎，而是使用操作系统提供的轻量级 WebView 引擎。这大大减少了应用的体积和资源消耗。

#### 3.2 事件与数据绑定
Wails 允许 Go 后端与前端 JavaScript 之间的互操作。开发者可以在前端调用后端的 Go 方法，后端则可以通过事件来通知前端更新视图。

- **前端调用后端**：通过 JavaScript，前端可以直接调用绑定的 Go 函数，并获取返回值。
- **后端调用前端**：Go 后端可以向前端发送事件（如通知或数据更新），让前端做出相应的处理。

#### 3.3 窗口管理
Wails 使用原生窗口系统来管理应用窗口。你可以自定义窗口的标题、大小、是否支持无边框等。与传统的 Web 应用不同，Wails 提供了与操作系统的更深层次集成，如访问系统文件、托盘、通知等。

---

### 4. Wails 的优点

- **轻量级**：与 Electron 相比，Wails 应用程序的体积要小得多，因为它依赖于操作系统的 WebView 而不是内嵌整个浏览器引擎。
- **高效的前后端交互**：通过 Go 后端与前端的紧密集成，Wails 提供了一个快速、高效的通信机制，使得开发桌面应用既高效又简洁。
- **跨平台**：Wails 支持 Windows、macOS 和 Linux，因此同一个代码库可以轻松编译为不同平台的应用程序。
- **现代前端支持**：Wails 允许你使用现代前端框架（如 Vue、React、Svelte）来构建桌面应用的 UI。

---

### 5. Wails 的使用场景

- **轻量级桌面应用**：与 Electron 相比，Wails 更适合那些不需要繁重渲染引擎的桌面应用，尤其是对于资源敏感的应用程序。
- **系统工具与自动化工具**：利用 Go 的系统访问能力，Wails 非常适合开发与文件系统、数据库或网络交互的桌面工具。
- **跨平台 GUI 应用**：Wails 的跨平台支持使它成为开发简单跨平台 GUI 应用的绝佳选择。

---

### 6. 启动函数Run

Wails 提供了一种现代化、轻量级的桌面应用开发方式，它将 Go 的高效后端与现代前端技术相结合。通过操作系统原生 WebView，它能够显著减小应用体积，并提供优秀的性能表现。对于需要开发跨平台桌面应用、并且对性能和体积敏感的开发者来说，Wails 是一个理想的选择。

`wails.Run()` 是 Wails 框架中的一个关键函数，它用于启动整个 Wails 应用。该函数会初始化应用的前端和后端部分，并且根据开发者在 `wails.Options` 中的配置，创建应用窗口、加载前端资源、绑定后端逻辑，最终呈现一个完整的桌面应用。

#### `wails.Run()` 的主要功能

1. **初始化 Wails 应用**：
   `wails.Run()` 负责初始化整个应用的各个部分，包括应用窗口、前端 WebView、前后端通信机制以及相关的事件系统。它是 Wails 应用的入口点，启动 Wails 框架的运行。

2. **应用窗口管理**：
   根据 `wails.Options` 提供的配置参数，`wails.Run()` 会创建并显示应用窗口。Wails 支持窗口的大小、标题、是否无边框等配置，所有这些窗口设置都通过 `wails.Options` 传递给 `wails.Run()`。

3. **加载前端资源**：
   Wails 会将前端资源（如 HTML、CSS、JavaScript）打包在应用中，`wails.Run()` 会加载这些前端资源并通过 WebView 显示在窗口中。前端代码通常由现代前端框架（如 Vue、React、Svelte）构建。

4. **绑定前后端通信**：
   `wails.Run()` 负责建立前端 JavaScript 与后端 Go 代码的通信桥梁。Go 代码可以通过 `wails.Bind()` 将函数暴露给前端调用。通过 Wails 的事件机制，前后端之间可以轻松交换数据和通知。

5. **启动生命周期管理**：
   在 Wails 应用启动和运行的过程中，`wails.Run()` 会触发不同的生命周期事件，例如：
   - `OnStartup`：在应用启动时执行，通常用于初始化应用。
   - `OnShutdown`：在应用关闭时执行，用于清理资源。
   - `OnDomReady`：当前端的 DOM 完全加载并准备好与后端交互时触发。

#### `wails.Run()` 的参数：`wails.Options`

`wails.Run()` 接受一个 `wails.Options` 结构体，开发者通过这个结构体定义应用的行为和配置。`wails.Options` 中的常见选项包括：

```go
err := wails.Run(&wails.Options{
    Title:            "My Wails App",    // 应用程序的标题
    Width:            1024,              // 窗口的宽度
    Height:           768,               // 窗口的高度
    MinWidth:         400,               // 窗口的最小宽度
    MinHeight:        300,               // 窗口的最小高度
    WindowStartState: options.Normal,    // 窗口的初始状态 (如最大化、最小化)
    HTML:             "frontend/dist/index.html", // 前端 HTML 文件
    Bind: []interface{}{app},            // 绑定到前端的 Go 对象或函数
    OnStartup:        app.startup,       // 启动时调用的函数
    OnDomReady:       app.domReady,      // DOM 加载完成后调用的函数
    OnShutdown:       app.shutdown,      // 应用关闭时调用的函数
})
```

#### 常用选项说明：

- **Title**：设置窗口的标题。
- **Width** 和 **Height**：窗口的初始宽高。
- **MinWidth** 和 **MinHeight**：窗口的最小尺寸。
- **WindowStartState**：窗口的启动状态，比如默认、最大化、最小化。
- **HTML**：前端的入口 HTML 文件路径，通常是前端打包后的静态文件（如 Vue、React 的 `index.html`）。
- **Bind**：绑定的 Go 对象或函数，它们可以在前端 JavaScript 中调用。
- **OnStartup**：启动时执行的回调，用于应用初始化。
- **OnDomReady**：前端 DOM 完全加载完成时触发的回调。
- **OnShutdown**：应用关闭时执行的回调，用于资源清理。

#### `wails.Run()` 的执行流程

1. **解析 `wails.Options`**：Wails 首先会读取 `wails.Options`，根据开发者的设置来配置应用窗口、加载的前端资源和绑定的后端逻辑。
2. **创建窗口和 WebView**：接着，Wails 会根据操作系统创建一个原生窗口（如 Windows 的 WebView2 或 macOS 的 WebKit），并在其中加载前端的 HTML 页面。
3. **前后端绑定**：`wails.Run()` 会通过 `wails.Bind()` 将 Go 后端函数暴露给前端 JavaScript，使前端可以调用这些函数。它还会建立前后端的事件机制，确保前后端可以进行数据通信。
4. **启动应用**：最后，`wails.Run()` 开始监听用户事件，运行主循环，保持应用的正常运行，直到用户关闭窗口。

#### 总结

`wails.Run()` 是 Wails 框架的核心启动函数，负责初始化应用、加载前端资源、配置窗口、并管理前后端通信。通过 `wails.Options`，开发者可以灵活配置应用的行为，如窗口属性、前端入口文件、绑定的后端函数等。

`wails.Run()` 不仅启动了整个应用程序，还提供了一个平台，使得现代前端技术和高效的 Go 后端能够无缝集成，从而实现高性能的桌面应用开发。