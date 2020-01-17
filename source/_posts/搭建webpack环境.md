---
title: " 搭建webpack环境\t\t"
tags:
  - webpack
url: 138.html
id: 138
comments: false
categories:
  - front
date: 2018-10-19 10:19:26
---

webpack安装，可以参考[webpack4.0安装](https://blog.csdn.net/qq_36407748/article/details/80968300)
最好是本地安装

```bash
npm/cnpm install --save-dev webpack
//如果是4.x还要安装webpack-cli
npm/cnpm install --save-dev webpack-cli
```
安装webpack-dev-server
    
```bash
npm/cnpm install --save-dev webpack-dev-server
```
在package.json里的scripts里面添加
    
```json
"dev":"webpack-dev-server --open --config webpack.config.js"
```
`--open`表示启动之后打开浏览器
`--config`表示配置文件的地址
在当前目录下新建`webpack.config.js`
写入如下内容
    

```javascript
var config = {};
moudule.exports = config;
```

使用命令行执行`npm run dev`就会启动本地服务器

webpack配置最重要也是必选的两项是入口(entry)和出口(output)，入口是告诉webpack从哪里开始寻找依赖，并且编译，出口则是用来配置编译后的文件存储位置和文件名
在目录下新建一个空的main.js作为入口文件，然后在webpack.config.js中进行配置
    

```javascript
var path = require('path');
var config = {
	entry: {
		main: './main'
	},
	output: {
		path: path.join(__dirname,'./dist'),
		publicPath: '/dist/',
		filename: 'main.js'
	}
};
module.exports = config;
```

- entry中的main是配置的单入口，webpack会从main.js开始工作
- output中的path选项用来存放打包文件的输出目录（必填）
- publicPath指定资源文件引用的目录，可以填CDN的网址
- filename指定输出文件的名称

然后在根目录下新建一个index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
	<div id="app">
		这是一个主页
	</div>
	<script type="text/javascript" src="/dist/main.js"></script>
</body>
</html>
```
执行`npm run dev`将本地服务器跑起来，会看到这是一个主页
然后在main.js中加入
    
```javascript
document.getElementById('app').innerHTML = 'Hello webpack';
```
再到网页上看，页面显示已经变成了 webpack，修改了main.js中的内容之后，webpack会自动编译并刷新浏览器
webpack通过一个websocket来实时响应代码的修改
没有编译之前，代码是存在于内存中的，可以使用下面的代码，按照之前的配置输出到目录
    
```bash
webpack --progress --hide-modules
```
## 完善配置  
webpack可以通过按照不同的加载器对不同后缀名的文件进行处理，如写css就要用到css-loader和style-loader
可以通过下面了命令安装
    
```bash
cnpm/npm insatll --save-dev css-loader
cnpm/npm insatll --save-dev style-loader
```
然后在配置文件里配置loader

```javascript
var path = require('path');

var config = {
	entry: {
		main: './main'
	},
	output: {
		path: path.join(__dirname,'./dist'),
		publicPath: '/dist/',
		filename: 'main.js'
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [
					'style-loader',
					'css-loader'
				]
			}
		]
	}
};
module.exports = config;
```
moudule的rules属性可以指定一系列的loaders，每个loader都必须包含test和use两个选项

npm i ..... 批量安装
-D 等同于 --save-dev
-S 等同于 --save