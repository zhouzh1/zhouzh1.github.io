---
title: js中的in操作符和Object.keys方法
tags: [JavaScript]
category: Frontend
filename: the in operator and object keys function
createdTime: 2019-12-10 16:42
---

### in操作符
`in`和`typeof`一样，只是操作符，不是函数，它用来判断对象是否拥有某个属性，例如：

```js
console.log('alert' in window); // true
console.log('xxoo' in window); // false
```

在js中，对象的属性查找和继承是基于原型链来实现的，可以使用`hasOwnProperty`方法判断一个属性是属于对象本身，还是属于它的上层原型：

```js
console.log(window.hasOwnProperty('alert')); // true
console.log(([]).hasOwnProperty('slice')); // false
```

`in`操作符只负责判断在目标对象的原型链上能否找到目标属性，能找到则返回true，否则返回false：

```js
console.log('slice' in ([])); // true
```

同理，`for ... in`循环也会遍历到对象原型链上的属性：

```js
function Foo() {
    this.foo = 'foo';
}
Foo.prototype.bar = 'bar';
Foo.prototype.__proto__.ox = 'ox';
const f = new Foo();
for (let key in f) {
    console.log(key); // foo bar ox
}
```

但是，像`__proto__`这样的私有属性是不会被遍历到的：

```js
const foo = {};
console.log(foo.hasOwnProperty('__proto__')); // false
```

当然，js是一门灵活的语言，当通过`JSON.parse`方法创建对象的时候，可以使得`__proto__`作为正常属性被遍历，而不是指向对象原型：

```js
const obj = JSON.parse('{"a":"a","__proto__":{"b":"b"}}');
for (let key in obj) {
    console.log(key); // a __proto__
}
```

使用上面描述的方法可能可以在某些代码中发现原型污染漏洞.

### Object.keys方法
Object.keys方法返回一个字符串数组，数组包含目标对象的所有属性名，`(o: object) => string[]`，但是只会包含对象本身的属性，上层原型对象上的属性不会包含在数组内：

```js
function Foo() {
    this.foo = 'foo';
    this.oof = 'oof';
}
Foo.prototype.bar = 'bar';
Foo.prototype.__proto__.ox = 'ox';
const f = new Foo();
console.log(Object.keys(f)); // ['foo', 'oof']
```
