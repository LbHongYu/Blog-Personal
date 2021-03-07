## 性能优化手段

### 开启服务器的缓存、压缩、HTTP2
* Cache-Control / Max-age、Last-Modified/If-Modified-Since 与 ETag/If-None-Match
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
* 使用 keep-alive(Vue)
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