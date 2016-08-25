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
## 闭包
#### 闭包是一种有效减少全局污染的方法
````html
一个内部函数除了可以访问自己的参数和变量，
同时它也能自由访问把它嵌套在其中的父函数的参数和变量。
通过函数字面量创建的函数对象包含一个连到外部上下文的连接。
这被称为闭包
````
## this(不同的调用模式，指向不一样)
#### 方法调用模式：当一个函数被保存为对象的一个属性时，我们称它为一个方法。当一个方法被调用时，this被绑定到该对象。
````javascript
var myObject = {
  value: 0,
  increment: function(inc){
    this.value += typeof inc === "number" ? inc : 1;
  }
}
myObject.increment();
myObject.value // 1
myObject.increment(2);
myObject.value // 3
````
#### 函数调用模式：当一个函数并非一个对象的属性时，那么它就是被当做一个函数来调用的。这是语言设计上的一个错误，解决方法
````javacript
var add = function(a,b){
  return a + b;
}
// 给myObject增加一个 double 方法
myObject.double = function(){
  var me = this;
  var helper = function(){
    me.value = add(me.value, me.value);
  }
  helper();
}
myObject.double();
myObject.value // 6
````
