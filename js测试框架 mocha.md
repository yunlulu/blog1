## 传送门：来自知乎话题：[如何进行前端自动化测试？](https://www.zhihu.com/question/29922082)  
## 传送门：mocha的入门教程，参考[阮一峰的博文](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)  
## 传送门：断言[expect语法](https://github.com/Automattic/expect.js)  
![图片](http://7xohgg.com2.z0.glb.qiniucdn.com/attachments/1452678117065/184403585.jpg)
### mocha简介  

### [`mocha`](https://mochajs.org/)（发音：摩卡），文艺名：抹茶。mocha是一个功能丰富的js测试框架，可以运行在node环境及浏览器上。语法上比较容易理解，跟loadrunner和QTP之类采用的断言、判断语法相似。mocha支持字符串diff测试、API测试、变量测试、运行时间等  

#### 测试脚本 add.js，建议测试脚本命名为 add.test.js，并放置在test文件夹中`mocha默认行为，命令行优先查找`  

#### 被测试的脚本  

```javascript
// add.js
function add(x, y) {
  return x + y;
}
module.exports = add;
```
#### 测试脚本  

```javascript
var add = require('./add.js');
var expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
  it('1 加 1 不能等于 3', function() {
    expect(add(1, 1)).to.be.not.equal(3);
  });
});
```
__详细语法介绍见[官网](https://mochajs.org/)及[阮一峰博文](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)__
