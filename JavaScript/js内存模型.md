### node的内存模型
#### 新生代、旧生代
新生代采用复制算法，因为对象存活周期不长。旧生代使用标记清除整理算法，旧生代中对象比较大，存活的时间比较长，使用复制算法的话会影响性能。

#### 内存泄漏情景
+ 闭包
```
'use strict';
const express = require('express');
const app = express();

//以下是产生泄漏的代码
let theThing = null;
let replaceThing = function () {
    let leak = theThing;
    // 在闭包中持有leak的引用，会导致内存无法释放
    let unused = function () {
        if (leak)
            console.log("hi")
    };
    
    // 不断修改theThing的引用
    theThing = {
        longStr: new Array(1000000),
        someMethod: function () {
            console.log('a');
        }
    };
};

app.get('/leak', function closureLeak(req, res, next) {
    // 每次访问都会创建一个leak对象，这个对象持有的内存由于闭包的缘故无法被释放
    replaceThing();
    res.send('Hello Node');
});

app.listen(8082);
```
解决方案
```
'use strict';
const express = require('express');
const app = express();
const easyMonitor = require('easy-monitor');
easyMonitor('Closure Leak');

let theThing = null;
let replaceThing = function () {
    let leak = theThing;
    //断掉leak的闭包引用即可解决这种泄漏
    let unused = function (leak) {
        if (leak)
            console.log("hi")
    };

    theThing = {
        longStr: new Array(1000000),
        someMethod: function () {
            console.log('a');
        }
    };
};

app.get('/leak', function closureLeak(req, res, next) {
    replaceThing();
    res.send('Hello Node');
});

app.listen(8082);
```
+ 匿名回调函数<br/>
回调函数内存泄漏是由于使用了监听者模式导致的。现在浏览器使用标记清除的垃圾回收机制可以实现对于回调函数的回收，不需要removeEventListener
```
const net = require('net');
let client = new net.Socket();

function connect() {
    // 使用匿名的回调函数
    client.connect(26665, '127.0.0.1', function callbackListener() {
    console.log('connected!');
});
}

//第一次连接
connect();

client.on('error', function (error) {
    // console.error(error.message);
});

client.on('close', function () {
    //console.error('closed!');
    //泄漏代码，在断开的时候无法移除回调函数，因为这种发布订阅模式都是将回调函数存储在内存的，没有移除的话会存在回调集合中
    client.destroy();
    setTimeout(connect, 1);
});
```
解决方案
```
const net = require('net');
const easyMonitor = require('easy-monitor');
easyMonitor('Socket Leak');
let client = new net.Socket();
// 具名的回调函数
function callbackListener() {
    console.log('connected!');
});

function connect() {
    client.connect(26665, '127.0.0.1', callbackListener}

connect();

client.on('error', function (error) {
    // console.error(error.message);
});

client.on('close', function () {
    //console.error('closed!');
    //断线时去掉本次侦听的connect事件的侦听器
    client.removeListener('connect', callbackListener);
    client.destroy();
    setTimeout(connect, 1);
});
```
+ 全局对象
```
function fn(){
    // 这里的a没有定义，所以等价与this.a，而this对象是全局的window对象，这样的话，执行调用函数的之后，会在全局对象window上挂载一个大对象a，造成内存泄漏
    a = new Array(1000000)
}
fn()
```
<a>https://cnodejs.org/topic/58eb5d378cda07442731569f</a><br/>
<a>https://jinlong.github.io/2016/05/01/4-Types-of-Memory-Leaks-in-JavaScript-and-How-to-Get-Rid-Of-Them/</a><br/>
<a>https://github.com/frontend9/fe9-library/issues/290</a><br/>
