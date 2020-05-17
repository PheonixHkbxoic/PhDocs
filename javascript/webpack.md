

[TOC]



## 利用webpack创建第一项目

1.mkdir webpackfirst

2.cnpm init -y 初始化项目生成package.json

```json

{
  "name": "webpackfirst",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```



3.项目全局安装webpack，如果已安装过则可省略
cnpm install webpack -g

4.安装项目依赖
cnpm install webpack --save-dev

可以发现package.json中多了一些配置

```json
"devDependencies": {
    "webpack": "^4.43.0"
  }
```

5.创建并配置webpack.config.js

```js
let path = require('path')
let webpack = require('webpack')
module.exports = {
    entry:'./index.js', // 入口js
    output:{
        path:path.join(__dirname,'dist'),
        filename:'bundle.js'
    },
    module:{
        rules:[
        ]
    }
}
```



6.创建index.js和index.html,一个为入口文件，一个为普通的html文件
在index.js里面写上这行测试用途的代码

```js
document.write("hello world");
```



7.通过npm安装webpack-dev-server
npm install webpack-dev-server --save-dev

8.webpack.config.js中热加载进行配置：module后面添加

```json
	,
	plugins:[
		new webpack.HotModuleReplacementPlugin()
	],
	devServer:{
		contentBase: './',
		historyApiFallback: true,
		inline: true,
		hot: true
		
	}
```



9.对webpack-dev-server的的启动进行一下设置：scripts里添加start这一项

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
	"start": "webpack-dev-server --inline"
 }
```



10.安装webpack-cli
cnpm install -g webpack-cli（全局安装，如果已经安装过则可以忽略）
cnpm install webpack webpack-cli -D（本地继续安装）

11.打包
webpack

12.index.html页面

```html
<!DOCTYPE html>
<html>
<head>
	<title>Webpack First</title>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no" />
	<meta name="keywords" content="第一个Webpack工程">
	<meta name="description" content="第一个Webpack工程: 利用Webpack创建项目">
</head>
<body>
	<script type="text/javascript" src="dist/bundle.js"></script>
</body>
</html>
```



13.启动项目

cnpm run start

浏览器访问 http://localhost:8080/



## 链接

原文链接：https://blog.csdn.net/weixin_40402681/article/details/88345319