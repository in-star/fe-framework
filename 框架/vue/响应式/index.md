##### 响应式原理
基于Object.defineProperty、Proxy进行属性代理。
+ 自定义set function
```
function logUtils() {
  let log_stack = [];
  Object.defineProperty(this, 'log', {
    set: function(val) {
      val && log_stack.push(val);
    }
  })
  this.print = function() {
    return log_stack;
  }
}
const logU = new logUtils();
logU.log = 'a is not defined';
logU.log = 'stack overflower';
console.log(logU.print);
```
+ 类全局属性扩展
```
class Global{}
let value = '';
Object.defineProperty(Global.prototype, 'x', {
  get: function() {
    return value;
  },
  set: function(val) {
    value = val;
  }
});
let global_1 = new Global();
let global_2 = new Global();
global_1.x = 'value';
console.log(global_2.x); // value
```
+ 私有属性
```
class Global{}
Object.defineProperty(Global.prototype, 'x', {
  set(val) {
    this.value = val;
  },
  get() {
    return this.value;
  }
});
const global_1 = new Global();
const global_2 = new Global();
global_1.x = 'value';
console.log(global_2.x); // undefined
```
