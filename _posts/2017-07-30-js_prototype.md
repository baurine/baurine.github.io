---
layout: post
title: "理解 JavaScript 的原型链"
description: ""
category: 
tags: [javascript, prototype]
---

参考：

1. [Javascript 王国的一次旅行，一个没有类的世界怎么玩转面向对象？](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513786&idx=1&sn=6f51ad314e3ef3e1575e032568477f3a)
1. [图解 Javascript 原型链](http://blog.rainy.im/2015/07/20/prototype-chain-in-js/)

   ![prototype]({{site.img_url}}/js-prototype.png)

上面这是 [ByVoid](https://github.com/byvoid) 画的一张图，理解了它，就理解了 js 的原型链。

假设 Foo / Bar 是构造函数：

    function Foo(name) {
      this.name = name
    }

    function Bar() {
      this.setAge = (age) => this.age = age
      this.showAge = () => console.log(this.age)
    }

    let foo = new Foo('foo')

总结一下，就是：

1. `__proto__` 属性属于对象，`prototpye` 属性属于构造函数。

    因此上例中，对象 foo 具有 `__proto__` 属性，但没有 `prototype` 属性。Foo 和 Bar 具有 `prototype` 属性。

        foo.__proto__  // 有值
        foo.prototype  // undefined

        Foo.prototype  // 有值
        Foo.__proto__  // ?? 有值还是 undefined，留个悬念

1. 对象的 `__proto__` 指向其构造函数的 `prototype`。

        foo.__proto__ === Foo.prototype  // true

   `__proto__` 总是指向 `prototype`，而 `prototype` 总是被 `__proto__` 所指向。

1. `prototype` 也是对象，因此它也有自己的 `__proto__` 属性。根据上面所言，`__proto__` 指向对象自身构造函数的 `prototype` 属性，而这里的 `Foo.prototype` 的构造函数是谁呢？因为 `Foo.protype` 是一个普通对象，Object，它是通过 `new Object()` 创造出来的，所以它的构造函数是 Object，所以它指向 `Object.protype`。

        Foo.prototype.__proto__ === Object.prototype  // true

   又因为 `foo.__proto__ = Foo.prototype`，所以，可得

        foo.__proto__.__proto__ === Object.prototpye  // true

   `Object.prototype`也是对象，那么它的 `__proto__` 又是多少呢，规定 `Object.prototype.__proto__ = null` (呃，有点特殊)，因此

        foo.__proto__.__proto__.__proto__ === null  // true

   一般来说，一个普通的自定义的构造函数生成的对象，通过向上查找三级 `__proto__` 就能得到 null 值。

1. 如上所示，因为对象是通过 `__proto__` 这条链条向上查找方法和变量，而 `__proto__` 指向的是各级构造函数的 `prototype` 属性。因此，为构造函数添加原型方法时，这些方法必须绑定在 `prototype` 属性上，这样才能被对象的 `__proto__` 所查找到。

        function Foo(name) {
            this.name = name
        }

        let foo = new Foo('foo')

        Foo.prototype.showName = function() {
            console.log(this.name)
        }

        foo.showName() // 'foo'

1. 上面说到，一般来说，一个普通的自定义的构造函数生成的对象，向上查找三级 `__proto__` 就得到了 null，它们的链条只有三级。如果我们想实现继承关系，比如让 Foo 继承自 Bar，让 foo 能用上 Bar 中的方法，那要怎么办呢。方法就是手动修改这条默认的 `__proto__` 链条。我们要让 `Foo.prototype.__proto__` 不再指向默认的 `Object.prototype`，而是指向 `Bar.prototype`。

        Foo.prototype.__proto__ = Bar.prototype

   那么就可以得到：

        foo.__proto__.__proto__ = Bar.prototype

   这样，foo 就可以用上 Bar 中的方法了:

        foo.setAge(18)
        foo.showAge()

   呃呃，但是实验之后，证明我还是太天真了，上面通过强行改变 `prototype.__proto__` 的方法并不管用，原因我想可能是因为 `__proto__` 这个值是一个内部值，并不是一个公开的值，从它的前缀是 2 个下划线可知，因此它可以被外部所读取，但不能被外部所修改，只能在内部被修改。为了达到相同的效果，该怎么做呢。前面我们得知：

        foo = new Foo('foo')
        // 得到 foo.__proto__ --> Foo.prototype

   那么：

        Foo.prototype = new Bar()
        // 得到 Foo.protype.__proto__ --> Bar.prototype

        Foo.protoype.__proto__ === Bar.prototype  // true

1. 现在我们知道了，`prototype` 也有一个 `__proto__` 属性，那它还包含什么值呢？我们在 chrome dev tool 中打印一下它。

        function Foo(name) {
            this.name = name
        }

        Foo.prototype.showName = function() {
            console.log(this.name)
        }

        > Foo.prototype
        < Object {showName: function, constructor: function}
            showName: function ()
            constructor: function Foo(name)
            __proto__: Object

   除了我们已知的 `__proto__` 和后来附加的原型方法 `showName`，还有一个属性，叫 `constructor`，从名字上看，它应该是表示构造函数，而它的值正是构造函数 `function Foo(name){...}`。所以得到：

        Foo.prototype.constructor === Foo  // true

1. 最后我们来看一下这个 Function 是个啥玩意。Foo / Object / String，它们是构造函数，用来构造对象。但是呢，从另一个角度来看呢，其实它们也是一种对象，它们是通过 `new Function() { execution code }` 生成的对象。(所以说，Function 才是终极大 Boss，是火种。)

        function Foo() {
            ...
        }

        // 等效于
        var Foo = new Function() {
            // execution code
        }

   因为它们也是对象，所以它们也有 `__proto__` 属性，因为它们的构造函数是 Function，所以它们的 `__proto__` 指向 `Function.prototype`。

        Foo.__proto__ === Function.prototype  // true
        Object.__proto__ === Function.prototype  // true

   因为 `Function.prototype` 是普通对象，所以很好理解，它的 `__proto__` 指向 `Object.prototype`。

        Function.prototype.__proto__ === Object.prototype  // true

   最后这一点就很有趣了，因为 Function 是构造函数，所以它和其它所有一切构造函数一样，它的 `__proto__` 将指向 `Function.prototype`，这是唯一一个 `__proto__` 和 `prototype` 相等的对象，突显了它造物主的特殊身份。

        Function.__proto__ === Function.prototype  // true

和 ruby 的对象模型联系起来看。

当得知 js 中一个普通的构造函数实际也是一个由顶层构造函数 (Function) 生成的对象时，我很震惊，因为想到和 Ruby 的对象模型简直是异曲同工。

在 Ruby 中，比如这样一段代码：

    class Foo
        def initialize(name)
            @name = name
        end

        def show_name
            puts @name
        end
    end

    foo = Foo.new('foo')
    foo.show_name  // 'foo'

Foo 在这里是一个类，它用来生成实例对象，但同时，它也是一个对象，它是由 Class 这个顶层类生成的对象。`class Foo` 表面上看是在声明一个类，实际上它等效于 `Foo = Class.new( execution code )`，它表示，Foo 是 Class 的实例对象。

    class Foo
        ...
    end 

    # 等效于 
    Foo = Class.new(
        # execution code
        ...
    )

再和 JavaScript 对比一下：

    function Foo() {
        ...
    }

    // 等效于
    var Foo = new Function() {
        // execution code
    }

是不是相似极了？惊不惊喜，意不意外。
