---
layout: post
title: （原创翻译）JS中的判断为真与相等（Truth, Equalit and JavaScript）
tags: JavaScript
---

以前也碰到过几次 `if(x)` 的时候出现预想外的错误，也翻看过许多相关资料，但是一直印象不深。

今天又碰到 `if({})` 为true的问题，找到一篇深入介绍JS中判断相等判断true原理的文章，正好闲来无事，就动手翻译一下吧。一是加深印象，而是练练英语。翻译的不好，大家将就看哈，实在觉得我的表达看不懂的可以参考原文！

[原文链接](https://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/)

<!--more-->

----------
### 译文开始
----------

哪怕你已经不是一个JS菜鸟了，你可能还是容易被下面的代码所迷惑

    if ([0]) {
        console.log([0] == true); //false
        console.log(!![0]); //true
    }

或者

    if ("potato") {
        console.log("potato" == false); //false
        console.log("potato" == true); //false
    }

好消息是，有一个所有浏览器都遵循的判断标准。一些人可能会告诉你要害怕强制类型转换，并且应该尽量避免使用它们。但是我希望说服你，强制类型转换是一种杠杆（或者理解为双刃剑？），你要做到的不是避免使用它们，你至少应该去理解它们。

x是true吗？ x==y吗？在JS中判断为真或者判断相等的问题，主要出现在这三种情况下：

1. 条件语句和操作符（if,三元运算符，&&,||等）
2. 相等判断符 `==`
3. 严格相等判断符 `===`

下面就让我们看看在这三个方面中各自会发生什么吧... 

#### 条件语句

在JS中，所有条件语句和操作符都遵循相同的规则。我们将用IF语句来做范例。

在IF语句中，它将利用一个抽象方法ToBoolean，强制将判断条件的结果转换为一个boolean类型的结果。由ES5 定义的ToBoolean方法的算法为：

<table cellpadding="0" cellspacing="0" style="border-width: 0px 0px 1px; border-bottom-style: solid; border-bottom-color: rgb(76, 76, 76); font-family: 'Droid Sans', Arial, sans-serif; font-size: 14px; margin: 0px 0px 1.4285714285em; outline: 0px; padding: 0px; vertical-align: baseline; border-spacing: 0px; width: 602px; color: rgb(76, 76, 76); line-height: 23px;">
    <tbody style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 110px; background-color: rgb(170, 170, 170); height: 24px;">
                Argument Type</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; background-color: rgb(170, 170, 170); height: 24px;">
                Result</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Undefined</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>false</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Null</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>false</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Boolean</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result equals the input argument (no conversion).</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result is&nbsp;<b>false</b>&nbsp;if the argument is&nbsp;<b>+0</b>,&nbsp;<b>−0</b>, or&nbsp;<b>NaN</b>;<br>
                otherwise the result is&nbsp;<b>true</b>.</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result is&nbsp;<b>false</b>&nbsp;if the argument is the empty String (its length is zero);<br>
                otherwise the result is&nbsp;<b>true</b>.</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 2px 0px; vertical-align: baseline;">
                <b>true</b>.</td>
        </tr>
    </tbody>
</table>

也就是说，在JS中，判断为true的有（true,"potato",36,[1,2,4] and {a:16}）。

判断为false的有(false,0,"",null and undefined)。

现在我们知道了，在我们最开始的例子中，`if([0])`，一个数组是一个对象，一个对象会被强制转为true。

下面是一些补充例子，对于一些结果可能会让你惊讶，不过它们都是遵循上面说到的规则的：

    var trutheyTester = function(expr) {
        return expr ? "truthey" : "falsey"; 
    }
     
    trutheyTester({}); //truthey (an object is always true)
     
    trutheyTester(false); //falsey
    trutheyTester(new Boolean(false)); //truthey (an object!)
     
    trutheyTester(""); //falsey
    trutheyTester(new String("")); //truthey (an object!)
     
    trutheyTester(NaN); //falsey
    trutheyTester(new Number(NaN)); //truthey (an object!)

#### 判断相等操作符(==)

用 `==` 用来判断相等比较宽容。

两个值会被判断为相等，哪怕它们不是相同的类型，因为操作符会在比较前强制将其中一个或者两个值转换为相同类型（通常是number类型）。

许多开发者认为这是一件相当可怕的事情，至少一位出名的JSer毫无疑问的推荐彻底的避免使用 `==` 操作符。

这种回避的策略让我很困扰，因为你在没有完全由内自外的理解一门语言之前，你都没法完全的掌控它，而恐惧与逃避是知识的天敌。

另外，假装 `==` 不存在并不能解决问题，因为强制类型转换在JS中比比皆是！

它在条件语句中有出现（前面展示过的），它在数组的索引中有出现，还有许多。

而且，当强制转换被安全的运用，它能让你的代码看起来更简洁，优雅，更有可读性。

好了，总而言之，下面我们来看看ECMA是怎么定义 `==` 工作的。

它真的没有那么的恐怖，只要记住的是 `undefined` 和 `Null` 能相互相等（并与其他任何都不相等），还有大部分类型都被强制转换为number类型以加速判断了：

<table cellpadding="0" cellspacing="0" style="border-width: 0px 0px 1px; border-bottom-style: solid; border-bottom-color: rgb(76, 76, 76); font-family: 'Droid Sans', Arial, sans-serif; font-size: 14px; margin: 0px 0px 1.4285714285em; outline: 0px; padding: 0px; vertical-align: baseline; border-spacing: 0px; width: 602px; color: rgb(76, 76, 76); line-height: 23px;">
    <tbody style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 110px; background-color: rgb(170, 170, 170); height: 24px;">
                Type(x)</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 110px; background-color: rgb(170, 170, 170); height: 24px;">
                Type(y)</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; background-color: rgb(170, 170, 170); height: 24px;">
                Result</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td colspan="2" style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x and y are the same type</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>See Strict Equality (===) Algorithm</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                null</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Undefined</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Undefined</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                null</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x ==&nbsp;<span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(204, 0, 153);">toNumber</span>(y)</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(204, 0, 153);">toNumber</span>(x) == y</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Boolean</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                (any)</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(204, 0, 153);">toNumber</span>(x) == y</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                (any)</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Boolean</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x ==&nbsp;<span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(204, 0, 153);">toNumber</span>(y)</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String or Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x ==&nbsp;<span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(0, 153, 0);">toPrimitive</span>(y)</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String or Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <span style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; color: rgb(0, 153, 0);">toPrimitive</span>(x) == y</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td colspan="2" style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                otherwise…</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>false</b></td>
        </tr>
    </tbody>
</table> 

如果算法的结果是一个表达式,它将被重新申请计算，直到结果是一个boolean值。

`toNumber` 和 `toPrimitive` 是内置方法，它将会把输入值按照下面规则转换：

 <table cellpadding="0" cellspacing="0" style="border-width: 0px 0px 1px; border-bottom-style: solid; border-bottom-color: rgb(76, 76, 76); font-family: 'Droid Sans', Arial, sans-serif; font-size: 14px; margin: 0px 0px 1.4285714285em; outline: 0px; padding: 0px; vertical-align: baseline; border-spacing: 0px; width: 602px; color: rgb(76, 76, 76); line-height: 23px; background-color: rgb(235, 204, 235);">
    <caption style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; text-align: left;">
        <strong style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">ToNumber</strong></caption>
    <tbody style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 110px; background-color: rgb(170, 170, 170); height: 24px;">
                Argument Type</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; background-color: rgb(170, 170, 170); height: 24px;">
                Result</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Undefined</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>NaN</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Null</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>+0</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Boolean</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result is&nbsp;<b>1</b>&nbsp;if the argument is&nbsp;<b>true</b>.<br>
                The result is&nbsp;<b>+0</b>&nbsp;if the argument is false.</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result equals the input argument (no conversion).</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                In effect evaluates Number(<i>string</i>)<br>
                “abc” -&gt; NaN<br>
                “123″ -&gt; 123</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Apply the following steps:
                <p style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px 0px 1.0714285714em; outline: 0px; padding: 0px; vertical-align: baseline;">
                    &nbsp;</p>
                <p class="color" style="border: 0px; font-family: inherit; font-style: inherit; margin: 4px 0px 3px; outline: 0px; padding: 0px; vertical-align: baseline;">
                    1. Let&nbsp;<i>primValue</i>&nbsp;be ToPrimitive(<i>input argument</i>, hint Number).<br>
                    2. Return ToNumber(<i>primValue</i>).</p>
            </td>
        </tr>
    </tbody>
</table>

<table cellpadding="0" cellspacing="0" style="border-width: 0px 0px 1px; border-bottom-style: solid; border-bottom-color: rgb(76, 76, 76); font-family: 'Droid Sans', Arial, sans-serif; font-size: 14px; margin: 0px 0px 1.4285714285em; outline: 0px; padding: 0px; vertical-align: baseline; border-spacing: 0px; width: 602px; color: rgb(76, 76, 76); line-height: 23px; background-color: rgb(204, 235, 204);">
    <caption style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; text-align: left;">
        <strong style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">ToPrimitive</strong></caption>
    <tbody style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 110px; background-color: rgb(170, 170, 170); height: 24px;">
                Argument Type</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; background-color: rgb(170, 170, 170); height: 24px;">
                Result</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                (in the case of equality operator coercion) if&nbsp;<code style="border: 0px; font-family: Monaco, Consolas, 'Andale Mono', 'DejaVu Sans Mono', monospace; font-size: 15px; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; line-height: normal;">valueOf</code>&nbsp;returns a primitive, return it. Otherwise if&nbsp;<code style="border: 0px; font-family: Monaco, Consolas, 'Andale Mono', 'DejaVu Sans Mono', monospace; font-size: 15px; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; line-height: normal;">toString</code>&nbsp;returns a primitive return it. Otherwise throw an error</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                otherwise…</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                The result equals the input argument (no conversion).</td>
        </tr>
    </tbody>
</table>

下面是一些例子，我将会用模拟算法来一步步展示强制转换的算法是怎么工作的：

    [0] == true;

    //EQUALITY CHECK...
    [0] == true; 
     
    //HOW IT WORKS...
    //convert boolean using toNumber
    [0] == 1;
    //convert object using toPrimitive
    //[0].valueOf() is not a primitive so use...
    //[0].toString() -> "0"
    "0" == 1; 
    //convert string using toNumber
    0 == 1; //false!
    “potato” == true;

    //EQUALITY CHECK...
    "potato" == true; 
     
    //HOW IT WORKS...
    //convert boolean using toNumber
    "potato" == 1;
    //convert string using toNumber
    NaN == 1; //false!
    “potato” == false;

    //EQUALITY CHECK...
    "potato" == false; 
     
    //HOW IT WORKS...
    //convert boolean using toNumber
    "potato" == 0;
    //convert string using toNumber
    NaN == 0; //false!
    object with valueOf

    //EQUALITY CHECK...
    crazyNumeric = new Number(1); 
    crazyNumeric.toString = function() {return "2"}; 
    crazyNumeric == 1;
     
    //HOW IT WORKS...
    //convert object using toPrimitive
    //valueOf returns a primitive so use it
    1 == 1; //true!
    object with toString

    //EQUALITY CHECK...
    var crazyObj  = {
        toString: function() {return "2"}
    }
    crazyObj == 1; 
     
    //HOW IT WORKS...
    //convert object using toPrimitive
    //valueOf returns an object so use toString
    "2" == 1;
    //convert string using toNumber
    2 == 1; //false!
 

#### 严格模式相等操作符===

这个就简单多了。当两值类型不同时，总是得到false。如果它们是相同类型，那么凭直觉测试得到相等的情况：对象必须指向相同的对象，字符串必须包含相同的字符，其他原始类型必须拥有相同的值。

NaN，null和undefined永远不会===其他类型，而NaN甚至===自己都为false。

<table border="1" cellpadding="0" cellspacing="1" style="border-width: 0px 0px 1px; border-bottom-style: solid; border-bottom-color: rgb(76, 76, 76); font-family: 'Droid Sans', Arial, sans-serif; font-size: 14px; margin: 0px 0px 1.4285714285em; outline: 0px; padding: 0px; vertical-align: baseline; border-spacing: 0px; width: 602px; color: rgb(76, 76, 76); line-height: 23px;">
    <tbody style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 150px; background-color: rgb(170, 170, 170); height: 24px;">
                Type(x)</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; width: 260px; background-color: rgb(170, 170, 170); height: 24px;">
                Values</td>
            <td style="border-width: 0px 0px 1px; border-top-style: none; border-bottom-style: solid; border-bottom-color: black; font-family: inherit; font-size: 13px; font-style: italic; font-weight: bold; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: middle; background-color: rgb(170, 170, 170); height: 24px;">
                Result</td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td colspan="2" style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Type(x) different from Type(y)</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>false</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td colspan="2" style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Undefined or Null</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Number</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x same value as y (but not&nbsp;<code style="border: 0px; font-family: Monaco, Consolas, 'Andale Mono', 'DejaVu Sans Mono', monospace; font-size: 15px; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline; line-height: normal;">NaN</code>)</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                String</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x and y are identical characters</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Boolean</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x and y are both true or both false</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                Object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                x and y reference same object</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>true</b></td>
        </tr>
        <tr style="border: 0px; font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0px; vertical-align: baseline;">
            <td colspan="2" style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                otherwise…</td>
            <td style="border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(76, 76, 76); font-family: inherit; font-style: inherit; margin: 0px; outline: 0px; padding: 0.5em 0.7142857142em 0.5em 0px; vertical-align: baseline;">
                <b>false</b></td>
        </tr>
    </tbody>
</table>

一些过分使用严格匹配的常见的例子： 

(2015.5.10补充：使用严格相等可以略微加快运算效率，在确定类型的情况下推荐使用===)

    //unnecessary
    if (typeof myVar === "function");
     
    //better
    if (typeof myVar == "function");
    因为typeof返回的是一个字符串，这个操作将会一直是比较两个字符串，所以==是100%coercion-proof(不知道怎么翻，大概是100%严格匹配？)

    //unnecessary
    var missing =  (myVar === undefined ||  myVar === null);
     
    //better
    var missing = (myVar == null);
    null和undefined是相互==的

提示：因为有（非常小）的出错可能，undefined变量被重新定义了，所以与null比较会相对安全些。

    //unnecessary
    if (myArray.length === 3) {//..}
     
    //better
    if (myArray.length == 3) {//..}
