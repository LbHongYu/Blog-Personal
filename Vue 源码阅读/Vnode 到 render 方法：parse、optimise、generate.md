## Vnode 到 render 方法：parse、optimise、generate

### parse
`parse` 的目标是把 `template` 模板字符串转换成 AST 树，它是一种用 JavaScript 对象的形式来描述整个模板。那么整个 parse 的过程是利用正则表达式顺序解析模板，当解析到开始标签、闭合标签、文本的时候都会分别执行对应的回调函数，来达到构造 AST 树的目的。

#### stack

用正则把 开始标签 中包含的数据（attrs, tagName 等）解析出来之后还要做一个很重要的事，就是要维护一个 `stack`。

这个 stack 是用来记录一个层级关系的，用来记录DOM的深度。当解析到一个 开始标签 或者 文本，无论是什么， stack 中的最后一项，永远是当前正在被解析的节点的 parentNode 父节点。

通过 stack 解析器就可以把当前解析到的节点 push 到 父节点的 children 中。
也可以把当前正在解析的节点的 parent 属性设置为 父节点。事实上也确实是这么做的。

`<input />` 这种 自闭和的标签 是不需要 push 到 stack 中的，因为 input 并不存在子节点。
所以当解析到一个标签的开始时，要判断当前被解析的标签是否是自闭和标签，如果不是自闭和标签才 push 到 stack 中。

#### 总结：
parse 过程主要是两部分内容，一部分是 截取 字符串，一部分是对截取之后的字符串做 解析每截取一段标签的开头就 push 到 stack中，解析到标签的结束就 pop 出来，当所有的字符串都截没了也就解析完了。

### optimise
优化器的目标是找出那些静态节点(DOM 不需要发生变化的节点)并打上标记
```HTML
<p>我是静态节点，我不需要发生变化</p>
```

标记静态节点有两个好处：
1. 每次重新渲染的时候不需要为静态节点创建新节点
2. 在 Virtual DOM 中 patching 的过程可以被跳过

优化器的实现原理主要分两步：
  1. 用递归的方式将所有节点添加 static 属性，标识是不是静态节点
  2. 标记所有静态根节点(子节点全是静态节点的节点就是静态根节点)
例如这里的ul 就是静态根节点。
```HTML
<ul>
  <li>我是静态节点1，我不需要发生变化</li>
  <li>我是静态节点2，我不需要发生变化</li>
</ul>
```
优化方法 optimize
```js
export function optimize (root: ?ASTElement, options: CompilerOptions) {
  if (!root) return
  isStaticKey = genStaticKeysCached(options.staticKeys || '')
  isPlatformReservedTag = options.isReservedTag || no
  // 通过 isStatic 方法判断一个节点是否静态节点，
  // 并添加 statics: true/false 属性
  markStatic(root)
  // 第二步：标记所有静态根节点
  markStaticRoots(root, false)
}
```

```js
function isStatic (node: ASTNode): boolean {
  if (node.type === 2) { // expression
    return false
  }
  if (node.type === 3) { // text
    return true
  }
  return !!(node.pre || (
    !node.hasBindings && // no dynamic bindings
    !node.if && !node.for && // not v-if or v-for or v-else
    !isBuiltInTag(node.tag) && // not a built-in
    isPlatformReservedTag(node.tag) && // not a component
    !isDirectChildOfTemplateFor(node) &&
    Object.keys(node).every(isStaticKey)
  ))
}
```

### generate
把优化后的 AST 树转换成可执行的代码。

generate 中调用了 genElement， genElement 通过递归去拼一个函数执行代码的字符串，递归的过程根据不同的节点类型调用不同的生成方法，如果发现是一个元素节点就拼一个 _c(tagName, data, children) 的函数调用字符串，然后 data 和 children 也是使用 AST 中的属性去拼字符串。

```js
export function genElement (el: ASTElement, state: CodegenState): string {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre
  }
    // 根据不同的节点类型调用不同的生成方法
  if (el.staticRoot && !el.staticProcessed) {
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    return genSlot(el, state)
  } else {
    // component or element
    let code
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      let data
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData(el, state)
      }

      // data 和 children 都是使用 AST 中的属性去拼字符串
      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${data ? `,${data}` : ''}${children ? `,${children}` : ''})`
    }

    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}
```