---
layout: post
title: Javascript中字符串直接量与new String的区别
tags: Javascript
description: 分析Javascript中字符串直接量与new String的区别
date: 2015-05-29 00:03
---

我们都知道JS中使用字符串可以直接赋值字符串字面量，或者是`new String("foo")`。

但是他们到底有什么区别？我们先看一段代码：

    var a = "foo";
    var b = new String("foo");
    
    console.log(a); // foo
    console.log(b); // String {0: "f", 1: "o", 2: "o", length: 3, [[PrimitiveValue]]: "foo"}

    console.log(typeof a); // "string"
    console.log(typeof b); // "object"
    
    console.log(a == b);  // true
    console.log(a === b); // false

    console.log(a.charAt === b.charAt); // true

可以看出，变量`a`是一个`primitive值（原生值）`，一个primitive值不会拥有自己的属性与方法。

而`b`是一个primitive类型（primitive类型：Undefined, Null, Boolean, String, Number）的实例，它有一个值为实例化String类型时所传入的参数的内置属性`[[PrimitiveValue]]`，这个String的实例享有`String.prototype`上所有的方法。

但是，为何当我们访问`a.charAt`时能够访问到`String.prototype.charAt`呢？

* 因为当我们尝试访问一个primitive值的属性时，JS引擎内部会调用一个内置[[toObject]] 方法，将字面量的"foo"转为一个[[PrimitiveValue]]为"foo"的String对象，然后从其原型链中尝试查找需要访问的属性，使用结束后再释放掉这个String对象。 

<!--more-->

关于`[[toObject]]`算法规则如下图：

<img src="{{ site.url }}/downloads/images/string/ECMA-262-5.1-toObject.jpg" alt="内置函数[[toObject]]的规则" style="width:100%;box-sizing: border-box;padding: 2px;border: 1px solid #999;">

[[toObject]]的计算规则（[ECMA-262/5.1 Edition](http://www.ecma-international.org/ecma-262/5.1/#sec-9.9)）

该逻辑适用于所有primitive类型的字面值(Undefined, Null, Boolean, String, Number)。

而对于为何 `a == b` 为 `true` ， 可以参照我之前译的[《原创翻译）JS中的判断为真与相等（Truth, Equalit and JavaScript）》](http://xiaoyuze88.github.io/blog/2013/04/13/JS%E4%B8%AD%E7%9A%84%E5%88%A4%E6%96%AD%E4%B8%BA%E7%9C%9F%E4%B8%8E%E7%9B%B8%E7%AD%89%EF%BC%88Truth,%20Equalit%20and%20JavaScript%EF%BC%89/) 文中介绍的关于`==`操作符的类型转换规则。

当一个Object与一个String/Number类型比较时，会首先对Object调用[[[toPrimitive]]方法](http://www.ecma-international.org/ecma-262/5.1/#sec-9.1)，对于所有primitive类型的实例（上面介绍那5个），调用`[[toPrimitive]]`方法将直接返回自身`[[PrimitiveValue]]`的值，对于我们的例子，也就是返回了字符串"foo"。 最后`"foo" == "foo"`， 所以返回`true`。

更为细节的说明请查看ECMA Script规范中关于[primitive value](http://www.ecma-international.org/ecma-262/5.1/#sec-4.3.2)、[[[toPrimitive]]](http://www.ecma-international.org/ecma-262/5.1/#sec-9.1)、[[[toObject]]](http://www.ecma-international.org/ecma-262/5.1/#sec-9.9)的介绍。

全文完...