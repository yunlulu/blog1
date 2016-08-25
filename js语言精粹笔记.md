## 引用
#### 对象通过引用来传递，它们永远不会被复制
````javascript
var x = stooge;//stooge是一对象
x.name = 'zhang';
var y = stooge.name;
  //因为x和stooge是指向同一个对象的引用，所以y为'zhang'

var a = {}, b = {}, c = {};
  //a b c 每个都引用一个不同的空对象
var a = b = c = {};
  //a b c 都引用同一个空对象
  
/*
 * 实战项目中经常会遇到这个问题，
 * 把后端返回的对象传递给一个函数处理下，然后再传递给别的函数
 * 处理之后的原数据也跟着变化，解决方式 深层copy
 * var dist = $.extend(true,{},src);
 */  
````
