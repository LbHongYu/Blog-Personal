async函数可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。

async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，
一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。


```
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

async关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个Promise对象。

//-------------
如果await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject。
```
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出错了
```
任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行。
```
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```
上面代码中，第二个await语句是不会执行的，因为第一个await语句状态变成了reject。

第一个await放在try...catch结构里面，这样不管这个异步操作是否成功，第二个await都会执行。

```
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}
```
另一种方法是await后面的 Promise 对象再跟一个catch方法，处理前面可能出现的错误。
```
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}
```
//-------------

多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```
//-------------

await命令只能用在async函数之中，如果用在普通函数，就会报错。
```
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```




