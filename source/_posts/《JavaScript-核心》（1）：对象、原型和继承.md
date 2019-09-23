---
title: 《JavaScript 核心》（1）：对象、原型和继承
date: 2019-09-23 23:37:52
tags: 翻译
---
本文译自：[JavaScript. The Core. - Dmitry Soshnikov](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/?source=post_page-----54102240a8b4----------------------)



## 对象
ECMAScript 是一门高度抽象化的面向对象语言，主要和*对象*打交道。虽然也有*原始值*，但是当需要的时候也会被转换为对象。

> 对象是一个*由属性组成的集合*，且有*单一的原型*。它的原型要么是一个对象，要么是 `null`。

我们来看一个简单的对象示例。一个对象的原型由内部的 `[[Prototype]]` 属性引用。但是在用户级的代码中，我们用 `__proto__` 来实现该引用，可以读作 'dunder proto' 。

```javascript
var foo = {
    x: 10,
    y: 20
}
```
我们会得到这样一种结构，它有两个显式的*自有*属性 `x`，`y`。还有一个隐式的 `__proto__` 属性，指向 `foo` 的原型。

{% asset_img prototype.png 图1.一个指向原型的对象 %}

原型有什么用？我们用*原型链*的概念来回答这个问题。

## 原型链
原型其实就是带有自有属性的对象。原型A指向它自身的原型——原型B，原型B再指向自身的原型——原型C，直到最终指向的原型为 `null`。这就称为*原型链*。

> 原型链是一个由对象组成的*有限*链，用来实现*继承*和*共享属性*。

假设我们有两个对象，它们只有很小一部分有区别，其余的部分都一样。显然，一个设计良好的系统会*重用*相似的功能/代码，而不是在每一个对象中重复一遍。在基于类的语言中，这种*代码重用*的语式称为*基于类的继承*——把相似的功能放入类 `A` ，再创建出继承自 `A` 且拥有自身额外小改动的 `B` 和 `C`。

ECMAScript 没有类的概念。不过代码重用的语式没有太大区别（在有些方面甚至比类更加灵活），通过*原型链*就可以实现。这种继承方式称作*委托继承*（或者更接近 ECMAScript 的说法是，*原型继承*）。

和类 `A` ，`B`，`C` 的例子相似，在 ECMAScript 中，你会创建对象 `a`, `b`, `c`。对象 `a` 中存放对象 `b` 和 `c` 的相同部分，`b` 和 `c` 中只存放它们自身的额外属性或方法。 

```javascript
var a = {
    x: 10,
    calculate: function (z) {
        return this.x + this.y + z
    }
}

var b = {
    y: 20,
    __proto__: a
}

var c = {
    y: 30,
    __proto__: a
}

// 调用继承方法
b.calculate(30) // 60
c.calculate(40) // 80
```

很简单对吧？我们可以看到 `b` 和 `c` 都能访问对象 `a` 中定义的 `calculate` 方法。这正是通过原型链来实现的。

原理很简单：如果一个属性或方法在对象自身中无法找到（比如对象没有*自有*属性），那么就尝试在原型链中寻找该属性/方法。如果在对象的原型中也找不到该属性，那么就在原型的原型中找，如此往复，直到遍历整个原型链（与基于类的继承做法完全一样，当解析一个继承*方法*时——我们也会找遍*类型链*）。第一个找到的同名属性/方法将被引用。找到的这个属性称作*继承*属性。如果在整个原型链中都找不到这个属性，则返回 `undefined` 。

注意，在调用继承方法时，其中的 `this` 绑定的是调用该方法的*原始*对象而不是该方法所在的原型对象。在上面的示例中 `this.y` 的值取自对象 `b` 和 `c` ，而不是 `a` 。不过 `this.x` 的值取自 `a` ，同样是通过*原型链*机制。

如果一个对象没有明确的指定其原型，则其 `__proto__` 默认指向原型 `Object.prototype` 。

原型 `Object.prototype` 自身也有 `__proto__` 属性，它指向原型链的*最后一环* `null`。

下图展示了对象 `a`，`b` 和 `c` 的继承结构。

{% asset_img prototype-chain.png 图2.原型链 %}

**注意：**
*  ES5中制定了另外一种原型继承的方法，使用 `Object.create` 函数：
    ```javascript
    var b = Object.create(a, {y: {value: 20}})
    var c = Object.create(a, {y: {value: 30}})
    ```
* 你可以在[这一章](http://dmitrysoshnikov.com/ecmascript/es5-chapter-1-properties-and-property-descriptors/#new-api-methods)中获取更多关于 ES5 API 的信息。

* ES6 已经将 `__proto__` 纳入标准，它可以用于对象的初始化。

我们经常会需要用到一些有*相同或相似声明结构*（比如相同属性）但*声明值*不同的对象。这种情况我们可以使用*构造函数*，它能用*特定的格式*创建对象。

## 构造函数
除了用特定格式创建对象，构造函数还有一个重要的作用 —— 它会为新创建的对象*自动指定一个原型*。这个原型就存放在 `ConstructorFunction.prototype` 属性里。

我们可以用构造函数重写前面例子中的对象 `b` 和 `c` 。这样，`Foo.prototype` 就扮演了对象 `a` 的角色：

```javascript
// 一个构造函数
function Foo(y) {
    // 可以用固定格式创建对象：
    // 他们有后生成的自有 'y' 属性
    this.y = y
}
// 同时 "Foo.prototype" 里存放着新创建对象的原型的引用,
// 所以我们可以用它来定义共享的/继承的属性或方法，于是和前面例子一样，我们创建：

// 继承属性 "x"
Foo.prototype.x = 10

// 还有继承方法 "calculate"
Foo.prototype.calculate = function (z) {
    return this.x + this.y + z
}

// 再来用“模板” Foo 创建对象 "b" 和 "c"
var b = new Foo(20)
var c = new Foo(30)

// 调用继承方法
b.calculate(30) // 60
c.calculate(40) // 80

// 来看看属性引用是否和预期的一样
console.log(
    b.__proto__ === Foo.prototype, // true
    c.__proto__ === Foo.prototype, // true

    // 同时 "Foo.prototype" 自动创建一个特殊属性 "constructor" ，
    // 指向构造函数本身；
    // 实例对象 "b" 和 "c" 可以透过委托找到该属性并且用它来查看它们的构造器。

    b.constructor === Foo, // true
    c.constructor === Foo, // true
    Foo.prototype.constructor === Foo, // true

    b.calculate === b.__proto__.calculate, // true
    b.__proto__.calculate === Foo.prototype.calculate // true
)
```
这段代码可以用下图的关系来表达：

{% asset_img constructor.png 图3.构造函数和对象间的关系 %}

这张图再次展示了每一个对象都有原型。构造函数 `Foo` 也有自己的 `__proto__` ，它指向 `Function.prototype` ，而 `Function.prototype` 又通过 `__proto__` 指向 `Object.prototype` 。`

`Foo.prototype` 就是 `Foo` 的一个显式属性。它是对象 `b` 和 `c` 的原型。

严格来说，如果要分类的话，构造函数和原型的组合可以称作“类”。实际上，像 Python 的*头等*动态类的实现和属性/方法这种解决方案完全一样。由此看来，Python 中的类其实是 ECMAScript 委托继承的一种语法糖。

**注意：**

* 在 ES6 中，类 “class” 的概念已经纳入标准，作为上面所述的构造函数的语法糖。由此看来，原型链成为了类继承的一种详细实现。
```javascript
// ES6
class Foo {
    constructor(name) {
        this._name = name;
    }

    getName() {
        return this._name;
    }
}

class Bar extends Foo {
    getName() {
        return super.getName() + ' Doe';
    }
}

var bar = new Bar('John');
console.log(bar.getName()); // John Doe
```
在 ES3 系列文章的第7章中可以找到这部分内容更完整和详细的解析。其中分为两个部分：[7.1.面向对象编程：概论](http://dmitrysoshnikov.com/ecmascript/chapter-7-1-oop-general-theory/)，在这部分中你可以找到各种面向对象编程的范例和语式，以及它们和 ECMAScript 的对比，还有 [7.2.面向对象编程：ECMAScript 实现](http://dmitrysoshnikov.com/ecmascript/chapter-7-2-oop-ecmascript-implementation/)，完全忠于 ECMAScript 中的面向对象编程实现。

现在我们已经了解了对象的基本面，继续来看*运行时程序执行*在 ECMAScript 中如何实现。这就是所谓的一个*执行上下文栈*，其中的每一个元素都可以抽象地用对象来代表。没错，ECMAScript 中几乎所有地方都用对象的概念运作。