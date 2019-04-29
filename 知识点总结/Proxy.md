Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改。

new Proxy()表示生成一个Proxy实例，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。
Proxy 对象的所有用法，都是上面这种形式，不同的只是handler参数的写法。

*要使得Proxy起作用，必须针对Proxy实例进行操作，而不是针对目标对象进行操作。

*Proxy 实例可以作为其他对象的原型对象。

Proxy 支持的拦截操作一览，一共 13 种。

1. get(target, propKey, receiver)：拦截对象属性的读取。
	* 如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。
	* 
2. set(target, propKey, value, receiver)：拦截对象属性的设置返回一个布尔值。
	* 如果目标对象自身的某个属性，不可写且不可配置，那么set方法将不起作用。
	* 严格模式下，set代理如果没有返回true，就会报错。

12. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如 proxyInstance(...args)、proxyInstance.call(object, ...args)、proxyInstance.apply(...)。

3. has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值(表示 in 运算符有无权利访问)。

4. deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。

5. ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
6. getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
7. defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
8. preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
9. getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
10. isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
11. setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

		* apply方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。

13. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。