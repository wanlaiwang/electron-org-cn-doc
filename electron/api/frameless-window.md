# Frameless Window

> 打开一个窗体没有工具栏，边框，或者其它图形化的"chrome"

无边框窗口指的是：除页面本身以外，不包含任何其它可视部分的窗口([chrome](https://developer.mozilla.org/en-US/docs/Glossary/Chrome))。
像工具栏，这不是页面的一部分。这些是[`BrowserWindow`](browser-window.md) 类的选项。

## 创建无边框窗口

为了创建一个无边框窗口，你需要设置[BrowserWindow](browser-window.md)的选项`frame`为`false`:

```javascript
const BrowserWindow = require('electron').BrowserWindow
var win = new BrowserWindow({ width: 800, height: 600, frame: false })
```

### macOS上的替代方案

在macOS 10.9 Mavericks或者更新的版本中，有一个替代方案，可以生成一个无边框窗口。设置`frame`为`false`，会隐藏标题栏以及窗口控制区域（最大化按钮、最小化按钮、还原按钮）。你可能想隐藏标题栏，也希望你的页面上下文能够继承全屏大小，也同时又想保持对窗口的控制("traffic lights")。你可以通过指定`titleBarStyle`这一选项来达到目的。

#### `hide`
你将可以得到一个隐藏的标题栏和一个全尺寸的内容窗体，然后在左上方仍然存在标准的窗体交通灯控制按钮("traffic lights")。

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({titleBarStyle: 'hidden'})
win.show()
```

#### `hiddenInset`
你会得到一个隐藏标题栏的另外一种外观，交通灯按钮的位置，比标准的情况下，更加远离窗口的边缘位置。

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({titleBarStyle: 'hiddenInset'})
win.show()
```

#### `customButtonsOnHover`
当激活窗体左上角的时候，将会显示一组定制的交通灯按钮组合，包括关闭、最小化、全屏按钮。这些定制的按钮可以避免在标准窗体工具栏按钮上的一些鼠标事件issue。这个选项只在无边框窗体中才会生效。

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({titleBarStyle: 'customButtonsOnHover', frame: false})
win.show()
```

## 透明窗口

通过设置`transparent` 选项为 `true`，你能使无边框窗口透明:

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({transparent: true, frame: false})
win.show()
```

### 限制

* 你无法点击透明的区域。我们正在采用一个新的API去设置窗口的外形以解决这个问题，
  详见[our issue](https://github.com/electron/electron/issues/1335)。
* 透明窗口是不可调整大小的。在某些平台上，设置`resizable`为`true`也许会造成这个透明窗口停止工作。
* `blur`滤光器器只适用于网页，所以没法将模糊效果用于窗口之下(比如：其它在用户的系统中打开的应用)。
* 在Windows操作系统中，当DWM（桌面窗口管理器）被禁用时，透明窗口不能正常工作。
* Linux用户需要在命令行中输入`--enable-transparent-visuals --disable-gpu`，来禁用GPU以及允许ARGB去渲染透明窗口，这是由于一个Linux上的上游bug[alpha channel doesn't work on some NVidia drivers](https://code.google.com/p/chromium/issues/detail?id=369209)所造成的
* 在Mac上，透明窗口的阴影不会被显示出来的。

## 可点透区域
为了创建一个可点透窗体，比如，让窗体忽略所有的鼠标时间，你可以调用[win.setIgnoreMouseEvents(ignore)](browser-window#winsetignoremouseeventsignore) API:
```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()
win.setIgnoreMouseEvents(true)
```

### Forwarding
Ignoring mouse messages makes the web page oblivious to mouse movement, meaning that mouse movement events will not be emitted. On Windows operating systems an optional parameter can be used to forward mouse move messages to the web page, allowing events such as `mouseleave` to be emitted:

```javascript
let win = require('electron').remote.getCurrentWindow()
let el = document.getElementById('clickThroughElement')
el.addEventListener('mouseenter', () => {
  win.setIgnoreMouseEvents(true, {forward: true})
})
el.addEventListener('mouseleave', () => {
  win.setIgnoreMouseEvents(false)
})
```

This makes the web page click-through when over `el`, and returns to normal outside it.


## 可拖动区域

默认情况下，无边框窗口是不可拖动的。应用在CSS中设置`-webkit-app-region: drag`
告诉Electron哪个区域是可拖动的(比如系统标准的标题栏)，应用也可以设置`-webkit-app-region: no-drag`
在可拖动区域中排除不可拖动的区域。需要注意的是，目前只支持矩形区域。

为了让整个窗口可拖动，你可以在`body`的样式中添加`-webkit-app-region: drag`:

```html
<body style="-webkit-app-region: drag">
</body>
```

另外需要注意的是，如果你设置了整个窗口可拖动，你必须标记按钮为不可拖动的(non-draggable)，
否则用户不能点击它们:

```css
button {
  -webkit-app-region: no-drag;
}
```

如果你设置一个自定义的标题栏可拖动，你同样需要设置标题栏中所有的按钮为不可拖动(non-draggable)。

## 文本选择

在一个无边框窗口中，拖动动作会与文本选择发生冲突。比如，当你拖动标题栏，偶尔会选中标题栏上的文本。
为了防止这种情况发生，你需要向下面这样在一个可拖动区域中禁用文本选择:

```css
.titlebar {
  -webkit-user-select: none;
  -webkit-app-region: drag;
}
```

## 上下文菜单

在一些平台上，可拖动区域会被认为是非客户端框架(non-client frame)，所有当你点击右键时，一个系统菜单会弹出。
为了保证上下文菜单在所有平台下正确的显示，你不应该在可拖动区域使用自定义上下文菜单。

