---
layout: post
title:  "js双向绑定原理"
date:   2017-01-03 22:14:54 GMT+0800
categories: JS
tag: 数据双向绑定原理
---

* content
{:toc}

> Angular、Vue、React、Ember等主流的mvvm前端框架都实现了数据双向绑定，关于怎么实现原理不好奇是不可能的 ~ 

> 其实这个问题也可以叫做，如何监听js变量变化。Angular的原理可以总结为对比数据差异，如果不同，即更新视图，嗯，叫做**脏值检测**，这次主要写一下vue的实现方式，ES5的getter和setter。

Getter和Setter
------------------

vue双向绑定的原理基于ES5定义的`Object.defineProperty(obj, prop, descriptor)`方法，这也是为什么vue不支持IE10以下的原因。其中`descriptor`中可传入getter和setter方法，顾名思义，就像从最开始C语言编程时的get和set函数一样，取值时候会触发getter，赋值会触发setter。

我们来实现一个最最简单的双向绑定 ~

首先来看下实现的效果

![输入框双向绑定]({{ '/styles/images/2017010301.png' | prepend: site.baseurl }})

input的值与下方的值是动态一致的。

html的部分很简单，就两个元素

    <input id="input" type="text">
    <div id="div"></div>

js的部分

    var a = {test: '1'};
    Object.defineProperty(a, 'test', {
        get: function() {
            console.log('invoke getter');
            return test;
        },
        set: function(value){
            console.log('invoke setter');
            test = value;
            // callback(value); 回调函数
            return test;
        }
    });

首先我们定义一个对象`a`，调用`Object.defineProperty`方法，设置getter和setter，这样当我们执行`a.test`时，会触发get方法，执行a.test = 'xxx'的时候会触发set方法。

既然我们想实现双向绑定，即input值改变了，`a.test`也要改变；`a.test`改变了，input的值也要改变。对于前者，非常简单，我们给input加事件监听

    var el = document.querySelector('#input');
    el.addEventListener('keyup', function() {
        a.test = this.value;
    });

这样input输入内容时，`a.test`随之改变。接下来实现后者

    function callback(value) {
        document.querySelector('#input').value = value;
        document.querySelector('#div').innerHTML = value;
    }

将这个回调函数填入set中，当`a.test`值改变时，会触发input和div的视图更新。

整体代码如下

    <body >

        <input id="input" type="text">
        <div id="div"></div>

        <script type="text/javascript">
            var el = document.querySelector('#input');
            el.addEventListener('keyup', function() {
                a.test = this.value;
            });

            var a = {test: '1'};
            Object.defineProperty(a, 'test', {
                get: function() {
                    console.log('invoke getter');
                    return test;
                },
                set: function(value){
                    console.log('invoke setter');
                    test = value;
                    callback(value);
                    return test;
                }
            });

            function callback(value) {
                document.querySelector('#input').value = value;
                document.querySelector('#div').innerHTML = value;
            }
        </script>
    </body>

在控制台中改变下`a.test`的值

![改变a.test的值]({{ '/styles/images/2017010302.png' | prepend: site.baseurl }})


可以看到两个视图都动态更新了 ~ 至此我们就实现了一个最最最简单的双向绑定，其原理就是这样 ~

还不够完美
-----------------

getter和setter虽然好用，还是有些不足，无法兼容IE10以下，并且数组的push，pop，shift，unshift等都不能够触发setter，这个时候还要写些黑科技来解决这些问题。

