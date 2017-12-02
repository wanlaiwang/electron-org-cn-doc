# desktopCapturer

> 使用`navigator.mediaDevices.getUserMedia`API，可以获取媒体源信息的权限。可用于：从桌面上捕获音频和视频功能。

进程：[渲染进程](../glossary.md#renderer-process)。

下面的例子，将要展示“如何从一个标题为`electron`的桌面窗体里面，捕获视频”：

```javascript
// 在渲染进程中
const {desktopCapturer} = require('electron')

desktopCapturer.getSources({types: ['window', 'screen']}, (error, sources) => {
  if (error) throw error
  for (let i = 0; i < sources.length; ++i) {
    if (sources[i].name === 'Electron') {
      navigator.mediaDevices.getUserMedia({
        audio: false,
        video: {
          mandatory: {
            chromeMediaSource: 'desktop',
            chromeMediaSourceId: sources[i].id,
            minWidth: 1280,
            maxWidth: 1280,
            minHeight: 720,
            maxHeight: 720
          }
        }
      }, handleStream, handleError)
      return
    }
  }
})

function handleStream (stream) {
  document.querySelector('video').src = URL.createObjectURL(stream)
}

function handleError (e) {
  console.log(e)
}
```

为了从一个由`desktopCapturer`提供的源里面捕获视频，传递给[`navigator.mediaDevices.getUserMedia`] 的约束信息必须包括`chromeMediaSource: 'desktop'`和`audio: false`。

为了从整个desktop捕获音频和视频，传递给[`navigator.mediaDevices.getUserMedia`] 的约束信息必须包含：`chromeMediaSource: 'desktop'`，对于音频和视频来说都是必须的。但是不能指定`chromeMediaSourceId`的约束条件。

```
const constraints = {
  audio: {
    mandatory: {
      chromeMediaSource: 'desktop'
    }
  },
  video: {
    mandatory: {
      chromeMediaSource: 'desktop'
    }
  }
}
```



## 方法

`desktopCapturer` 模块有如下方法:

### `desktopCapturer.getSources(options, callback)`

* `options` Object
  * `types` Array - 一个 String 数组，列出了可以捕获的桌面资源类型, 可用类型为 `screen` 和 `window`.
  * `thumbnailSize` Object (可选) - 建议缩略可被缩放的 size, 默认为 `{width: 150, height: 150}`.
* `callback` Function
  *  `error`错误
  *  `sources` [DesktopCapturerSource](structures/desktop-capturer-source.md)

  发起一个获取所有桌面资源的请求，当请求完成的时候，请使用 `callback(error, sources)` 调用  `callback` .

`sources` 是一个[DesktopCapturerSource](structures/desktop-capturer-source.md)对象数组, 每个 `DesktopCapturerSource r` 表示了一个可以被捕获的屏幕或单独窗口。