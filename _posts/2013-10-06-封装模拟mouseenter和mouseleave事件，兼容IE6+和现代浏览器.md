---
layout: post
title: 封装模拟mouseenter和mouseleave事件，兼容IE6+和现代浏览器
tags: JavaScript
date: 2013-10-06 13:50
---

最近要写一个使用原生JS的页面，需要用到 `mouseenter` 和 `mouseleave` 事件。

（补充：因为如果直接使用 `mouseout` 和 `mouseover` ，当鼠标经过你注册元素的子元素时，也会触发这俩事件并冒泡给你注册的元素，导致多次触发。）

以前就有写过模拟 `mouseenter` 和 `mouseleave` 事件的函数，主要原理是还是给元素注册` mouseout` ， `mouseover` 事件，但是当触发时判断事件的 `e.relatedTarget` ，判断是否是由子元素冒泡上来的，如果不是，才调用 `handler` 。

但是在IE下测试时发现问题，原因是IE不支持 `event.relatedTarget` ，需要使用 `mouseout` 的 `event.toElement` 和 `mouseover` 的 `event.fromElement` 。

于是乎重新封装了下，测试成功，兼容IE6+和其他现代浏览器。

<!--more-->

直接上源代码：


    //首先需要一个兼容的注册事件的函数
    function on(o, event, handler) {
        if (window.addEventListener) {
            o.addEventListener(event, handler, false);
        }
        else {
            o.attachEvent('on' + event, function (e) {
                //这里主要是将handler内的this指向当前对象，坏处是无法删除注册事件，如果需要删除注册事件请直接
                // o.attachEvent('on' + event,handler)
                e = e || window.event;
                return handler.call(o, e);
            });
        }
    }

    //然后是判断触发mouseout事件的是否是该注册元素的子元素，是子元素返回false,不是返回true
    function checkMouseoutTo(that, e) {
        e = e || window.event;
        // 现代浏览器是e.relatedTarget,IE下是toElement
        var parent = e.relatedTarget || event.toElement || null;
        try {
            while (parent && parent !== that) {
                parent = parent.parentNode;
            }
            return (parent !== that);
        }
        catch (e) {}
    }

    //判断触发mouseto事件的是否是注册元素的子元素
    function checkMouseoverFrom(that, e) {
        e = e || window.event;
        var parent = e.relatedTarget || event.fromElement || null;
        try {
            while (parent && parent !== that) {
                parent = parent.parentNode;
            }
            return (parent !== that);
        }
        catch (e) {}
    }

    //最后是绑定mouseout和mouseover的函数，当然你也可以分别注册这两个事件
    function mouseEnterLeave(o, enterCallback, leaveCallback) {
        on(o, 'mouseover', function (e) {
            if (checkMouseoverFrom(this, e)) {
                enterCallback.call(o, e);
            }
        });
        on(o, 'mouseout', function (e) {
            if (checkMouseoutTo(this, e)) {
                leaveCallback.call(o, e);
            }
        });
    }

    //调用方法：
    var element = document.getElementById('yourelement');

    function onMouseenter(e) {
        //do something when mouseenter
    }

    function onMouseleave(e) {
        //do something when mouseleave
    }

    mouseEnterLeave(element, onMouseenter, onMouseleave);

源代码就在上面，就不多解释了，都有注释！ 欢迎交流！