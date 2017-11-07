
今天给大家分享一下H5开发，关于在微信和支付宝端开发期间遇到的坑和对应的解决办法。


## 手机端H5开发
* 手机端设计图
* meta
* 媒体查询@media
* 响应式布局单位
* 手机端常用库
* 愉快地踩坑吧
* 手机端页面调试技巧


## 手机端设计图

设计图都有个基准，一般iPhone5 、6 ，图是两倍图，除以2之后的尺寸就是实际页面里要显示的尺寸。


## meta

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=0" />
```

## 媒体查询@media

经常用来设置页面基准、不同屏幕大小下图片的适配、响应式页面布局。

```html
html {font-size:10px}
@media screen and (max-width:320px) {
	.div1{
	   background:url(xxx@2x.png) no-repeate;
	}
	.block-left{
	    width:100%;
	}
	.block-right{
	    width:100%;
	}
}
@media screen and (min-width:321px) and (max-width:375px) {
	.div1{
    	background:url(xxx@3x.png) no-repeate;
    }
   .block-left{
   	    width:30%;
   	}
   	.block-right{
   	    width:60%;
   	}
}
```


## 响应式布局单位

%     em     rem


## 【设置页面基准，两种方式：】

> 1、用@media：

```html
html {font-size:10px}
@media screen and (max-width:320px) {
	html {
		font-size: 13px
	}
}
@media screen and (min-width:321px) and (max-width:375px) {
	html {
		font-size: 15px
	}
}
```

> 2、用自执行函数（又名立即执行函数）

```javascript
(function (doc, win) {
    var docEl = doc.documentElement,
        resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
        recalc = function () {
            var clientWidth = docEl.clientWidth;
            if (!clientWidth) return;
            docEl.style.fontSize = 20 * (clientWidth / 375) + 'px';
        };
    if (!doc.addEventListener) return;
    win.addEventListener(resizeEvt, recalc, false);
    doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);
```

## 手机端常见样式重置

> 对于字体

IOS 、Android （以及 WinPhone），各个手机系统都有自己的默认字体，且都不支持微软雅黑，如无特殊需求，手机端无需定义中文字体，
使用系统默认即可，英文字体和数字字体可使用 Helvetica

```css
body{
   font-family:Helvetica;
}
```


> 在iOS上，去掉点击时灰色半透明背景色

```css
a,button,input,textarea{
    -webkit-tap-highlight-color: rgba(0,0,0,0)
}
```


> 在iOS上，输入框默认有内部阴影，但无法使用 box-shadow 来清除

```css
input,textarea {
	border: 0; /* 方法1 */
	-webkit-appearance: none; /* 方法2 */
}
```


## 手机端常用库

jquery
zepto.js   tap事件也是为了解决在click的延迟问题
fastclick   可以解决在手机上点击事件的300ms延迟
ontouchstart 、ontouchend  也可以加快触发
swiper.js   手机轮播图、上拉刷新、下拉加载等等


## 愉快地踩坑吧

> 支付宝中两个常见CSS坑

vw & vh  css视口单位 ，支付宝端失效，微信端正常。

页面里第一个非body元素设置margin-top会失效。微信端正常。


> flex & box & tabel + table-cell盒子布局

手机端不兼容flex 布局，要配合老版本流布局-webkit-box 一起用以做兼容

```css
.berth-block {
    width: 100%;
    height: 2.5rem;
    line-height: 2.5rem;
    margin-top: 0.6rem;
    display: -moz-box;
    display: -webkit-box;
}

.berth-block > span {
    display: block;
    width: 1px;
    -moz-box-flex: 1;
    -webkit-box-flex: 1;
    box-flex: 1;
}
```


> iconfont

iconfont 用脚本的方式载入页面，页面初始化之后会发现iconfont有的没有显示出来，高频重现率。

解决办法：把使用了字体图标的布局代码直接放html里。


> ios 系统时间格式化

ios系统中，用new Date() 格式化时间，传入参数带有"-"，如："2017-11-5"，则会返回 NaN ，查资料发现不支持 这种日期格式，
必须先把"-"改为"/"之后再格式化才会返回正常的值。

具体原因不太清楚，个人推测可能是ios的Safari浏览器不支持Date参数带"-"格式的写法。


> 【手机端禁用iframe标签】
容易被XSS（跨站脚本攻击）


> fixed定位 + input

问题：文本框聚焦的时候，fixed定位的布局会被键盘顶上来，失去焦点则还原。常用场景：搜索框和底部菜单同时存在

解决方案：

```javascript
//屏幕理论初始化高度
var initHeight = window.innerHeight;
//屏幕理论初始化宽度
var initWidth = window.innerWidth;
//屏幕初始状态0-竖屏,1-横屏
var initScreenState;
if (initWidth == window.screen.width) {
    initScreenState = 0;
}
else {
    initScreenState = 1;
}
//屏幕分辨率
var screenHeight = window.screen.height;
var screenWidth = window.screen.width;
//resize监听
$(window).resize(function () {
    //当前页面高度
    var nowHeight = window.innerHeight;
    var nowWidth = window.innerWidth;
    //竖屏
    if (nowWidth == screenWidth) {
        if (initScreenState == 1) {
            location.reload();
        }
    }
    //横屏
    else if (nowWidth == screenHeight) {
        if (initScreenState == 0) {
            location.reload();
        }
    }
    if (nowHeight != initHeight) {
        $(".nav-bottom-contain").hide();
        $("#authorizedSignBlock").hide();
    }
    else {
        $(".nav-bottom-contain").show();
        $("#authorizedSignBlock").show();
    }
});
```


> 关于手机端的支付
不管在哪里触发支付接口，要做请求限制，比如点击完显示loading，或者置灰，不然会造成因请求次数过多导致支付失败的情况。


## 手机端页面调试技巧

Chrome浏览器 手机模拟器模式

tomcat / nodejs简易web服务器 + ngrok + 二维码生成器

用alert代替console输出调试信息

调试支付：支付宝支付可在Chrome模拟器上直接调试支付，登录了支付宝账号即可。微信支付只能在外网下访问调试。


## 浏览器的控制台

console.log('%c吃吃吃%c货~~','font-size:60px;color:orange','font-size:25px;color:green');


