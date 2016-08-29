对象（Object）
---

### 概述

对象中包含一系列属性，这些属性是无序的。每个属性都有一个字符串`key`和对应的`value`。

---
 
### 对象结构
	1.key（键值）	
	2.对象属性（可增加删除）
		* writable
		* enumerable
		* configurable
		* value
	3.对象原型
		* prototype
	4.对象种类（类型）
		* class
	5.标签（是否允许增加新属性）
		* extensible	
		
---


#### 对象键值

对象中包含一系列属性，这些属性是无序的。每个属性都有一个字符串key和对应的value。


* 访问属性，可以通过`.`和`[]`，`[]`中间为字符串，如果不是字符串，会自动转化成字符串。

		var obj = {x : 1, y : 2};
		obj.x; // 1
		obj["y"]; // 2
	
		obj[{}] = true;
		obj[{x : 1}] = true;
		obj; // Object {1: 2, [object Object]: true}

* 属性读写异常，可以通过判断进行避免报错也可以巧用`&&`。

		var obj = {x : 1};
		obj.y; // undefined
		var yz = obj.y.z; // TypeError: Cannot read property 'z' of undefined
		obj.y.z = 2; // TypeError: Cannot set property 'z' of undefined
		
		var yz;
		if (obj.y) {
		    yz = obj.y.z;
		}
		
		var yz = obj && obj.y && obj.y.z;

---

#### 对象原型

每个`JS`对象都有原型，除了`Object.prototype`，因为它是原型链的顶端。（`__proto__`是隐藏属性，`ie`下不兼容）

* 访问对象属性的值时，如果对象本身上没有改属性则会沿着原型链进行查找。如果到原型链的末端还没查到则会返回`undefined`。

		unction foo(){};
		foo.prototype.z = 3;
		
		var obj =new foo();
		obj.x = 1;
		
		obj.x; // 1
		obj.z; // 3
		obj.h; // undefined

* `in`操作可以判断该对象是否存在某个属性，但是不能判断是否上对象本身具有还是其原型链拥有的。可以使用`hasOwnProperty()`进行判断。

		'z' in obj; // true
		obj.hasOwnProperty('z'); // false

* 给对象赋值，如果不存在赋值的属性，则在该对象上新增该属性，如果存在的话就修改改属性的值。


		obj.z = 5;
		obj.hasOwnProperty('z'); // true
		foo.prototype.z; // still 3
		obj.z; // 5
		
		obj.z = undefined;
		obj.z; // undefined

* `delete`删除方法只能对象上的属性，而不能删除对象原型链上的属性。`prototype`属性不能够被删除。另外`delete`删除方法只是删除该属性的值，并没有删除该属性。

		delete obj.z; // true
		obj.z; // 3
		
		delete obj.z; // true
		obj.z; // still 3!!!
		
		delete obj.prototype; // false
		
		var p = {a: 2};
		delete p.a; // true
		p.a; // undefined

---

#### 属性操作


###### `Object.defineProperty()`方法

* `Object.defineProperty`，设置对象的属性（ES5的语法，IE8存在一定的兼容性问题）。接收三个参数，对象名、属性名和具体属性描述。

		var cat = new Object;
		Object.defineProperty(cat, 'price', {configurable：true, enumerable : false, writable: true, value : 1000});

	1. `configurable`值为布尔值，默认为`true`，可删除。判断对象的属性能否用`delete`删除掉,严格模式下`configurable`为`false`时，`delete`会报错。

	2. `enumerable`表示是否能够通过`for…in`语句来枚举出属性，默认是true，true可以枚举，false不可。可以通过用propertyIsEnumerable来获取Enumerable的值。

			cat.propertyIsEnumerable('price'); // false

	3. `writable`表示属性值是否可以修改，默认为`true`。如果`writable`被设置成`false`，修改属性值时将没有效果，而在严格模式下会报错。但是该对象为数组，利用数组的方法来添加或修改子项却能够成功。

			var test = {};
			Object.defineProperty(test, "array", {
			    writable: false,
			    value: ["c", "u"]
			});
			container.array = ["array"];
			console.log(container.array); // ["c", "u"]
			 
			container.array.push("value");
			console.log(container.array); // ["c", "u", "value"]

	4. `value`设置该属性的值。

	5. `get`、`set`在读取或写入属性时调用的函数，默认都是是`undefined`。`get`,`set`方法只是属性的特性 ，不是对象方法，决定属性能否、怎么读。

		* 当对象或原型链上存在`get`或`set`方法时，给对象属性赋值并不能修改该属性的值。可以通过`Object.defineProperty`的`value`来修改。

				function foo() {};
				Object.defineProperty(foo.prototype, 'z', 
				    {get : function(){return 1;}});
				var obj = new foo();
				obj.z; // 1
				obj.z = 10;
				obj.z; // still 1
				
				Object.defineProperty(obj, 'z', 
				{value : 100, configurable: true});
				obj.z; // 100;
				delete obj.z;
				obj.z; // back to 1

		* 也可以通过set来实现

				function Foo(val){
					this.value = val;//定义了value属性 并没有定义_value
				}
				Foo.prototype={
					set value(val){//注意方法名和属性名相同，在prototype里定义了value属性
						this._value=val;
					},
					get value(){//方法名和属性名相同，在prototype里面定义了value属性和它的get 特性
						return this._value;
					}
				}; 　　　　
				var obj=new Foo("hello"); 　　　　
				obj.value;//"hello" 　　　　
				obj.value="yehoo"; 　　　　
				obj.value;//"yehoo"


###### `Object.getOwnPropertyDescriptor()`方法

* 接收两个参数，对象和属性名，返回一个关于该属性名所有对象属性集合


		Object.getOwnPropertyDescriptor({pro : true}, 'pro');
		// Object {value: true, writable: true, enumerable: true, configurable: true}
		
		Object.defineProperties(person, {
		    title : {value : 'fe', enumerable : true},
		    corp : {value : 'BABA', enumerable : true},
		    salary : {value : 50000, enumerable : true, writable : true},
		    luck : {
		        get : function() {
		        return Math.random() > 0.5 ? 'good' : 'bad';
		        }
		    },
		    promote : {
		        set : function (level) {
		            this.salary *= 1 + level * 0.1;
		        }
		    }
		});
		
		Object.getOwnPropertyDescriptor(person, 'salary');
		// Object {value: 50000, writable: true, enumerable: true, configurable: false}
		Object.getOwnPropertyDescriptor(person, 'corp');
		// Object {value: "BABA", writable: false, enumerable: true, configurable: false}
		person.salary; // 50000
		person.promote = 2;
		person.salary; // 60000

---

#### 对象种类

* 每个对象都有`toString`方法，`toString`方法是属于对象原型链末端的`object.prototype`上的。可以利用`toString`方法判断数据类型。

		var toString = Object.prototype.toString;
		function getType(o){return toString.call(o).slice(8,-1);};
		
		toString.call(null); // "[object Null]"
		getType(null); // "Null"
		getType(undefined); // "Undefined"
		getType(1); // "Number"
		getType(new Number(1)); // "Number"
		typeof new Number(1); // "object"
		getType(true); // "Boolean"
		getType(new Boolean(true)); // "Boolean"
		getType(new Function); // "Funtion"
		getType(new Array); // "Array"

* `Object.create({x: 1})`会创建一个指向原型链的对象，接收一个参数。创建的属性在该对象的原型链上。

		var obj = Object.create({x : 1});
		obj.x; // 1
		typeof obj.toString; // "function"
		obj.hasOwnProperty('x'); // false

		//该对象的原型直接为null，就不存在`toString`的方法
		var obj = Object.create(null);
		obj.toString // undefined


###### extensible标签

* `extensible`判断对象能否被拓展（`writable`表示能否修改，`configurable`表示能否删除）。`isExtensible`判断能否被拓展，`preventExtensions`设置该对象不能拓展。

		var obj = {x : 1, y : 2};
		Object.isExtensible(obj); // true
		Object.preventExtensions(obj);
		Object.isExtensible(obj); // false
		obj.z = 1;
		obj.z; // undefined, add new property failed
		Object.getOwnPropertyDescriptor(obj, 'x');
		// Object {value: 1, writable: true, enumerable: true, configurable: true}
		
		Object.seal(obj);
		Object.getOwnPropertyDescriptor(obj, 'x');
		// Object {value: 1, writable: true, enumerable: true, configurable: false}
		Object.isSealed(obj); // true
		
		Object.freeze(obj);
		Object.getOwnPropertyDescriptor(obj, 'x');
		// Object {value: 1, writable: false, enumerable: true, configurable: false}
		Object.isFrozen(obj); // true
		// [caution] not affects prototype chain!!!


###### 序列化

* `JSON.stringify()`是常用的序列化对象的方法，如果存在某一项值为`undefined`则忽略该项。如果该项值为`NaN`或`Infinity`则转化为`null`。

		var obj = {x : 1, y : true, z : [1, 2, 3], nullVal : null};
		JSON.stringify(obj); // "{"x":1,"y":true,"z":[1,2,3],"nullVal":null}"
		
		obj = {val : undefined, a : NaN, b : Infinity, c : new Date()};
		JSON.stringify(obj); // "{"a":null,"b":null,"c":"2015-01-20T14:15:43.910Z"}"
		
		obj = JSON.parse('{"x" : 1}');
		obj.x; // 1
		
		
		var obj = {
		    x : 1,
		    y : 2,
		    o : {
		        o1 : 1,
		        o2 : 2,
		        toJSON : function () {
		            return this.o1 + this.o2;
		        }
		    }
		};
		JSON.stringify(obj); // "{"x":1,"y":2,"o":3}"

* `JSON.stringify()`接收三个参数，第一项是`value`，表示输入的对象或数组；第二项是`replacer`，表示用什么做分隔符；第三项是`space`，表示用什么做分隔符。