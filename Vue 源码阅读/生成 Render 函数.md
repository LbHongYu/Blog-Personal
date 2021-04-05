##  生成 Render 函数

调用Vue 构造函数，做了那些事情：
1. 在 Vue.prorotype 上定义方法(准备工作、event 相关、实例的更新销毁、创建Vnode)
2. 定义与platforms 相关的方法、是否带编译器
3. `Vue.prototype._init`、`compile ？`、`$mount(mountComponent)`、`_render`、`_update`、`__patch__`


render 函数是 template 经过编译从templet 到 ast 再到 render 函数。render 函数再用于生成 VNode。

步骤3里的  `compile`， 如果 option 中的是 render 函数而不是 template，就需要再编译了。平时的开发中，使用的是 Vue 单组件文件，经过 Vue-loader 处理之后就是一个包含组件信息的对象，其中的template 已经被编译称 render 函数。
```js
{
  components: {}
  created: ƒ ()
  data: ƒ ()
  methods: {}
  name: "comp-name"
  props: {prop1: {}}
  render: ƒ ()
  staticRenderFns: []
  watch: {}
  _Ctor: {0: ƒ}
  __file: "src/components/comp-1.vue"
  _compiled: true
  _scopeId: "data-v-330374fa"
  __proto__: Object
}

```

Runtime + Compiler 的 Vue.js 入口是 src/platforms/web/entry-runtime-with-compiler.js,
带 Compiler 版本的 Vue 包含将template 编译为 render 函数的代码，`vue-template-compiler` 的核心代码就是来自 Compiler。

Vue.js 在不同的平台下都会有编译的过程，因此编译过程中的依赖的配置 baseOptions 会有所不同。
而编译过程会多次执行，但这同一个平台下每一次的编译过程配置又是相同的.

为了不让这些配置在每次编译过程都通过参数传入，Vue.js 利用了函数柯里化的技巧很好的实现了 baseOptions 的参数保留。

$mount 方法中通过调用  ```compileToFunctions``` 从 template 得到 render 方法。
```js
const { render, staticRenderFns } = compileToFunctions(template, {
  outputSourceRange: process.env.NODE_ENV !== 'production',
  shouldDecodeNewlines,
  shouldDecodeNewlinesForHref,
  delimiters: options.delimiters,
  comments: options.comments
}, this)
options.render = render
options.staticRenderFns = staticRenderFns
```

`compileToFunctions` 内部调用的是 `baseCompile`:
```js
function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options) // 解析模板字符串生成 AST
  if (options.optimize !== false) {
    optimize(ast, options) // 优化语法树
  }
  const code = generate(ast, options) // 生成代码
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
}
```