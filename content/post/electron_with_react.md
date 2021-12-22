---
title: "【浅尝辄止】在 Electron 中使用 React"
date: 2021-12-22T09:53:06+08:00
draft: true
tags: [JavaScript,React,Electron]
categories: [浅尝辄止]
---

这是一个“Hello World”级别的教程，目的是基于 React 技术来构建 Electron 工程。主要参考资料是[ Electron 官方文档的快速入门（Quick Start）](https://www.electronjs.org/zh/docs/latest/tutorial/quick-start)和这里的一篇文章 [Getting Started with Electron by Creating a React App](https://www.section.io/engineering-education/desktop-application-with-react/) 不过这篇文章只说明了如何在开发模式下运行，通过 Electron 官方教程使用的打包工具 electron-forge 打包后的 App 是无法运行的，这里给出了解决方案，保证最终通过 electron-forge 打包后的 App 也可以顺利运行起来。

## 新建一个 React 工程

首先通过 `create-react-app` 来新建一个空白的工程。

```shell
npx create-react-app electron-react-app
```

接下来把 Electron 添加进来

```shell
cd electron-react-demo

npm i -D electron

npm i -S electron-is-dev
```

这里特别提一下 `electron-is-dev` 这个依赖。因为 React 在开发模式下会在本地默认启动一个 3000 端口的 HTTP 服务，而正是打包的时候是不需要的，所以这个依赖是用来判断 Electron 运行在开发模式还是打包模式下的。 [Getting Started with Electron by Creating a React App](https://www.section.io/engineering-education/desktop-application-with-react/) 这篇文章里添加这个依赖的时候使用了 `--save-dev` 这会导致在 Electron 打包的时候无法正确引入这个依赖，从而在打包时无法判断 Electron 的运行模式。

## Electron 入口 electron.js

```javascript
const { app, BrowserWindow } = require('electron')
const path = require('path')
const isDev = require('electron-is-dev');

function createWindow () {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    }
  })

  // win.loadFile('index.html')
  win.loadURL(
    isDev
      ? 'http://localhost:3000'
      : `file://${path.join(__dirname, '../build/index.html')}`
  );
  // Open the DevTools.
  if (isDev) {
    win.webContents.openDevTools({ mode: 'detach' });
  }
}

app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})
```

这基本上就是在[ Electron 官方文档的快速入门（Quick Start）](https://www.electronjs.org/zh/docs/latest/tutorial/quick-start)中提供的 `main.js` 基础上做了几处修改。

首先 Electron 默认的入口文件叫做  `main.js` 这个需要改个名字叫做  `electron.js` 并放到  `public` 目录下面。

其次就是  `win.loadFile('index.html')` 这里需要使用 `electron-is-dev` 进行判断，如果是开发模式则加载本地默认 3000 端口上的 HTTP 服务，如果是打包模式则直接加载 React 构建产生的 `build/index.html`

最后别忘记修改 `package.json` 文件配置好 Electron 的入口位置

```json
{
    "main": "public/electron.js"
}
```

## 开发模式启动 Electron

首先安装两个依赖

```shell
npm i -D concurrently wait-on
```

然后编辑 `package.json` 分别添加两个命令

```json
{
    "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test",
        "eject": "react-scripts eject",
        "dev": "concurrently -k \"BROWSER=none npm start\" \"npm:electron\"",
        "electron": "wait-on tcp:3000 && electron ."
    }
}
```

主要是最后两个。`dev` 命令首先以开发模式运行 React 项目，然后调用后面的 `electron` 命令。而  `electron` 命令等待本地的 3000 端口启动，然后运行 Electron

如果一切顺利弹出的 Electron 程序窗口就可以显示出 React 的 HelloWorld 页面了。

## 打包运行 Electron

首先先构建 React 工程

```shell
npm run build
```

接着按照[ Electron 官方文档的快速入门（打包并分发您的应用程序）](https://www.electronjs.org/zh/docs/latest/tutorial/quick-start#%E6%89%93%E5%8C%85%E5%B9%B6%E5%88%86%E5%8F%91%E6%82%A8%E7%9A%84%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F)的章节操作先安装 `electron-forge` 依赖

```shell
npm i -D @electron-forge/cli
```
然后进行初始化

```shell
npx electron-forge import
```

最后进行打包

```shell
npm run make
```

 打包成功后运行你打包好的程序，这时候你发现会报错说找不到模块。这是由于我们构建的 React 工程对文件的引入路径有问题，解决办法很简单，只需要在 `package.json` 中设置 `homepage` 即可
 
```json
{
    "main": "public/electron.js",
    "homepage": "./"
}
```

再次重新构建 React 工程并重新进行 Electron 打包，一切顺利的话你打包出来的程序也可以正常运行并显示出 React 的 HelloWorld 页面了。
