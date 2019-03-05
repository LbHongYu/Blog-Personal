函数可通过一下四种方式进行调用：
1. 函数调用
2. 方法调用
3. call, apply方法调用
4. 构造函数方式调用

+ 通过函数调用，在严格模式下，this 指向undefined, 在非严格模式下，this 指向window。 
这里所说的 "严格模式下", 是指 this 所在的函数采用的是严格模式。如果一个函数采用的非严格模式，
这个函数用到了 this,即使这个函数在严格模式被调用了，那么其中的 this 指向的是 window。
```
"use strict";
function fn(){		
	console.log(this); //undefined
}
function fn2(){	
	fn();
}
fn2()
```

```
function fn(){	
    "use strict";	
	console.log(this);  //undefined
}
function fn2(){	
	fn();
}
fn2();
```

```
function fn(){		
	console.log(this); //window
}
function fn2(){
	"use strict";
	fn();
}
fn2()
```

+ 通过方法调用，这个时候函数中this一般指向所在的对象。
```
var person = {
    firstName: "Willion",
    getName: function(){
        console.log(this.firstName);
    }
}

person.getName(); //"Willion"
```
```
var newPerson = person.getName;
newPerson(); //undefined
```
+ call, apply方法调用, this指向调用第一个参数，如果第一个参数不是对象，比如 null。那么this指向window。
```
var firstName = "Willion(Global)",
    person = {
        firstName: "Willion"
    };

function fn(){
    console.log(this.firstName);
}
fn.call(person);  //"Willion"
fn.call(null);  //"Willion(Global)"
```
+ 构造函数方式调用,this指向函数返回的实例：
```
function Person(name, age) {
    this.name = name;
    this.age = age;
    cosnole.log(this)
}

var person = new Person("Willion", 24); //Person {name: "Willion", age: 24}
```

//模拟实现 new 操作
使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1、创建（或者说构造）一个新对象。
2、这个新对象会被执行[[Prototype]]连接。
3、这个新对象会绑定到函数调用的this。
4、如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

function selfConstructor(fn){
	//创建一个对象
	var obj = {};

	//[[Prototype]]连接
	obj.__proto__ = fn.prototype;

	//往对象中添加属性
	let newObj = fn.apply(obj, Array.prototype.slice.call(arguments).slice(1,arguments.length));

	//返回这个对象
	return newObj instanceof Object ? newObj : obj;
}

+ 最后补充一点， 在浏览器事件中，IE事件处理程序(IE8-)中 this 指向的 是window, DOM事件处理程序中this指向的是绑定的DOM元素。

```
<div id="app">
	<div id="child-1"></div>
</div>
```

```
var child1 = document.getElementById("child-1");

if(child1.attachEvent){
	child1.attachEvent('onclick', function(){
		console.log("IE");
		console.log(this);  // window
	})
}

child1.onclick = function(){
	console.log(this); // <div id="child-1"></div>
}

if(child1.addEventListener){
	child1.addEventListener('click',function(){
		console.log(this); // <div id="child-1"></div>
	},false)
}
```