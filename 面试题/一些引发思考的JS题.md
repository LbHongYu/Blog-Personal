##### 一、数组的 push 方法的原理
```
var obj = {
    '2': 3,
    '3': 4,
    'length': 2,
    'splice': Array.prototype.splice,
    'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```
解析：

涉及知识点：

类数组（ArrayLike）：
一组数据，由数组来存，但是如果要对这组数据进行扩展，会影响到数组原型，ArrayLike的出现则提供了一个中间数据桥梁，
ArrayLike有数组的特性， 但是对ArrayLike的扩展并不会影响到原生的数组。

push方法：
push 方法有意具有通用性。该方法和 call() 或 apply() 一起使用时，可应用在类似数组的对象上。push 方法根据 length 属性
来决定从哪里开始插入给定的值。如果 length 不能被转成一个数值，则插入的元素索引为 0，包括 length 不存在时。当 length 不存在时，
将会创建它。唯一的原生类数组（array-like）对象是 Strings，尽管如此，它们并不适用该方法，因为字符串是不可改变的。

对象转数组的方式：
Array.from()、splice()、concat()等。

题分析：
这个obj中定义了两个key值，分别为splice和push分别对应数组原型中的splice和push方法，因此这个obj可以调用数组中的push和splice方法，
调用对象的push方法：push(1)，因为此时obj中定义length为2，所以从数组中的第二项开始插入，也就是数组的第三项（下表为2的那一项），
因为数组是从第0项开始的，这时已经定义了下标为2和3这两项，所以它会替换第三项也就是下标为2的值，第一次执行push完，
此时key为2的属性值为1，同理：第二次执行push方法，key为3的属性值为2。


##### 二、 ".", "=" 运算符的优先级
```
var a = {n: 1};
var b = a;
a.x = a = {n: 2};

console.log(a.x)     
console.log(b.x)
```
解析：
```
var a = {n: 1};
var b = a;
a.x = a = {n: 2};

a.x     // --> undefined
b.x     // --> {n: 2}
```
答案已经写上面了，这道题的关键在于

1、优先级。.的优先级高于=，所以先执行a.x，堆内存中的{n: 1}就会变成{n: 1, x: undefined}，改变之后相应的b.x也变化了，
  因为指向的是同一个对象。
2、赋值操作是从右到左，所以先执行a = {n: 2}，a的引用就被改变了，然后这个返回值又赋值给了a.x，需要注意的是这时候a.x是
  第一步中的{n: 1, x: undefined}那个对象，其实就是b.x，相当于b.x = {n: 2}


#####  三、异步方案

改造下面的代码，使之输出0 - 9，写出你能想到的所有解法。
```
for (var i = 0; i< 10; i++){
    setTimeout(() => {
        console.log(i);
    }, 1000)
}
```
解法1：
```
// 通过给setTimeout 传入第三个参数的方式
for (var i = 0; i< 10; i++){
   setTimeout((i) => {
         console.log(i);
   }, 1000,i)
}
```
解法2：
```
for (var i = 0; i< 10; i++){
	(function(i){
    setTimeout(() => {
      console.log(i);
    }, 1000)
	})(i)
}
```
解法3：
```
for (let i = 0; i< 10; i++){
  setTimeout(() => {
    console.log(i);
  }, 1000)
}
```
解法4：
```
[0,1,2,3,4,5,6,7,8,9].forEach(d=>{
  setTimeout(() => {
    console.log(d);
  }, 1000)	
})
```
