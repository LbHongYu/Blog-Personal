## 性能优化手段

### 资源请求
* 开启缓存
  ```
  Cache-Control/Max-age
  Last-Modified/If-Modified-Since
  ETag/If-None-Match
  ```
* 开启 dns-prefetch
  ```
  // 方式一：
  <link rel="dns-prefetch" href="https://fonts.googleapis.com/">

  // 方式二：
  Link: <https://fonts.gstatic.com/>; rel=dns-prefetch
  ```
  最佳实践
  在跨域源使用：dns-prefetch 仅对跨域域上的 DNS查找有效，因此请避免使用它来指向您的站点或域。
  与 preconnect 配用：将 dns-prefetch 与 preconnect(预连接)提示配对。尽管 dns-prefetch 仅执行 DNS查找，但 preconnect 会建立与服务器的连接。如果站点是通过HTTPS服务的，则此过程包括DNS解析，建立TCP连接以及执行TLS握手。将两者结合起来可提供进一步减少跨域请求的感知延迟的机会。preconnect 提示最好仅用于最关键的连接。
  ```
    <link rel="preconnect" href="https://fonts.gstatic.com/" crossorigin>
    <link rel="dns-prefetch" href="https://fonts.gstatic.com/">
  ```
* 开启 preload: 本页面使用的资源使用 preload，跨域资源要加上 crossorigin 属性否则会请求两次（请求优先级）。

* 开启 prefetch: 下一个页面要使用的资源使用 prefetch

* 开启 HTTP2

### 资源加载的优化
* 首页尽量减少不必要的资源加载
* 异步加载路由组件
* 将项目中引用的工具打入单独的 chunk用于异步加载
* 以“内容hash”命名打包的项目文件
* 使用 polyfill 服务，根据不同浏览器动态引入不同的polyfill
* 压缩图片、使用雪碧图、图片懒加载、小图片处理成base64、使用icon替代小图片

### 首屏优化：
* 尽量减少不必要的资源加载 （工具、CSS等等）
* 内敛样式
* loading 提示、Skeleton
* 使用本地缓存将可缓存的首页数据缓存起来

### 代码：
* 页面中暂时不用的功能不加载
* 防抖节流
* 缓存不会变化的数据
* 使用 keep-alive (Vue)
* 尽量避免重排重绘
* 移除没用的CSS

### webpack 打包优化
* 压缩 HTML\CSS\JS:
* HTML 中的 js(HtmlWebpackPlugin: args[0].minify.minifyJS = true;)、
* CSS(optimize-css-assets-webpack-plugin)、
* JS(TerserPlugin)

### 构建优化
* 使用 externals 属性，将某些包放在 cdn 上
* 使用 resolve.alias
* 使用 exclude 和 include缩小打包范围
* 使用 DllPlugin 分包
* babel-loader，开始多进程和缓存
* thread-loader， 放置在其他 loader 之前，在此 loader 之后的 loader 会在一个独立的 worker 池中运行。这些loader会有一些限制
  * 不能生成新的文件。
  * 不能使用自定义的 loader API（也就是说，不能通过插件来自定义）。
  * 无法获取 webpack 的配置
* cache-loader，在一些性能开销较大的 loader 之前添加此 loader，以将结果缓存到磁盘里
* terser-webpack-plugin 开启缓存
 * 使用 hard-source-webpack-plugin 为模块提供一个中间缓存步骤，提升二次构建速度