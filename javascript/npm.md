

[TOC]



## 1.npm基本命令

npm help vue

npm install -g webpack 全局安装

npm update -g webpack 全局更新

npm init [-y] 当前项目下创建package.json文件 -y会忽略默认的提示

npm list --depth=1 --global
–depth 表示深度，我们使用的模块会有依赖，深度为零的时候，不会显示依赖模块

这个指令可以用来 显示 出我们的项目中安装了哪些模块，其实就是 package.json 文件中 的 dependencies 和 devDependencies 的和


npm list --global  这个指令用来查看全局安装了哪些工具

npm list <packagename>
这个指令用来查看某个模块是否安装了



## 2.附加优化设置

设置全部缓存与模块安装位置， 不然默认都在C盘 很占C盘空间。。。

npm config set prefix "D:\nodejs\node_global"

npm config set cache "D:\nodejs\node_cache"//缓存



安装cnpm 使用淘宝镜像来加速模块的安装更新等

npm install -g cnpm --registry=https://registry.npm.taobao.org



## 3.devDependencies与dependencies

**安装到生产环境(开发环境也能使用) dependencies**

npm install query --save 



**安装到开发环境devDependencies**

npm install webpack --save-dev

npm install webpack -D

