# electron-builder [![npm version](https://img.shields.io/npm/v/electron-builder.svg?label=latest)](https://yarn.pm/electron-builder) [![downloads per month](https://img.shields.io/npm/dm/electron-builder.svg)](https://yarn.pm/electron-builder)
electron-builder是一个完整的解决方案，对于macos、windows、linux下的electron app，它可以提供打包及构建的相关功能。同时，它还提供开箱即用的“自动更新”功能支持。

* NPM 包管理:
  * [原生应用依赖](https://electron.atom.io/docs/tutorial/using-native-node-modules/) 编译 (包括 [Yarn](http://yarnpkg.com/) 支持).
  * Development dependencies are never included. You don't need to ignore them explicitly.
* [Code Signing](code-signing.md) on a CI server or development machine.
* [Auto Update](auto-update.md) ready application packaging.
* 众多目标格式:
  * 所有平台: `7z`, `zip`, `tar.xz`, `tar.lz`, `tar.gz`, `tar.bz2`, `dir` (unpacked directory).
  * [macOS](configuration/mac.md#MacConfiguration-target): `dmg`, `pkg`, `mas`, `mas-dev`.
  * [Linux](configuration/linux.md#LinuxConfiguration-target): [AppImage](http://appimage.org), [snap](http://snapcraft.io), debian package (`deb`), `rpm`, `freebsd`, `pacman`, `p5p`, `apk`.
  * [Windows](configuration/win.md#WindowsConfiguration-target): `nsis` (Installer), `nsis-web` (Web installer), `portable` (portable app without installation), AppX (Windows Store), Squirrel.Windows.
* [Two package.json structure](tutorials/two-package-structure.md) is supported, but you are not forced to use it even if you have native production dependencies.  
* [Build version management](configuration/configuration.md#build-version-management).
* [Publishing artifacts](/configuration/publish) to GitHub Releases, Amazon S3, DigitalOcean Spaces and Bintray.
* Pack in a distributable format [already packaged app](#pack-only-in-a-distributable-format).
* 独立的 [构建步骤](https://github.com/electron-userland/electron-builder/issues/1102#issuecomment-271845854).
* Build and publish in parallel, using hard links on CI server to reduce IO and disk space usage.
* [electron-compile](https://github.com/electron/electron-compile) support (compile for release-time on the fly on build).
* [Docker](/multi-platform-build#docker) images to build Electron app for Linux or Windows on any platform.

| 问题                      | 回答                                       |
| ----------------------- | ---------------------------------------- |
| 我想要配置 electron-builder” | [查看选项](/configuration/configuration.md)  |
| “我有一个问题”                | [提交一个issue](https://github.com/electron-userland/electron-builder/issues) 或者 [参与讨论](https://slackin.electron.build) |
| “我发现了一个bug”             | [提交一个issue](https://github.com/electron-userland/electron-builder/issues/new) |
| “我想要捐钱”                 | [捐献](/donate.md)                         |

真实项目实例 — [onshape-desktop-shell](https://github.com/develar/onshape-desktop-shell).

## 安装
[强烈推荐](https://github.com/electron-userland/electron-builder/issues/1147#issuecomment-276284477) 您使用 [Yarn](https://yarn.org.cn/) ，来取代npm。

`yarn add electron-builder --dev`

Platform specific `7zip-bin-*` packages are `optionalDependencies`, which may require manual install if you have npm configured to [not install optional deps by default](https://docs.npmjs.com/misc/config#optional).

## 模版

* [electron-webpack-quick-start](https://github.com/electron-userland/electron-webpack-quick-start) — A bare minimum project structure to get started developing with [electron-webpack](https://github.com/electron-userland/electron-webpack). Recommended.
* [electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate) A boilerplate for scalable cross-platform desktop apps.
* [electron-react-redux-boilerplate](https://github.com/jschr/electron-react-redux-boilerplate) A minimal boilerplate to get started with Electron, React and Redux.
* [electron-boilerplate](https://github.com/szwacz/electron-boilerplate) A minimalistic yet comprehensive boilerplate application.

## 快速设置向导

[electron-webpack-quick-start](https://github.com/electron-userland/electron-webpack-quick-start) is a recommended way to create a new Electron application.

1. Specify the standard fields in the application `package.json` — [name](/configuration/configuration.md#Metadata-name), `description`, `version` and [author](https://docs.npmjs.com/files/package.json#people-fields-author-contributors).

2. 在下面的`package.json`文件中，如下指明 [构建](/configuration/configuration.md#configuration) 配置：
```json
"build": {
      "appId": "your.id",
      "mac": {
        "category": "your.app.category.type"
      }
    }
```
参见 [所有选项](/configuration/configuration.md#configuration)。

3. 添加 [图标](/icons.md).

4. 添加 [scripts](https://docs.npmjs.com/cli/run-script) 内容到开发版的`package.json`中:

    然后你就可以运行命令 `yarn dist` (to package in a distributable format (e.g. dmg, windows installer, deb package)) 或者运行命令 `yarn pack` (only generates the package directory without really packaging it. This is useful for testing purposes).

    为了确保你的原生依赖总是搭配electron的版本号，最简单的的办法就是添加如下命令： `"postinstall": "electron-builder install-app-deps"` 到你的 `package.json`文件之中。

```json
    "scripts": {
      "pack": "electron-builder --dir",
      "dist": "electron-builder"
    }
```

5. 如果你的应用里面，包含有自己的原生addon (并不是依赖项)，那么请设置 [nodeGypRebuild](/configuration/configuration#Configuration-nodeGypRebuild)为`true`。

6. 如果你不是 macOS 10.12+ 系统的话，您需要安装 [必须的系统包](/multi-platform-build.md)。请注意， [默认情况](configuration/configuration.md#Configuration-asar)下，所有的文件都会被打包到asar压缩包中。对于一个将要马上投入生产的app，你应该对你的应用进行签名。参见 [哪里可以购买签名证书](/code-signing.md#where-to-buy-code-signing-certificate).

## 编程使用
请参见 `node_modules/electron-builder/out/index.d.ts`。为 TypeScript 提供了 Typings。

```js
"use strict"

const builder = require("electron-builder")
const Platform = builder.Platform

// Promise is returned
builder.build({
  targets: Platform.MAC.createTarget(),
  config: {
   "//": "build options, see https://goo.gl/ZhRfla"
  }
})
  .then(() => {
    // handle result
  })
  .catch((error) => {
    // handle error
  })
```

## 只在发布版本下打包

您可以用electron-builder只打包下面的某种格式。 AppImage, Snaps, Debian package, NSIS, macOS installer component package (`pkg`) 以及其他发行格式。.

```
./node_modules/.bin/build --prepackaged <packed dir>
```

`--projectDir` (指向project目录) 选项也是很有用的。

## 社区

Slack的[electron-builder](https://slackin.electron.build) 频道  (请使用 [threads](https://get.slack.help/hc/articles/115000769927-Message-threads)).
无需注册，公共 [打包](http://electron-builder.slackarchive.io) .