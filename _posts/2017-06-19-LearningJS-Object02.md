---
layout: post
title: "JS学习之对象继承"
date: 2017-06-19
categories: [JavaScript, MyNotes]

---

## 原型链继承
原型链继承的原理就是子类的原型指向父类的一个实例，这样，父类的原型作为原型链最顶级，其中的属性和方法子类所有实例均可访问，父类的实例由于成为了子类的原型，其实例属性与实例方法也成为了公共属性和方法。子类不但有自己的属性和方法，还可以访问父类的所有属性和方法。
```
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
}
// 创建子类
function SubType(){
	this.subProperty = false;
}
// 继承父类
SubType.prototype = new SuperType();
// 定义公共方法
SubType.prototype.getSubValue = function(){
	return this.subProperty;
};
var instance = new SubType();
console.log(instance.getSubValue());
```
### 确定原型和实例的关系
- instanceof操作符
- isPrototypeOf()

**注意**：通过原型链实现继承时，不能使用对象字面量

### 缺点
使用原型链继承，有两个缺点：
1. 父类的实例属性将会变成子类的公共属性，而当实例属性为引用类型时，在某个实例中修改引用类型时，所有其他实例中的该共享属性也被修改。
2. 创建子类型的实例时，不能向超类型的构造函数中传递参数。

### 关于父类的实例属性变成子类的公共属性
```
SubType.prototype = new SuperType();
```
s而原型链继承中的缺点之一，对引用类型的原型属性做修改，只是修改其中的值，并没有修改属性指针。若是修改了属性指针，那么也将会在实例中创建一个新的同名属性。

## 借用构造函数
借用构造函数的原理是在子类的构造函数中调用父类的构造函数，这样可以将父类的实例属性全部在子类中创建，并且还可以向父类的构造函数中传入参数。然后将子类的原型对象指向父类的实例，这样虽然父类的实例属性同样变成了原型属性，但是在子类中调用父类构造函数时，已经将父类所有的实例属性添加到子类实例中，实现了对原型属性的覆盖。
```
function SuperType(){
	this.colors = ["red", "blue", "green"];
}
function Subtype(){
	SuperType.call(this);
}
```
**缺点**：
1. 每个方法都会在创建实例时创建一遍，没有实现函数的复用。

## 组合继承
组合集成是结合使用原型链集成和借用构造函数的方式，公共的方法可以在原型上定义，而实例属性则在借用构造函数中继承。

## 原型式继承
该继承的原理是基于一个现有对象，创建一个新的对象。
```
function object(o){
	function F(){};
    F.prototype = o;
    return new F();
}
```
ES5有``Object.create()``方法用来实现原型式继承。该方法可以接受两个参数，第一个是用作新对象原型的对象，第二个是为新对象定义额外属性的对象，与``Object.defineProperty()``方法接受的对象相同。

## 寄生式继承
寄生式继承就是在利用原型式继承创建对象后，为对象添加方法和属性，最后将该对象返回。将这些代码封装起来，就是寄生式继承。

## 寄生组合式继承
组合继承是最常用的继承模式，但是也有自己的不足，在整个过程中，调用了两次父类的构造函数。
寄生组合式继承是去除了组合继承中的下面这行代码
```
SubType.prototype = new SuperType();
```
即不对父类进行实例化，而是直接令子类的原型的原型等于父类的原型。这样就实现了SubType继承SuperType的效果。
```
var prototype = object(SuperType.prototype);
prototype.constructor = SubType;
Subtype.prototype = prototype;
```
将上述代码进行封装
```
function inheritPrototype(SubType, SuperType){
	var prototype = object(SuperType.prototype);
	prototype.constructor = SubType;
	SubType.prototype = prototype;
}
```
寄生组合式继承代码如下：
```
function SuperType(name){
	this.name = name;
    this.colors = [1,2];
}

SuperType.prototype.sayName = function(){
	console.log(this.name);
}

function SubType(name, age){
	SuperType.call(this, name);
    this.age = age;
}
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
	console.log(this.age);
}
```
其实如果不考虑实例的constructor属性和子类原型对象的指向，直接令子类原型等于父类原型也可以达到同样的目的，只是那样，在子类原型上添加的方法也就添加到了父类的原型上，也就更改了父类原型，而继承也就无从谈起了。

## Object常用的相关方法总结
### 对象属性及其特性
#### Object.defineProperty()
该方法接收三个参数: 要定义属性的对象，属性名和描述符对象。
```
var book = {
	_year: 2005,
    edition: 1
};
Object.defineProperty(book, "year", {
	configurable: false,
    enumerable: true,
    get: function(){return 3;},
    set: function(value){this._year = value;}
});
```

#### Object.defineProperties()
该方法接收两个参数： 要添加属性的方法和属性及其描述符对象
```
var book = {};
Object.defineProperties(book, {
	_year: {
    	writable: true,
        value: 2005
    },
    edition: {
    	writable: true,
        value: 2
    },
    year: {
    	get: function(){return this._year;},
        set: function(value){this._year = value;}
    }
});
```

#### Object.getOwnPropertyDescriptor()
该方法可以获得对象中属性的描述符。
接收两个参数：属性所在的对象和要读取其描述符的属性名称。

### 原型相关
#### isPrototypeOf()
该方法用于判断当前对象是否是参数对象的原型
```
var person = {};
console.log(Object.prototype.isPrototypeOf(person));     // true
```

#### hasOwnProperty()
该方法用于判断给定属性是否是当前对象的实例属性。
该方法接收一个参数： 要判断的属性名。

#### in操作符
in操作符用于判断属性是否存在于对象中，而无论属性是原型属性还是实例属性

#### Object.keys()
该方法接受一个对象作参数，返回一个包含所有可枚举属性的字符串的数组。

#### Object.getOwnPropertyNames()
可以得到所有的实例属性，包括不可枚举属性。

#### Object.create()
该方法用于原型式继承，可接受两个参数：作为新对象原型的对象和一个为新对象定义额外属性的对象。