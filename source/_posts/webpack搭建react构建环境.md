---
title: webpack搭建react构建环境
date: 2016-12-15 18:36:11
tags: 
- react
- webpack 
categories:
- web前端
---

# Babel和Webpack
事实上react并不需要这些工具就能运行；但是为了充分使用 [ES6](https://github.com/lukehoban/es6features)、[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) 和bunding

<!--more-->
# 让我们重新配置
新建一个react-hello-world文件夹并用npm初始化
```
mkdir react-hello-world
cd react-hello-world
npm init
```

# 安装并配置webpack 
[Webpack](https://facebook.github.io/react/docs/jsx-in-depth.html)可以根据配置文件将依赖文件和资源文件打包在一起
```
# 安装并保存webpack到package.json里面
npm install webpack —save-dev
# 安装webpack的全局命令
npm install -g webpack-cli
```
webpack需要一些配置来指定他如何工作，最佳实践就是把配置写进去webpack.config.js
{% codeblock webpack.config.js lang:js %}
var webpack = require('webpack');
var path = require('path');

var BUILD_DIR = path.resolve(__dirname, 'build');
var APP_DIR = path.resolve(__dirname, 'src');

var config = {
  entry: {
    'index_bundle.js': path.resolve(APP_DIR, 'index.js'),
  }
  output: {
    path: BUILD_DIR,
    // [name]指的是entry对象里面的key
    filename: '[name]'
  }
};

module.exports = config;
{% endcodeblock %}
以上是webpack配置文件的最小化配置，只需要提供entry(入口)和output(输出)

``APP_DIR``是源码文件夹，``BUILD_DIR``是最终打包生成输出的文件夹

在src下创建``index.js``验证下此配置
{% codeblock src/index.js lang:js %}
console.log('Hello World!');
{% endcodeblock %}
运行命令
```
webpack -d
```
上面命令运行了webpack的开发模式，它会生成build/index_bundle.js和关联build/index_bundle.js的映射文件index_build.js.map

为了得到更多的交互我们创建index.html
{% codeblock index.html lang:html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React.js use jsx and es6</title>
</head>
<body>
    <div id="app"></div>
    <script src="bundle/index_bundle.js"></script>
</body>
</html>
{% endcodeblock %}
现在我们打开index.html，会在浏览器的console中看到输出

NOTE: [html-loader](https://github.com/webpack/html-loader)允许我们自动生成上述index.html文件

# 激活babel
jsx和es6让react能让有更高的开发效率，但是目前几乎所有浏览器都不支持。

因此我们需要个工具来转换他们，令到浏览器能识别，它就是[babel](http://babeljs.io/)


{% codeblock 转换流程 %}

>>>>>>>>>>>>>>>>>>          >>>>>>>>>>>>>>>>>>       >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>  <p>Hello</p>  >  ===>    >  babel-loader  >  ===> >  React.createElement('p', ...)  >
>>>>>>>>>>>>>>>>>>          >>>>>>>>>>>>>>>>>>       >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
     index.js                    webpack                       index_bundle.js

{% endcodeblock %}

安装babel
```
npm i babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
```
``babel-preset-es2015``和``babel-preset-react``是``babel-loader``的插件，用来转换es6和jsx

babel-loader是读取一个配置文件，所以能在配置文件中声明使用转换es6和react的插件
{% codeblock .babelrc lang:js %}
{
  "presets" : ["es2015", "react"]
}
{% endcodeblock %}


下一步我们使用babel-loader来打包文件
{% codeblock webpack.config.js lang:js %}
// Existing Code ....
var config = {
  // Existing Code ....
  module : {
    loaders : [
      {
        test : /\.js$/,
        include : APP_DIR,
        loader : 'babel'
      }
    ]
  }
}
{% endcodeblock %}

``loaders``接收一组loader，我们只使用了babel-loader。每个loader都会处理匹配``test``内容的文件。 这里我们配置了babel-loader处理.js后缀的文件。loader会在``include``指定的文件夹中遍历寻找符合``test``规则的文件。

目前所有配置都准备就绪了。让我们写点react吧

# Hello React
安装react和react-dom
```
npm i react react-dom --save
```

{% codeblock src/index.js lang:js %}
import React from 'react';
import {render} from 'react-dom';

class App extends React.Component {
  render () {
    return <p> Hello React!</p>;
  }
}

render(<App/>, document.getElementById('app'));
{% endcodeblock %}
如果还打开着index.html页面，刷新下会看到*Hello React!*

# webpack监听文件更改
每次更改都需要手动运行命令，这不是一个高效的工作流。我们可以很简单的使用以下命令来监听文件修改
```
webpack -d --watch
```
现在以watch模式运行着webpack，当文件更改或者删除的时候会自动生成bundel。我们可以试试修改src/index.js，然后刷新浏览器看效果

# 让一切"自动"起来
为了能更专心coding，我们希望能``所编即所见``。我们需要用到[webpack-dev-server](https://webpack.github.io/docs/webpack-dev-server.html)
{% codeblock 安装webpack-dev-server命令行工具 %}
npm i -g webpack-dev-server
{% endcodeblock %}
webpack-dev-server能提供server服务和监听修改刷新浏览器，让我们配置下它
{% codeblock webpack.coding.js lang:js %}
// Existing Code ....
var config = {
  output: {
    // Existing Code ....
    filename: '[name]',
    // webpack-dev-server 输出路径
    publicPath: '/build/'
  }
  // Existing Code ....
}
{% endcodeblock %}
``publicPath``是webpack-dev-server输出文件的路径，这只是个映射，映射到内存中的一部分，并不真的会在build下生成文件。

启动服务
```
webpack-dev-server
```
我们就可以通过[http://localhost:8080/webpack-dev-server/](http://localhost:8080/webpack-dev-server/)来访问到index.html文件。现在试下做下修改看浏览器有什么变化？
