---
layout: post
title: 你不知道的JavaScript RegExp对象的一些细节
tags: Developing
description: 你不知道的JavaScript RegExp对象的一些细节
date: 2016-08-26 15:11
---

JS对于字符串的匹配有两个方法，分别是 `String.prototype.match(regexp)` 与 `RegExp.prototype.exec(string)` ，首先我们可以看一个问题：


    var someText = "web2.0 .net2.0",
        pattern = /(\w+)(\d)\.(\d)/g;

    var res1 = pattern.exec(someText);
    var res2 = someText.match(pattern);

    var res3 = pattern.exec(someText);
    var res4 = pattern.exec(someText);
    var res5 = pattern.exec(someText);

    // What is these values?
    // 1. res1[1]
    // 2. res2[1]
    // 3. res3[0]
    // 4. res4[0]
    // 5. res5


如果能够知道答案，说明你对 `exec` 与 `match` 非常熟悉。如果不知道也没有关系，因为今天的重点不是介绍用法。

把上述代码执行一下，可以看到如下结果：

<div>
  <p><img src="{{ site.url }}/downloads/images/regExpIntro/result.jpg" style="width:400px;box-sizing: border-box;padding: 2px;border: 1px solid #999;"></p>
</div>

这里面其实涉及到两个点：

1. `someText.match(pattern)` 与 `pattern.exec(someText)` 返回结果的差异；
2. 当 `pattern.exec(someText)` 当 `pattern` 带有 `global flag` 时，每次执行后仅返回一个匹配，更新 `pattern.lastIndex` ，下一次执行将会从 `lastIndex` 处开始往后匹配。

那么现在问题来了，仔细看res3/4/5，既然 `pattern.exec` 会更新 `pattern.lastIndex` ，那为什么res3的结果还是 `["web2.0", "web", "2", "0"]` 而不是 `["net2.0", "net", "2", "0"]` 呢？是不是 `someText.match(pattern)` 干了什么？

<!--more-->

在上述代码中的每一步执行后的 `pattern.lastIndex` 打印出来，可以看到结果如下：


    var someText = "web2.0 .net2.0",
    pattern = /(\w+)(\d)\.(\d)/g;

    var res1 = pattern.exec(someText);
    console.log(pattern.lastIndex); // 6
    var res2 = someText.match(pattern);
    console.log(pattern.lastIndex); // 0

    var res3 = pattern.exec(someText);
    console.log(pattern.lastIndex); // 6
    var res4 = pattern.exec(someText);
    console.log(pattern.lastIndex); // 14
    var res5 = pattern.exec(someText);
    console.log(pattern.lastIndex); // 0


可以看到，`someText.match(pattern)` 执行完后将 `pattern.lastIndex` 置为0了。我查阅了许多关于 `String.prototype.match` 方法描述的文档，最后终于在[ES5规范](http://www.ecma-international.org/ecma-262/5.1/#sec-15.5.4.10)中找到了答案。

规范中对match的实现算法做了介绍，翻译成伪代码如下：

    String.prototype.match = function (reg) {
      var S = toString(this);
      var rx = reg;
      var global = rx.global;
      var exec = rx.exec;

      if (global !== true) {
        return exec(S);
      } else {
        // 当global为true，遍历往后调用reg.exec
        rx.lastIndex = 0;
        var A = new Array();

        var previousLastIndex = 0;
        var n = 0;
        var lastMatch = true;

        while (lastMatch === true) {
          var result = exec(S);
          if (result === null) {
            lastMatch = false;
          } else {
            var thisIndex = rx.lastIndex;
            if (thisIndex === previousLastIndex) {
              rx.lastIndex = thisIndex + 1;
              previousLastIndex = thisIndex + 1;
            } else {
              previousLastIndex = thisIndex;
            }
            var matchStr = result[0];
            defineProperties(A, toString(n), {
              value: matchStr,
              writable: true,
              enumerable: true,
              configurable: true
            }, false);
            n++;
          }
        }
        if (n == 0) return null;

        return A;
      }
    };


简单点来说，就是`String.prototype.match(regexp)`方法的底层其实还是调用的是RegExp.prototype.exec。

当 `regexp` 带有 `global` 标识时，会循环的调用 `regexp.exec` 方法，并把返回数组中的第一个结果塞进一个新数组，直到 `regexp.exec` 返回 `null` 为止，而当`regexp.exec` 执行返回 `null` 的话，`regexp.lastIndex` 将会被重置为0，这就是为什么执行 `someText.match(pattern)` 后 `pattern.lastIndex` 会变为0。

### 总结

简单的举几个使用场景：

1. 当仅需要判断字符串是否匹配正则表达式而不需要取得匹配结果时，可以直接使用 `pattern.test(someText)`（注意 `pattern.test` 的背后其实仍是调用的 `pattern.exec` 方法，根据 `exec` 返回值是否为 `null` 而返回 `false` 或是 `true`。所以当 `pattern.global` 时，执行多次执行 `pattern.test` 仍会修改 `pattern.lastIndex` 从而导致结果可能会有所不同）。
2. 当 `pattern.global == false` 时，使用 `pattern.exec` 与 `someText.match`的效果一模一样，从上面的伪代码可以知道他们内部调用的同一个方法。
3. 当 `pattern.global == true`，且仅需要取得字符串匹配正则结果的第一个结果，并且对其余结果并不关心时，使用 `exec` 与 `match` 都能够获得同样的结果。
4. 当 `global == true`，且关注每一个匹配的各个捕获，仅能够使用`exec`。

当使用 `RegExp.prototype.exec` 去匹配带`global`标识的正则对象的时候必须要小心，你必须十分清楚你的使用场景与你的预期是否匹配。

### 补充（JS中涉及到正则的若干方法简述）

#### 1. RegExp.prototype.exec(String) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.10.6.2)
注意点伪代码：

    if(reg.global)
      update reg.lastIndex // 仅当global==true才会更新lastIndex

#### 2. RegExp.prototype.test(String) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.10.6.3)

实现伪代码：

    RegExp.prototype.test = function(String) {
      return this.exec(reg) ? true : false
    }

#### 3. String.prototype.match(RegExp) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.5.4.10)

具体伪代码实现参见上文，简述：

    if (reg.global)
        loop reg.exec(String)
    else
        return reg.exec(String)

#### 4. String.prototype.search(RegExp) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.5.4.12)

注意点：

1. 返回第一个匹配时的偏移量
2. 忽略global与lastIndex(不会更新lastIndex)

#### 5. String.prototype.replace(searchValue, replaceValue) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.5.4.11)

注意点：

当searchValue为RegExp时，匹配的底层逻辑与String.prototype.match一致，包括对reg.lastIndex的更新

#### 6. String.prototype.split(separator, limit) [规范描述](http://www.ecma-international.org/ecma-262/5.1/#sec-15.5.4.14)

注意点：

1. 匹配时的底层调用方法与exec一致，都是内部方法[`[[Match]]`](http://www.ecma-international.org/ecma-262/5.1/#sec-8.6.2)
2. 不会影响reg.lastIndex

伪代码实现：

    String.prototype.split = function(separator, limit) {
      let S = ToString(this);
      let A = new Array();
      let lengthA = 0;
      let lim;
      if (limit === undefined) {
        lim = Math.pow(2, 23) - 1;
      }
      else {
        lim = ToUint32(limit);
      }
      let s = S.length;
      let p = 0;

      let R;
      if (pattern instanceof RegExp) {
        R = separator;
      }
      else {
        R = ToString(separator);
      }

      if (lim == 0) return A;

      if (separator === undefined) return [S];

      if (s == 0) {
        let z = SplitMatch(S, 0, R);
        if (z) return A;
        return [S];
      }

      let q = p;

      while(q != s) {
        let z = SplitMatch(S, q, R);
        if (!z) {
          q = q + 1;
        } else {
          let e = z.endIndex;
          let cap = z;
          if (e == q) {
            q = q + 1;
          } else {
            let T = S.substring(p, q);
            A.push(T);
            lengthA++;
            if (lengthA == lim) return A;
            let p = e;
            let i = 0;
            while(i != cap.length) {
              i++;
              A.push(cap[i]);
              lengthA =+;
              if (lengthA == lim) return A;
            }
            let q = p;
          }
        }
      }

      let T = S.substring(p, s);

      A.push(T);
      return A;
    };

    /**
     *
     * @param {String} S
     * @param {Number} q
     * @param {String|RegExp} pattern
     * @return {MatchResult}
     */
    function SplitMatch(S, q, R) {
      if (R instanceof RegExp) {
        return R.match(S, q);
      }
    }
