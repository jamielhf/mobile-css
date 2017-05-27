# 移动端自适应布局的两种方式

## 1、 rem+js  

### scss 代码  

``` 
//基础font-size
$font:16;
//设计稿宽度
$screen:750;


@function px2rem($n){
  @return #{$n/($screen*$font/320)}rem
}

```

### js  代码 (在head引入)  

``` js
(function (doc, win) {
var docEl = doc.documentElement,
resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
recalc = function () {
var clientWidth = docEl.clientWidth;
if (!clientWidth) return;
docEl.style.fontSize = 16 * (clientWidth / 320) + 'px';
            };
if (!doc.addEventListener) return;
    win.addEventListener(resizeEvt, recalc, false);
    doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);

```

> 写scss的时候，就可以根据设计稿的尺寸来写，如：

```
div{
  width:px2rem(100);  //设计稿宽度是100px
  font-size:px2rem(16) ; //设计稿字体是16px
}
```
## 2、 vw的方式

### scss 代码

```
$screen:750;

@function px2vw($n){
  @return #{($n/$screen)*100}vw
}

```

> 写scss的时候，就可以根据设计稿的尺寸来写，如：

```
div{
  width:px2vw(100);  //设计稿宽度是100px
  font-size:px2vw(16) ; //设计稿字体是16px
}
```


## 第一种的方式在页面加载的时候会有个重置css的过程，导致了初始化的时候会出现页面从小变大的闪烁

### 以下是另一种方法，比较完善

scss文件
```
@charset "UTF-8";


/* 设计图宽度 
如设计稿750，分10份， 1rem = 75px
base：https://github.com/amfe/lib-flexible */
$baseWidthRate: 36;

/* px转换为rem */
@function r($num) {
    @return $num / $baseWidthRate + rem;
}


/* === 全局变量配置 === */
$base-font-size          : r(14) !default; /* // 字号 */
$base-font-family        : "Microsoft YaHei",Tahoma, Arial, "Helvetica Neue", Helvetica, sans-serif !default; /* // 字体族 */
$base-line-height        : 1.5 !default; /* // 行号 */
$base-color              : #333 !default; /* // 字体颜色 */
$base-background-color   : #F2F2F2 !default; /* // 背景颜色 */
$base-font-weight		 : normal; /* //大小 */


```
js文件
```
/*!
 * from: https://github.com/amfe/lib-flexible
 */
;
(function(win, lib) {
    var doc = win.document;
    var docEl = doc.documentElement;
    var metaEl = doc.querySelector('meta[name="viewport"]');
    var flexibleEl = doc.querySelector('meta[name="flexible"]');
    var dpr = 0;
    var scale = 0;
    var tid;
    var flexible = lib.flexible || (lib.flexible = {});

    if (metaEl) {
        console.warn('将根据已有的meta标签来设置缩放比例');
        var match = metaEl.getAttribute('content').match(/initial\-scale=([\d\.]+)/);
        if (match) {
            scale = parseFloat(match[1]);
            dpr = parseInt(1 / scale);
        }
    } else if (flexibleEl) {
        var content = flexibleEl.getAttribute('content');
        if (content) {
            var initialDpr = content.match(/initial\-dpr=([\d\.]+)/);
            var maximumDpr = content.match(/maximum\-dpr=([\d\.]+)/);
            if (initialDpr) {
                dpr = parseFloat(initialDpr[1]);
                scale = parseFloat((1 / dpr).toFixed(2));
            }
            if (maximumDpr) {
                dpr = parseFloat(maximumDpr[1]);
                scale = parseFloat((1 / dpr).toFixed(2));
            }
        }
    }

    if (!dpr && !scale) {
        var isAndroid = win.navigator.appVersion.match(/android/gi);
        var isIPhone = win.navigator.appVersion.match(/iphone/gi);
        var devicePixelRatio = win.devicePixelRatio;
        if (isIPhone) {
            // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
            if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {
                dpr = 3;
            } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)) {
                dpr = 2;
            } else {
                dpr = 1;
            }
        } else {
            // 其他设备下，仍旧使用1倍的方案
            dpr = 1;
        }
        scale = 1 / dpr;
    }

    docEl.setAttribute('data-dpr', dpr);
    if (!metaEl) {
        metaEl = doc.createElement('meta');
        metaEl.setAttribute('name', 'viewport');
        metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
        if (docEl.firstElementChild) {
            docEl.firstElementChild.appendChild(metaEl);
        } else {
            var wrap = doc.createElement('div');
            wrap.appendChild(metaEl);
            doc.write(wrap.innerHTML);
        }
    }

    function refreshRem() {
        var width = docEl.getBoundingClientRect().width;
     
        var rem = width / 10;

        if(window.__max_html_width_) {
            rem = Math.min(__max_html_width_, rem);
        }

        docEl.style.fontSize = rem + 'px';
        flexible.rem = win.rem = rem;
    }

    win.addEventListener('resize', function() {
        clearTimeout(tid);
        tid = setTimeout(refreshRem, 300);
    }, false);
    win.addEventListener('pageshow', function(e) {
        if (e.persisted) {
            clearTimeout(tid);
            tid = setTimeout(refreshRem, 300);
        }
    }, false);

    if (doc.readyState === 'complete') {
        doc.body.style.fontSize = 12 * dpr + 'px';
    } else {
        doc.addEventListener('DOMContentLoaded', function(e) {
            doc.body.style.fontSize = 12 * dpr + 'px';
        }, false);
    }


    refreshRem();

    flexible.dpr = win.dpr = dpr;
    flexible.refreshRem = refreshRem;
    flexible.rem2px = function(d) {
        var val = parseFloat(d) * this.rem;
        if (typeof d === 'string' && d.match(/rem$/)) {
            val += 'px';
        }
        return val;
    }
    flexible.px2rem = function(d) {
        var val = parseFloat(d) / this.rem;
        if (typeof d === 'string' && d.match(/px$/)) {
            val += 'rem';
        }
        return val;
    }

})(window, window['lib'] || (window['lib'] = {}));
```







