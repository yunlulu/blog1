# H5-WebApp 自适应方案  - rem
>对于WebApp来说，为了更通用地满足各机型屏幕的自适应布局要求，我们目前采用rem布局方案。

## rem
>rem是相对于根元素(html)字体大小的单位，它只是一种相对单位。不同于另一个相对单位em，em是相对于父元素的字体大小，而rem则相对于根元素(html)，与父元素的字体大小无关。

## 等比例适配所有屏幕
不论传统的px绝对像素布局，还是流式布局、固定宽度和响应式做法，都有其缺陷，并不能完全做到自适应所有屏幕。

但是，rem方案可以比较容易地做到等比例适配所有屏幕，保证各屏幕的显示效果与原始设计稿一致。

我们看看rem是如何工作的。

举个例子：（图片来自网络）

    html{
        font-size: 20px;
    }
    .btn {
        width: 6rem;
        height: 3rem;
        line-height: 3rem;
        font-size: 1.2rem;
        display: inline-block;
        background: #06c;
        color: #fff;
        border-radius: .5rem;
        text-decoration: none;
        text-align: center;    
    }

上面代码结果如下图：

![demo](http://520ued.com/uploads/20141218/1418899506.jpeg)

如果我们把html的font-size改改，再看看效果。

    html{
        font-size: 40px;
    }

![demo](http://520ued.com/uploads/20141218/1418898055.jpeg)

可以看到，按钮的`width`、`height`、`font-size`和`border-radius`都被放大了一倍，我们只需要改变`html`的`font-size`，就能改变按钮在页面上的显示大小，而不必再去重设按钮的样式规则。

__按钮的大小 = 按钮的rem * html.font-size__

于是，利用这个特性，我们可以这样实现等比例适配所有屏幕。

### 基准屏幕宽度
以设计稿宽度作为最理想的基准屏幕宽度，假设设计稿宽度是`750px`，那么基准屏幕宽度就是`750px`，宽度大于`750px`的屏幕，等同于等比例放大了页面，小于`750px`的屏幕，等同于等比例缩小了页面。

把设计稿750px十等分一下，每等分=`75px`，我们可以把这个`75px`当做`1个rem单位`，那么`750px`宽度就等于是`10rem`。

设置html的`font-size: 75px`，即`1rem`，也就是`rem的基准px=75px`。

设计稿上的元素尺寸换算公式：`原始px值 / rem基准px`；例如`240px * 120px`的元素，其实就是`3.2rem * 1.6rem`。

### 适配任意屏幕
我们可以通过js来取得当前机型屏幕的宽度值，然后10等分得出当前屏幕1个rem应该代表的绝对像素值，再将这个`rem基准px`动态设置到`html.font-size`，核心代码如下：

    var docEl = document.documentElement;
    var width = docEl.getBoundingClientRect().width;
    var rem = width / 10;
    docEl.style.fontSize = rem + 'px';

我们还可以通过less预处理工具，编写一个绝对px转rem的函数，自动转换px值，省却我们手动计算rem的麻烦，核心代码如下：

    @design-width: 750px; // 设计稿宽度
    @rem: @design-width / 10; // 10等分得到的rem基准px

    .px2rem(@attr; @px) when (ispixel(@px)) {
        @{attr}: unit(@px / @rem, rem);
    }

    // 处理非px值
    .px2rem(@attr; @px) when (default()) {
        @{attr}: @px;
    }

    .px2rem(@attr; @px1; @px2) when (ispixel(@px1)) and (ispixel(@px2)) {
        @{attr}: unit(@px1 / @rem, rem) unit(@px2 / @rem, rem);
    }

    .px2rem(@attr; @px1; @px2) when (ispixel(@px1)) and not (ispixel(@px2)) {
        @{attr}: unit(@px1 / @rem, rem) @px2;
    }

    .px2rem(@attr; @px1; @px2) when not (ispixel(@px1)) and (ispixel(@px2)) {
        @{attr}: @px1 unit(@px2 / @rem, rem);
    }

    // 处理非px值
    .px2rem(@attr; @px1; @px2) when (default()) {
        @{attr}: @px1 @px2;
    }

然后用法如：

    .px2rem(width; 240px);
    .px2rem(padding; 10px; 20px);

### 对于Retina高清屏幕的处理
对于2倍和3倍的高清屏幕，统一做2倍处理。

处理方法是：重设viewport，将viewport缩小一倍。

核心代码如：

    var dpr = 1;
    var scale = 1;

    if (window.devicePixelRatio >= 2) {
        scale *= 0.5; // viewport缩小一倍
        dpr *= 2;
    }

    docEl.setAttribute('data-dpr', dpr);
    metaEl.setAttribute('content', 'initial-scale=' + scale + ', width=device-width, maximum-scale=' + scale + ', user-scalable=no');

### 字号不用rem
字号大小不推荐用rem作为单位，否则可能会有文字排版问题。因此，字号仍旧使用px作为单位，并配合`data-dpr`自定义属性来在普通屏和高清屏设置不同的`font-size`。

__高清屏的`font-size`=设计稿的font-size，普通屏是设计稿font-size的一半。__

处理方式是：设定`body`的`font-size`，此后页面上所有元素的字号大小都是相对于body的`font-size`，而不是html的`font-size`。

核心代码如：
    
    var fontBase = 16; // 普通屏基准字号：16px，高清屏基准字号：16px * 2
    if (doc.readyState === 'complete') {
        document.body.style.fontSize = fontBase * dpr + 'px';
    } else {
        document.addEventListener('DOMContentLoaded', function(e) {
            document.body.style.fontSize = fontBase * dpr + 'px';
        }, false);
    }

    // 默认普通屏，设计稿字号的一半
    h3 {
        font-size: 18px;
    }

    // 高清屏：设计稿的字号
    [data-dpr="2"] h3 {
        font-size: 36px;
    }

## 完整代码
    <meta name="viewport" content="initial-scale=1, width=device-width, maximum-scale=1, user-scalable=no" />
    <script type="text/javascript">
        var FS = {};
        (function(win, FS) {
            var doc = win.document;
            var docEl = doc.documentElement;
            var metaEl = doc.querySelector('meta[name="viewport"]');
            var dpr = 1;
            var scale = 1;
            var fontBase = 16;

            if (window.devicePixelRatio >= 2) {
                scale *= 0.5;
                dpr *= 2;
            }

            docEl.setAttribute('data-dpr', dpr);
            metaEl.setAttribute('content', 'initial-scale=' + scale + ', width=device-width, maximum-scale=' + scale + ', user-scalable=no');

            function refreshRem() {
                var width = docEl.getBoundingClientRect().width;
                var rem = width / 10;
                docEl.style.fontSize = rem + 'px';
                FS.rem = rem;
            }

            var tid = null;
            win.addEventListener('resize', function() {
                clearTimeout(tid);
                tid = setTimeout(refreshRem, 300);
            }, false);

            if (doc.readyState === 'complete') {
                doc.body.style.fontSize = fontBase * dpr + 'px';
            } else {
                doc.addEventListener('DOMContentLoaded', function(e) {
                    doc.body.style.fontSize = fontBase * dpr + 'px';
                }, false);
            }

            refreshRem();

            FS.dpr = dpr;
            FS.rem2px = function(d) {
                var val = parseFloat(d) * FS.rem;
                if (typeof d === 'string' && d.match(/rem$/)) {
                    val += 'px';
                }
                return val;
            }
            FS.px2rem = function(d) {
                var val = parseFloat(d) / FS.rem;
                if (typeof d === 'string' && d.match(/px$/)) {
                    val += 'rem';
                }
                return val;
            }
        })(window, FS);
    </script>

__From Fxiaoke__
