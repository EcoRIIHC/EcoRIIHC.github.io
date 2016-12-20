---
layout: post
title:  "JS并行加载顺序执行问题"
date:   2016-12-20 23:44:21 GMT+0800
categories: 项目实战
tag: JS动态加载
---

* content
{:toc}

> 今天在项目实战中遇到了久闻的Js并行加载执行顺序的问题，在之前的实践中鲜有碰到，忽视了这个问题，就将问题及解决思路整理一下。

JS并行加载的问题
---------------------
医院OA的项目需要集成新的工作流，系统的前台是vue写的单页面应用，需要集成的工作流系统是多页面应用，本身单页面的css和js需要全部打包一次性加载，首页的负担已经很重，把这部分的js代码放入原来系统一起打包显然是不科学的，从性能考虑，应当将这部分js动态加载，用户只有到工作流页面的时候，才加载相关的js代码。

思路很简单，把这些js文件存到一个数组里，循环加到body最下面，不要问为什么有这么多Js为啥不合并到一起。

``` javascript
// 渲染js
var body = document.querySelector('body');
var jsScript = ['1.js','2.js','3.js','4.js','core.js','4.js', '5.js','6.js'];
for(let i = 0, item; item = jsScript[i++]; ) {
    var script = document.createElement('script');
    script.src = item;
    headTag.appendChild(script);
}
```

但是问题来了，页面不能正常加载，查看控制台，发现各种undefined找不到，于是乎大致看了一下，这几个js之间是有依赖的，原因就在于`现代浏览器都会并行下载js文件，但并不能保证顺序执行`，比如2.js依赖于1.js，2.js下载完开始执行时，1.js还没有下载完，导致报错。

解决方法
---------------------

问题已经很明了，那么要处理的就是让js并行下载的同时，能够顺序执行。

参数还是数组，将需要顺序执行的js传入，进行递归，通过`onload`回调来保证js能够顺序执行。

``` javascript
    var loadScript = function(arr) {
        var script = document.createElement('script');
        script.src = arr.shift();
        script.onload = function() {
            if(arr.length === 0) {
                return;
            }     
            loadScript(arr);   
        }
        document.querySelector('body').appendChild(script);
    }
    var jsScript = ['1.js','2.js','3.js','4.js','core.js','4.js', '5.js','6.js'];
    loadScript(jsScript);
``` 

其他方法
-------------------
LABjs貌似就是干专门干这个事的0.0
