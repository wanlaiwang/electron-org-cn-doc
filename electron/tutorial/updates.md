# 更新程序体

这里有很多种方法更新一个Electron程序。最简单的官方支持的方式，就是利用内置的 [Squirrel](https://github.com/Squirrel) 框架和Electron的[自动更新](../api/auto-updater.md)模块。

## 部署一台更新服务器

作为一个好的开端，您首先需要部署一台服务器，用于[自动更新](../api/auto-updater.md)模块将要下载模块的有力支持。

根据实际需求，您可能有如下选择：

- [Hazel](https://github.com/zeit/hazel) ，一款简单的开源的APP，您可以在这里拉取到对应源码[GitHub Releases](https://help.github.com/articles/creating-releases/) 
。现在还可以在这里 [免费部署](https://zeit.co/now)。
- [Nuts](https://github.com/GitbookIO/nuts) ，您也可以从
[GitHub Releases](https://help.github.com/articles/creating-releases/)里面找到对应源码，主要特色是：可以把APP的更新缓存到硬盘上，同时支持私人仓库。
- [electron-release-server](https://github.com/ArekSredzki/electron-release-server) ，提供一个处理发型版本的面板。

如果您的app是用 [electron-builder][electron-builder-lib] 打包的话，那么您可以使用[electron-updater] 模块，它不需要提供更新专用服务器，支持从 S3 、Github 或者其它静态文件服务器获得更新。

## 在您的APP之中实现更新

一旦您部署了您的更新服务器，接下来要做的就是，在您的 APP 中引入必要的模块了。接下来的代码，对于不同的服务器软件，可能会有所不同。但是工作原理，都是和使用使用[Hazel](https://github.com/zeit/hazel)是一样的。

**重要提示:** 请确保下面的程序是运行在您的打包app上面的，而不是开发环境。您可以使用[electron-is-dev](https://github.com/sindresorhus/electron-is-dev) 用于检测是否是开发环境。

```js
const {app, autoUpdater, dialog} = require('electron')
```

接下来，我们需要组装一个服务器更新的链接URL，然后把这个信息通知给[自动更新](../api/auto-updater.md) 模块:

```js
const server = 'https://your-deployment-url.com'
const feed = `${server}/update/${process.platform}/${app.getVersion()}`

autoUpdater.setFeedURL(feed)
```

作为最后一部，请检查更新。下面的范例代码，将会每分钟就执行一次，是否更新检查。

```js
setInterval(() => {
  autoUpdater.checkForUpdates()
}, 60000)
```

一旦您的程序体被 [封装](../tutorial/application-distribution.md) 好之后, 它将会收到您发布的每条 [Github发行版](https://help.github.com/articles/creating-releases/) 更新通知。

## 应用更新

现在，您已经为您的程序配置好了基本的更新机制，当有更新发布的时候，您需要确保每个用户都会收到提示信息。这一点将使用 自动更新API [events](../api/auto-updater.md#events) 来达成目的：

```js
autoUpdater.on('update-downloaded', (event, releaseNotes, releaseName) => {
  const dialogOpts = {
    type: 'info',
    buttons: ['Restart', 'Later'],
    title: 'Application Update',
    message: process.platform === 'win32' ? releaseNotes : releaseName,
    detail: 'A new version has been downloaded. Restart the application to apply the updates.'
  }

  dialog.showMessageBox(dialogOpts, (response) => {
    if (response === 0) autoUpdater.quitAndInstall()
  })
})
```

同时需要确保错误信息[被处理](../api/auto-updater.md#event-error)。这里有个范例用于把这些错误信息记录到 `标准错误输出（stderr）`之中：

```js
autoUpdater.on('error', message => {
  console.error('There was a problem updating the application')
  console.error(message)
})
```

[electron-builder-lib]: https://github.com/electron-userland/electron-builder
[electron-updater]: https://www.electron.build/auto-update