---
title: 利用reduce方法实现map方法
tags: [JavaScript]
category: Frontend
filename: use reduce to implement map
createdTime: 2020-03-19 22:12
---
```js
/**
 * 利用reduce方法实现map方法
 * @param {function} callback map回调函数
 * @param {object} context callback执行时的this上下文对象
 */
Array.prototype.mapImplementedByReduce = function (callback, context = null) {
	const reducer = function (accumulator, currentValue, index, array) {
		const result = callback.call(context, currentValue, index, array);
		accumulator.push(result);
		return accumulator;
	}
	return this.reduce(reducer, []);
}

/**
 * test
 */
const arr = [1, 3, 5];
const callback = function (number, index, array) {
	return Math.pow(number, 3);
}

console.log(arr.mapImplementedByReduce(callback)); // [ 1, 27, 125 ]
```