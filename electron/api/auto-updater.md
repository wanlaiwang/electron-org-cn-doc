# autoUpdater（自动更新）

> 使程序具有自动更新的能力。

进程: [Main](../glossary.md#main-process)

`autoUpdater`模块提供一个基于[Squirrel](https://github.com/Squirrel)框架的接口。
你可以使用以下任一项目，快速启动多平台版本发布服务器，来分发你的应用程序：

*   [nuts](https://github.com/GitbookIO/nuts)：您的应用程序的智能版本服务器，使用 GitHub 作为后端。使用 [Squirrel](https://github.com/Squirrel)（Mac和Windows）自动更新。
*   [electron-release-server](https://github.com/ArekSredzki/electron-release-server)：一个功能齐全的自主发布的 electron app 服务器，与自动更新功能兼容。
*   [squirrel-updates-server](https://github.com/Aluxian/squirrel-updates-server)：用于 Squirrel.Mac 和 Squirrel.Windows 的简单 node.js 服务器，它使用 GitHub 发布版本。
*   [squirrel-release-server](https://github.com/Arcath/squirrel-release-server)：一个用于 Squirrel.Windows 的简单PHP应用程序。它从文件夹读取更新，支持增量更新。

[你还可以在这里找到一份详细教程](../tutorial/updates.md)，讲述如何在你的程序里面实现更新功能。

## 平台提示
目前，只有macOS和windows这2个平台支持自动更新。在Linux系统上，目前没有内置的自动更新支持，所以，进一步来说，在不同的平台上，这里有一些微妙的区别：

### macOS
在MacOS系统上，`autoUpdater`模块是建立在[Squirrel.Mac](https://github.com/Squirrel/Squirrel.Mac)基础之上的，这就意味着使之生效的话，你并不需要做什么额外的工作。对于服务器端的要求，你可以查看文章`"[服务器支持](https://github.com/Squirrel/Squirrel.Mac#server-support)"`。注意，应用程序传输安全性（ATS）作为自动更新的一部分，适用于全部自动更新的请求。需要禁用ATS的应用程序，可以把这个属性 `NSAllowsArbitraryLoads` 添加到程序的plist之中。

注意：在MacOS系统中，你的程序必须签名。这个是[Squirrel.Mac](https://github.com/Squirrel/Squirrel.Mac)的要求。

### Windows
在winwos系统中，在使用自动更新之前，您必须先保证将你的的应用已经安装到了用户的计算机上，所以比较推荐的方法是用 [electron-winstaller][installer-lib], [electron-forge][electron-builder-lib] 或者 [grunt-electron-installer][installer] 模块，来自动生成一个 Windows 安装包。

当使用 [electron-winstaller][installer-lib] 或者 [electron-forge][electron-builder-lib] 的时候，要确保在[首次运行](https://github.com/electron/windows-installer#handling-squirrel-events)的时候，您不会尝试更新应用程序 （另见 [this issue for more info](https://github.com/electron/electron/issues/7155)）。 我们推荐您使用 [electron-squirrel-startup](https://github.com/mongodb-js/electron-squirrel-startup) 获取应用程序的桌面快捷方式。

Squirrel 自动生成的安装向导，会生成一个带 [Application User Model ID][app-user-model-id] 的快捷方式。Application User Model ID 的格式是 `com.squirrel.PACKAGE_ID.YOUR_EXE_WITHOUT_DOT_EXE`, 比如
像 `com.squirrel.slack.Slack` 和 `com.squirrel.code.Code` 这样的。你应该在自己的应用中，使用 `app.setAppUserModelId` API设置相同的ID，否则， Windows 将不能正确地把你的应用固定在任务栏上。

并不像Squirrel.Mac，windows版本的Squirrel 可以在S3或者其他静态文件主机中，提供更新服务。
您可以阅读 [Squirrel.Windows][squirrel-windows] 这个文档来获得详细信息。

### Linux

Linux 下没有任何的自动更新支持，所以我们推荐用各个 Linux 发行版的包管理器来分发你的应用。

## 事件列表

`autoUpdater` 对象会触发如下事件：

### 事件：'error'

返回：

* `error` Error

当更新发生错误的时候触发。

### 事件：'checking-for-update'

当开始检查更新的时候触发。

### 事件：'update-available'

当发现一个可用更新的时候触发，更新包下载会自动开始。

### 事件：'update-not-available'

当没有可用更新的时候触发。

### 事件：'update-downloaded'

返回：

* `event` Event
* `releaseNotes` String - 新版本更新公告
* `releaseName` String - 新的版本号
* `releaseDate` Date - 新版本发布的日期
* `updateURL` String - 更新地址

在更新下载完成的时候触发。

在 Windows 上只有 `releaseName` 是有效的。

## 方法列表

`autoUpdater` 对象有以下的方法：

### `autoUpdater.setFeedURL(url[, requestHeaders])`

* `url` String
* `requestHeaders` Object _macOS_ (可选) - HTTP请求头。

设置检查更新的 `url`，并且初始化自动更新模块。

### `autoUpdater.getFeedURL()`

返回 `String` - 当前更新提要的 URL。

### `autoUpdater.checkForUpdates()`

向服务端查询：现在是否有可用的更新。在调用这个方法之前，必须要先调用 `setFeedURL`。

### `autoUpdater.quitAndInstall()`

在下载完成后，重启当前的应用并且安装更新。这个方法应该仅在 `update-downloaded` 事件触发后被调用。

**注意：** `autoUpdater.quitAndInstall()` 将先关闭所有应用程序窗口，并将结束调用后，在 `app` 上触发 `before-quit` 事件。这不同于正常退出的事件序列。

[squirrel-mac]: https://github.com/Squirrel/Squirrel.Mac
[server-support]: https://github.com/Squirrel/Squirrel.Mac#server-support
[squirrel-windows]: https://github.com/Squirrel/Squirrel.Windows
[installer]: https://github.com/electron/grunt-electron-installer
[installer-lib]: https://github.com/electron/windows-installer
[electron-builder-lib]: https://github.com/electron-userland/electron-forge
[app-user-model-id]: https://msdn.microsoft.com/en-us/library/windows/desktop/dd378459(v=vs.85).aspx
[electron-release-server]: https://github.com/ArekSredzki/electron-release-server
[squirrel-updates-server]: https://github.com/Aluxian/squirrel-updates-server
[nuts]: https://github.com/GitbookIO/nuts
[squirrel-release-server]: https://github.com/Arcath/squirrel-release-server
