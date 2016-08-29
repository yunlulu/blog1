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
