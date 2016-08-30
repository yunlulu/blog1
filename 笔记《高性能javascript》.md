## 高性能javascript
#### defer 运行在window.onload之前
````javascript
<script defer>
  alert('defer')
</script>
<script>
  alert('script')
</script>
<script>
window.onload = function(){
  alert('load')
}
</script>

弹出顺序 script defer load
````
#### 性能优化之动态加载脚本
````javascript
/*
* 动态加载脚本凭借着它在浏览器兼容性和易用的优势，成为最通用的无阻塞加载解决方案。
*/
function loadScript(url,callback){
  var script = doc.createElement('script');
  script.type = "text/javascript";
  if( script.readyState ){// IE
    script.onreadystatechange = function(){
      if( script.readyState == 'loaded' || script.readyState == 'complete' ){
        script.onreadystatechange = null;
        callback();
      }
    }
  }else{
    script.onload = function(){
      callback();
    }
  }
  script.src = "file1.js";
  doc.getElementsByTagName('head')[0].appendChild(script); 
}
/*
* 动态加载脚本，可以尽可能多的加载，如果要确保加载顺序，如下
* 更好的做法是 把它们按顺序合并为一个文件，因为过程是异步的，文件大一点，不会影响
*/
loadScript('file1.js',function(){
  laodScript('file2.js',function(){
    loadScript('file3.js',function(){
      alert('all files loaded!')
    })
  })
})
````
#### 性能优化之XMLHttpRequest脚本注入
````javascript
/*
* 优点是兼容性强，各浏览器都支持。
* 不足是不支持跨域，也就不能从CDN下载
*/
var xhr = new XMLHttpRequest();
xhr.open('get','file1.js',true);
xhr.onreadystatechange = function(){
  if( xhr.readyState == 4 ){
    if( xhr.status >= 200 && xhr.status < 300 || xhr.status == 304 ){
      var script = doc.createElement('script');
      script.type = 'text/javascript';
      script.text = xhr.responseText;
      doc.body.appendChild(script);
  }
};
xhr.send(null);
````
#### 推荐的无阻塞模式
````javascript
/*
* 添加大量script做法，只需两步
* 先添加动态加载所需的代码，然后加载初始化页面所需的剩下的代码
* 因为第一部分代码尽量精简，它下载执行很快，所以不会对页面有太多影响
* 记得把这些代码压缩到最小尺寸
*/
<script>
function loadScript(url,callback){
  var script = doc.createElement('script');
  script.type = "text/javascript";
  if( script.readyState ){// IE
    script.onreadystatechange = function(){
      if( script.readyState == 'loaded' || script.readyState == 'complete' ){
        script.onreadystatechange = null;
        callback();
      }
    }
  }else{
    script.onload = function(){
      callback();
    }
  }
  script.src = "file1.js";
  doc.getElementsByTagName('head')[0].appendChild(script); 
}

loadScript('the-rest.js',function(){
  Application.init();// Application是the-rest.js里面的对象
})
</script>
````
#### loadScript增加版 Yahoo工程师编辑了延迟加载工具，压缩后（gzip）之前，只有1.5k[github地址](https://github.com/rgrove/lazyload/)
````javascript
/*
* LazyLoad同样可以动态加载css文件，这没有太大意义，因为css文件已是并行下载，不会阻塞页面的其他进程
*/
<script type="tetxt/javascript" src="lazyload-min.js"></script>
<script>
  LazyLoad.js(['first-file.js','the-rest.js'],function(){
    Application.init();
  })
</script>
````
#### LABjs 另一个开源的无阻塞脚本加载工具 对加载过程更加精细，压缩后只有4.5k（非gzip）
````javascript
<script src="lab.js" type="text/javascript"></script>
<script>
  $LAB.script("first-file.js").script("the-rest.js").wait(function(){
    Application.init();
  })
</script>
````
