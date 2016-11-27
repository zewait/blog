---
title: react-native笔记-基础篇
date: 2016-11-23 03:46:29
tags: react react-native
categories:
- react native
---

#Component
* 定义组件首字必须大小，用于区分普通html元素

# props
可用于定制组件，等同于htm中的attribute。
贯穿整个生命周期，不得做任何改变
<!--more-->
## 主要属性
* [defaultProps](https://facebook.github.io/react/docs/react-component.html#defaultprops)
* [proptypes](https://facebook.github.io/react/docs/react-component.html#proptypes)
* [props](https://facebook.github.io/react/docs/react-component.html#props)
  this.props.children该组件下的子组件

# state
跟props一样用于定制组件用，不同的是state可以被修改
在构造器初始化，当你想改变组件状态的时候调用setState，
组件重新render，除非shouldComponentUpdate()返回false
## 主要属性
* [state](https://facebook.github.io/react/docs/react-component.html#state)
## 主要方法
* [setState](https://facebook.github.io/react/docs/react-component.html#setstate)
  此方法不是重新设置state，而是合并

## 参考文档
* [State and Lifecycle](https://facebook.github.io/react/docs/state-and-lifecycle.html)
* [更好的控制状态Redux](http://redux.js.org/index.html)

# style
跟html的style差不多，不同的是background-color这些要写成backgroundColor。
style能是普通的js对象，也能是使用StyleSheet.create来创建。
更推荐的是使用StyleSheet.create在组件的同一个地方创建
## StyleSheet.create vs 普通js对象
* StyleSheet.create创建返回的是一个id，通过StyleSheetRegistry.getStyleByID获取该id
指向的对象
* 用StyleSheet.create创建代替每次都创建新的对象





