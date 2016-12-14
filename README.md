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

