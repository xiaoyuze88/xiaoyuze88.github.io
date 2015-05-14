---
layout: post
title: 几句话整理下Javascript原型链与继承
tags: Javascript
description: 对Javascript原型链与继承简单总结
date: 2015-05-15 23:58
---

很多东西久了不碰就生疏了，今天突然发现在NodeJS中直接用prototype继承不起作用，只能用util.inherits，于是想比较下浏览器中prototype继承与Node中util.inherits的区别。

但却突然发现prototype继承有点生了，在这里简单整理下思路。

<!--more-->

首先简单介绍下JS中的原型链，与其他语言不同，JavaScript是基于原型链继承的。 

#### 什么是原型链呢？ 

发现很多介绍原型链的文章都是先讲prototype，我这里用另外一个思路来介绍，我们先讲`__proto__`。

JS中，每个对象（JS中所有变量都是对象除了`null`和`undefined`）都有一个私有的只读变量`__proto__`。

当我们调用一个对象的某个方法或者属性时，无法在这个对象本身上查找到，那么就会去查找这个`__proto__`变量，这个`__proto__`所指向的，就是这个对象的原型。

如果仍然在`__proto__`无法找到，那么就会继续查找`__proto__`的`__proto__`，这个过程可以叫做遍历原型链，直到原型链的顶端，也就是Object.prototype，如果整个原型链上都找不到，那么返回`undefined`。

这整个链状向上遍历的关系，就叫做原型链，JS就是基于这个特性实现的继承。

不详细说了，具体关于原型链的介绍可以上网搜到很多，或者可以查看MDN上[关于原型链与继承的介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)。

下图就是对原型链的一个简单的展示（Object.prototype指向的一个空对象`{}`）：

<img src="{{ site.url }}/downloads/images/prototype/prototype.jpg" style="width:400px;box-sizing: border-box;padding: 2px;border: 1px solid #999;" alt="原型链的简单例子">

还要注意的是，这个`__proto__`是怎么指定的，再看一个例子：

当我们有一个构造函数Parent，它的原型`prototype`上有一个方法`sayHi`，当我们new一个Parent的实例parent后，发生的事情就是: `parent.__proto__指向了Parent.prototype`。

    function Parent() {
        this.name = "I'm parent";
    }

    Parent.prototype.sayHi = function() {
        console.log('Hi!');
    }

    var parent = new Parent();

    console.log(parent.__proto__);  // Parent.prototype

而通过我们上面介绍的往上遍历`__proto__`的规则，当我们调用`parent.sayHi`时，会首先查找parent自身，然后查询`__proto__`，然后从Parent.prototype中找到sayHi这个方法。

我们简单的通过hasOwnProperty即可知道sayHi到底是不是在parent身上。

    parent.hasOwnProperty('sayHi');             // false
    Parent.prototype.hasOwnProperty('sayHi');   // true

### 如何实现继承？

假设我们现在有另外一个构造函数Child，它啥都没有，现在我们需要让它的实例继承得Parent的所有属性和方法。

    function Child() {}

考虑到上面说到的`prototype`和`__proto__`的关系，我们可以简单的这样做：

    Child.prototype = new Parent();
    Child.prototype.constructor = Child;

这样做的效果是什么呢？根据上面的规则，相当于：

    Child.prototype.__proto__ == Parent.prototype;

这个时候我们new Child时，

    var child = new Child();

他们的关系就变成这样了：
    
    child.__proto__ == Child.prototype;
    child.__proto__.__proto__ == Child.prototype.__proto__ = Parent.prototype;

根据向上遍历原型链的规则，当我们访问child.sayHi时，会依次查询`child自身`、`Child.prototype`、`Parent.prototype`， 最终找到`sayHi`方法。

（注意的是，整个原型链是动态的，也就是无论Child已经生成了多少实例，更新原型链上的属性、方法，将使得所有实例中的对应对象也发生改变。实际上，他们指向的同一个引用--引用传递）

当然要简单的实现遍历原型链能找到Parent.prototype.sayHi，还可以直接这样：

    Child.prototype = Parent.prototype;

但是这样的缺点很明显，一旦你改写Child.prototype中的值，Parent.prototype也将改变。

还可以利用ES5的Object.create()方法串起原型链：

    Child.prototype = Object.create(Parent.prototype);

但是这样做的缺点是，无法继承Parent构造函数中定义的属性（如Parent中的`this.name`），因为并不是将Parent的实例链上Child.prototype。

可以通过这样解决：

    function Child() {
        Parent.call(this);
    }

### 总结

#### 一、实现继承的三种方法：

1. `Child.prototype = new Parent();`

2. `Child.prototype = Object.create(Parent.protype);` + `Parent.call(this);`

3. `Child.prototype = Parent.prototype`;

#### 二、搞清楚向上遍历原型链的规则（向上遍历`__proto__`），一切都将迎刃而解。

全文完.