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
 ```json
    "scripts": {
      "pack": "electron-builder --dir",
      "dist": "electron-builder"
    }
```
    Then you can run `yarn dist` (to package in a distributable format (e.g. dmg, windows installer, deb package)) or `yarn pack` (only generates the package directory without really packaging it. This is useful for testing purposes).

    To ensure your native dependencies are always matched electron version, simply add script `"postinstall": "electron-builder install-app-deps"` to your `package.json`.

5. If you have native addons of your own that are part of the application (not as a dependency), set [nodeGypRebuild](/configuration/configuration#Configuration-nodeGypRebuild) to `true`.

6. Install the [required system packages](/multi-platform-build.md) if you are not on macOS 10.12+.

Please note that everything is packaged into an asar archive [by default](configuration/configuration.md#Configuration-asar).

For an app that will be shipped to production, you should sign your application. See [Where to buy code signing certificates](/code-signing.md#where-to-buy-code-signing-certificate).

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

## Pack Only in a Distributable Format

You can use electron-builder only to pack your electron app in a AppImage, Snaps, Debian package, NSIS, macOS installer component package (`pkg`) 
and other distributable formats.

```
./node_modules/.bin/build --prepackaged <packed dir>
```

`--projectDir` (the path to project directory) option also can be useful.

## 社区

[electron-builder](https://slackin.electron.build) on Slack (please use [threads](https://get.slack.help/hc/articles/115000769927-Message-threads)).
Public [archive](http://electron-builder.slackarchive.io) without registration.