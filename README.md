# 移动端兼容问题

## 设置圆角(border-radius:50%;)部分手机显示为椭圆

### 原因：

使用 rem 做屏幕适配

### 解决方案：

设置圆角时，设置具体的数据，不用百分比的形式

## 安卓手机 line-height 和 height 相等,文案垂直不居中

### 原因：

推测可能是 Android 在排版计算的时候参考了 primyfont 字体的相关属性（即 HHead Ascent、HHead Descent 等），而 primyfont 的查找是看`font-family`里哪个字体在 fonts.xml 里第一个匹配上，而原生 Android 下中文字体是没有 family name 的，导致匹配上的始终不是中文字体，所以解决这个问题就要在`font-family`里显式申明中文，或者通过什么方法保证所有字符都 fallback 到中文字体

### 解决方案：

设置字体为系统字体，在不是要求一定使用特殊字体的情况下可以参考以下字体的设置([张鑫旭大神的配置方案](https://www.bilibili.com/video/BV1b54y1Z7pu?spm_id_from=333.999.0.0))

```css
// 通用设置
body {
    font-family: system-ui, —apple-system, Segoe UI, Roboto, Emoji, Helvetica, Arial,
        sans-serif;
}

// emoji字体
@font-face {
    font-family: Emoji;
    src: local("Apple Color Emojiji"), local("Segoe UI Emoji"), local(
            "Segoe UI Symbol"
        ), local("Noto Color Emoji");
    unicode-range: U+1F000-1F644, U+203C-3299;
}

// 衬线字体
.font-serif {
    font-family: Georgia, Cambria, "Times New Roman", Times, serif;
}

// 等宽字体
.font-mono {
    font-family: Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
        monospace;
}
```

## 安卓手机部分版本 input 的 placeholder 偏上

### 原因：

未知

### 解决方案：

设置 input 的 line-heigh 为 normal

```css
input {
    line-heigh: normal;
}
```

## 输入框在页面较底部时，安卓手机弹出的键盘会遮挡且点击原生键盘的关闭按钮收回键盘时，输入框没有失焦

### 原因：

未知

### 解决方案：

Element.scrollIntoView()和 Element.scrollIntoViewIfNeeded()方法让当前的元素滚动到浏览器窗口的可视区域内。

使用 Element.scrollIntoView()和 Element.scrollIntoViewIfNeeded()解决遮挡问题，监听输入框聚焦事件，调用上面的方法将激活的元素（输入框）滚动到可视区域

通过监听页面 resize 事件来解决点击原生键盘的关闭按钮收回键盘时，输入框没有失焦的问题

可参考以下代码（使用 vue 实现）

```vue
data() { return { originHeight: 0, isAndroid:
/Android/gi.test(navigator.userAgent) || /adr/gi.test(navigator.userAgent),
resizeTimer: null }; }, methods: { resizeFn() { //
防止部分手机触发两次resize事件导致无法拉起键盘 if (this.resizeTimer) return;
this.resizeTimer = setTimeout(() => { let resizeHeight =
document.documentElement.clientHeight || document.body.clientHeight; if
(this.originHeight > resizeHeight) { //
拉起键盘会有动画，所以需要加延时，否则不管用 setTimeout(() => { if
("scrollIntoView" in document.activeElement) {
document.activeElement.scrollIntoView(); } else {
document.activeElement.scrollIntoViewIfNeeded(); } }, 0); } else {
document.activeElement.blur(); document.removeEventListener("resize",
this.resizeFn); } clearTimeout(this.resizeTimer); }, 100); } }, mounted() { if
(this.isAndroid) { this.originHeight = document.documentElement.clientHeight ||
document.body.clientHeight; window.addEventListener("resize", this.resizeFn); }
}
```

## ios 手机父元素设置了 overflow:hidden 和 border-radius，子元素超出部分不隐藏

### 原因：

未知

### 解决方案：

在父元素加 transform: rotate(0deg)属性

```css
.father {
    transform: rotate(0deg);
}
```

## ios 手机页面滚动时动画停止

### 原因：

iOS 的事件处理机制有关，iOS 最先响应屏幕反应。响应顺序依次为 Touch——Media——Service——Core 架构，当用户只要触摸接触了屏幕之后，系统就会最优先去处理屏幕显示也就是 Touch 这个层级，然后才是媒体（Media），服务（Service）以及 Core 架构。所以说，当系统接收到 Touch 事件之后会优先响应，此时会暂停屏幕上包括 js、css 的渲染。这个时候不光是 css 动画不动了，哪怕页面没有加载完如果你手指头还停留在屏幕上那么页面也不会继续加载，直到你的手松开

### 解决方案：

给动画元素设置 transform: translate3d(0, 0, 0);

```css
.animation {
    transform: translate3d(0, 0, 0);
}
```

## ios 手机刘海屏和底部小黑条适配

### 原因：

苹果公司提出的安全区域概念（safe area），简单的说就是我们的移动端页面可操作区域应该避开刘海区域和小黑条，因为在这两处地方的操作是不会响应我们的页面，即如果我们的按钮在这两块区域范围，那我们的点击就不会触发按钮上的事件

### 解决方案：

官方给出的适配方案
iOS11 同时新增了一个特性，constant(safe-area-inset-\*)，这是 Webkit 的一个 CSS 函数，用于获取安全区域与边界的距离，有四个预定义的变量（单位 px）:

safe-area-inset-left：安全区域距离左边界距离，横屏时适配

safe-area-inset-right：安全区域距离右边界距离，横屏时适配

safe-area-inset-top：安全区域距离顶部边界距离，竖屏下刘海屏为 44px，iphone6 系列 20px，竖屏刘海适配关键

safe-area-inset-bottom：安全区域距离底部边界距离，竖屏下为 34px，竖屏小黑条适配关键

一般使用 safe-area-inset-top，safe-area-inset-bottom，

```html
// 让页面占满全屏 <meta name="viewport" content="viewport-fil=cover" />
```

```css
// 使用@supports查询机型是否支持constant()或env()实现兼容代码隔离，个别安卓也会成功进入这个判断，因此加上-webkit-overflow-scrolling: touch的判断可以有效规避安卓机。
env()
    是为了防止大于IOS11版本不支持constant()
    @supports
    ((height: constant(safe-area-inset-top)) or (height: env(safe-area-inset-top)))
    and
    (-webkit-overflow-scrolling: touch) {
    .fullscreen {
        /* 适配齐刘海 */
        padding-top: constant(safe-area-inset-top);
        padding-top: env(safe-area-inset-top);

        /* 适配底部小黑条 */
        padding-bottom: costant(safe-area-inset-bottom);
        padding-bottom: env(safe-area-inset-bottom);
    }
}
```

## ios 手机滚动卡顿

### 原因：

未知

### 解决方案：

给滚动的区域设置-webkit-overflow-scrolling: touch 属性

```css
.scroll {
    overflow: scroll;
    -webkit-overflow-scrolling: touch;
}
```

## ios 手机快速滚动页面卡死

### 原因：

未知

### 解决方案：

使用[better-scroll](https://better-scroll.gitee.io/docs/zh-CN/)插件做页面的滚动

项目使用 vue 框架的也可以使用我封装的一个滚动组件[scroll-view](https://www.npmjs.com/package/@husandao/scroll-view)，具备基本的滚动功能，同时增加了下拉刷新和上拉加载的扩展功能

## ios 手机最后一个子元素设置 margin-bottom 无效

### 原因：

未知

### 解决方案：

将 margin-bottom 改成 padding-bottom

## ios 手机上输入框无法选中聚焦

### 原因：

手贱设置了全部元素无法被选中（需求上不允许用户复制图片和文字）

```css
* {
    webkit-user-select: none;
}
```

### 解决方案：

输入框元素不设置这个属性，如果还存在问题，尝试在输入框和其父元素设置-webkit-user-select:text !important 属性

## ios 系统 14 以上，设置特殊字体（UI 提供的用在数字上的字体），钱币符号和数字设置相同的颜色，页面显示的颜色不一样

### 原因：

字体原因

### 解决方案：

1.使用系统字体

2.设置颜色时透明度设置为 0.99

```css
p {
    color: rgba(#b96e16, 0.99);
}
```

## ios 系统 14 以上，使用 transform 实现动画时，会出现闪屏的情况

### 原因：

未知

### 解决方案：

在做动画的元素的父元素设置 transform: translate3d(0, 0, 1);
backface-visibility: hidden;

```css
.father {
    transform: translate3d(0, 0, 1);
    backface-visibility: hidden;
}
```

## ios 手机，图片使用 transform rotateY()不显示的问题

### 原因：

猜测是 ios 手机又视角概念，使用 transform rotateY()旋转后，视角出现问题

### 解决方案：

在父元素增加 perspective: 1;

```css
.father {
    perspective: 1;
}
```

## <div id="iosInput">ios 手机，输入框聚焦不灵敏([推荐使用另一方案解决](#delay))</div>

### 原因：

为解决移动端 300ms 延迟的问题，我们引入了 fastclick.js 这个库，ios11.3 对事件支持 {passive: false}被动模式，passive: false 会永远不调用 event.preventDefault()，如果调用客户端会报错，fastclick 是采用拦截 click 和监听 touch 事件去实现的，里面包括对 tagetElement 的 focus 方法重写，因此在 11.3 之前可能 event.preventDefault 生效了，同时用 setSelectionRange 是可以聚焦 input 的

### 解决方案：

```js
FastClick.attach(document.body);
FastClick.prototype.focus = function (targetElement) {
    let length;
    //兼容处理:在iOS7中，有一些元素（如date、datetime、month等）在setSelectionRange会出现TypeError
    //这是因为这些元素并没有selectionStart和selectionEnd的整型数字属性，所以一旦引用就会报错，因此排除这些属性才使用setSelectionRange方法
    if (
        targetElement.setSelectionRange &&
        targetElement.type.indexOf("date") !== 0 &&
        targetElement.type !== "time" &&
        targetElement.type !== "month"
    ) {
        length = targetElement.value.length;
        targetElement.setSelectionRange(length, length);
        targetElement.focus();
    } else {
        targetElement.focus();
    }
};
```

## transform 导致 z-index 失效

### 原因：

[元素样式包含 transform 时形成新的堆叠上下文](https://www.zhangxinxu.com/wordpress/2016/08/safari-3d-transform-z-index/)

### 解决方案：

1.父级，任意父级，非 body 级别，设置 overflow:hidden 可恢复和其他浏览器一样的渲染。

2.元素设置 transform: translateZ(120px)。

## 使用 border-image 后，border-radius 无效

### 原因：

未知

### 解决方案：

设置父级元素背景渐变，设置 padding，元素覆盖在上面营造成渐变边框的效果

## 块级元素内嵌套图片，图片上下不居中

### 原因：

未知

### 解决方案：

1.在 div 内设置 font-size 和行高为 0，使用 flex 布局居中

2.设置图片 diaplay：block

## 多个子元素设置为圆形时（测试结果为超过 3 个），或出现部分呈现椭圆形

### 原因：

未知

### 解决方案：

宽高等属性设置为两倍，然后缩放 0.5

## webview 前端上传图片被旋转

### 背景：

用户上传图片，前端将这张图片绘制到 canvas 画布上

### 问题：

绘制在 canvas 上的图片出现旋转(ios 版本大于等于 13.4.1 的手机不需要前端对其调整图片方向，无论倒着拍，还是旋转拍，图片上传后的方向都是正确的，所以需要对 ios 的版本进行判断)

### 原因：

在手机中默认横排才是正确的拍照姿势，如果我们手机竖着拿然后逆时针旋转 90° 这才是正确的拍照姿势，这时候拍出来的照片展示在 canvas 中是不会被旋转的。如果以其他角度拍搜时，就会发生旋转

### 解决方案：

```js
function imgToCanvasWithOrientation(img, width, height, orientation) {
    let canvas = document.createElement("canvas");
    let ctx = canvas.getContext("2d");
    canvas.width=width
    canvas.height=height
    if (判断机型系统不高于ios13.4.1) {
        switch (orientation) {
            case 3:
                ctx.rotate(Math.PI)
                ctx.drawImage(img, -width, -height, width, height);
                break;
            case 6:
                ctx.rotate(0.5 * Math.PI)
                ctx.drawImage(img, 0, -height, width, height);
                break;
            case 8:
                ctx.rotate(3 * Math.PI / 2)
                ctx.drawImage(img, -width, 0, width, height);
                break;
            default:
                ctx.drawImage(img, 0, 0, width, height);
                break
        }
    }

    return canvas;
}
```

## 浏览器置于后台（移动端表现为去聊微信了），倒计时不准问题

### 原因：

浏览器的“休眠”模式，页面未激活状态时，浏览器为节省性能，会停止或减少定时任务

### 解决方案：

使用 visibilitychange 监听页面是否可见（激活）去重新拉取后台时间，使用 setTimeout 去进行倒计时，setTimeout 会有误差，每次执行需要计算出减去误差后的时间作为下次执行的间隔

## ios 手机上 input 输入框设置 opacity 会导致聚焦拉起键盘时，不会把输入框挤到可视区域

### 原因：

设置 opacity 小于等于 0.01 时，就会出现这个问题

### 解决方案：

设置 opacity 大于 0.01

```css
.box {
    opacity: 0.011;
}
```

## <div id="delay">移动端 300ms 延迟问题</div>

### 原因：

历史包袱问题：以前网站都是为大屏幕电脑设计的，手机上预览就会导致内容被缩小了，为解决这个问题，就约定双击屏幕就将网页等比例放大缩小，如何判断用户是否是双击了呢？那就在首次点击后等待 300 毫秒，判断用户是否再次点击了屏幕，点击了就判断是双击。这也是会有上述 300 毫秒延迟的主要原因

### 解决方案：

1.在 HTML 文档头部添加如下 meta 标签,添加了 user-scalable=no 会禁止缩放

```html
<meta name="viewport" content="width=device-width,user-scalable=no" />
```

2.CSS touch-action 属性

```css
html {
    touch-action: none;
}
```

3.(不推荐使用，[作者在 github 有说明](https://github.com/ftlabs/fastclick)，并且还会有在[ios 手机上 input 输入框聚焦不灵敏的问题](#iosInput))使用 fastclick.js，这是一个轻量级库文件。我们引入库文件之后，添加如下代码就可以了。

```js
window.addEventListener(
    "load",
    function () {
        FastClick.attach(document.body);
    },
    false
);
```

## 设置元素溢出隐藏时（overflow:hidden），部分安卓手机会把文字的头部切掉

### 原因：

因为使用了 rem 的原因

### 解决方案：

给元素设置 line-height 的值大于设置的 font-size 的值

```css
.test {
    font-size: 20px;
    line-height: 24px;
}
```

## vite3 创建的 vue 项目本地开发服务在 ios12.1 及以下版本的手机上白屏（感谢叶同学提供此问题及解决方案）

### 原因：

globalThis 为 undefined 的原因，[globalThis 的兼容性](https://caniuse.com/?search=globalThis)

### 解决方案：

给在入口的 html 文件将 globalThis 指向 window

```html
<script>
    if (globalThis === undefined) {
        var globalThis = window;
    }
</script>
```

## ios 手机上将图片转成 base64 失败（感谢国梁同学提供此问题）

### 原因：

转换需要给图片设置允许跨域，但是在 ios 手机上允许跨域和给 src 赋值有顺序的区别

### 解决方案：

先给 Image 对象设置允许跨域，再给 Image 对象的 src 赋值

## vite 创建的项目使用可选链操作符（?.）本地启动服务在 ios13.4 以下版本的手机白屏报错

### 原因：

vite 在启动本地服务时，只处理语法转译不包含 polyfill，可选链操作符语法在 ios13.4 以下版本不支持，所以会报错白屏

### 解决方案：

使用*rollup-plugin-esbuild*将可选链操作符语法降级到兼容低版本浏览器。如果是生产环境使用*@vitejs/plugin-legacy*

```shell
npm install rollup-plugin-esbuild -D
```

```ts
// vite.config.ts
import vue from "@vitejs/plugin-vue";
import legacy from "@vitejs/plugin-legacy";
import esbuild from "rollup-plugin-esbuild";

export default defineConfig(({ command }) => {
    const plugins = [vue()];
    if (command === "build") {
        plugins.push(
            legacy({
                targets: ["Android >= 8", "iOS >= 10"],
            })
        );
    }
    if (command === "serve") {
        plugins.push(
            esbuild({
                target: "ios12",
                loaders: { ".vue": "js", ".ts": "js" },
            })
        );
    }
    return {
        plugins,
    };
});
```
