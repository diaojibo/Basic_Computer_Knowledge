# Electron开发
Electron是一个跨平台的界面程序开发框架，使用js html css可以方便开发出桌面端应用。

### 开始开发
开发electron应用其实很简单，只要在npm项目加入这个库就ok。

```
npm install --save-dev electron
```

一个简单的electron项目只用两个主要文件就够了：

 - index.html(页面布局)
 - main.js(启动脚本)

并且我们需要修改package.json,我们需要用electron启动app而不是用node来启动。

以下为package.json

```
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "electron ."
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "electron": "^3.0.12"
  }
}

```


我们一个实例启动脚本如下:

```
const { app, BrowserWindow } = require('electron')

function createWindow () {   
  // 创建浏览器窗口
  win = new BrowserWindow({ width: 800, height: 600 })

  // 然后加载应用的 index.html。
  win.loadFile('index.html')
}

app.on('ready', createWindow)
```

然后写好hello world html：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
</head>
<body>
<h1>Hello World!</h1>
We are using node <script>document.write(process.versions.node)</script>,
Chrome <script>document.write(process.versions.chrome)</script>,
and Electron <script>document.write(process.versions.electron)</script>.
</body>
</html>
```

最后在终端执行：

```
npm start
```

即会看到应用启动

### 参考

[Electron官方开发文档](https://electronjs.org/docs/tutorial/first-app)