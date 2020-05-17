## webpack-dev-server

1.安装webpack与webpack-dev-server

```cmd
cnpm i webpack -D
cnpm i webpack-dev-server -D
```

2.package.json的scripts中配置启动脚本

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --inline --progress --open --colors --config ./config/webpack.dev.js",
    "start": "npm run dev",
    "build": "webpack --config ./config/webpack.prod.js"
 }
```

说明：

```txt
--open：代码修改保存完，直接打开 http://localhost:8080
--port：修改端口号：8080，为端口号：3000
--contentBase：代码修改保存完运行或是浏览器输入 http://localhost:8080，如果指定为src
则直接打开 http://localhost:8080 下的 src文件，展示项目网页, 也就是说src下的文件改动了的话热更新会更新页面
--hot：无需刷新页面，即可看见修改代码保存后的效果；而且只是局部更新打包，减少不必要的打包，速度变快
--config：指定配置文件
--inline/iframe: 指定刷新模式
--quiet 控制台中不输出打包的信息
--compress 开启gzip压缩
--progress 显示打包的进度
```





## html-webpack-plugin

我们的index.html文件中引入我们打包后的buddle.js文件，有2种方式：

1.手动引用对应的js

2.利用html-webpack-plugin,先指定我们的html的文件是哪个，自动在内存中生成一个对应的页面并把js自动注入到内存html中

```js
cnpm i html-webpack-plugin -D

const HtmlWebpackPlugin = require('html-webpack-plugin');

new HtmlWebpackPlugin({
    template: './src/index.html',
    inject: true
}),
```





## 热更新

1.添加 devServer：

```js
devServer: {
    // 这是配置 dev-server 命令参数的第二种形式，相对来说麻烦一些
    open: true, // 自动打开浏览器
    port: 3000, // 设置启动时候的运行端口
    contentBase: 'src', // 指定托管的根目录
    hot: true // 启动热更新的第一步
}
```

2.启用热更新：

```js
// 启用热更新的第二步
var webpack = require('webpack');
```



3.配置插件的节点：

```js
plugins:[
    // 配置插件的节点
    // new 一个热更新的模块对象，这是启用热更新的第三步
   new webpack.HotModuleReplacementPlugin()
]
```
