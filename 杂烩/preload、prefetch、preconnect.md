### 1、Preload

preload 提供了一种声明式的命令，让浏览器提前加载指定资源(加载后并不执行)，在需要执行的时候再执行。最好使用 preload 来加载你最重要的资源，比如图像，CSS，JavaScript 和字体文件。
使用方式：
```
<!-- 1、link 标签静态标记-->
<link rel="preload" href="/path/to/style.css" as="style">

<!-- 2、脚本动态创建link标签后插入到 head 头部 -->
<script>
  const link = document.createElement('link');
  link.rel = 'preload';
  link.as = 'style';
  link.href = '/path/to/style.css';
  document.head.appendChild(link);
</script>

<!-- 3、HTTP 响应头的 Link 字段创建 -->
Link: <https://example.com/other/styles.css>; rel=preload; as=style
```

Preload 与 prefetch 不同的地方就是 Preload 专注于当前的页面，并以高优先级加载资源，Prefetch 专注于下一个页面将要加载的资源并以低优先级加载。同时也要注意 preload 并不会阻塞 window 的 onload 事件。

在不支持 preload 的浏览器环境中，会忽略对应的 link 标签，而若需要做特征检测的话，则：
```
const isPreloadSupported = () => {
  const link = document.createElement('link');
  const relList = link.relList;
  if (!relList || !relList.supports) {
    return false;
  }
  return relList.supports('preload');
};
```

### 2、Prefetch

Prefetch 是一个低优先级的资源提示，允许浏览器在后台（空闲时）获取将来可能用得到的资源，并且将他们存储在浏览器的缓存中。当用户点击了一个带有 prefetched 的连接，它将可以立刻从缓存中加载内容。有三种不同的 prefetch 的类型，link，DNS 和 prerendering

#### Link Prefetching

浏览器会寻找 HTML <link> 元素或者 HTTP 头中的 prefetch，然后获取资源并将他们存储在缓存中。
```
<!-- HTML -->
<link rel="prefetch" href="/uploads/images/pic.png">
<!--  HTTP Header -->
Link: </uploads/images/pic.png>; rel=prefetch
```

#### DNS Prefetching

DNS prefetching 允许浏览器在用户浏览页面时在后台运行 DNS 的解析。DNS 的解析在用户点击一个链接时已经完成，所以可以减少延迟。可以在一个 link 标签的属性中添加 rel="dns-prefetch' 来对指定的 URL 进行 DNS prefetching
```
<link rel="dns-prefetch" href="//fonts.googleapis.com">
<link rel="dns-prefetch" href="//www.google-analytics.com">
```

####  Prerendering

Prerendering 和 prefetching 非常相似，它们都优化了可能导航到的下一页上的资源的加载，区别是 prerendering 在后台渲染了整个页面，整个页面所有的资源。如下：

```
<link rel="prerender" href="https://www.keycdn.com">
```

### 3、Preconnect

preconnect 允许浏览器在一个发送 HTTP 请求前预先执行一些操作，这包括 DNS 解析，TLS 协商，TCP 握手。

preconnect 可以直接添加到 HTML 中 link 标签的属性中或者通过 JavaScript 生成，也可以写在 HTTP 头中。
```
<link href="https://cdn.domain.com" rel="preconnect" crossorigin>
```
注意点：

避免混用 preload 和 prefetch：混用 preload 和 prefetch ，并不会复用资源，而是会重复加载。

避免错用 preload 加载跨域资源：
对跨域的文件进行 preload 的时候，我们必须加上 crossorigin 属性，否则会加载这个文件两次。

Preload links for CORS enabled resources, such as fonts or images with a crossorigin attribute, must also include a crossorigin attribute, in order for the resource to be properly used.