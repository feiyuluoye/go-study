## Wails 学习笔记：创建第一个项目

[TOC]

Wails 是一个用于构建跨平台桌面应用程序的框架，允许开发者使用前端技术（如 HTML、CSS、JavaScript）以及 Go 语言来开发桌面应用。本文基于官方文档《[Wails 入门指南 - 创建第一个项目](https://wails.io/zh-Hans/docs/gettingstarted/firstproject)》总结 Wails 的项目创建过程。

### 1. 安装 Wails

在开始使用 Wails 之前，你需要在系统中安装它。按照文档的说明，使用以下命令安装 Wails：

```bash
go install github.com/wailsapp/wails/v2/cmd/wails@latest
```

确保你已经安装了 Go 语言，并将其添加到环境变量中。安装完成后，可以通过运行以下命令验证 Wails 是否成功安装：

```bash
wails doctor
```

```shell
git:(feat-wails) ✗ wails docker
Wails CLI v2.9.2

Available commands:

   build      Builds the application 
   dev        Runs the application in development mode 
   doctor     Diagnose your environment 
   init       Initialises a new Wails project 
   update     Update the Wails CLI 
   show       Shows various information 
   generate   Code Generation Tools 
   version    The Wails CLI version 

Flags:

  -help
        Get help on the 'wails' command.
```

此命令将检查系统的依赖环境，并确保一切正常运行。

### 2. 创建 Wails 项目

创建项目的过程相当简单，只需运行以下命令即可：(PS: 最新版本需要-n参数指定项目名)

```bash
wails init -n wails-demo
```

这会启动一个交互式的 CLI 提示，你可以选择项目的类型。通常有两种模板供选择：

1. **Vue 模板**：适合那些喜欢使用 Vue.js 构建前端的开发者。
2. **React 模板**：适合那些习惯于使用 React 的开发者。

选择其中一种模板并输入项目名称后，Wails 会自动生成一个包含前端和后端代码的基本项目结构。

| 框架    | 项目                               | 指定TypeScript                        |
| ------- | ---------------------------------- | ------------------------------------- |
| svelte  | wails init -n myproject -t svelte  | wails init -n myproject -t svelte-ts  |
| react   | wails init -n myproject -t react   | wails init -n myproject -t react-ts   |
| vue     | wails init -n myproject -t vue     | wails init -n myproject -t vue-ts     |
| preact  | wails init -n myproject -t preact  | wails init -n myproject -t preact-ts  |
| lit     | wails init -n myproject -t lit     | wails init -n myproject -t lit-ts     |
| vanilla | wails init -n myproject -t vanilla | wails init -n myproject -t vanilla-ts |

### 3. 项目结构

```makefile
.
├── build/
│   ├── appicon.png
│   ├── darwin/
│   └── windows/
├── frontend/
├── go.mod
├── go.sum
├── main.go
└── wails.json
```

项目生成后，主要目录结构如下：

- **frontend/**：前端代码，通常使用你选择的框架（Vue/React）来编写。
- **go.mod**：Go 项目的模块定义文件。
- **main.go**：主要的 Go 语言入口文件，负责管理应用的后端逻辑。
- **wails.json**：Wails 项目的配置文件。

这个结构清晰地将前端和后端分离，使得开发者能够分别处理界面和逻辑部分。

`frontend` 目录没有特定于 Wails 的内容，可以是您选择的任何前端项目。

`build` 目录在构建过程中使用。 这些文件可以修改以自定义您的构建。 如果从 build 目录中删除文件，将重新生成默认版本。

`go.mod` 中的默认模块名称是“changeme”。 您应该将其更改为更合适的内容。

### 4. 运行项目

项目初始化完成后，可以通过以下命令运行项目：

```bash
wails dev
```

这个命令会启动开发服务器，并在默认浏览器中打开应用。在开发过程中，任何前端代码的修改都会被自动编译和更新，方便实时预览效果。

该程序执行以下操作：

```makefile
- 构建您的应用程序并运行它
- 将您的 Go 代码绑定到前端，以便可以从 JavaScript 调用它
- 使用 Vite 的强大功能，将监视您的 Go 文件中的修改并在更改时重新构建/重新运行
- 启动一个 网络服务器 通过浏览器为您的应用程序提供服务。 这使您可以使用自己喜欢的浏览器扩展。 你甚至可以从控制台调用你的 Go 代码。
```

| 标志                         | 描述                                                         | 默认                |
| ---------------------------- | ------------------------------------------------------------ | ------------------- |
| -appargs "参数"              | 以 shell 样式传递给应用程序的参数                            |                     |
| -assetdir "./path/to/assets" | 从给定目录提供资产，而不是使用提供的资产 FS                  | `wails.json` 中的值 |
| -browser                     | 在启动时打开浏览器到 `http://localhost:34115`                |                     |
| -compiler "编译器"           | 使用不同的 go 编译器来构建，例如 go1.15beta1                 | go                  |
| -debounce                    | 检测到资产更改后等待重新加载的时间                           | 100 (毫秒)          |
| -devserver "host:port"       | 将 wails 开发服务器绑定到的地址                              | "localhost:34115"   |
| -extensions                  | 触发重新构建的扩展（逗号分隔）                               | go                  |
| -forcebuild                  | 强制构建应用程序                                             |                     |
| -frontenddevserverurl "url"  | 使用 3rd 方开发服务器 url 提供资产，例如：Vite               | ""                  |
| -ldflags "标志"              | 传递给编译器的额外 ldflags                                   |                     |
| -loglevel "日志级别"         | 要使用的日志级别 - Trace, Debug, Info, Warning, Error        | Debug（调试）       |
| -nocolour                    | 关闭彩色命令行输出                                           | false               |
| -noreload                    | 资产更改时禁用自动重新加载                                   |                     |
| -nosyncgomod                 | 不同步 go.mod 中的 Wails 版本                                | false               |
| -race                        | 使用 Go 的竞态检测器构建                                     | false               |
| -reloaddirs                  | 触发重新加载的附加目录（逗号分隔）                           | `wails.json` 中的值 |
| -s                           | 跳过前端构建                                                 | false               |
| -save                        | 将指定的 `assetdir`、 `reloaddirs`、 `wailsjsdir`、 `debounce` 、 `devserver` 和 `frontenddevserverurl` 标志的值保存到 `wails.json` 以成为后续调用的默认值。 |                     |
| -skipbindings                | 跳过 bindings 生成                                           |                     |
| -tags "额外标签"             | 传递给编译器的构建标签（引号和空格分隔）                     |                     |
| -v                           | 详细级别 (0 - silent, 1 - standard, 2 - verbose)             | 1                   |
| -wailsjsdir                  | 生成生成的Wails JS模块的目录                                 | `wails.json` 中的值 |

```shell
示例：

wails dev -assetdir ./frontend/dist -wailsjsdir ./frontend/src -browser

此命令将执行以下操作：

构建应用程序并运行它（更多细节在 这里）
在 ./frontend/src 中生成 Wails JS 模块
监听 ./frontend/dist 中文件的更新并在更改时重新加载
打开浏览器并连接到应用程序
```



### 5. 构建项目

开发完成后，你可以通过以下命令构建桌面应用：

```bash
wails build
```

这个命令会生成适用于目标操作系统的可执行文件。生成的文件可以直接分发和安装，无需用户单独安装任何依赖。

| 标志               | 描述                                                         | 默认                                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -clean             | 清理 `build/bin` 目录                                        |                                                              |
| -compiler "编译器" | 使用不同的 go 编译器来构建，例如 go1.15beta1                 | go                                                           |
| -debug             | Retains debug information in the application and shows the debug console. 允许在应用程序窗口中使用 devtools |                                                              |
| -devtools          | Allows the use of the devtools in the application window in production (when -debug is not used). Ctrl/Cmd+Shift+F12 may be used to open the devtools window. *NOTE*: This option will make your application FAIL Mac appstore guidelines. Use for debugging only. |                                                              |
| -dryrun            | 打印构建命令但不执行它                                       |                                                              |
| -f                 | 强制构建应用                                                 |                                                              |
| -garbleargs        | 传递给 garble 的参数                                         | `-literals -tiny -seed=random`                               |
| -ldflags "标志"    | 传递给编译器的额外 ldflags                                   |                                                              |
| -m                 | 编译前跳过 mod tidy                                          |                                                              |
| -nopackage         | 不打包应用程序                                               |                                                              |
| -nocolour          | 在输出中禁用颜色                                             |                                                              |
| -nosyncgomod       | 不同步 go.mod 中的 Wails 版本                                |                                                              |
| -nsis              | 为 Windows 生成 NSIS 安装程序                                |                                                              |
| -o 文件名          | 输出文件名                                                   |                                                              |
| -obfuscated        | 使用 [garble](https://github.com/burrowers/garble) 混淆应用程序 |                                                              |
| -platform          | 为指定的 [平台](https://wails.io/zh-Hans/docs/reference/cli#平台)（逗号分割）构建，例如： `windows/arm64`。 `windows/arm64`。 注意，如果不给出架构，则使用 `runtime.GOARCH`。 | 如果给定环境变量 platform = `GOOS` 否则等于 `runtime.GOOS`。 如果给定环境变量 arch = `GOARCH` 否则等于 `runtime.GOARCH`. |
| -race              | 使用 Go 的竞态检测器构建                                     |                                                              |
| -s                 | 跳过前端构建                                                 |                                                              |
| -skipbindings      | 跳过 bindings 生成                                           |                                                              |
| -tags "额外标签"   | 构建标签以传递给 Go 编译器。 必须引用。 空格或逗号（但不能同时使用）分隔 |                                                              |
| -trimpath          | 从生成的可执行文件中删除所有文件系统路径。                   |                                                              |
| -u                 | 更新项目的 `go.mod` 以使用与 CLI 相同版本的 Wails            |                                                              |
| -upx               | 使用 “upx” 压缩最终二进制文件                                |                                                              |
| -upxflags          | 传递给 upx 的标志                                            |                                                              |
| -v int             | 详细级别 (0 - silent, 1 - default, 2 - verbose)              | 1                                                            |
| -webview2          | WebView2 安装策略：download,embed,browser,error.             | download                                                     |
| -windowsconsole    | 保留Windows构建控制台窗口                                    |                                                              |

```shell
示例：

wails build -clean -o myproject.exe

信息
在 Mac 上，应用程序将被绑定到 Info.plist，而不是 Info.dev.plist。

苹果芯片上的 UPX
在苹果芯片上使用 UPX 相关的 问题。

Windows 上的 UPX
一些防病毒软件供应商误将 upx 压缩的二进制文件标记为病毒，请查看相关 问题。
```



### 6. 部署和发布

构建完成的应用可以直接发布。根据操作系统的不同，你可以打包成可安装文件（如 Windows 的 `.exe`，MacOS 的 `.dmg` 等）。文档还提供了一些打包工具的建议，如 `electron-builder`，方便对应用进行进一步的包装和签名。

支持的平台有：

| 平台             | 描述                                          |
| ---------------- | --------------------------------------------- |
| darwin           | MacOS + architecture of build machine         |
| darwin/amd64     | MacOS 10.13+ AMD64                            |
| darwin/arm64     | MacOS 11.0+ ARM64                             |
| darwin/universal | MacOS AMD64+ARM64 universal application       |
| windows          | Windows 10/11 + architecture of build machine |
| windows/amd64    | Windows 10/11 AMD64                           |
| windows/arm64    | Windows 10/11 ARM64                           |
| linux            | Linux + architecture of build machine         |
| linux/amd64      | Linux AMD64                                   |
| linux/arm64      | Linux ARM64                                   |

---

### 总结

Wails 通过结合前端技术和 Go 后端，提供了一个高效的方式来开发跨平台的桌面应用。其简单的项目初始化和清晰的结构使得开发者能够快速上手。通过 Wails，可以充分利用 Web 开发技能，同时获得原生桌面应用的优势。