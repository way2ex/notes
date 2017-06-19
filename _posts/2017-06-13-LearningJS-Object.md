---
layout: post
title: "JS学习之创建对象"
date: 2017-06-13
categories: [JavaScript, MyNotes]

---

## 理解对象
### 对象属性的类型
JS中有两种属性: 数据属性和访问器属性。

#### 数据属性
数据属性包含一个数据值的位置。有四个描述其行为的特性
- [[Configurable]]: 表示能否使用delete, 能否修改属性的特性，能否更改为访问器属性。直接在对象上定义的属性，默认值为true
- [[Enumerable]]: 表示能否通过for-in循环返回属性，直接在对象上定义的属性，默认值为true
- [[Writable]]: 表示能否修改属性的值。直接在对象上定义的属性，默认为true
- [[Value]]: 包含这个属性的数据值。直接在对象上定义的属性，默认为undefined

可以通过``Object.definedProperty()``方法来修改属性。该方法接收三个参数，要修改的对象，要修改对象的属性名，以及一个描述符对象。描述符对象的属性必须是: configurable, enumerable, writable, value。可以设置一个或多个值。

也可以通过``Object.defineProperty()``来新增属性，但是这样新增的属性的configurable,enumerable,writabel特性均为false。

#### 访问器属性
访问器属性不包含数据值，他们包含一对``getter()``和``setter()``函数，在读取访问器属性时，会调用``getter()``函数，该函数负责返回有效值。在写入访问器属性时，会调用``setter()``函数，该函数负责处理数据。
访问器有四个特性:
- [[Configurable]]: 表示能否使用delete, 能否修改属性的特性，能否更改为访问器属性。直接在对象上定义的属性，默认值为true
- [[Enumerable]]: 表示能否通过for-in循环返回属性，直接在对象上定义的属性，默认值为true
- [[Get]]: 读取属性时的函数
- [[Set]]: 写入时调用的函数
**注意**: 访问器属性不能直接定义，必须使用``Object.defineProperty()``来定义

```
var book = {
	_year: 2004,	// 下划线是一种常用的记号，表示只能通过对象的方法来访问。
	edition: 1
};

Object.defineProperty(book, "year", {
	get: function(){
		return this._year;
	},
	set: function(newValue){
		if(newValue > 2004){
			this._year = newValue;
			this.edition += newValue - 2004;
		}
	}
});
```

#### 定义多个属性
使用``Object.definedProperties()``方法可以一次定义多个属性。
```
var book = {};
Object.defineProperties(book, {
	_year: {
		writable: true,
		value: 2004
	},
	edition: {
		writable: true,
		value: 1
	},
	year: {
		get: function(){
			return this._year;
		},
		set: function(newValue){
			if( newValue > 2004 ){
				this._year = newValue;
				this.edition += newValue - 2004;
			}
		}
	}
});
```

#### 读取属性的特性
可以使用``Object.getOwnPropertyDescriptor()``方法来取得给定属性的描述符。接收两个参数，对象和属性名。
接上面的代码
```
console.log(Object.getOwnPropertyDescriptor(book, 'year'))
// Object {enumerable: false, configurable: false, get: function, set: function}
```

## 创建对象
### 工厂模式
所谓工厂模式就是把创建对象的代码封装起来，然后返回一个新创建的对象.
```
function createPerson(name, age){
	var o = new Object();
	o.name = name;
	o.age = age;
	o.sayName = function(){
		console.log(this.name);
	}
	return o;
}
```
**缺点** 无法解决对象的识别问题。不能使用instanceof判断对象的类型

### 构造函数模式
创建一个构造函数，使用new操作符来创建新的对象。
```
function Person(name,age){
	this.name = name;
	this.age = age;
	this.sayName = function(){
		console.log(this.name);
	}
}
var person1 = new Person('John', 12);
person1.sayName()    // John
```
使用new操作符调用函数时实际上会做一下事情：
1. 创建一个新的对象
2. 将构造函数的作用于，即this的指针指向该对象
3. 执行构造函数中的代码
4. 返回新对象

所以，经历过上述过程，
```
var person1 = new Person('John', 12);
```
实际上是在内存中创建了一个新的对象，然后添加了各个属性和方法，得到了这样一个对象：
```
{
	name: "John",
    age: 12,
    sayName: function(){
    	console.log(this.name);
    }
}
```
然后将该对象返回，该对象的指针赋值给person1.可以使用instanceof操作符来确定对象的类型。
```
console.log(person1 instanceof Person )  // true
console.log(person1 instanceof Object )  // true
```
### instanceof操作符判断的具体过程
在JavaScript中，创建新的对象类型是通过声明函数实现的。每当声明一个函数的时候，就会创建一个该函数的原型对象，该对象有一个constructor属性，指向该函数。而该函数对象中，也有一个属性指向这个原型对象。
在创建的每一个实例对象中，会有一个隐藏的指针，指向创建该实例对象时，所调用的构造函数的原型对象。比如
```
var person = new Person();
```
则person创建时，使用的是Person构造函数，该构造函数的prototype指向了一开始创建的原型对象，所以person的__proto__也指向了那个原型对象。此后，person和Person就没有任何关系了，如果非要说有关系的话，就是双方都有一个指向原型对象的属性。
而在instanceof操作符做判断时，会先找出instanceof操作符前面的实例对象的__proto__指向的原型对象，这里会一层一层往上找，即该原型对象中若是也有__proto__属性，也会继续往上找，直到到达最顶端。
然后再查找instanceof操作符后面的构造函数名的prototype所指向的对象。如该对象存在于第一次查找的原型链中，则instanceof操作符返回true，否则返回false。
下面的代码结果就可以这样解释
```
Function instanceof Object             // true
Object instanceof Function  		   // true
```
<img src="</assets/blog_images/20170613-01.png">
看下面的代码
```
function A(){}
var a = {};
A.prototype = a;
var person = {};
person1.__proto__ = a;
console.log(person1 instanceof A )        // true
```

**构造函数的缺点**在构造函数模式中，创建的对象都是独立的，其中的属性，方法也都是独立的，也就是说每次实例化一个对象，都要把各种属性和方法都创建一遍。有的方法执行的内容是一样的，是可以复用的，没有必要每次都新建一个方法对象。
```
function Person(name,age){
	this.name = name;
	this.age = age;
	this.sayName = function(){
		console.log(this.name);
	}
}
```
上面的代码可以改写为复用sayName方法。
```
function Person(name,age){
	this.name = name;
	this.age = age;
	this.sayName = sayName;
}
function sayName(){
	console.log(this.name);
}
```
但是这样很不好，在全局作用域声明的函数，实际上只能又被某个对象调用。而且当方法多了，就要定义好多函数。

### 原型模式
#### 理解原型
1. 在原型中定义的属性和方法是由所有实例所共享的；
2. ``isPrototypeOf()``方法可以用来判断对象是否是传入参数的原型
3. ``Object.getPrototypeOf()``方法可以用来获取对象的原型对象
4. 当读取实例中的某个属性时，会先从对象实例本身开始搜索，若搜索到所需的属性，则返回该属性的值，若没有，则继续搜索原型指针指向的原型对象，若找到，则返回该属性的值，若没有找到，则说明实例中没有这个属性。
5. 不能通过实例对象来重写原型对象中的属性值，但可以通过在实例中添加属性来覆盖它。
6. 可以通过delete操作符来删除实例和原型中的属性
7. ``hasOwnProperty()``方法可以接收一个字符串参数，然后判断该字符串表示的属性是不是存在于实例对象中。
8. in 操作符可以用来判断属性是否存在于该对象中，不论是实例中还是原型中。
	```
    function Person(){};
    Person.prototype.name = "Nic";
    var person = new Person();
    person.age = 12;
    console.log(person.hasOwnProperty("age"))          // true
    console.log(person.hasOwnProperty("name"))          // false
    console.log("age" in person)			   // true
    console.log("name" in person)             // true
    ```
9. 使用``hasOwnProperty()``和in操作符来创建一个判断属性是否是原型属性的函数
	```
    function hasPrototypeProperty(object, name){
    	return !object.hasOwnProperty(name) && (name in object) ;
    }
    ```
10. 使用for-in循环时，返回的是能够通过实例对象访问的、可枚举的属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。
11. 按照规定，所有开发人员定义的属性都是可枚举的属性。
12. 要取得对象上所有**可枚举**的**实例**属性，可以使用``Object.keys()``方法。该方法接收一个对象做参数，返回所有可枚举实例属性所组成的数组。
13. 如果想得到所有的实例属性，无论其是否可枚举，都可以使用``Object.getOwnPropertyNames()``，接收一个对象作为参数。

#### 原型模式创建对象
```
function Person(){}
Person.prototype.name = "xiaoming";
Person.prototype.age = 12;
Person.sayName = function(){
	console.log(this.name);
}

var person = new Person();
console.log(person.name);   // xiaoming
```
这样创建的对象由于是在原型上添加属性和方法，所以这些属性和方法是可以由所有实例所共享的

#### 原型模式的问题
1. 没有传递参数进行初始化的功能
2. 所有的属性都是确定的，共享的。这样当属性为引用类型时，修改实例中的属性，将会影响所有其他的实例。

基于以上原因，很少单独使用原型模式，而是组合使用构造函数模式和原型模式。

### 组合使用构造函数模式和原型模式
这种方法原理很简单，共享的属性和方法在原型中定义，单独的属性和方法在构造函数中定义。
```
function Person(name, age){
	this.name = name;
    this.age = age;
}
Person.prototype = {
	constructor: Person,
    sayName: function(){
    	console.log(this.name);
    }
}
```

### 动态原型模式
上面的方式中，分别写了构造函数和原型，代码并没有在一起。动态原型模式就是将原型属性的设置转移到构造函数中。原理是在构造方法中判断某个应该存在的属性或者方法是否存在，根据这样来决定是否初始化原型。
```
function Person(name, age){
	this.name = name;
    this.age = age;
    if(typeof this.sayName != function){
    	Person.prototype.sayName = function(){
        	console.log("this.name");
        };
    }
}
```
这样，对于原型的设置只会在第一次创建实例对象时起作用，并且能反映到所有的实例中。

### 寄生构造函数模式
寄生模式就是将创建对象的代码封装到一个函数中，然后在函数中创建一个对象，最后返回该新建的对象。
```
function Person(name, age){
	var o = new Object();
    o.name = name;
    o.age = age;
    o.sayName = function(){
    	console.log(this.name);
    };
    return o;
}

var person = new Person("Bob", 18);
person.sayName();     //  Bob
```
寄生模式可以用在希望能对某个对象来进行扩展的情况。比如想创建一个具有额外方法的数组对象，就可以将上面代码中的Object()替换为Array()，然后对新建的对象添加属性和方法。

### 稳妥构造函数模式
稳妥对象，指的是没有公共属性，其方法也不引用this的对象。
```
function Person(name, age){
	var o = new Object();
    // 可以在这里定义私有变量和函数
    
    // 添加方法
    o.sayName = function(){
    	console.log(name);   // 没有使用this，而是直接访问私有变量
    }
    return o;      // 返回对象
}

var person = Person("Bob", 19);
```