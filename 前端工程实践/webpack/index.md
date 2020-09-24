##### source map
+ eval <br/>

在模块的尾部加上source url，指明原始代码的位置，不会生成.map文件。（eval是通过source url来定位源代码的位置，其他的模式通过.map文件来指明代码位置）eval指向的代码不是源码，而是经过babel编译之后的代码，所以如果使用到es6的写法（箭头函数），则调试的时候，不会有相应的箭头函数。

![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/eval.png?raw=true)
