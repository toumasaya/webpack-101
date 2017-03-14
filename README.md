# Webpack 2 實作筆記

建議可以先看這篇文章： [A Detailed Introduction To Webpack](https://www.smashingmagazine.com/2017/02/a-detailed-introduction-to-webpack/) 

作者用很淺顯易懂的範例讓你了解 webpack 在病蝦咪忙。

接著可以看這系列的教學影片，時間都蠻短的，講的也很淺顯易懂，跟著做一次就可以上手基本的 webpack。

教學影片：
* [Webpack 2 Tutorial - Installation and Config](https://www.youtube.com/watch?v=JdGnYNtuEtE)
* [Webpack 2 - HTML Webpack Plugin](https://www.youtube.com/watch?v=cKTDYSK0ArI)
* [Webpack 2 - Style, CSS and Sass loaders](https://www.youtube.com/watch?v=m7V0OackwxY&t=16s)
* [Webpack 2 with Webpack Dev Server](https://www.youtube.com/watch?v=gH4LxB6NkNc)
* [Webpack 2 with Webpack Dev Server Configuration](https://www.youtube.com/watch?v=soI7X-7OSb4)
* [Webpack 2 - How to install React and Babel](https://www.youtube.com/watch?v=zhA5LNA3MxE)
* [Webpack 2 - Multiple templates options and RimRaf](https://www.youtube.com/watch?v=OvjB2Sfq9ZU)

---

以下是影片的筆記。

我是使用 [Yarn](https://yarnpkg.com/zh-Hans/)，用 NPM 也可以。

## Installation and Config

建立專案目錄並且進到目錄，然後初始化 `yarn`：

```shell
$ mkdir webpack-101 && cd webpack-101
$ yarn init
```

安裝 `webpack`：

```shell
$ yarn add -D webpack
```

預計目錄結構：

```
| - src/
    | - app.js
| - dist/
    | - app.bundle.js
| - webpack.config.js
| - package.json
| - yarn.lock
```

新增 `src/app.js`，加上：

```javascript
console.log('Hello World!');
```

新增 `webpack.config.js`，加上：

```javascript
module.exports = {
  entry: './src/app.js', // 要輸出的檔案入口
  output: {
    filename: './dist/app.bundle.js' //最終的目的檔案
  }
}
```

因為是把 webpack 裝在專案底下，無法直接透過 `webpack` 指令在 terminal 執行，但可以透過 npm script 來設定，在 `package.json`：

```javascript
...
"script": {
  "dev": "webpack -d --watch", // -d 是開發模式，--watch 是監聽檔案變更
  "build": "webpack -p" // -p 是線上模式，預設會壓縮檔案
},
...
```

執行 `yarn run dev` 和 `yarn run build` 觀看結果。

## HTML Webpack Plugin

安裝 `html-webpack-plugin`：

```shell
$ yarn add -D html-webpack-plugin
```

設定 `webpack.config.js`：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/app.js',
  output: {
    path: 'dist', 
    filename: 'app.bundle.js'
  },
  plugins: [
    new HtmlWebpackPlugin()
  ],
}
```

其中原本的：

```
filename: './dist/app.bundle.js'
```

拆成：

```
path: 'dist',
filename: 'app.bundle.js'
```

按照 `html-webpack-plugin` 官方文件要拆成兩段 `path` 和 `filename`，這樣 plugin 才能知道要在 `dist` 目錄產生 HTML，並且在 HTML 自動注入 `app.bundle.js`。

### 其他選項設定

```javascript
...
plugins: [
  new HtmlWebpackPlugin({
    minify: {
      collapseWhitespace: true, // 壓縮 HTML
    },
    hash: true, // 會幫 app.bundle.js 加上 hash
    template: './src/index.html' // 自訂 template，後面會講怎麼使用 pug，自訂完要自己產生一個 index.html
  })
],
...
```

## Style, CSS and Sass loaders

### CSS

安裝 `css-loader` 和 `style-loader`：

```shell
$ yarn add -D css-loader style-loader
```

新增一個 `src/app.css`，加上：

```css
body {
  background: pink;
}
```

設定 `webpack.config.js`：

```javascript
...
module: {
  rules: [
    { test: /\.css$/, use: ['style-loader', 'css-loader'] }
  ],
},
...
```

`rules` 的值是陣列，裡面會放一些設定。

`test` 的值通常用正規表示式，用來過濾符合條件的檔案。

`use` 裡面放要使用的 loader，執行順序會是由右到左，所以會先執行 `css-loader` 然後再執行 `style-loader`。

`css-loader` 會被注入到 `app.bundle.js` 裡面，然後再透過 `style-loader` 把 CSS 寫入到 HTML `style` tag 裡面。

這不是個好方法，不過目前先這樣。

最後在 `app.js` 加上：

```javascript
const css = require('./app.css');
```

### Sass

安裝 `sass-loader` 和 `node-sass`：

```shell
$ yarn add -D sass-loader node-sass
```

設定 `webpack.config.js`：

```javascript
...
module: {
  rules: [
    { test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }
  ],
},
...
```

`test` 條件改成 `.scss`。

`use` 多增加一個 `sass-loader`，因為在右邊，所以會先被執行。

設定 `app.js`，改成：

```javascript
const css = require('./app.scss');
```

最後把原本的 `app.css` 改成 `app.scss`，然後寫一些 Sass ：

```sass
body {
  background: pink;
  p {
    color: green;
  }
}
```

執行 `yarn run dev` 可以看到結果。

### 把 CSS 檔案抽離成一個單獨檔案

前面的做法會讓 CSS 直接注入在 HTML 的 `style` tag 裡面，這其實不是很好的做法，可以透過 `extract-text-webpack-plugin` 來把 CSS 檔案抽出來。

安裝 `extract-text-webpack-plugin`：

```shell
$ yarn add -D extract-text-webpack-plugin
```

設定 `webpack.config.js`：

```javascript
...
const ExtractTextPlugin = require('extract-text-webpack-plugin');

...
module: {
  rules: [
    { 
      test: /\.scss$/, 
      use: ExtractTextPlugin.extract({
        fallback: 'style-loader',
        use: ['css-loader', 'sass-loader'],
        publicPath: './dist'
      }), 
    },
  ],
},
plugins: [
  ...
  new ExtractTextPlugin({
    filename: 'app.css',
    disable: false,
    allChunks: true
  }),
],
...
```

webpack 1.x 的寫法跟 2.x 不一樣，如果設定遇到問題可以查一下官方文件。

執行 `yarn run build`，可以看到 `index.html` 的輸出大概會長這樣：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  <link href="app.css?85bd7a55f6b243abc006" rel="stylesheet"></head>
  <body>
  <script type="text/javascript" src="app.bundle.js?85bd7a55f6b243abc006"></script></body>
</html>
```

## Webpack 2 with Webpack Dev Server

安裝 `webpack-dev-server`：

```shell
$ yarn add -D webpack-dev-server
```

設定 `package.json`，把 `dev` 改成：

```  javascript
...
"scripts": {
  "dev": "webpack-dev-server",
  "build": "webpack -p"
},
...
```

執行 `yarn run dev`，結果出現錯誤：

```shell
Error: `output.path` needs to be an absolute path or `/`.
```

`output` 的路徑要設成絕對路徑，參考 [webpack 官方文件](https://webpack.js.org/configuration/output/#output-publicpath)，改成：

```javascript
...
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'app.bundle.js'
},
...
```

再執行 `yarn run dev`，就可以到 locallhost:8080 瀏覽頁面。

## Webpack 2 with Webpack Dev Server Configuration

原本在 `package.json` 的 `dev` 是使用：

```
"dev" : "webpack -d --watch"
```

這個指令會去搜索 `dist` 資料夾，當 `src/app.scss` 變更，`dist/app.css` 就會變更。

後來改成：

```
"dev": "webpack-dev-server"
```

`webpack-dev-server` 是靠記憶體快取運作，所以變更 `src/app.scss` ，`dist/app.css` 不會跟著變，但如果重整瀏覽器可以看到變化。

可以利用 `devServer` 的選項做更多設定：

```javascript
...
devServer: {
  contentBase: path.join(__dirname, "dist"), // 要跑 server 的目錄
  compress: true, // 用 gzip 壓縮檔案
  stats: "errors-only", // 只輸出錯誤訊息
  open: true // 第一次啟動會自動打開瀏覽器
},
...
```

## How to install React and Babel

安裝 `react` 和 `react-dom`：

```shell
$ yarn add react react-dom
```

安裝 Babel 相關套件：

```shell
$ yarn add -D babel babel-preset-react babel-preset-es2015
```

安裝 Babel webpack loader：

```shell
$ yarn add -D babel-loader babel-core
```

在 `src/app.js` 加上 React 語法：

```javascript
...
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

然後在 `src/index.hml` 加上：

```html
<div id="root"></div>
```

設定 `webpack-config.js`：

```javascript
module: {
  rules: [
    ...
    {
      test: /\.js$/,
      exclude: /node_modules/,
      use: 'babel-loader'
    },
  ],
},
```

執行 `yarn run dev` 就可以到 localhost:8080 看到成果。

## Multiple templates options and RimRaf

### 在每次 build 之前清除舊的 dist 目錄

在每次執行 `yarn run build` 之前，刪除 `./dist`。

安裝 `rimraf`：

```shell
$ yarn add -D rimraf
```

設定 `package.json`：

```javascript
...
"scripts": {
  "dev": "webpack-dev-server",
  "build": "yarn run clean && webpack -p",
  "clean": "rimraf ./dist/*"
},
...
```

### 建立多個 template

在 `webpack.config.js` 增加一個 Contact template：

```javascript
...
plugins: [
  new HtmlWebpackPlugin({
    hash: true,
    template: './src/index.html'
  }),
  new HtmlWebpackPlugin({
    hash: true,
    filename: 'contact.html', // 指定 template 要輸出的路徑
    template: './src/contact.html'
  }),
  ...
],
...
```

其中 `filename: 'contact.html'`，可以指定 template 要輸出的路徑，因為 `output` 設定的路徑就是 `path: path.resolve(__dirname, 'dist')`，也就是 `dist/`，所以最終檔案就是會被輸出到那裡。

也可以按需求改變路徑，例如：

```
filename: './../contact.html
```

會被輸出到專案根目錄底下。

接著再建立一個 `src/contact.html`：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <h1>Contact page</h1>
</body>
</html>
```

執行 `yarn run dev` ，到 localhost:8080/contact.html 可以看到成果。再執行 `yarn run build` 就可以看到 `contact.html` 被輸出到 `dist`。

### 如何在不同的 template 注入不同的 bundle.js

建立一個 `src/contact.js`，加上：

```javascript
console.log('Contact me Baby');
```

多了一個新的 entry file，如果想要多個 output files 多個 bundle files，就要重新設定 `entry`，加上多個 entry files。

修改 `webpack.config.js`：

```javascript
...
entry: {
  app: './src/app.js',
  contact: './src/contact.js'
},
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: '[name].bundle.js'
},
...
```

把原本的 `entry: './src/app.js'` 改成物件：

```
entry: {
  app: './src/app.js',
  contact: './src/contact.js'
}
```

這樣就可以加入多個 entry files。

因為想要一一輸出每個 entry files，所以把 `filename: 'app.bundle.js'` 的檔名改成能動態產生相對應的檔名：

```
filename: '[name].bundle.js'
```

`[name]` 會對應到 `entry` 的 key。

執行 `yarn run dev` ，到 localhost:8080 瀏覽頁面，不過看起來目前兩個 template 都注入兩個 bundle files，`app.bundle.js` 和 `contact.bundle.js`：

![](https://i.imgur.com/XtSV6vx.png)

為了要讓 `index.html` 只注入 `app.bundle.js`，而 `contact.html` 只注入 `contact.bundle.js`，要修改一下設定：

在 `webpack.config.js`：

```javascript
...
plugins: [
  new HtmlWebpackPlugin({
    hash: true,
    excludeChunks: ['contact'], // 排除 contact.bundle.js
    template: './src/index.html'
  }),
  new HtmlWebpackPlugin({
    hash: true,
    chunks: ['contact'], // 指定 contact.bundle.js
    filename: 'contact.html',
    template: './src/contact.html'
  }),
  ...
],
...
```

再執行一次 `yarn run dev`，就可以看到各就各位了：

![](https://i.imgur.com/aMFCJ2p.png)

![](https://i.imgur.com/tF1HIqX.png)



## VS Code 小技巧

可以透過設定 Workspace settings ，讓 `node_modules` 或 `.vscode` 資料夾隱藏起來，按 `command + shift + P`，搜尋 `Workspace settings`：

![](https://i.imgur.com/neTLC30.png)

然後搜尋 `exclude`：

![](https://i.imgur.com/VwWINEe.png)

把左邊的 `exclude` 設定貼到右邊的使用者設定，然後多加兩個條件：

```
"node_modules": true,
".vscode": true
```

這樣這兩個資料夾就會被隱藏起來。

---



