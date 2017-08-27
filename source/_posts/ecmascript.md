---
layout: post
title: JavaScript
date: 2017-08-27 15:26:29
tags: JavaScript
categories: JavaScript
---

学习记录一下javascript的知识点。

参考：[重新介绍 JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript)

***

### 类型
类型：`Number  String  Boolean  Function  Object  Symbol`

特殊的对象：` undefined  null  Array  Function...`

<!-- more -->

### 创建对象
```JavaScript
    var obj = new Object();
    var obj = {};

    //对象原型
    function Person(name, age) {
      this.name = name;
      this.age = age;
    }

    // 对象实例
    var You = new Person("You", 24); 
    var name = obj.name;
    var name = obj["name"];

```
### 数组
```JavaScript
    var a = new Array();
    a[0] = "dog";
    a[1] = "cat";
    a.length; // 2

    var a = ["dog", "cat", "hen"];
    a[100] = "fox";
    a.length; // 101

    //ECMAScript 5 增加了遍历数组的另一个方法 forEach()
    ["dog", "cat", "hen"].forEach(function(currentValue, index, array) {
        // Do something with currentValue or array[index]
    });

    //返回一个包含数组中所有元素的字符串，每个元素通过逗号分隔。
    a.toString()

    //返回一个包含数组中所有元素的字符串，每个元素通过指定的 sep 分隔。
    a.join(sep)

    //删除并返回数组中的最后一个元素
    a.pop()

    //将 item1、item2、……、itemN 追加至数组 a。
    a.push(item1, ..., itemN)

    //数组逆序（会更改原数组 a）。
    a.reverse()

    //删除并返回数组中第一个元素。
    a.shift()

    //返回子数组，以 a[start] 开头，以 a[end] 前一个元素结尾。
    a.slice(start, end)

    //	依据 cmpfn 返回的结果进行排序，如果未指定比较函数则按字符顺序比较（即使元素是数字）。
    a.sort([cmpfn])

    //	将 item 插入数组头部，返回数组新长度（考虑 undefined）。
    a.unshift([item])

```

### 函数
* 函数内部对象 arguments，表示传入的实参数组。

* fun.apply(thisArg, [argsArray])，this是在fun 函数运行时指定的 this 值，argsArray是向函数传递的参数作为一个数组。

* 匿名函数：
```JavaScript
    //匿名函数隐藏局部变量
    var a = 1;
    var b = 2;
    (function() {
        var b = 3;
        a += b;
    })();

    a; // 4
    b; // 2
```

* 递归调用。递归在处理树形结构（比如浏览器 DOM）时非常有用。
```JavaScript

    function countChars(elm) {
        if (elm.nodeType == 3) { // 文本节点
            return elm.nodeValue.length;
        }
        var count = 0;
        for (var i = 0, child; child = elm.childNodes[i]; i++) {
            count += countChars(child);
        }
        return count;
    }
```

* 自定义对象
``` JavaScript
    function makePerson(first, last) {
        return {
            first: first,
            last: last,
            fullName: function() {
                return this.first + ' ' + this.last;
            },
            fullNameReversed: function() {
                return this.last + ', ' + this.first;
            }
        }
    }
    s = makePerson("Simon", "Willison");
    s.fullName(); // Simon Willison
    s.fullNameReversed(); // Willison, Simon
    
    //如果在一个对象上使用点或者方括号来访问属性或方法，这个对象就成了 this。
    //如果并没有使用“点”运算符调用某个对象，那么 this 将指向全局对象（global object）。
    s = makePerson("Simon", "Willison");
    var fullName = s.fullName;
    fullName(); // undefined undefined


    //使用关键字 this 改进已有的 makePerson函数：
    function Person(first, last) {
        this.first = first;
        this.last = last;
    }
    Person.prototype.fullName = function() {
        return this.first + ' ' + this.last;
    }
    Person.prototype.fullNameReversed = function() {
        return this.last + ', ' + this.first;
    }
    var s = new Person("Simon", "Willison");
    //Person.prototype 是一个可以被Person的所有实例共享的对象。
```

### 闭包
* 一个闭包就是一个函数和被创建的函数中的作用域对象的组合。
```JavaScript
    //将函数对象和其所处的环境形成一个闭包集合
    function makeAdder(a) {
        return function(b) {
            return a + b;
        }
    }
    var x = makeAdder(5);
    var y = makeAdder(20);
    x(6); // 11
    y(7); // 27


    //示例应用，设置字体大小
    function makeSizer(size) {
    return function() {
        document.body.style.fontSize = size + 'px';
    };
    }

    var size12 = makeSizer(12);
    var size14 = makeSizer(14);
    var size16 = makeSizer(16);

    document.getElementById('size-12').onclick = size12;
    document.getElementById('size-14').onclick = size14;
    document.getElementById('size-16').onclick = size16;

```
* 内存泄漏
在 IE 中，每当在一个 JavaScript 对象和一个本地对象之间形成循环引用时，就会发生内存泄露。
```JavaScript
    function leakMemory() {
        var el = document.getElementById('el');
        var o = { 'el': el };
        el.o = o;
    }


    function addHandler() {
        var el = document.getElementById('el');
        el.onclick = function() {
            el.style.backgroundColor = 'red';
        }
    }
```

