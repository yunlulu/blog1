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
