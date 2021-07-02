### Class 的继承

class 的继承方式是子类和父类通过 extends 关键字连接实现继承。
```
class Parent{
	constructor(name, age){
		this.name = name;
		this.age = age;
		
	}

	sayName(){	
		console.log(this.name);
	}

}

class Child extends Parent{
	constructor(name, age, classes){
		// this.classes = classes;  只有调用super之后，才可以使用this关键字
		super(name, age); //子类必须在constructor方法中调用super方法
		this.job = "cook";
	}

	sayAge(){	
		console.log(this.age);
	}	
}

let child = new Child("nike", 16);
console.log(child);
```
* 如果子类没有定义constructor方法，这个方法会被默认添加。且会在constructor上中调用 super
* 通过class 继承父类的实例上的属性和方法会被继承到子类上，子类可以通过原型链访问父类原型上的方法(同时继承实例和原型上的方法和属性)。
* 这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象，新建实例时会报错。

* 在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有super方法才能调用父类实例。

* 父类的静态方法，也会被子类继承
```
class Parent {
  static sayHi() {
    console.log('hello world');
  }
}

class Child extends Parent {
}

Child.sayHi()
```

* 往Object.getPrototypeOf()中传入一个实例，可以返回这个实例的原型，但通过 extends 继承的 class 子类，我们可以使用 Object.prototypeOf() 方法得到父类的class。 ES5 中的继承，子类的__proto__ 属性指向的Function.prototype
```
class Parent { }
class Child extends Parent { }
Object.getPrototypeOf(Child) === Parent; // true
Child.__proto__ 	=== Parent; // true
```

```
function Parent(){}
function Child(){}
Child.prototype = new Parent();
Child.__proto__ === Function.prototype; //true
```
#### super 关键字

super这个关键字，既可以当作函数使用，也可以当作对象使用。

1. super作为对象时，
	1. 在子类的普通方法中，指向<b>父类的原型对象</b>；
		 * 定义在父类实例上的方法或属性是无法通过super 获取的。
		 * super虽然代表了父类A的构造函数，但是返回的是子类Child的实例，即super内部的this指的是Child的实例，
       相当于A.prototype.constructor.call(this)。
	2. 在子类的静态方法中，指向<b>父类</b>。
		 * 方法内部的this指向当前的子类，而不是子类的实例。
		 * 在子类的static 方法中通过super调用父类的static 方法时，父类的 static 方法内部的 this 指向当前的子类，而不是子类的实例。

* 在子类的普通方法和静态方法中使用super：

```
class Parent{
	constructor(name, age){
		this.name = "Parent name";		
		this.job = "cook";		
	}

	static sayName(){
		return "staticSayName";
	}

	static sayAge(){
		return this.age;
	}

	sayName(){	
		return "sayName";
	}
}

class Child extends Parent{
	constructor(age){
		super();
		this.name = "Child name";
		this.age = age;
	}
	
	static sayParentName(){
		// super在静态方法之中指向父类
		// static_super_sayName: staticSayName
		console.log("static_super_sayName:", super.sayName());	
	}

	static sayParentAge(){
		console.log("staticAge: ", super.sayAge()); //staticAge: 18 sayAge 中的this指向的是子类，所以返回18
	}

	sayParentName(){	
		console.log("super: ",super.name); //super:  undefined 父类实例上属性是无法通过super获取
		console.log("super_sayName:", super.sayName()); //super_sayName: sayName
	}	
}

Child.age = 18;

let child = new Child(16);
child.sayParentName();
Child.sayParentName();  
Child.sayParentAge();  
```


* super() 调用的是父类的构造函数，但是其中的this是指向的子类，所以super()返回的子类实例
```
class Parent {
	constructor() {
		console.log("target name: ", new.target.name); // new.target指向当前正在执行的函数
		this.name = "Parent Name";
	}
	sayName(){
		console.log(this.name);	
	}
}

class Child extends Parent {
  constructor() {
    var obj = super(); // Parent.prototype.constructor.call(this)
		this.name = "Child Name";
		console.log("obj_instanceof: ", obj instanceof Child); // true

		super.sayName(); // "Child Name"
  }
}
let a = new Parent(); // new.target.name ---> Parent
let b = new Child(); // new.target.name ---> Child

// 在super()执行时，它指向的是子类Child的构造函数，super()内部的this指向的是Child。
```


2. super 关键字的正确使用方式：
		1. super()只能用在子类的构造函数之中，用在其他地方就会报错。 
		2. super 不能直接被使用，只用它来调用父类构造函数 或者 获取父类原型上的方法
```
1. 
class Parent {}

class Child extends Parent {
  m() {
    // Uncaught SyntaxError: 'super' keyword unexpected here
    super(); 
  }
}

2. 
class Parent {}

class Child extends Parent {
  constructor(){
  	super();

  	//Uncaught SyntaxError: 'super' keyword unexpected here
  	//super 不能直接被使用
  	console.log(super);  

  	//Uncaught SyntaxError: 'super' keyword unexpected here
  	console.log(super === Parent.prototype); 
  }
}

let b = new Child();
```

3. 通过super 向子类添加属性。
   * super指向的父类的原型， 但是通过 super 对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。
```
class Parent {
	constructor(){
		this.job = "cook";
	}
}

class Child extends Parent {
  constructor(){
  	super();
		this.job = "student";
		super.job = "developer";

		console.log(super.job); //undefined 
														//(这里读取的父类原型的job 属性，由于父类原型上没有这个属性，所以返回undefined)

		console.log(this.job); // developer (修改的实例上的属性)
  }
}

let b = new Child();
console.log(b);
```

4. 对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。
```
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); 
```