---
title: underscore源码学习二
date: 2018-08-08 11:31:00
tags: [underscore]
categories: underscore
---
### 实现面向对象调用
underscore支持函数式调用，也支持面向对象式调用
```javascript
// 函数式
_.each([1,2,3], function (item) {
	console.log(item)
})
// 面向对象式
_([1,2,3]).each(function (item) {
	console.log(item)
})
```
这是如何实现的？
首先看一下underscore each 函数，这一节中并不关心内部的实现，所以只看函数结构，由源码可见，each支持函数式调用，那么面向对象怎么支持？
```javascript
_.each = _.forEach = function(obj, iteratee, context) {
	...
    return obj;
  };
```
#### _ 函数
接下来看看_的源码定义
```javascript
  var _ = function(obj) {
    if (obj instanceof _) return obj;
    if (!(this instanceof _)) return new _(obj);
    this._wrapped = obj;
  };
```
原来 _ 是函数，js中函数也是对象，因此underscore中的其它方法可以挂在 _ 上，我们来仔细研究下 _ 函数。
_([1,2,3]) 等价于 `new _([1,2,3])`, 这个新实例我们给它个名字：var instance =  new _([1,2,3]). 这里出现了一个关键点：**new** ，构造函数和新实例，由此可以联想到 `原型链、原型对象`
![](/images/underscore.png)
如上图，_ 作为构造函数new了一个新实例，新实例原型链指向原型对象，因此new出的新实例可以调用原型对象上的方法。那么定义在 _ 上的函数如何挂载到其原型对象上的呢？这就要看mixin方法了
#### mixin
```javascript
  _.mixin = function(obj) {
    _.each(_.functions(obj), function(name) {
      var func = _[name] = obj[name];
      _.prototype[name] = function() {
        var args = [this._wrapped];
        push.apply(args, arguments);
        return func.apply(_, args)); // 这里先不管链式调用
      };
    });
    return _;
  };

 _.mixin(_);
```
我们看下原型对象上的each函数是怎样的
```
>_.prototype.each
>ƒ () {
        var args = [this._wrapped];
        push.apply(args, arguments);
        return func.apply(_, args);
      }
```
先看第一行 var args = [this._wrapped]; 回到  _  函数，最后一行 this. _wrapped = obj; 将传入的对象（这里是[1,2,3]）放在新实例instance. _wrapped 属性中，原面向对象调用方式我们转化为：
```javascript
var instance =  new _([1,2,3])
instance.each(function (item) {
	console.log(item)
})
```
instance.each() 其实调用的是 _ 原型对象上的each 函数即_.prototype.each。
上面的函数再转化一下
```javascript
var instance =  new _([1,2,3])
instance.each(function (item) {
	console.log(item)
})
// 进入each函数栈
ƒ (function (item) {
	console.log(item)
}) {
        var args = [this._wrapped];// this._wrapped = [1,2,3]
        push.apply(args, arguments); // arguments = function (item) {console.log(item)}
        return func.apply(_, args);// args = [[1,2,3], function (item) {console.log(item)}]
      }
      // func = function(obj, iteratee, context) {
			...
		    return obj;
		  };
```
运行到此，obj = [1,2,3], iteratee = function (item) {console.log(item)}]，实现了面向对象调用。