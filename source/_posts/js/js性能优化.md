---
title: 雅虎前端优化规则
date: 2020-04-01 11:28:59
---

如何让web页面更快，雅虎团队实践总结了7类35条规则

<!-- more -->

## 1 Content

### 1.1 减少/最小化 http 请求数

Minimize HTTP Requests
到终端用户的响应时间80%花在前端：大部分用于下载组件（js/css/image/flash等等）。减少组件数就是减少渲染页面所需的http请求数。这是更快页面的关键。

### 1.2 减少DNS查询

Reduce DNS Lookups

### 1.3 避免跳转

Avoid Redirects

### 1.4 让Ajax可缓存

Make Ajax Cacheable

+ gzip组件
+ 减少DNS查找
+ 压缩JS
+ 避免跳转
+ 设置ETags

### 1.5 延迟加载组件

Post-load Components

再看看你的页面然后问问自己，“什么是页面初始化必须的？”。剩下的内容和组件可以延迟。

JavaScript是理想的（延迟）候选者，可以切分到onload事件之前和之后。比如拖放的js库可以延迟，因为拖动必须在页面初始化之后。其它可延迟的包括隐藏的内容，折叠起来的图片等等。

### 1.6 预加载组件

Preload Components

有几种预加载类型：

+ 无条件预加载：一旦onload触发，你立即获取另外的组件。比如谷歌会在主页这样加载搜索结果页面用到的雪碧图。
+ 有条件预加载：基于用户动作，你推测用户下一步会去哪里并加载相应组件。
+ 预期的预加载：在发布重新设计（的网站）前提前加载。在旧网页预加载新网页的部分组件，那么切换到新网页时就不会是没有任何缓存了。

### 1.7 减少dom数

Reduce the Number of DOM Elements

一个复杂的页面意味着更多的内容要下载，以及更慢的dom访问。比如在有500dom数量的页面添加事件处理就和有5000dom数量的不同。

如果你的页面dom元素很多，那么意味着你可能需要删除无用的内容和标签来优化。

### 1.8 把组件分散到不同的域名

Split Components Across Domains

把组件分散到不同的域名允许你最大化并行下载数。由于DNS查询的副作用，最佳的不同域名数是2-4。

### 1.9 最小化iframe的数量

Minimize the Number of iframes

\<iframe>优点：

+ 帮助解决缓慢的第三方内容的加载，如广告和徽章
+ 安全沙盒
+ 并行下载脚本
  
\<iframe>缺点：

即使空的也消耗（资源和时间）
阻塞了页面的onload
非语义化（标签）

### 不要404

No 404s

http请求是昂贵的，所以发出http请求但获得没用的响应（如404）是完全不必要的，并且会降低用户体验。

一些网站会有特别的404页面提高用户体验，但这仍然会浪费服务器资源。特别坏的是当链接指向外部js但却得到404结果。这样首先会降低（占用）并行下载数，其次浏览器可能会把404响应体当作js来解析，试图从里面找出可用的东西。

## Server

### 使用CDN

Use a Content Delivery Network

### 加Expires或者Cache-Control头部

Add an Expires or a Cache-Control Header

这条规则有两个方面：

+ 对静态组件：通过设置Expires头部来实现“永不过期”策略。
+ 对动态组件：用合适的Cache-Control头部来帮助浏览器进行有条件请求。

页面越来越丰富，意味着更多脚本，样式，图片等等。第一次访问的用户可能需要发出多个请求，但使用Expires可以让这些组件被缓存。这避免了访问子页面时没必要的http请求。Expires一般用在图片上，但应该用在所有的组件上。

浏览器（以及代理）使用缓存来减少http请求数，加快页面加载。服务器使用http响应的Expires头部来告诉客户端一个组件可以缓存多久。比如下面：

Expires: Thu, 15 Apr 2010 20:00:00 GMT //2010-04-15之前都是稳定的
注意，如果你设置了Expires头部，当组件更新后，你必须更改文件名。

### 传输时用gzip等压缩组件

Gzip Components

从HTTP/1.1开始，客户端通过http请求中的Accept-Encoding头部来提示支持的压缩：

Accept-Encoding: gzip, deflate

如果服务器看到这个头部，它可能会选用列表中的某个方法压缩响应。服务器通过Content-Encoding头部提示客户端：

Content-Encoding: gzip

gzip一般可减小响应的70%。尽可能去gzip更多（文本）类型的文件。html，脚本，样式，xml和json等等都应该被gzip，而图片，pdf等等不应该被gzip，因为它们本身已被压缩过，gzip它们只是浪费cpu，甚至增加文件大小。

### 实体标记

Configure ETags

是服务器和浏览器之间判断浏览器缓存中某个组件是否匹配服务器端原组件的一种机制。实体就是组件：图片，脚本，样式等等。ETag被当作验证实体的比最后更改（last-modified）日期更高效的机制
服务器这样设置组件的ETag：

```md
HTTP/1.1 200 OK
Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
ETag: "10c24bc-4ab-457e1c1f"
Content-Length: 12195
```

之后，如果浏览器要验证组件，它用If-None-Match头部来传ETag给服务器。如果ETag匹配，服务器返回304：

```md
GET /i/yahoo.gif HTTP/1.1
Host: us.yimg.com
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
If-None-Match: "10c24bc-4ab-457e1c1f"
HTTP/1.1 304 Not Modified
```

ETag的问题是它们被构造来使它们对特定的运行这个网站的服务器唯一。浏览器从一个服务器获取组件，之后向另一个服务器验证，ETag将不匹配。然而服务器集群是处理请求的通用解决方案。

### 早一点刷新buffer（尽早给浏览器数据）

Flush the Buffer Early

当用户请求一个页面，服务器一般要花200-500ms来拼凑整个页面。这段时间，浏览器是空闲的（等数据返回）。在php，有个方法flush()允许你传输部分准备好的html响应给浏览器。这样的话浏览器就可以开始下载组件，而同时后台可以继续生成页面剩下的部分。这种好处更多是在忙碌的后台或轻前端网站可以看到。

一个比较好的flush的位置是在head之后，因为浏览器可以加载其中的样式和脚本文件，而后台继续生成页面剩余部分。

```php
<!-- css, js -->
</head>
<?php flush(); ?>
<body>
<!-- content -->
```

### ajax请求用get

Use GET for AJAX Requests

Yahoo! Mail团队发现当使用XMLHttpRequest，POST 被浏览器实现为两步：首先发送头部，然后发送数据。所以使用GET最好，仅用一个TCP包发送（除非cookie太多）。IE的url长度限制是2K。

POST但不提交任何数据根GET行为类似，但从语义上讲，获取数据应该用GET，提交数据到服务器用POST

### 避免空src的图片

Avoid Empty Image src

浏览器会向你的服务器发请求。

+ IE，向页面所在的目录发请求。
+ Safari和Chrome，请求实际的页面。
+ FireFox3及之前和Safari/Chrome一样，但从3.5开始修复问题，不再发请求。
+ Opera遇到空图片src不做任何事。

这种行为的根源是uri解析发生在浏览器。RFC 3986 定义了这种行为，空字符串被当作相对路径，Firefox, Safari, 和 Chrome都正确解析，而IE错误。总之，浏览器解析空字符串为相对路径的行为被认为是符合预期的。

html5在_4.8.2_添加了对标签src属性的描述，指导浏览器不要发出额外的请求。

幸运的是将来浏览器不会有这个问题了（在图片上）。不幸的是，\<script src="">和\<link href="">没有这样的规范。

## Cookie

### 减小cookie的大小

Reduce Cookie Size

http cookie的使用有多种原因，比如授权和个性化。cookie的信息通过http头部在浏览器和服务器端交换。尽可能减小cookie的大小来降低响应时间。

+ 消除不必要的cookie。
+ 尽可能减小cookie的大小来降低响应时间。
+ 注意设置cookie到合适的域名级别，则其它子域名不会被影响。
+ 正确设置Expires日期。早一点的Expires日期或者没有会尽早删除cookie，优化响应时间。

### 用没有cookie的域名提供组件

Use Cookie-free Domains for Components

当浏览器请求静态图片并把cookie一起发送到服务器时，cookie此时对服务器没什么用处。所以这些cookie只是增加了网络流量。所以你应该保证静态组件的请求是没有cookie的。可以创建一个子域名来托管所有静态组件

## CSS

### 把样式放在顶部

Put Stylesheets at the Top

研究雅虎网页性能时发现把样式表移到\<head>里会让页面更快。这是因为把样式表移到\<head>里允许页面逐步渲染。

把样式表放在文档底部的问题是它阻止了许多浏览器的逐步渲染，包括IE。这些浏览器阻止渲染来避免在样式更改时需要重绘页面元素。所以用户会卡在白屏。

### 避免CSS表达式

Avoid CSS Expressions

CSS表达式的问题是它们可能比大多数人预期的计算的更频繁。它们不仅在页面载入和调整大小时重新计算，也在滚动页面甚至是用户在页面上移动鼠标时计算。比如在页面上移动鼠标可能轻易计算超过10000次。

### 选择\<link>而不是@import

Choose \<link> over @import

### 避免使用（IE）过滤器

## JavaScript

### 把脚本放到底部

Put Scripts at the Bottom

脚本引起的问题是它们阻塞了并行下载。HTTP1.1规范建议浏览器每个域名下不要一次下载超过2个组件。如果你的图片分散在不同服务器，那么你能并行下载多个图片。但当脚本在下载，浏览器不会再下载其它组件，即使在不同域名下。

有些情况下把脚本移动到底部并不简单。比如，脚本中用了document.write来插入内容，它就不能被移动到底部。另外有可能有作用域问题。但大多数情况，有方法可以解决这些问题。

一个替代建议是使用异步脚本。defer属性表明脚本不包含document.write，是提示浏览器继续渲染的线索。不幸的是，Firefox不支持。如果脚本能异步，那么也就可以移动到底部。

### 使用外部JS和CSS

Make JavaScript and CSS External

真实世界中使用外部文件一般会加快页面，因为JS和CSS文件被浏览器缓存了。內连的JS和CSS怎在每次HTML文档下载时都被下载。內连减少了http请求，但增加了HTML文档大小。另一方面，如果JS和CSS被缓存了，那么HTML文档可以减小大小而不增加HTTP请求。

核心因素，就是JS和CSS被缓存相对于HTML文档被请求的频率。尽管这个因素很难被量化，但可以用不同的指标来计算。如果网站用户每个session有多个pv，许多页面重用相同的JS和CSS，那么有很大可能用外部JS和CSS更好。

### 压缩JS和CSS

Minify JavaScript and CSS

压缩就是删除代码中不必要的字符来减小文件大小，从而提高加载速度。当代码压缩时，注释删除，不需要的空格（空白，换行，tab）也被删除。
混淆是对代码可选的优化。它比压缩更复杂，并且可能产生bug。在对美国top10网站的调查，压缩可减小21%，而混淆可减小25%。
除了外部脚本和样式，內连的脚本和样式同样应该被压缩。

### 删除重复的脚本

Remove Duplicate Scripts

### 最小化DOM访问

Minimize DOM Access

用JS访问DOM元素是缓慢的，所以为了响应更好的页面，你应该：

+ 缓存访问过的元素的引用
+ 在DOM树外更新节点，然后添加到DOM树
+ 避免用JS实现固定布局

### 开发聪明的事件处理

Develop Smart Event Handlers

有时候页面看起来不那么响应（响应速度慢），是因为绑定到不同元素的大量事件处理函数执行太多次。这是为什么使用_事件委托_是一种好方法。

另外，你不必等到onload事件来开始处理DOM树，DOMContentLoaded更快。大多时候你需要的只是想访问的元素已在DOM树中，所以你不必等到所有图片被下载。

## Images

### 优化图片

Optimize Images

在设计师建好图片后，在上传图片到服务器前你仍可以做些事：

+ 检查gif图片的调色板大小是否匹配图片颜色数。
+ 可以把gif转成png看看有没有变小。除了动画，gif一般可以转成png8。
+ 运行pngcrush或其它工具压缩png。
+ 运行jpegtran或其它工具压缩jpeg。

### 优化CSS雪碧图

Optimize CSS Sprites

+ 把图片横向合并而不是纵向，横向更小。
+ 把颜色近似的图片合并到一张雪碧图，这样可以让颜色数更少，如果低于256就可以用png8.
+ "Be mobile-friendly"并且合并时图片间的间距不要太大。这对图片大小影响不是太大，但客户端解压时需要的内存更少。100×100是10000个像素，1000×1000是1000000个像素。

### 不要在html中缩放图片

Don't Scale Images in HTML

不要因为你可以设置图片的宽高就去用比你需要的大得多的图片。如果你需要

```html
<img width="100" height="100" src="mycat.jpg" alt="My Cat" /> 
```

那么，就用100x100px的图片，而不是500x500px的。

### favicon.ico小且缓存

Make favicon.ico Small and Cacheable

favicon.ico是在你服务器根路径的图片。邪恶的是即使你不关心它，浏览器仍然会请求它。所以最好不要响应404。另外由于在同一服务器，每次请求favicon.ico时也会带上cookie。这个图片还会影响下载顺序，比如在IE，如果你在onload时下载额外的组件，fcvicon会在这些组件之前被下载。

+ 小，最好1K以下
+ 设置Expires头部。也许可以安全地设置为几个月。

## Mobile

### 保持组件小于25K

Keep Components under 25K

这个限制与iPhone不缓存大于25K的组件相关。注意，这是非压缩（uncompressed）的文件大小。在这里minification（压缩，不要与compress混淆）很重要，因为gzip无法满足（iPhone）。

### 打包组件到一个多部父文档

Pack Components into a Multipart Document

打包组件到一个多部父文档类似于带附件的邮件。它帮助你在一个http请求中获取多个组件，但注意，iPhone不支持。