---
title: js中的对象继承方式以及优劣对比分析
tags: [JavaScript]
category: Frontend
filename: js-inheritance
createdTime: 2019-10-08 18:23
---
### 原型链继承
```js
// 父类构造函数
function Parent() {
    this.name = 'John';
    this.habits = ['basketball', 'cook', 'boxing'];
}
// 父类原型
Parent.prototype.sayHello = function sayHello() {
    console.log('hello, world');
}
// 子类构造函数
function Child() {
    // @empty
}
// 将子类的原型指向父类的一个实例
Child.prototype = new Parent();

const child_1 = new Child();
console.log(child_1.name); // John
console.log(child_2.habits); // ['basketball', 'cook', 'boxing']
const child_2 = new Child();
console.log(child_1.habits === child_2.habits); // true
child_1.habits.push('running');
console.log(child_1.habits); // ['basketball', 'cook', 'boxing', 'running']
```
**缺点**：父类实例中的引用类型对象由所有的子类实例共享，只要其中一个子类实例对其进行了修改，便会影响其他子类实例，容易“牵一发动全身”，造成容易忽略的BUG.

### 构造函数继承
```js
function Parent(name) {
    this.name = name;
    this.habits = ['boxing'];
    this.sayHello = function sayHello() {
        console.log('hello, world!');
    }
}

function Child() {
    Parent.call(this, 'frank');
}

const child_1 = new Child();
const child_2 = new Child();
child_1.habits.push('cooking');
console.log(child_1.habits); // ['boxing', 'cooking']
console.log(child_2.habits); // ['boxing']
console.log(child_1.sayHello === child_2.sayHello); // false
```
**优点**：
1. 与原型链的方式相比，解决了不同的子类实例共享原型链上的引用类型对象的问题，因为所有的子类实例都拥有属于自身的属性；
2. 在子类构造函数中调用父类构造函数的同时，可以传递参数，实现不同的子类实例上的属性保存不同的值

**缺点**：对象方法附加在实例上，每创建一个子类实例，就会将相关的方法也创建一遍，而函数的创建需要占用内存空间

### 原型链-构造函数组合式继承
```js
function Parent(name) {
    this.name = name;
    this.habits = ['boxing'];
}
Parent.prototype.sayHello = function sayHello() {
    console.log('hello, world');
}

function Child(name) {
    Parent.call(this, name);
}

Child.prototype = new Parent();

const child_1 = new Child('trump');
const child_2 = new Child('willianm');
console.log(child_1.sayHello === child_2.sayHello); // true
```
**优点**：结合了原型链和构造函数两种方式的优势，既解决了子类实例共享引用类型属性的问题，又实现了对象方法只需要创建一次即可
**缺点**：
1. 每创建一个子类实例，需要调用两次父类构造函数；
2. 子类实例和子类实例的原型上存在同名属性，造成冗余.

### 原型式继承
```js
function createObj(parentObj) {
    function F() {}
    F.prototype = parentObj;
    return new F();
} 
const parent = {
    habits: ['boxing']
};
const child = createObj(parent);
```
本质上和ES6的`Object.create`方法一样，只是改变了子类构造函数的原型对象，缺点和基于原型链的继承方式一样.

### 寄生式继承
```js
function createObj(parentObj) {
    const childObj = Object.create(parentObj);
    childObj.sayHello = function sayHello() {
        console.log('hello, world');
    }
}
```
和基于构造函数实现继承的方式相比，省略了父类构造函数的调用，但是两种方式缺点一样.

### 寄生-组合式继承
```js
function Parent() {
    this.firstName = 'gates';
}

Parent.prototype.sayHello = function sayHello() {
    console.log('hello, world');
}

function Child(lastName) {
    Parent.call(this);
    this.lastName = lastName;
}

const prototype = Object.create(Parent.prototype);
Child.prototype = prototype;
prototype.constructor = Child;

// 寄生-组合式方法封装
function inheritance(Child, Parent) {
    const prototype = Object.create(Parent);
    Child.prototype = prototype;
    prototype.constructor = Child;
}
```
从名字就可以看出，这种方式集成了寄生和组合方式的优点，绕过了组合方式需要两次调用父类构造函数以及属性冗余的问题.