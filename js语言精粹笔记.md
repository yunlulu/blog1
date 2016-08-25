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
````
