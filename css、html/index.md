
#### 一、盒子模型
> W3C盒子宽高为content的宽高； box-sizing: content-box<br/>
IE盒子宽高，content + padding + border； box-sizing: border-box
```
/**
* 对于content-box，设置width为100px，那么div的宽度就是100px，这也是浏览器默认
* 的模型。
*/
.box1 {
    width: 100px; 
    height: 100px;
    background: red;
    padding: 10px;
    border: 10px solid yellow;
    box-sizing: content-box;
}
/**
* 对于border-box，设置100px，盒子的真正的大小其实还要减去padding和border的大
* 小，即div真正能容得下的内容只有width为60px，这种情况会比上面的模型能容纳的
* 内容小。
*/
.box1 {
    width: 100px;
    height: 100px;
    background: red;
    padding: 10px;
    border: 10px solid yellow;
    box-sizing: border-box;
}
```
