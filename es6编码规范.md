## ES6编程风格

	1. 块级作用域
	2. 字符串
	3. 解构赋值
	4. 对象
	5. 数组
	6. 函数
	7. Map结构
	8. Class
	9. 模块
	10. ESLint的使用

#### 块级作用域
 
1. let取代var，`let`完全可以取代`var`，因为其没有副作用。
		
		'use strict';

		for (let i = 0; i < 10; i++) {
		  console.log(i);
		}		

2. 在`let`和`const`之间，建议优先使用`const`，尤其是在全局环境，不应该设置变量，只应设置常量。
	
		const [a, b, c] = [1, 2, 3];		


#### 字符串

1. 静态字符串一律使用单引号或反引号，不使用双引号。动态字符串使用反引号。
	
		const a = 'foobar';
		const b = `foo${a}bar`;


#### 解构赋值

1. 使用数组成员对变量赋值时，优先使用解构赋值。

		const arr = [1, 2, 3, 4];
		//bad
		const first = arr[0];
		const second = arr[1];
		//good
		const [first, second] = arr;

2. 函数的参数如果是对象的成员，优先使用解构赋值。

		// good
		function getName(obj){
			const {firstName, lastName} = obj;
		}
		// best
		function getName({firstName, lastName}){}
		
3. 如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。这样便于以后添加返回值，以及更改返回值的顺序。

		// bad
		function processInput(input){
			return [left, right, top, bottom];
		}
		// good
		function processInput(input){
			return {left, right, top, bottom};
		}


#### 对象

1. 单行定义的对象，最后一个成员不以逗号结尾。多行定义的对象，最后一个成员以逗号结尾。
		
		const a = {k1: v1, k2: v2};
		const b = {
			k1: v1,
			k2: v2,
		};

2. 对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用`Object.assign`方法。

		const a = {};
		Object.assign(a, {x: 3});
		// good
		const a = {x: null};
		a.x = 3;

3. 对象的属性和方法，尽量采用简洁表达法，这样易于描述和书写。
	
		const atom = {
			value: 1,
			addValue(value){
				return atom.value;
			},
		};


#### 数组 

1. 使用扩展运算符（...）拷贝数组。

		const arr = [1, 2, 3, 4];
		//good
		const items = [...arr];

2. 使用`Array.from`方法，将类似数组的对象转为数组。

		const foo = document.querySelectorAll('.foo');
		const node = Array.from(foo);


#### 函数

1. 立即执行函数可以写成箭头函数的形式。

		(() => {
			console.log('aaa');
		})();
	
2. 那些需要使用函数表达式的场合，尽量用箭头函数代替。因为这样更简洁，而且绑定了`this`。

		//good
		[1, 2, 3].map((x) => {
			return x * x;
		});
		//best
		[1, 2, 3].map(x => x * x);

3. 箭头函数取代`Function.prototype.bind`，不应再用`self`/`_this`/`that`绑定`this`。简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体较为复杂，行数较多，还是应该采用传统的函数写法。

		//acceptable
		const bound = method.bind(this);
		//best
		const bound = (...params) => method.apply(this, params);

4. 所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数。

		//bad
		function divide(a, b, option = false){}
		//good
		function divide(a, b, {option = false} = {}){}

5. 不要在函数体内使用`arguments`变量，使用`rest`运算符（...）代替。`arguments`是一个类似数组的对象，而`rest`运算符可以提供一个真正的数组。

		function concat(...args){
			return args.join('');
		}

6. 使用默认值语法设置函数参数的默认值。

		//bad
		function handle(option){
			option = option || {};
		}
		//good
		function handle(option = {}){}


#### Map结构

1. 注意区分`Object`和`Map`，只有模拟现实世界的实体对象时，才使用`Object`。如果只是需要`key: value`的数据结构，使用`Map`结构。因为`Map`有内建的遍历机制。

		let map = new Map(arr);
		for (let key of map.keys()) {
		  console.log(key);
		}
		for (let value of map.values()) {
		  console.log(value);
		}
		for (let item of map.entries()) {
		  console.log(item[0], item[1]);
		}


#### Class

1. 总是用`Class`，取代需要`prototype`的操作。因为`Class`的写法更简洁，更易于理解。

		//bad 
		function Queue(contents = []){
			this._queue = [...contents];
		}
		Queue.prototype.pop = function(){
			const value = this._queue[0];
			this._queue.splice(0,1);
			return value;
		}
		
		//good
		class Queue{
			constructor(contents = []){
				this._queue = [...contents];
			}
			pop(){
				const value = this._queue[0];
				this._queue.splice(0, 1);
				return value;
			}
		}

2. 使用`extends`实现继承，因为这样更简单，不会有破坏`instanceof`运算的危险。
		
		//bad
		const inherits = require('inherits');
		function PeekQueue(contents){
			Queue.apply(this, contents);
		}
		inherits(PeekQueue, Queue);
		PeekQueue.prototype.peek = function(){
			return this._queue[0];
		}
		//good
		class PeekQueue extends Queue{
			peek(){
				return this._queue[0];
			}
		}


#### Class

1. Module语法是JavaScript模块的标准写法，坚持使用这种写法。使用`import`取代`require`。

		//bad
		const moduleA = require('moduleA');
		//good
		import {func1, func2} from 'moduleA';

2. 使用`export`取代`module.exports`。  
 如果模块只有一个输出值，就使用export default，如果模块有多个输出值，就不使用export default，不要export default与普通的export同时使用。

		//commonJSx写法
		var React = require('react');
		var Breadcrumbs = React.createClass({
			render(){
				return <nav />;
			}
		});
		module.exports = Breadcrumbs;
		//es6写法
		import React from 'react';
		const Breadcrumbs = React.createClass({
			render(){
				return <nav />;
			}
		});
		export default Breadcrumbs

3. 不要在模块输入中使用通配符。因为这样可以确保你的模块之中，有一个默认输出（`export default`）。


		// bad
		import * as myObject './importModule';
		// good
		import myObject from './importModule';

4. 如果模块默认输出一个函数，函数名的首字母应该小写。

		function makeStyleGuide() {
		}
		export default makeStyleGuide;

5. 如果模块默认输出一个对象，对象名的首字母应该大写。

		const StyleGuide = {
		  es6: {
		  }
		};
		export default StyleGuide;