 我们在初始化 VueX 之前，如果没有没有调用Vue.use 方法 ```Vue.use(Vuex)``` 将Vuex注入Vue中就会报错这么一条信息： ```[vuex] must call Vue.use(Vuex) before creating a store instance.```

 因为在初始化 store 的这个 class 中使用到了Vue中的一些特性，比如使用Vue.set 方法使 store 的属性能够是响应的。

 在Vue 安装 VueX 和 关于stroe class 在同一个文件src/store.js， 有一个 Vue 全局变量用来保存 Vue 的， 在安装 VueX 的 install 方法中，这个 Vue 全局变量获取了 Vue。 

```
 export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

我们是如何在Vue实例中得到 store 的呢？ 这就得助于 applyMixin 这个方法，这个方法的代码来自与src/mixin.js 这个文件。里面干的活就是和Vue.mixin() 一样的。 在创建 Vue 实例之前(beforeCreated), 将 store 注入 (调用 vuexInit 方法) 到 Vue 实例的 $store 的 属性上，这样我们就可以在component 中使用 this.$store 来获取 store 的属性，以及调用 Getter, Mutation, Action。

我们一般会将Stroe 实例添加在顶层App component， 在创建子级component 的时候逐级向下注入。 且子组件每一次实例化都需要从父级获取 store。
```
  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) { // 只有顶层 App component 才能通过这个判断
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) { //所有的子 component 才走这个判断
      this.$store = options.parent.$store
    }
  }
```