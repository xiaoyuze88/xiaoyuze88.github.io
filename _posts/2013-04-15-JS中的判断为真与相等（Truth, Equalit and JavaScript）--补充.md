---
layout: post
title: （原创翻译）JS中的判断为真与相等（Truth, Equalit and JavaScript）--补充
tags: JavaScript
date: 2013-04-15 20:30
---

补充一下我的前一篇文章[（原创翻译）JS中的判断为真与相等（Truth, Equalit and JavaScript）]({% post_url 2013-04-13-JS中的判断为真与相等（Truth, Equalit and JavaScript） %})。


文中其他地方都好理解，主要是当 `object` 与其他类型做 `==` ， `+` 等操作时，强制转换类型时调用的 `toPrimitive` 方法多多少少会让人比较迷惑。

我又翻阅了另一篇资料[http://www.adequatelygood.com/Object-to-Primitive-Conversions-in-JavaScript.html](http://www.adequatelygood.com/Object-to-Primitive-Conversions-in-JavaScript.html)

中间详细讲解了 `toPrimitive` 的运作过程。

我在这里就凭自己的理解总结下，有兴趣的朋友可以去原文中阅读更多内容。

（另外不得不吐槽下，JS真的是一门很神奇的语言，是那么的灵活，那么的吸引人...但同时也是那么的蛋疼。）

<!--more-->

----------
### 正文开始
----------

首先一步步来，JS中的基本类型与引用类型，不是很清楚的同学可以Google下或者翻看我以前的文章

[原创翻译--JS中的判断为真与相等（Truth, Equalit and JavaScript）]({{site.url}}/blog/2013/04/13/%E5%8E%9F%E5%88%9B%E7%BF%BB%E8%AF%91--JS%E4%B8%AD%E7%9A%84%E5%88%A4%E6%96%AD%E4%B8%BA%E7%9C%9F%E4%B8%8E%E7%9B%B8%E7%AD%89%EF%BC%88Truth,%20Equalit%20and%20JavaScript%EF%BC%89/)

当我们对一个对象与一个基本类型使用 `+` 操作符时，它会对对象使用内置的 `ToPrimitive` 方法。

这个方法会优先调用对象的 `valueOf` 方法，如果不返回基本类型，再调用对象的 `toString` 方法，如果还不能获得基本类型，报 `TypeError错误` 。

#### toString方法

所有对象都从Object.prototype中继承了一个toString方法，它返回 `[object Object]` 字符串，我们可以重写这个方法，包括直接写在每个实例中或者写进它的prototype。

原文中写法如下：

    function population(country, pop) {
        return {
            country: country,
            pop: pop,
            
            toString: function () {
                return "[Population " + 
                    "\"" + country + "\" " +
                    pop +
                "]";
            }
        }
    }

#### toValue方法

同样的，所有对象也从Object.prototype中继承了valueOf方法，它返回这个对象本身。

同样的，我们经常会将它重写成我们希望的结果，返回一个基本类型，好让它能够工作在 `+` 操作符中。

原文中写法如下：

    function population(country, pop) {
        return {
            country: country,
            pop: pop,
            
            toString: function () {
                return "[Population " + 
                    "\"" + country + "\" " +
                    pop +
                "]";
            },
            
            valueOf: function () {
                return pop;
            }
        };
    }

我们看一个例子，它同时重写了foo对象的toString与valueOf方法：

    var foo = {
        toString: function () {
            return "foo";
        },
        valueOf: function () {
            return 5;
        }
    };

    alert(foo + "bar"); // 5bar
    alert([foo, "bar"].join("")); // foobar

可以看到，当使用 `+` 操作符导致对对象强制转换为基本类型时，优先调用的是 `valueOf` 方法。

值得注意的是，当你重写这两个方法的时候，JS并不会检查返回值的类型，只要它们是一个基本类型。

所以你可以这样写：

    var foo = {
        toString: function () {
            return 5;
        },
        valueOf: function () {
            return "foo";
        }
    };
    alert(foo.toString() + 1); // 6 (bad!)
    alert(foo + 1); // "foo1" (no good!)
    alert(+foo); // NaN (the worst!)

可以看到，如果将valueOf或者toString重写为返回其他类型的值，非常容易导致错误！

所以，请不要在重写对象的toString方法时返回不是字符串类型的返回值！

#### 总结下两篇文章吧...其实啰里啰嗦的，总结起来很简单：

- 在if()等条件判断语句中

为true的有：（true,"potato",36,[1,2,4]，{a:16}），其中对象永远为true，包括{}，

为false的有： (false,0,"",null and undefined)。

- 在判断相等“==”操作符中

除了null和undefined相互相等外，其他大部分类型都是先强制转为了数字（toNumber），再比较。

如果操作符中出现了对象，对对象使用ToPrimitive方法，既优先调用valueOf，不返回基本类型的话再调用toString(除了Date对象是优先调用toString)。

转为基本类型后再做比较。NaN不与任何东西相等，包括自己。

- 严格判断相等“===”中

null和undefined永远不会===其他类型，NaN不会===任何东西，包括自己。

对对象使用===时，只有两个对象指向同一实例时才能相等，哪怕是一模一样的两个实例都不===。

- 在使用“+”操作符时

根据《Javascript权威指南》上的说法，当两边都是数字或者都是字符串时，结果显而易见。

对于其他情况，要进行类型转换，+号的转换规则优先考虑字符串连接，如果其中一个值为字符串，那么另外一个转为字符串。如果都不是类字符串(string-like),则会进行算数加法。

如果其中一个为对象，就会首先根据上面所提的toPrimitive方法转为基本类型，转换完后，再按上面的规则进行操作。

一些例子：

    "1" + 2// "12"

    1+{} // "1[object Object]"

    true + true// 2

    2 + null // 2

    2 + undefined // NaN   ，注意  undefined转换为数字是NaN，而null转为数字是0

