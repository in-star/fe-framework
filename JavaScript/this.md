##### 前言
在JavaScript中this对象必须在函数调用的时候才能确定this的指向，有时候会造成一些很奇怪的问题，例如在回调函数中的this的指向等等问题。下面介绍this指向的几种情况。
##### 默认绑定
在没有明确指定调用函数的对象的时候，此时this对象指向的是window对象，在严格模式下，如果没有明确指定调用函数的对象的时候，this为undefined。在node中默认绑定会失效，因为在node中默认是严格模式。
```
// 非严格模式下
function foo(){
	console.log(this.a); // window
}
var a = 2;
foo(); // 2
// 严格模式下
function foo(){
  'use strict';
	console.log(this); // undefined
}
var a = 2;
foo(); // 2
// 严格模式
function foo(){
  'use strict';
	console.log(this); // window,在严格模式下必须明确指定函数的调用对象
}
var a = 2;
window.foo(); // 2
```
##### 隐式绑定
在对象中声明一个属性指向调用方法，此时方法中的this对象为该对象。
```
// 正常情况下：
function foo() {
	console.log(this.a);
}	
let obj = {
	a: 2,
	foo
}
obj.foo(); // 2
// 【问题1】在隐式绑定下会出现绑定对象丢失。如下，此时obj1是window对象的属性，所以此时的this是指向window。
function foo() {
	console.log(this);
}	
let obj = {
	a: 2,
	foo
}
let obj1 = obj.foo;
obj1.foo(); // window
// 【问题2】作为函数的入参的时候，会丢失指向，其实跟上面的原理类似。
function foo(){
	console.log(this.a);
}
function doFoo(fn) {// 在js中，将一个引用作为函数参数，其实是创建一个新的引用，将该引用指向入参的引用的对象实例，所以不是真正意义上的引用传递，只是两个引用指向同一份内存。
	fn();
}
let obj = {
	a: 2,
	foo
}
var a = 'global variable';
doFoo( obj.foo ); // "oops, global"
```
##### 显示绑定
利用call和apply来指定函数中的this对象的指向。call调用的函数的入参是一个跟在this对象后面，apply则是将所有的参数形成一个数组，统一入参。
```
function apply() {
	console.log(Object.prototype.toString.apply(arguments)); // 该方法可用于对象类型的判定
}
apply(1, 2);
```
##### 硬绑定
使用显示绑定的时候可以显示指定this对象，这可以解决在调用过程中this指向的不明确，但是如果此时如果是作为函数的入参的话，这种方式还是不能解决this的确定性，因为显示绑定之后是立即执行函数调用的，无法作为入参。为了解决这个问题，所以引入硬绑定。
```
function foo(){
	console.log(this.a);
}
let obj = {
	a: 2
}
var bar = function() { // 因为显示绑定的话会立即执行，这里的话利用一个函数将其包裹起来不让他马上执行，这样的话就能作为入参，保证foo函数的this对象不会被串改。
	foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100); // 2
// 硬绑定的 bar 不可能再修改它的this
bar.call( window ); // 2
```
##### bind绑定
```
bind是es6的一种新语法，跟上面的硬绑定其实差不多。会返回一个绑定函数。
let obj = Object.prototype.toString.bind(arguments);
```
##### new绑定
利用构造函数创建一个对象，此时构造函数中的this指向就是这个新建的对象。
new的原理
	1、调用new的时候会创建一个对象，其实也是一块内存。
	2、将这个对象与原型对象进行绑定，其实所谓的原型对象就是构造函数实例的内存快，可以看成是一个共享的内存快。
	3、利用绑定将当前的实例对象绑定到构造函数上，所以此时构造函数中的this对象就是新建的对象
	4、默认构造函数会返回当前新建的对象。
其实利用new创建一个对象的时候，跟构造函数关系不大，只是在new的时候会通过像constructor.bind(obj)这样的模式去调用一次构造函数，执行其中的代码，并将其中的this指向当前的实例对象。
##### 箭头函数
在es6中引入了箭头函数，箭头函数中的this对象跟函数的调用位置无关，this的指向是最靠近构造函数的函数的作用域中this的指向一致。
##### 结束语
es6之前，this的指向一致都很难判断，有时候会this的指向的问题导致一系列出乎意料的问题。有时候最直接的解决方案就是不直接使用this，而是使用别的变量来替代this，例如let _that = this;这种方式，可以提前指定this的指向，也能直接使用箭头函数来避免this指向的不确定，避免带来问题。
