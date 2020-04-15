---
title: 实现一个jQuery的API
date: 2020-04-16 01:56:51
tags:
---
最近学到了 DOM —— Document Object Model（文档对象模型）。它通过映射文档的结构，将 web 页面与脚本或编程语言关联起来。DOM 用逻辑树来表示一个文档，每个分支的末端是一个节点（node），每一个节点（node）包含着对象。使用 DOM 的 API 可以程序化访问这个树，你可以用他们操纵文档的结构，样式或内容。

jQuery 是一个高效简洁的 JavaScript 库，它通过对 DOM 节点进行包装及对一些通用重复的功能进行封装，使得操作 DOM 变得简单高效，而且大大缓解了早期各浏览器之间不兼容的问题。

### 如何实现 jQuery 的 API
这次的作业内容是：
```javascript
window.jQuery = ???
window.$ = jQuery

var $div = $('div')
$div.addClass('red') // 可将所有 div 的 class 添加一个 red
$div.setText('hi') // 可将所有 div 的 textContent 变为 hi
```
需要将 ？？？的内容补全。
我们先从 addClass() 和 setText() 这两个函数切入。
```javascript
// addClass()
function addClass(nodeList, className) {
  for(var i = 0; i < nodeList.length; i++) {
    nodeList[i].classList.add(className)
  }
}

// setText()
function setText(nodeList, text) {
  for(var i = 0; i < nodeList.length; i++) {
    nodeList[i].textContent = text
  }
}
```
以往当我们需要调用这两个函数实现想要的效果时，我们是这样做的：
```javascript
// 先获取要操作的元素
var divList = document.querySelectorAll('div')
// 再将其带入函数执行
addClass(divList, 'red')
setText(divList, 'hi')
```
或者用命名空间的方式，将你写的很酷的一些函数打包在一个自己命名的对象里，增加辨识度：
```javascript
var Yui = {
  addClass: function(nodeList, className) {
    for(var i = 0; i < nodeList.length; i++) {
      nodeList[i].classList.add(className)
    }
  },
  setText: function(nodeList, text) {
    for(var i = 0; i < nodeList.length; i++) {
     nodeList[i].textContent = text
    }
  }
} 

Yui.addClass(divList, 'red')
Yui.setText(divList, 'hi')
```

*但是上面这两种函数调用的方法都有点反直觉...*

我们调用函数的方式一般都是 **对象.函数名(参数)** 这种形式。而上面两个函数之所以不能这么用的原因是：
我们获取到的元素 `divList` 它的原型是 `NodeList` ，在他的原型链上是没有 addClass 和 setText 这两个函数的，`NodeList` 没有提供，我们也并没有把这两个函数添加到它的原型链上。
不仅是 `divList` ，在开发的过程中我们需要获取各种各样的元素对象，他们的原型链都各不相同。我们如果想对他们执行原型链上没有的方法，就不得不像上面一样先写一些函数，再把他们作为参数传入，然后使用他们现有的 API 组合以达到想要的功能。

有什么办法让我们可以直接对所有获取到的元素统一用 `divList.addClass('red')` 这种形式来调用函数么？
#### 方案1：扩展现有原型的 API
我们都知道所有的对象原型链的最后一环都是 Object ，那么按理说只要我们在 Object.prototype 上添加 addClass 和 setText 这两个函数，那么获取到的任何元素都可以直接调用它们。
```javascript
// 在 Object 原型上添加 addClass
Object.prototype.addClass = function(className) {
  for(var i = 0; i < this.length; i++) {
    this[i].classList.add(className)
  }
}
// 在 Object 原型上添加 setText
Object.prototype.setText = function(text) {
  for(var i = 0; i < this.length; i++) {
    this[i].textContent = text
  }
}

// divList 就可以直接调用 addClass 和 setText 了
divList.addClass('red')
divList.setText('hi')
```
但是这样做有很多缺陷，比如有可能会不小心起了和别的 API 同样的名字，这样就会覆盖原有的 API 或者别人写的方法。
#### 方案2：把获取到的元素包装成一个新的对象

假设我们在 `window` 上添加一个函数 jQuery，用它来接收我们获取的对象 A，然后返回一个重新包装好的对象 \$A。在函数 jQuery 中，我们可以预定义各种常用但是写起来繁琐的功能函数，添加在返回的对象 \$A 上，这样 \$A 就可以直接使用这些函数。
```javascript
var divList = document.querySelectorAll('div')
window.jQuery = function(divList) {
  //返回一个重新包装的对象
  return {
    addClass: function(className) {
      for(var i = 0; i < divList.length; i++) {
        divList[i].classList.add(className)
      }
    },
    setText: function(text) {
      for(var i = 0; i < divList.length; i++) {
        divList[i].textContent = text
      }
    },
    xxx: f(),
    xxx: f(),
    ...
  }
}
// 调用 jQuery 返回一个新包装的对象
var $divList = jQuery(divList)
$divList.addClass('red')
$divList.setText('hi')
```
只要把 jQuery 中的函数扩展得足够多和强大，以后我们就可以直接引入它，然后随心所欲地使用它提供的功能。

#### 再对 jQuery 做些改进
上面这样写有些缺陷，jQuery 接收的参数只能是特定类型的元素，而且每次调用 jQuery 之前还得先获取到元素，十分麻烦。我们希望只需一个元素 id，一个类名，或者标签名就能直接得到 jQuery 返回的包装好的对象 \$A。只要在 jQuery 内部做些改进：
```javascript
window.jQuery = function(nodeOrSelector) {
  // 创建一个新的空对象，作为存放包装后对象的容器
  var nodes = {}
  // 判断传入的元素类型，若是字符串，则表明希望用该字符串作为选择器参数获取相应元素
  if (typeof nodeOrSelector === 'string') {
    var temp = document.querySelectorAll(nodeOrSelector)
    // querySelectorAll() 方法返回的是 NodeList 类型的对象，有一些自带的属性和方法。
    // 我们希望 nodes 保持纯净，里面只有元素列表和我们自己的方法
    for (var i = 0; i < temp.length; i++) {
      nodes[i] = temp[i]
    }
    nodes.length = temp.length
    // 经过上面的重构，nodes 就是一个纯净的类数组对象
    // 里面只有对象列表和 length ，原型链直连 Object，没有多余的属性和方法

    // 如果传入的元素就是一个单一的节点，就直接添加到 nodes 的 0 键上
  } else if (nodeOrSelector instanceof Node) {
    nodes = {0: nodeOrSelector, length: 1}
  }
  
  // 然后就可以在 nodes 上添加 setClass 和 setText 等各种函数了 
  nodes.addClass = function(className) {
    for(var i = 0; i < divList.length; i++) {
      divList[i].classList.add(className)
    }
  }

  nodes.setText = function(text) {
    for(var i = 0; i < divList.length; i++) {
      divList[i].textContent = text
    }
  }

  // 最后返回包装好的 nodes 对象
  return nodes
}

  // 再为 jQuery 添加一个缩写 $
window.$ = jQuery
```

这样就实现了一个简易版的 jQuery。