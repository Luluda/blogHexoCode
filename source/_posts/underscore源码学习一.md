---
title: underscore源码学习一
date: 2018-07-25 21:10:00
tags: [underscore]
categories: underscore
---
## 前言
第一篇记录源码阅读的博客，underscore对我说比较合适，相对于vue等框架更容易入手。希望坚持下去把这一系列写（ziliaozhenghe）完。
underscore v1.9.1
## 不同环境下的全局对象
underscore把封装的方法挂在全局对象的_下，首先获取全局对象赋值给root
```javascript
// 旧版本root值
var root = this;
// 新版本root值
var root = typeof self == 'object' && self.self === self && self ||
            typeof global == 'object' && global.global === global && global ||
            this ||
            {};
```
为什么新版本的underscore不直接用this?
```javascript
"use strict";
(function () {
    console.log(this)
})()
```
上面代码输出是undefined，原因是**严格模式下，在全局作用域调用的函数中this指向undefined**
严格模式下this的指向可以参考博客[JavaScript严格模式下的this的几种指向](https://segmentfault.com/a/1190000010108912)
### self与window
{% blockquote 对于web页面，在默认状况下，下面4个写法都是等同的 https://www.zhangxinxu.com/wordpress/2017/07/js-window-self/ 博客%}
{% endblockquote %}
```javascript
window === self                  // true
window.window === window.self    // true
window.self === self             // true
window.window === self           // true
```
说实话在看underscore之前完全不知道有self，既然window与self都指向全局作用域，为什么源码中要用self不用this？最常见的就是用在Service Workers或者Web Workers中
#### Service Workers, Web Workers下的self
{% blockquote %}
无论是Web Workers或者说是Service Workers，本质上都是开启了另外的线程。
但是，Workers开辟的新线程是没有“窗体”这个概念的，都是在浏览器背后悄悄运行的线程，没有窗体的概念也就意味着没有window对象。
{% endblockquote %}
而此时self仍指向全局作用域，这就是为什么underscore源码中用self的原因（兼容Workers下的全局作用域）
关于Service Worker可以参考博客[深入了解 Service Worker ，看这篇就够了](https://zhuanlan.zhihu.com/p/27264234)
[Demo](https://github.com/mqyqingfeng/Blog/tree/master/demos/web-worker)这是大神写的webworker中打印全局对象demo

知道了用self代替window的原因，那么
```javascript
typeof self == 'object' && self.self === self && self
```
在返回self前要判断typeof和self.self，这样写的原因是什么呢？个人认为这是为了一定程度保证此self即我们想要的全局对象。self、window、global很有意思，self.self.self.self.self === self 为true,还可以继续往下写， window和global同理。这个实现其实很简单：
```javascript
self.self = self
```
但**self不是js的保留关键字**，chrome 67.0.3396.99版本实验，self可以被改写！虽然增加了两个条件并不能保证self就是全局作用域，算是加了层保障。另`|| this`也可以一定程度避免self被改写（但不符合typeof self == 'object' && self.self === self）情况下可以获取到全局作用域。
self不是js保留关键字，但为window保留关键字，所以要警惕以后变量名要避开各种保留关键字才不会出现诡异的问题。
### this
上面说的修改self算是作死会出现的问题，增加了this最主要的原因应该是
{% blockquote %}
在 node 的 vm 模块中，也就是沙盒模块，runInContext 方法中，是不存在 window，也不存在 global 变量的。
但是我们却可以通过 this 访问到全局对象
{% endblockquote %}

### {}
那么最后为什么还会有`|| {}`呢？
{% blockquote %}
因为在微信小程序中，window 和 global 都是 undefined，加上又强制使用严格模式，this 为 undefined，挂载就会发生错误
{% endblockquote %}

underscore第一句代码解读就整理了一大块，也是体会了源码的博大精深。后面继续说说自己在underscore刚学到的两个写法
## void 0
源码中频繁出现void 0例如：
```javascript
if (context === void 0) return func;
```
那么void 0究竟是什么？实践一下:
```javascript
void 0
//返回 undefined
```
既然是undefined为什么不直接用undefined？
最主要的原因是：**undefined 在低版本 IE 中能被重写**，同时在局部作用域中，还是可以被重写
```javascript
(function () {
	var undefined = 1;
	console.log(undefined);
})();
// 输出1
```
**void**是js的运算符
{% blockquote %}
The void operator evaluates the given expression and then returns undefined.
{% endblockquote %}
就是说void后接任意表达式都返回undefined, 而0是所有表达式最简短的了。

## == null
与void 0类似，源码中经常可以看到
```javascript
if (value == null) return _.identity;
```
这样的用法，一开始很疑惑，不是应该尽量避免使用双等号==么？为什么源码中还频繁用到了？让我们来看看双等号转换法则：
![](/images/==.png)
如果A == null , A只能是null 或 undefined !
对比以下写法
```javascript
value === null || value === undefined
value == null
```
源码中的写法简洁很多~

## 总结
源码的阅读刚刚开始，整理第一篇博文更感觉任重道远。每次学习源码都感觉开了眼界，原来全局对象获取会有这么多坑，原来void 0可以更安全地代替undefined, 原来==null可以让代码更简洁

参考文档：
[underscore 系列之如何写自己的 underscore](https://github.com/mqyqingfeng/Blog/issues/56)
[为什么用「void 0」代替「undefined」](https://github.com/hanzichi/underscore-analysis/issues/1)
