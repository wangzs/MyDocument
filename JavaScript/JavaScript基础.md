# JavaScript基础
## 一、一些基本概念
### 1. Converting strings to numbers
 * [parseInt()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)
 * [parseFloat()](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/parseFloat)
 * 一元操作符`+`，从string中获取number： `(+"1.1") + (+"1.1") = 2.2`

### 2. typeof使用
```javascript
typeof 123 		// "number"
typeof "123"	   // "string"
typeof false 	  // "boolean"
typeof NaN 		// "number"

function f(){}	// 定义一个空函数
typeof f		  // "function"

typeof undefined	 // "undefined"

v					// ReferenceError: v is not defined
typeof v			 // "undefined"

// 其它均为object
typeof window 		// "object"
typeof {} 			// "object"
typeof [] 			// "object"
typeof null		   // "object"
```

### 3. null 与 undefined区别
* null 表示没有对象，一般用在：
> * 作为函数参数，表示该函数参数不是对象
  * 作为对象原型链的终点

* 表示"缺少值"，就是此处应该有一个值，但是还未定义
> * 变量被声明了，但没有赋值时，为undefined
  * 调用函数时，应该提供的参数没有提供时，为undefined
  * 对象没有赋值的属性时，为undefined
  * 函数没有返回值时

### 4. 表示false的值
* false
* 0
* null
* undefined
* NaN
* ""

## 二、数值相关概念
### 1. 注意浮点的计算
```js
0.1 + 0.2 == 0.3 	// false
0.1 + 0.2 === 0.3 	// false
1.1 + 1.2 == 2.3	// true
```
### 2. JS自动转为科学计数
* 小数点前的数字多于21位
	```js
	1234567890123456789012
	// 1.2345678901234568e+21

	123456789012345678901
	// 123456789012345680000
	```
* 小数点后的零多于5个
	```js
	0.0000003 // 3e-7
	0.000003 // 0.000003
	```

### 3. 正负零的差异
直接作对比时，相等
```js
0 === -0
0 === +0
+0 === -0
```
如果作为分母时：
```js
1 / -0     // -Infinity
1 / +0 	// Infinity
1 / 0 	 // Infinity
```
### 4. NaN
NaN是JavaScript的特殊值，表示“非数字”（Not a Number），主要出现在将字符串解析成数字出错的场合。
一些数学函数的参数和一些计算有问题，结果也出现NaN;
```JS
Math.acos(2)  // NaN
Math.log(-1)  // NaN
Math.sqrt(-1) // NaN

0 / 0 // NaN
```
==NaN不等于任何值，包括它本身。==
```js
NaN === NaN		// false
```
* 判断NaN的方法： isNaN函数
* isFinite函数判断某个值是否为正常值

## 三、字符串相关概念
### 1. 一种生产多行字符串的方法
```js
(function() { /*
Line 1
Line 2
*/}).toString().split('\n').slice(1,-1).join('\n')

// "Line 1
// Line 2"
```
### 2. Base64转码
* JavaScript原生提供两个方法，用来处理Base64转码：btoa方法将字符串或二进制值转化为Base64编码，atob方法将Base64编码转化为原来的编码
	```js
	// 只适合用于ASCII码字符
	window.btoa("Hello World")		 // "SGVsbG8gV29ybGQ="

	window.atob("SGVsbG8gV29ybGQ=")	// "Hello World"
	```
* 非ASCII码字符转为Base64编码，必须中间插入一个浏览器转码的环节，再使用这两个方法
```js
function b64Encode( str ) {
    return window.btoa(unescape(encodeURIComponent( str )));
}
function b64Decode( str ) {
    return decodeURIComponent(escape(window.atob( str )));
}
// 使用方法
b64Encode('你好') // "5L2g5aW9"
b64Decode('5L2g5aW9') // "你好"
```

## 四、对象相关概念
#### 1).基本概念
对象，就是一种无序的数据集合，由若干个“键值对”（key-value）构成。
```js
// 键名可以不见双引号，除非键名不符合标识名条件
var o = {
  p: "Hello World"
};
```
==属性==
键名又称为属性（property），键值可以为任何数据类型。

#### 2).生产方法
下面方法是等价的
```js
var o1 = {};
var o2 = new Object();
var o3 = Object.create(null);
```
属性读取：
```js
a = {0.7: "Hello",
	cc: "World"};
a.cc		   // "World"
a['cc']		// "World"
a[0.7]		 // 0.7会转为'0.7'字符

// 查看对象的所有keys
Object.keys(a)	// ["0.7", "cc"]

// 删除属性(delete命令只能删除对象本身的属性，不能删除继承的属性)
delete a[0.7]	// true
// 如果删除一个不存在的属性，delete不报错，而且返回true
var o = {};
delete o.p 	// true

////////////////////////////////////////////////////////////////
// 只有一种情况，delete命令会返回false，那就是该属性存在，且不得删除。
var o = Object.defineProperty({}, "p", {
        value: 123,
        configurable: false
});
o.p // 123
delete o.p // false
```

## 五、数组相关概念
#### 1).定义
```js
var arr = ['a', 'b', 'c'];
```
除了在定义时赋值，数组也可以先定义后赋值。
```js
var arr = [];
arr[0] = 'a';
arr[1] = 'b';
arr[2] = 'c';
```
数组对应的键只有整数键，不是整数键的为数组属性，不影响length的长度
使用delete命令删除一个值，会形成空位。
```js
var a = [1,2,3];
delete a[1];
delete a[2];
a 			// [1, undefined, undefined]
a.length 	 // 3
```
通过空值生成，使用数组的forEach方法或者for...in结构进行遍历，空位就会被跳过。
```js
var a = [,,,];
a.forEach(function (x, i) { console.log(i+". "+x) })		// 不产生任何输出
for (var i in a){console.log(i)}							// 不产生任何输出
```
如果空位是通过显式定义undefined生成，遍历的时候就不会被跳过。
#### 2).Array构造函数
除了直接使用方括号创建，数组还可以使用JavaScript内置的Array构造函数创建。
```js
var a = new Array();
a 				// []
a.length		// 0

var a = new Array(1);
a 				// [undefined × 1]
a.length 		// 1

var a = new Array(1,2);
a 				// [1,2]
a.length 		// 2
```

## 六、函数相关概念
#### 1).函数作用域
Javascript只有两种作用域：一种是全局作用域，变量在整个程序中一直存在；另一种是函数作用域，变量只在函数内部存在。

#### 2).默认参数的使用
```js
function f(a){
    (a !== undefined && a != null)?(a = a):(a = 1);
    return a;
}

f('') // ""
f(0)  // 0
```

#### 3).参数传递方式
JavaScript的函数参数传递方式是传值传递（passes by value），这意味着，在函数体内修改参数值，不会影响到函数外部。
```js
// 修改复合类型的参数值
var o = [1,2,3];

function f(o){
    o = [2,3,4];
}

f(o);
o // [1, 2, 3]
```

虽然参数本身是传值传递，但是对于复合类型的变量来说，属性值是传址传递（pass by reference），也就是说，属性值是通过地址读取的。所以在函数体内修改复合类型变量的属性值，会影响到函数外部。
```js
// 修改数组的属性值
var a = [1,2,3];
function f(a){
    a[0]=4;
}
f(a);
a // [4,2,3]
```
需要对==某个变量==达到传址传递的效果，可以将它写成全局对象的属性。
```js
var a = 1;
function f(p){
    window[p]=2;
}
f('a');
a // 2
```

#### 4).arguments对象
```js
var f = function(one) {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
}
f(1, 2, 3)
```
可以通过arguments对象的length属性，判断函数调用时到底带几个参数。





























































































