##### source map
+ eval <br/>

在模块的尾部加上source url，指明原始代码的位置，不会生成.map文件。（eval是通过source url来定位源代码的位置，其他的模式通过.map文件来指明代码位置）eval指向的代码不是源码，而是经过babel编译之后的代码，所以如果使用到es6的写法（箭头函数），则调试的时候，不会有相应的箭头函数。

![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/eval.png?raw=true)

+ source map  

生成.map文件，能映射到对应的源文件

+ cheap

调试定位的时候，不会定位到具体某一列，只会定位到某行的代码

##### 环境配置
+ 开发环境
  * cheap-eval-source-map
+ 正式环境
  * none
  * hidden-source-map  
  
  参考
  https://segmentfault.com/a/1190000008315937

##### babel
+ babel-loader、@babel/core
编译指定的后缀文件，此时对于es6的语法还是无法直接转译成低版本浏览器能够识别的语法。
+ @babel/preset-env
转译es6新语法糖，一个大礼包，不需要对每一个特定的语法糖使用对应的plugin进行转译。
+ @babel/polyfill
转译一些新的api（Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise）、新的函数（Object.assign）。  
![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/babel-polyfill.png?raw=true)

1、将@babel/polyfill直接写在entry中进行引入的话，打包出来的文件会很大。  
![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/entry-polyfill.png?raw=true)
![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/entry-polyfill-wp-cfg.png?raw=true)

2、在.babelrc中配置，结合useBuiltIns则可以不需要手动在入口文件中引入polyfill包，webpack会自动根据需要引入对应的转译包，同时包的资源会小很多。  
2.1、使用eval进行打包，源码会被追加到main.js中，会造成文件比较大。
2.2、使用source-map则可以将其进行分离到.map文件中。
![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/polyfill-import-require.png?raw=true)
![](https://github.com/1415757704/fe-framework/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5/webpack/polyfill-pg-in-main.png?raw=true)
