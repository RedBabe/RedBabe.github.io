## 1. 开始

-

## 2. 性能优化的指标和工具

### Chrome DevTools

#### Network panel

在Chrome的Network面板，可以看到waterfall(瀑布图)，上面会打印资源的加载信息：

```t
# CONNECTION START
Queued: 资源排队时间
DNS Lookup: DNS查找，就是由域名查找到IP
Initial connection: TCP建立连接
SSL: https请求的SSL协商

# REQUEST/RESPONSE
Request sent: 发送请求的时间
Waiting(TTFB): 请求发出到请求返回（在这个期间，页面是白屏）
Content Download: 内容下载时间
```

Network中，**蓝线**是DOM加载完成的时间，**红线**是页面资源加载完成的时间

> 可以通过`Save all as HAR with content`来保存Network面板信息为`har`文件。

#### Lighthouse

-

> **lighthouse也可以通过npm安装使用**，npm i -g lighthouse

#### 其他

异步请求要快，多快呢？**1s以内**

`command + shift + p`：搜索*frame*，找到`show frames per second (FPS) meter`，查看页面的FPS

搜索`block`，找到`show request blocking`，可以阻止指定的文件加载(可以用正则)

### RAIL模型(页面的理想优化结果)

> Response(页面对用户的响应): 处理事件要在50ms以内完成
>
> Animation: 每10ms产生一帧
>
> Idle: 尽可能增加空闲时间
>
> Load: 在5s内完成内容家在并可以交互
>
> 在Performance面板可以看到主线程(Main)的详情，以优化主线程的空闲时间(Idle)

### WebPageTest

> [webpagetest](https://webpagetest.org/)
>
> 多测试地点、全面性能报告

也可以本地安装：

```bash
docker pull webpagetest/server
docker pull webpagetest/agent
```

测试：

```bash
docker run -d -p 4000:80 webpagetest/server
docker run -d -p 4001:80 --network="host" -e "SERVER_URL=http://localhost:4000/work" -e "LOCATION=Test" webpagetest/agent
```

本地安装自己查吧

### 利用浏览器API

#### 获取网站的可交互时间

> DNS 解析耗时: domainLookupEnd - domainLookupStart
> TCP 连接耗时: connectEnd - connectStart
> SSL 安全连接耗时: connectEnd - secureConnectionStart
> 网络请求耗时 (TTFB): responseStart - requestStart
> 数据传输耗时: responseEnd - responseStart
> DOM 解析耗时: domInteractive - responseEnd
> 资源加载耗时: loadEventStart - domContentLoadedEventEnd
> First Byte时间: responseStart - domainLookupStart
> 白屏时间: responseEnd - fetchStart
> 首次可交互时间: domInteractive - fetchStart
> DOM Ready 时间: domContentLoadEventEnd - fetchStart
> 页面完全加载时间: loadEventStart - fetchStart
> http 头部大小： transferSize - encodedBodySize
> 重定向次数：performance.navigation.redirectCount
> 重定向耗时: redirectEnd - redirectStart

```js
window.addEventListener('load', () => {
  // 
  let timing = performance.getEntriesByType('navigation')[0];
  // time to interactive 可交互时间
  let tti = timing.domInteractive - timing.fetchStart;
})
```

#### 监听长任务（耗时长）

```js
// 创建一个监听器
let observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.log(entry);
  }
});
observer.observe({
  // 选择监听的类型：long tasks
  entryTypes: ['longtask']
});
```



#### 监听用户是否在当前页面

```js
// 页面离开事件在不同的浏览器有不同的实现
let vEvent = 'visibilitychange';
if (document.webkitHidden !== undefined) {
  // webkit 事件名称
  vEvent = 'webkitvisibilitychange';
}

function handleVisibilityChange() {
  // 页面不可见
  if (document.hidden || document.webkitHidden) {
    console.log('Web page is hidden');
  } else {
    console.log('Web page is visible');
  }
}

document.addEventListener(vEvent, handleVisibilityChange, false);
```

#### 获取用户网络状态

```js
const connection = navigator.connection || navigator.mozConnection || navigator.webkitConnection;
function updateConnectionStatus() {
	const type = connection.effectiveType;
  console.log('Connection type changed to ' + type);
}
connection.addEventLlistener('change', updateConnectionStatus);
```

## 3. 渲染优化

### 浏览器渲染原理和关键渲染路径(critical rendering path)

> 关键渲染路径：
>
> JavaScript (利用JS/Web animation API等更新页面)
>
> -> Style (重新计算样式)
>
> -> Layout (布局)
>
> -> Paint (绘制)
>
> -> Composite (把页面拆分图层进行绘制再进行复合：所以动画最好使用css3 transform)

对html文件：构建DOM

对css文件：构建CSSOM

合并为：Render Tree

### 回流(reflow)与重绘(repaint)

> 当Render Tree的一部分需要重新构建，称为**回流**。
>
> 当Render Tree的一部分需要更新外观，而不影响布局，称为**重绘**。

#### 影响回流的因素

添加/删除元素

操作styles

display: none

offsetLeft, scrollTop, clientWidth

移动元素位置

修改浏览器大小，字体大小

#### 避免布局抖动(layout thrashing)

> 通过Performance中，onLoad之后检查layout变化。
>
> 要消除带有红色标记的Long tasks

#### 减少回流

尽可能避免上述情况。

如果要进行动画，尽量使用CSS3属性，而不是`position left`。

### FastDom

借助fastdom使得读（如读取元素offsetTop）写（如修改元素width）分离

### 避免重绘

> 在ChromeDevTools搜索`render`，找到`show rendering`，勾选`Paint flashing`

总结：多用`transform`和`opacity`。

对于要使用`transform`的元素，可以给它们的父盒子加上`will-change: transform;`，来告诉浏览器这个盒子要放到一个单独的图层中（最好是动画完毕后去掉这个属性，比如hover时加上，unhover时去掉）。

要不要加这个属性，可以多测测。

### Life of a frame

> Input events
>
> JS
>
> Begin frame
>
> rAF
>
> Layout
>
> Paint

比如做一个简单交互动画（卡顿版）

```js
window.addEventListener('pointermove', e => {
  const posX = e.clientX;
  // 给图片设置宽
});
```

升级版

```js
let ticking = false;
window.addEventListener('pointermove', e => {
  const posX = e.clientX;
  if (ticking) return;
  ticking = true;
  window.requestAnimationFrame(() => {
    // 给图片设置宽
    ticking = false;
  });
});
```

## 4. 代码优化

### v8引擎

> 源码 -> 抽象语法树 -> 字节码bytecode -> 机器码
>
> 编译过程会进行优化。如果优化是错误的，就会进行反优化。

### 变量

尽量不要修改变量的类型。这样会极大增加v8引擎的反优化过程。

### 函数

函数定义都是懒解析的(lazy parsing)的。但是有时候定义完函数就立刻使用，需要让v8引擎对其先懒解析再饥饿解析(eager parsing)。

实现方法就是：在函数定义时最外层加上括号，如`const fun = (() => {})`

由于打包时一些打包工具或插件会自动把括号去掉，所以可以使用`optimize-js`库。

### html优化

#### 减少iframes使用

它有自己的加载过程，还会阻塞父文档的加载。

使用优化（延迟加载iframe）：

```html
<iframe id="iframe"></iframe>
<script>
  // 当页面加载完毕后，再加载iframe
	function loadIframe() {
    document.getElementById('iframe').setAttribute('src', 'url');
  }
</script>
```

#### 压缩空白符和删除注释

使用某些库

#### 避免节点深层级嵌套

节点少点还是好

#### 避免table布局

使用不灵活，开销很大

#### css和js尽量外链

除非做首屏优化

#### 删除元素默认属性

默认属性写了跟没写是没有区别的

#### 借助工具

如`html-minifier`

### CSS

`contain`属性：[介绍](https://www.webhek.com/post/css-contain-property.html)

`font-display`属性：下一小节会介绍

## 5. 资源优化

### 资源压缩

html压缩：html-minifier

css压缩：clean-css

js压缩和混淆：webpack

### 图片优化

jpg格式图片的特点：有损压缩，色彩好。压缩工具：imagemin

png格式图片的特点：无损，可以做图标。压缩工具：imagemin-pngquant

webP

svg

### 图片加载优化

原生懒加载：`<img loading="lazy">`的属性

库：`react-lazy-load-image-component`, `verlok/lazyload`, `yall.js`, `Blazy`

### 渐进式图片

> Baseline JPEG: Loads from top-to-bottom
>
> Progressive JPEG: Loads from low-quality to high-quality

自己制作渐进式图片的工具：`progressive-image`, `ImageMagick`, `libjpeg`, `jpegtran`, `jpeg-recompress`, `imagemin`

### 响应式图片

> vw是view port width

使用`img`的`srcset`属性

```html
<img src="p-200.jpg" sizes="100vw" srcset="p-100.jpg 100w, p-200.jpg 200w, p-500.jpg 500w"
```

还可以使用`<picture>`。

### 字体问题

`FOIT`: flash of invisible text

`FOUT`: flash of unstyled text

#### 使用 `font-display` 属性

`auto`: 自动选择

`block`: 3s内不显示文字(FOIT)，3s后如果还未下载完，显示浏览器默认字体(FOUT)，直至下载完毕

`swap`: 直接用浏览器默认字体显示，直至下载完毕

`fallback`: 100ms内不显示文字，100ms后如果还未下载完，显示浏览器默认字体，直至下载完毕

`optional`: 100ms内不显示文字，判断用户网络速度进行选择：使用用户字体 / 浏览器默认字体。这个不会更改了

自定义@font-face：略

## 6. 构建优化

> 使用webpack时进行的优化

### Tree-shaking

条件：上下文未用到的代码(dead code)，且是以ES6语法导出导入的语法

可以在`package.json`中设置`sideEffects`属性，来使得`webpack`的`terserPlugin`在做压缩工作使用`tree shaking`时，过滤掉一些指定类型的文件。如

```js
"sideEffects": [
  "*.css"
]
```

### 优化打包速度

#### noParse

通知webpack忽略较大的库，如`lodash`

#### dllPlugin

极大提高打包速度。一般在开发时使用

### Code splitting

> 代码拆分，以达到首屏加载优化的效果

在webpack.config.js中：

```js
{
  output: {
    path: `${__dirname}/build`,
    filename: '[name].[hash].bundle.js',
    // 8位的hash。其实默认长度(20)就ok
    chunkFilename: '[name].[chunkhash:8].bundle.js'
  },
  // 代码拆分的文件
	optimization: {
    splitChunks: {
      cacheGroups: {
        // vendor通常用来指第三方库
        vendor: {
          name: 'vendor',
          test: /[\\/]node_modules[\\/]/,
          // 最小的大小
          minSize: 0,
          // 最小的段数
          minChunks: 1,
          // 优先级
          priority: 10,
          // 这个字段表示支持的引入方式：initial支持静态引入，async支持动态引入，all支持两种
          chunks: 'initial'
        },
        common: {
          name: 'common',
          test: /[\\/]src[\\/]/,
          chunks: 'all'
        }
      }
    }
  },
  plugins: [
    // css提取插件
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
      chunkFilename: '[id].[contenthash:8].css'
    })
  ]
}
```

在React项目中，可以用Suspense和lazy实现动态引入

### 基于webpack的文件压缩(Minification)

Terser: JS
mini-css-extract-plugin: CSS（从js中提取css）

HtmlWebpackPlugin

### webpack检测和分析工具

#### webpack-chart

> [webpack-chart](https://alexkuz.github.io/webpack-chart/)

生成项目的`stats.json`文件，上传到网址就好了

#### source-map-explorer

> 它是一个npm库，可以直接安装
>
> 它基于项目的source-map文件，所以打包时需要生成source-map文件

#### webpack-bundle-analyzer

webpack的插件。没有上一个看的清晰

#### speed-measure-webpack-plugin

> 测量速度的插件

### React按需加载

> 粒度来说，通常按照路由来进行组件的按需加载

除了使用`React.Suspense`和`React.lazy`，还可以使用库`@loadable/component`，就是`react-router`的按需加载库。

## 7. 传输加载优化

### Gzip

这里使用nginx

```bash
brew install nginx
brew services start nginx
```

配置

```nginx
gzip on;
# 当文件达到1k时才压缩
gzip_min_length 1k;

# 文件压缩等级
gzip_comp_level 6;

# 压缩的文件类型
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/xml text/javascript application/json;

#
gzip_static on;
gzip_vary on;
gzip_buffers 4 16k;
gzip_http_version 1.1;
```

### Keep Alive

> 它对应的是`Network`的`Initial connection`(HTTP连接)。开启之后，只有第一个请求会有这个时间的消耗。

这个选项在HTTP1.1是默认开启的。

对`nginx`进行keep-alive相关参数配置

```nginx
# 延时为65s
keepalive_timeout 65;

# 一次最多可以发送100个请求
keepalive_requests 100;
```

### nginx做缓存

```nginx
if ($request_filename ~* .*\.(?:htm|html)$)
{
  # 禁止浏览器缓存(http1.1)
  add_header Cache-Control "no-cache, must-revalidate";
  # 兼容低版本http1.0
  # 禁止缓存
  add_header "Pragma" "no-cache";
  # 立即过期
  add_header "Expires" "0";
}
if ($request_filename ~* .*\.(?:js|css)$)
{
  expires 7d;
}
if ($request_filename ~* .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$)
{
  expires 7d;
}
```

`Etag`+`If-None-Match`在nginx是默认开启的。

`cache-control: max-age=0`：禁止缓存

`cache-control: private, no-store`: private是禁止二级缓存(只有用户浏览器可以缓存)；no-store是禁用任何形式的缓存，所以`etag`会失效，一定会从服务器拉取最新的内容

### Service Worker

> 可以为网页提供离线访问功能。
>
> 只能在localhost或https下访问。

在webpack中使用：`workbox-webpack-plugin`和`manifest-plugin`

#### http2

略

## 8. 优化方案

### flex布局

给子元素加`float: left`效率比较低，替代方案为：

给父元素加

```css
display: flex;
flex-flow: row wrap;
```

### 优化资源加载顺序

preload：指定当前页面优先加载的资源

> preload后，资源在Network显示的优先级仍是`low`
>
> preload字体时，必须要设置crossorigin="anonymous"

```html
<head>
  <link rel="preload" href="img/product.svg" as="image">
</head>
```

prefetch

> 页面所有事干完后，prefetch一个资源。它用来提前拉取下一个页面用的资源的

```html
<head>
  <link rel="prefetch">
</head>
```

webpack也有相应的配置。

### 预渲染

`react-snap`开箱即用

### 窗口化(Windowing)提高列表性能

`react-window`可以实现windowing

> **只渲染可见的行**，渲染和滚动的性能都会提升。它包括 **lazy loading 和 主动释放** 的两个过程。

`react-window`可以实现二维的windowing，即横向和纵向都滚动。

### 使用骨架组件(skeleton)减少布局移动

使用`react-placeholder`库

注意：在`Chrome DevTools`可以打开`Layout Shift Regions`，来查看布局变化。

## 9. 其他

### 首屏优化

什么是首屏？above the fold，报纸折叠的上面部分。

**首屏优化的几个名词：**

First Contentful Paint(FCP)

Largest Contentful Paint(LCP)

Time to Interactive(TTI)

**解决方案：**

资源压缩：html, css, js, img等

传输压缩：gzip

代码拆分：按照路由分出更多的js包，包括第三方库和自己写的懒加载组件

tree shaking：去掉无用代码

缓存：加速访问

**若首页内容太多：**

对组件或内容懒加载，或者预渲染，inline-css

### 垃圾回收(garbage collection)

局部变量：函数执行完毕时，如果没有闭包引用，则会被释放

全局变量：直至浏览器卸载页面才会释放

垃圾回收的两种方式：引用计数（可能会有循环引用的问题）；标记清除（现代浏览器基本都用这个方法）

