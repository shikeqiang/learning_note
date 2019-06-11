# 一、简介

## 1.Babel 转码器

​	[Babel](https://babeljs.io/) 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而在现有环境执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。下面是一个例子。

```javascript
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```

上面的原始代码用了箭头函数，Babel 将其转为普通函数，就能在不支持箭头函数的 JavaScript 环境执行了。

```bash
#安装babel
$ npm install --save-dev @babel/core
```

### 配置文件.babelrc

​	Babel 的配置文件是`.babelrc`，存放在项目的根目录下。使用 Babel 的第一步，就是配置这个文件。

该文件用来设置转码规则和插件，基本格式如下。

```javascript
{
  "presets": [],
  "plugins": []
}
```

==`presets`字段设定转码规则==，官方提供以下的规则集，你可以根据需要安装。

```bash
# 最新转码规则
$ npm install --save-dev @babel/preset-env

# react 转码规则
$ npm install --save-dev @babel/preset-react
```

然后，将这些规则加入`.babelrc`。

```javascript
  {
    "presets": [
      "@babel/env",
      "@babel/preset-react"
    ],
    "plugins": []
  }
```

注意，以下所有 Babel 工具和模块的使用，都必须先写好`.babelrc`。

### 命令行转码

Babel 提供命令行工具`@babel/cli`，用于命令行转码；它的安装命令如下：

```bash
$ npm install --save-dev @babel/cli
```

基本用法如下：

```bash
# 转码结果输出到标准输出
$ npx babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ npx babel example.js --out-file compiled.js
# 或者
$ npx babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ npx babel src --out-dir lib
# 或者
$ npx babel src -d lib

# -s 参数生成source map文件
$ npx babel src -d lib -s
```

### babel-node

`@babel/node`模块的`babel-node`命令，提供一个支持 ES6 的 REPL 环境。它支持 Node 的 REPL 环境的所有功能，而且可以直接运行 ES6 代码。

首先，安装这个模块。

```bash
$ npm install --save-dev @babel/node
```

然后，执行`babel-node`就进入 REPL 环境。

```bash
$ npx babel-node
> (x => x * 2)(1)
2
```

`babel-node`命令可以直接运行 ES6 脚本。将上面的代码放入脚本文件`es6.js`，然后直接运行。

```bash
# es6.js 的代码
# console.log((x => x * 2)(1));
$ npx babel-node es6.js
2
```

### @babel/register 模块

`@babel/register`模块改写`require`命令，**为它加上一个钩子。此后，每当使用`require`加载`.js`、`.jsx`、`.es`和`.es6`后缀名的文件，就会先用 Babel 进行转码。**

```bash
$ npm install --save-dev @babel/register
```

使用时，必须首先加载`@babel/register`。

```bash
// index.js
require('@babel/register');
require('./es6.js');
```

然后，就不需要手动对`index.js`转码了。

```bash
$ node index.js
2
```

​	需要注意的是，`@babel/register`只会对`require`命令加载的文件转码，而不会对当前文件转码。另外，由于它是实时转码，所以只适合在开发环境使用。

### babel API

如果某些代码需要调用 Babel 的 API 进行转码，就要使用`@babel/core`模块。

```javascript
var babel = require('@babel/core');

// 字符串转码
babel.transform('code();', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

配置对象`options`，可以参看官方文档<http://babeljs.io/docs/usage/options/>。

下面是一个例子。

```javascript
var es6Code = 'let x = n => n + 1';
var es5Code = require('@babel/core')
  .transform(es6Code, {
    presets: ['@babel/env']
  })
  .code;

console.log(es5Code);
// '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
```

上面代码中，`transform`方法的第一个参数是一个字符串，表示需要被转换的 ES6 代码，第二个参数是转换的配置对象。

## 2.Traceur 转码器

Google 公司的[Traceur](https://github.com/google/traceur-compiler)转码器，也可以将 ES6 代码转为 ES5 代码。

### 直接插入网页

Traceur 允许将 ES6 代码直接插入网页。首先，必须在网页头部加载 Traceur 库文件。

```html
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
<script type="module">
  import './Greeter.js';
</script>
```

上面代码中，一共有 4 个`script`标签。第一个是加载 Traceur 的库文件，第二个和第三个是将这个库文件用于浏览器环境，第四个则是加载用户脚本，这个脚本里面可以使用 ES6 代码。

注意，第四个`script`标签的`type`属性的值是`module`，而不是`text/javascript`。这是 Traceur 编译器识别 ES6 代码的标志，编译器会自动将所有`type=module`的代码编译为 ES5，然后再交给浏览器执行。

除了引用外部 ES6 脚本，也可以直接在网页中放置 ES6 代码。

```javascript
<script type="module">
  class Calc {
    constructor() {
      console.log('Calc constructor');
    }
    add(a, b) {
      return a + b;
    }
  }

  var c = new Calc();
  console.log(c.add(4,5));
</script>
```

正常情况下，上面代码会在控制台打印出`9`。

如果想对 Traceur 的行为有精确控制，可以采用下面参数配置的写法。

```javascript
<script>
  // Create the System object
  window.System = new traceur.runtime.BrowserTraceurLoader();
  // Set some experimental options
  var metadata = {
    traceurOptions: {
      experimental: true,
      properTailCalls: true,
      symbols: true,
      arrayComprehension: true,
      asyncFunctions: true,
      asyncGenerators: exponentiation,
      forOn: true,
      generatorComprehension: true
    }
  };
  // Load your module
  System.import('./myModule.js', {metadata: metadata}).catch(function(ex) {
    console.error('Import failed', ex.stack || ex);
  });
</script>
```

上面代码中，首先生成 Traceur 的全局对象`window.System`，然后`System.import`方法可以用来加载 ES6。加载的时候，需要传入一个配置对象`metadata`，该对象的`traceurOptions`属性可以配置支持 ES6 功能。如果设为`experimental: true`，就表示除了 ES6 以外，还支持一些实验性的新功能。

# 二、let和var命令

## 1.let 命令

### 基本用法

​	ES6 新增了`let`命令，用来声明变量。它的用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效。

```javascript
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

​	上面代码在代码块之中，分别用`let`和`var`声明了两个变量。然后在代码块之外调用这两个变量，结果`let`声明的变量报错，`var`声明的变量返回了正确的值。这表明，`let`声明的变量只在它所在的代码块有效。

`for`循环的计数器，就很合适使用`let`命令。

```javascript
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
```

上面代码中，计数器`i`只在`for`循环体内有效，在循环体外引用就会报错。下面的代码如果使用`var`，最后输出的是`10`。

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上面代码中，**变量`i`是`var`命令声明的，在全局范围内都有效，所以全局只有一个变量`i`。**每一次循环，变量`i`的值都会发生改变，而循环内被赋给数组`a`的函数内部的`console.log(i)`，==里面的`i`指向的就是全局的`i`。==也就是说，所有数组`a`的成员里面的`i`，指向的都是同一个`i`，导致运行时输出的是最后一轮的`i`的值，也就是 10。

如果使用`let`，声明的变量仅在块级作用域内有效，最后输出的是 6。

```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

​	上面代码中，变量`i`是`let`声明的，当前的`i`只在本轮循环有效，所以**每一次循环的`i`其实都是一个新的变量，所以最后输出的是`6`。** JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量`i`时，就在上一轮循环的基础上进行计算。

另外，`for`循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

​	上面代码正确运行，输出了 3 次`abc`。这表明函数内部的变量`i`与循环变量`i`不在同一个作用域，有各自单独的作用域。

### 不存在变量提升

`var`命令会发生“变量提升”现象，即变量可以在声明之前使用，值为`undefined`。`let`命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

```javascript
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

上面代码中，变量`foo`用`var`命令声明，会发生变量提升，即脚本开始运行时，变量`foo`已经存在了，但是没有值，所以会输出`undefined`。变量`bar`用`let`命令声明，不会发生变量提升。这表示在声明它之前，变量`bar`是不存在的，这时如果用到它，就会抛出一个错误。

### 暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```javascript
var tmp = 123;
if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

​	上面代码中，存在全局变量`tmp`，但是块级作用域内`let`又声明了一个局部变量`tmp`，导致后者绑定这个块级作用域，所以在`let`声明变量前，对`tmp`赋值会报错。

​	==ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。==

总之，在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

上面代码中，在`let`命令声明变量`tmp`之前，都属于变量`tmp`的“死区”。

“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作。

```javascript
typeof x; // ReferenceError
let x;
```

上面代码中，变量`x`使用`let`命令声明，所以在声明之前，都属于`x`的“死区”，只要用到该变量就会报错。因此，`typeof`运行时就会抛出一个`ReferenceError`。

作为比较，如果一个变量根本没有被声明，使用`typeof`反而不会报错。

```javascript
typeof undeclared_variable // "undefined"
```

上面代码中，`undeclared_variable`是一个不存在的变量名，结果返回“undefined”。所以，在没有`let`之前，`typeof`运算符是百分之百安全的，永远不会报错，现在这一点不成立了。

有些“死区”比较隐蔽，不太容易发现。

```javascript
function bar(x = y, y = 2) {
  return [x, y];
}
bar(); // 报错
```

​	上面代码中，调用`bar`函数之所以报错（某些实现可能不报错），**是因为参数`x`默认值等于另一个参数`y`，而此时`y`还没有声明，属于“死区”。如果`y`的默认值是`x`，就不会报错，因为此时`x`已经声明了。**

```javascript
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2, 2]
```

另外，下面的代码也会报错，与`var`的行为不同。

```javascript
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined
```

上面代码报错，也是因为暂时性死区。使用`let`声明变量时，只要变量在还没有声明完成前使用，就会报错。上面这行就属于这个情况，在变量`x`的声明语句还没有执行完成前，就去取`x`的值，导致报错”x 未定义“。

ES6 规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。

总之，==暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。==

### 不允许重复声明

`let`不允许在相同作用域内，重复声明同一个变量。

```javascript
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

因此，**不能在函数内部重新声明参数。**

```javascript
function func(arg) {
  let arg;
}
func() // 报错

function func(arg) {
  {
    let arg;
  }
}
func() // 不报错
```

## 2.块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景：

> - 第一种场景，内层变量可能会覆盖外层变量。
>
> ```javascript
> var tmp = new Date();
> 
> function f() {
>   console.log(tmp);
>   if (false) {
>     var tmp = 'hello world';
>   }
> }
> 
> f(); // undefined
> ```
>
> ​	上面代码的原意是，`if`代码块的外部使用外层的`tmp`变量，内部使用内层的`tmp`变量。但是，函数`f`执行后，输出结果为`undefined`，原因在于变量提升，导致内层的`tmp`变量覆盖了外层的`tmp`变量。
>
> - 第二种场景，用来计数的循环变量泄露为全局变量。
>
> ```javascript
> var s = 'hello';
> 
> for (var i = 0; i < s.length; i++) {
>   console.log(s[i]);
> }
> 
> console.log(i); // 5
> ```
>
> 上面代码中，变量`i`只用来控制循环，但是循环结束后，它并没有消失，**泄露成了全局变量。**

### ES6 的块级作用域

`let`实际上为 JavaScript 新增了块级作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

​	上面的函数有两个代码块，都声明了变量`n`，运行后输出 5。**这表示外层代码块不受内层代码块的影响。**如果两次都使用`var`定义变量`n`，最后输出的值才是 10。

ES6 允许块级作用域的任意嵌套。

```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};
```

上面代码使用了一个五层的块级作用域，每一层都是一个单独的作用域。第四层作用域无法读取第五层作用域的内部变量。

内层作用域可以定义外层作用域的同名变量。

```javascript
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```

### 块级作用域与函数声明

ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。

```javascript
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch(e) {
  // ...
}
```

上面两种函数声明，根据 ES5 的规定都是非法的。

但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，因此上面两种情况实际都能运行，不会报错。

==ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用==。

```javascript
function f() { console.log('I am outside!'); }
(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }
  f();
}());
```

上面代码在 ES5 中运行，会得到“I am inside!”，**因为在`if`内声明的函数`f`会被提升到函数头部，**实际运行的代码如下：

```javascript
// ES5 环境
function f() { console.log('I am outside!'); }
(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```

​	ES6 就完全不一样了，理论上会得到“I am outside!”。因为块级作用域内声明的函数类似于`let`，对作用域之外没有影响。但是，如果你真的在 ES6 浏览器中运行一下上面的代码，是会报错的，这是为什么呢？

```javascript
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

上面的代码在 ES6 浏览器中，都会报错。

原来，如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在[附录 B](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-block-level-function-declarations-web-legacy-compatibility-semantics)里面规定，浏览器的实现可以不遵守上面的规定，有自己的[行为方式](http://stackoverflow.com/questions/31419897/what-are-the-precise-semantics-of-block-level-functions-in-es6)。

- 允许在块级作用域内声明函数。
- **函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。**
- 同时，函数声明还会提升到所在的块级作用域的头部。

注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作`let`处理。

根据这三条规则，浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于`var`声明的变量。上面的例子实际运行的代码如下。

```javascript
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

考虑到环境导致的行为差异太大，**应该避免在块级作用域内声明函数。**如果确实需要，也应该写成函数表达式，而不是函数声明语句。

```javascript
// 块级作用域内部的函数声明语句，建议不要使用
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 块级作用域内部，优先使用函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

另外，还有一个需要注意的地方。**ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。**

```javascript
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```

上面代码中，第一种写法没有大括号，所以不存在块级作用域，而`let`只能出现在当前作用域的顶层，所以报错。第二种写法有大括号，所以块级作用域成立。

函数声明也是如此，严格模式下，函数只能声明在当前作用域的顶层。

```javascript
// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';
if (true)
  function f() {}
```

## 3.const 命令

### 基本用法

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。

```javascript
const PI = 3.1415;
PI // 3.1415
PI = 3;	// TypeError: Assignment to constant variable.
```

`const`声明的变量不得改变值，这意味着，`const`一旦声明变量，就**必须立即初始化**，不能留到以后赋值。

```javascript
const foo;	// SyntaxError: Missing initializer in const declaration
```

==`const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效。==

```javascript
if (true) {
  const MAX = 5;
}
MAX // Uncaught ReferenceError: MAX is not defined
```

**`const`命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。**

```javascript
if (true) {
  console.log(MAX); // ReferenceError，在常量MAX声明之前就调用，结果报错。
  const MAX = 5;
}
```

==`const`声明的常量，也与`let`一样不可重复声明。==

```javascript
var message = "Hello!";
let age = 25;
// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```

### 本质

​	==`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。==对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），**变量指向的内存地址，保存的只是一个指向实际数据的指针，`const`只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。**因此，将一个对象声明为常量必须非常小心。

```javascript
const foo = {};
// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```

​	上面代码中，常量`foo`储存的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把`foo`指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。

```javascript
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

​	上面代码中，常量`a`是一个数组，这个数组本身是可写的，但是如果将另一个数组赋值给`a`，就会报错。

如果真的想将对象冻结，应该使用`Object.freeze`方法。

```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

上面代码中，常量`foo`指向一个冻结的对象，所以添加新属性不起作用，严格模式时还会报错。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

### ES6 声明变量的六种方法

​	ES5 只有两种声明变量的方法：`var`命令和`function`命令。ES6 除了添加`let`和`const`命令，后面章节还会提到，另外两种声明变量的方法：`import`命令和`class`命令。所以，ES6 一共有 6 种声明变量的方法。

## 4.顶层对象的属性

​	顶层对象，在浏览器环境指的是`window`对象，在 Node 指的是`global`对象。ES5 之中，顶层对象的属性与全局变量是等价的。

```javascript
//顶层对象的属性赋值与全局变量的赋值，是同一件事。
window.a = 1;
a // 1
a = 2;
window.a // 2
```

​	顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。这样的设计带来了几个很大的问题，首先是没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；其次，程序员很容易不知不觉地就创建了全局变量（比如打字出错）；最后，顶层对象的属性是到处可以读写的，这非常不利于模块化编程。另一方面，`window`对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。

​	ES6 为了改变这一点，一方面规定，为了保持兼容性，`var`命令和`function`命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，`let`命令、`const`命令、`class`命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

```javascript
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1

let b = 1;
window.b // undefined
```

​	上面代码中，全局变量`a`由`var`命令声明，所以它是顶层对象的属性；全局变量`b`由`let`命令声明，所以它不是顶层对象的属性，返回`undefined`。

## 5.globalThis 对象

​	JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的。

- 浏览器里面，顶层对象是`window`，但 Node 和 Web Worker 没有`window`。
- 浏览器和 Web Worker 里面，`self`也指向顶层对象，但是 Node 没有`self`。
- Node 里面，顶层对象是`global`，但其他环境都不支持。

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用`this`变量，但是有局限性。

- 全局环境中，`this`会返回顶层对象。但是，Node 模块和 ES6 模块中，`this`返回的是当前模块。
- 函数里面的`this`，如果函数不是作为对象的方法运行，而是单纯作为函数运行，`this`会指向顶层对象。但是，严格模式下，这时`this`会返回`undefined`。
- 不管是严格模式，还是普通模式，`new Function('return this')()`，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全策略），那么`eval`、`new Function`这些方法都可能无法使用。

综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。

```javascript
// 方法一
(typeof window !== 'undefined'
   ? window
   : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
     ? global
     : this);

// 方法二
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```

​	现在有一个[提案](https://github.com/tc39/proposal-global)，在语言标准的层面，引入`globalThis`作为顶层对象。也就是说，任何环境下，`globalThis`都是存在的，都可以从它拿到顶层对象，指向全局环境下的`this`。

垫片库[`global-this`](https://github.com/ungap/global-this)模拟了这个提案，可以在所有环境拿到`globalThis`。

# 三、变量的解构赋值

- 左右两边结构必须一致，比如左右都是数组
- 右边必须是个东西，比如是个数组，或者json等
- 声明和赋值赋值不能分开，必须在一句话里

## 1.数组的解构赋值

### 基本用法

ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

以前，为变量赋值，只能直接指定值。

```javascript
let a = 1;
let b = 2;
let c = 3;
```

ES6 允许写成下面这样。

```javascript
let [a, b, c] = [1, 2, 3];	//可以从数组中提取值，按照对应位置，对变量赋值。
```

​	本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。下面是一些使用嵌套数组进行解构的例子。

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

如果解构不成功，变量的值就等于`undefined`。

```javascript
let [foo] = [];
let [bar, foo] = [1];
```

以上两种情况都属于解构不成功，`foo`的值都会等于`undefined`。

​	**另一种情况是不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。**解构依然可以成功。

```javascript
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```

上面两个例子，都属于不完全解构，但是可以成功。

如果等号的右边不是数组（或者严格地说，不是可遍历的结构，参见《Iterator》一章），那么将会报错。

```javascript
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

​	上面的语句都会报错，因为等号右边的值，要么转为对象以后不具备 Iterator 接口（前五个表达式），要么本身就不具备 Iterator 接口（最后一个表达式）。对于 Set 结构，也可以使用数组的解构赋值：

```javascript
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
```

​	事实上，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。

```javascript
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```

​	上面代码中，`fibs`是一个 Generator 函数（参见《Generator 函数》一章），原生具有 Iterator 接口。解构赋值会依次从这个接口获取值。

### 默认值

解构赋值允许指定默认值。

```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

​	注意，ES6 内部使用严格相等运算符（`===`），判断一个位置是否有值。所以，只有当一个数组成员严格等于`undefined`，默认值才会生效。

```javascript
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];	// 如果一个数组成员是null，默认值就不会生效，因为null不严格等于undefined。
x // null
```

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

```javascript
function f() {
  console.log('aaa');
}
let [x = f()] = [1];
```

上面代码中，因为`x`能取到值，所以函数`f`根本不会执行。上面的代码其实等价于下面的代码。

```javascript
let x;
if ([1][0] === undefined) {
  x = f();
} else {
  x = [1][0];
}
```

默认值可以引用解构赋值的其他变量，但该变量必须已经声明。

```javascript
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```

上面最后一个表达式之所以会报错，是因为`x`用`y`做默认值时，`y`还没有声明。

## 2.对象的解构赋值

​	解构不仅可以用于数组，还可以用于对象。

```javascript
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: 'aaa', bar: 'bbb' };	
baz // undefined，变量没有对应的同名属性，导致取不到值，最后等于undefined。
```

​	对象的解构与数组有一个重要的不同。**数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。**

​	对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。

```javascript
// 例一，将Math对象的对数、正弦、余弦三个方法，赋值到对应的变量上
let { log, sin, cos } = Math;

// 例二，将console.log赋值到log变量
const { log } = console;
log('hello') // hello
```

​	如果变量名与属性名不一致，必须写成下面这样。

```javascript
let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

​	这实际上说明，对象的解构赋值是下面形式的简写（参见《对象的扩展》一章）。

```javascript
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
```

​	也就是说，**==对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者(冒号后面的变量)，而不是前者。==**

```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"
foo // error: foo is not defined
```

​	上面代码中，`foo`是匹配的模式，`baz`才是变量。真正被赋值的是变量`baz`，而不是模式`foo`。

与数组一样，解构也可以用于嵌套结构的对象。

```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```

​	注意，这时`p`是模式，不是变量，因此不会被赋值。如果`p`也要作为变量赋值，可以写成下面这样。

```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]
```

下面是另一个例子。

```javascript
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```

​	上面代码有三次解构赋值，分别是对`loc`、`start`、`line`三个属性的解构赋值。注意，最后一次对`line`属性的解构赋值之中，只有`line`是变量，`loc`和`start`都是模式，不是变量。

下面是嵌套赋值的例子。

```javascript
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}
arr // [true]
```

如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错。

```javascript
// 报错
let {foo: {bar}} = {baz: 'baz'};
```

​	上面代码中，等号左边对象的`foo`属性，对应一个子对象。该子对象的`bar`属性，解构时会报错。原因很简单，因为`foo`这时等于`undefined`，再取子属性就会报错。

注意，对象的解构赋值可以取到继承的属性。

```javascript
const obj1 = {};
const obj2 = { foo: 'bar' };
Object.setPrototypeOf(obj1, obj2);

const { foo } = obj1;
foo // "bar"
```

​	上面代码中，对象`obj1`的原型对象是`obj2`。`foo`属性不是`obj1`自身的属性，而是继承自`obj2`的属性，解构赋值可以取到这个属性。

### 默认值

对象的解构也可以指定默认值。

```javascript
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

​	默认值生效的条件是，对象的属性值严格等于`undefined`。

```javascript
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```

​	上面代码中，属性`x`等于`null`，因为`null`与`undefined`不严格相等，所以是个有效的赋值，导致默认值`3`不会生效。

### 注意点

（1）如果要将一个已经声明的变量用于解构赋值，必须非常小心。

```javascript
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error
```

​	上面代码的写法会报错，因为 JavaScript 引擎会将`{x}`理解成一个代码块，从而发生语法错误。只有不将大括号写在行首，避免 JavaScript 将其解释为代码块，才能解决这个问题。

```javascript
// 正确的写法
let x;
({x} = {x: 1});
```

​	上面代码将整个解构赋值语句，放在一个圆括号里面，就可以正确执行。

（2）解构赋值允许等号左边的模式之中，不放置任何变量名。因此，可以写出非常古怪的赋值表达式。

```javascript
({} = [true, false]);
({} = 'abc');
({} = []);
```

上面的表达式虽然毫无意义，但是语法是合法的，可以执行。

（3）由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。

```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

​	上面代码对数组进行对象解构。数组`arr`的`0`键对应的值是`1`，`[arr.length - 1]`就是`2`键，对应的值是`3`。

## 3.字符串的解构赋值

​	字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有一个`length`属性，因此还可以对这个属性解构赋值。

```javascript
let {length : len} = 'hello';
len // 5
```

## 4.数值和布尔值的解构赋值

​	解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

​	上面代码中，数值和布尔值的包装对象都有`toString`属性，因此变量`s`都能取到值。

**==解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于`undefined`和`null`无法转为对象，所以对它们进行解构赋值，都会报错。==**

```javascript
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 5.函数参数的解构赋值

函数的参数也可以使用解构赋值。

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

​	上面代码中，函数`add`的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量`x`和`y`。对于函数内部的代码来说，它们能感受到的参数就是`x`和`y`。

```javascript
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

函数参数的解构也可以使用默认值。

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

上面代码中，函数`move`的参数是一个对象，通过对这个对象进行解构，得到变量`x`和`y`的值。如果解构失败，`x`和`y`等于默认值。

注意，下面的写法会得到不一样的结果。

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

上面代码是为函数`move`的参数指定默认值，而不是为变量`x`和`y`指定默认值，所以会得到与前一种写法不同的结果。

`undefined`就会触发函数参数的默认值。

```javascript
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

## 6.圆括号问题

​	解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。由此带来的问题是，如果模式中出现圆括号怎么处理。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，**建议只要有可能，就不要在模式中放置圆括号。**

### 不能使用圆括号的情况

以下三种解构赋值不得使用圆括号。

（1）变量声明语句

```javascript
// 全部报错，因为它们都是变量声明语句，模式不能使用圆括号。
let [(a)] = [1];

let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};

let { o: ({ p: p }) } = { o: { p: 2 } };
```

（2）函数参数

函数参数也属于变量声明，因此不能带有圆括号。

```javascript
// 报错
function f([(z)]) { return z; }
// 报错
function f([z,(x)]) { return x; }
```

（3）赋值语句的模式

```javascript
// 全部报错，将整个模式放在圆括号之中，导致报错。
({ p: a }) = { p: 42 };
([a]) = [5];
```

```javascript
// 报错，将一部分模式放在圆括号之中，导致报错。
[({ p: a }), { x: c }] = [{}, {}];
```

### 可以使用圆括号的情况

==可以使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。==

```javascript
[(b)] = [3]; // 正确
({ p: (d) } = {}); // 正确
[(parseInt.prop)] = [3]; // 正确
```

​	上面三行语句都可以正确执行，**因为首先它们都是赋值语句，而不是声明语句；**其次它们的圆括号都不属于模式的一部分。第一行语句中，模式是取数组的第一个成员，跟圆括号无关；第二行语句中，模式是`p`，而不是`d`；第三行语句与第一行语句的性质一致。

## 7.用途

**（1）交换变量的值**

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```

**（2）从函数返回多个值**

​	函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。

```javascript
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

**（3）函数参数的定义**

**解构赋值可以方便地将一组参数与变量名对应起来。**

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

**（4）提取 JSON 数据**

​	==解构赋值对提取 JSON 对象中的数据，尤其有用。==

```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;	//number为data的属性，作为后面被赋值的对象

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

**（5）函数参数的默认值**

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。

**（6）遍历 Map 结构**

​	任何部署了 Iterator 接口的对象，都可以用`for...of`循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。

```javascript
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

**（7）输入模块的指定方法**

加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```

# 四、字符串

## 1.字符的 Unicode 表示法

​	ES6 加强了对 Unicode 的支持，允许采用`\uxxxx`形式表示一个字符，其中`xxxx`表示字符的 Unicode 码点。

```javascript
"\u0061"		// "a"
```

这种表示法只限于码点在`\u0000`~`\uFFFF`之间的字符。超出这个范围的字符，必须用两个双字节的形式表示。

```javascript
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// " 7"
```

​	上面代码表示，如果直接在`\u`后面跟上超过`0xFFFF`的数值（比如`\u20BB7`），JavaScript 会理解成`\u20BB+7`。由于`\u20BB`是一个不可打印字符，所以只会显示一个空格，后面跟着一个`7`。

ES6 对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

```javascript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

上面代码中，最后一个例子表明，大括号表示法与四字节的 UTF-16 编码是等价的。

有了这种表示法之后，JavaScript 共有 6 种方法可以表示一个字符。

```javascript
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

## 2.字符串的遍历器接口

ES6 为字符串添加了遍历器接口，使得字符串可以被`for...of`循环遍历。

```javascript
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

除了遍历字符串，**这个遍历器最大的优点是可以识别大于`0xFFFF`的码点，**传统的`for`循环无法识别这样的码点。

```javascript
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```

​	上面代码中，字符串`text`只有一个字符，但是`for`循环会认为它包含两个字符（都不可打印），而`for...of`循环会正确识别出这一个字符。

- `startsWith`
- `endsWith`

```js
var url = 'http://qq.com'
console.log(url.startsWith('http'))
console.log(url.endsWith('com'))
// 都是 true
```

- 字符串模版
  - 使用反引号  `，${变量}，读取变量，拼接字符串
  - 可以折行

```js
let a = 12
let str1 = `asdf${a}`
console.log(str1)	--> asdf12

let title = '标题'
let content = '内容'
let str = `<div>
<h1>${title}</h1>
<p>${content}</p>
`
console.log(str)
<div>
<h1>标题</h1>
<p>内容</p>
```

# 五、函数

## 箭头函数

- 箭头函数，就是函数的简写
  - 如果只有一个参数，`()` 可以省
  - 如果只有一个`return`，`{}`可以省

```js
// 普通函数
function name() {
}
// 箭头函数，去掉 function， 加上 =>
() => {
}
let show1 = function () {
    console.log('abc')
}
let show2 = () => {
    console.log('abc')
}
show1() // 调用函数
show2()
let show4 = function (a) {
    return a*2
}
let show5 = a => a * 2  // 简洁，类似python lambda 函数
console.log(show4(10))
console.log(show5(10))
```

## 函数-参数

- 参数扩展／展开	...args
  - 收集剩余的参数，必须当到最后一个参数位置
  - 展开数组，简写，效果和直接把数组的内容写在这儿一样
- 默认参数

```js
function show(a, b, ...args) {
    console.log(a)
    console.log(b)
    console.log(args)
}
console.log(show(1, 2, 3, 4, 5))   --> 弹出1； 2； 3，4，5

let arr1 = [1, 2, 3]
let arr2 = [4, 5, 6]
let arr3 = [...arr1, ...arr2]
console.log(arr3)	--> 弹出 1，2，3，4，5，6

function show2(a, b=5, c=8) {	// 默认参数
    console.log(a, b, c)
}
show2(88, 12)	-- >弹出 88，12，8
```

# 六、数组

- 新增4个方法
- map 映射 一个对一个

```js
let arr = [12, 5, 8]
let result = arr.map(function (item) {
    return item*2		// 返回
})
let result2 = arr.map(item=>item*2) // 简写，箭头函数也可以直接赋值
console.log(result)
console.log(result2)

let score = [18, 86, 88, 24]
let result3 = score.map(item => item >= 60 ? '及格' : '不及格')
console.log(result3)

// 结果
[ 24, 10, 16 ]
[ 24, 10, 16 ]
[ '不及格', '及格', '及格', '不及格' ]
```

- reduce 汇总 一堆出来一个，每次会减少某些计算的数
  - 用于比如，算总数，算平均数

```js
var arr = [1, 3, 5, 7]
var result = arr.reduce(function (tmp, item, index) {
    //tmp 上次结果，item当前数，index次数1开始，这里的“上次结果”即为第一个数
    console.log(tmp, item, index)
    return tmp + item
})
console.log(result)		--> 1，3，1 ； 4，5，2 ； 9，7 ，3 ；16

var arr = [1, 3, 5, 7]
var result = arr.reduce(function (tmp, item, index) {
    if (index != arr.length - 1) { // 不是最后一次，求和
        return tmp + item
    } else {
        return (tmp + item)/arr.length		// 求平均数
    }
})
console.log(result)  // 平均值
```

- filter 过滤器 保留为true的

```js
var arr = [12, 4, 8, 9]
// 三目运算符
var result = arr.filter(item => (item % 3 === 0) ? true : false)
console.log(result)
var result = arr.filter(item => item % 3 === 0)
console.log(result)

var arr = [
    { title: '苹果', price: 10 },
    { title: '西瓜', price: 20 },
]
var result = arr.filter(json => json.price >= 20)
console.log(result)
```

- forEach 循环迭代

```js
var arr = [12, 4, 8, 9]
var result = arr.forEach(item => console.log(item))
var result = arr.forEach((item, index)=>console.log(item, index))
```

# 七、面向对象

- 原来写法
  - 类和构造函数一样
  - 属性和方法分开写的

```js
// 老版本
function User(name, pass) {
    this.name = name
    this.pass = pass
}

User.prototype.showName = function () {
    console.log(this.name)
}
User.prototype.showPass = function () {
    console.log(this.pass)
}

var u1 = new User('able', '1233')
u1.showName()
u1.showPass()
// 老版本继承
function VipUser(name, pass, level) {
    User.call(this, name, pass)
    this.level = level
}
VipUser.prototype = new User()
VipUser.prototype.constructor = VipUser
VipUser.prototype.showLevel = function () {
    console.log(this.level)
}
var v1 = new VipUser('blue', '1234', 3)
v1.showName()
v1.showLevel()
```

- 新版面向对象
  - 有了 class 关键字、构造器(constructor)
  - class 里面直接加方法
  - 继承，super 超类==父类

```js
class User {
    constructor(name, pass) {
        this.name = name
        this.pass = pass
    }
    showName() {
        console.log(this.name)
    }
    showPass() {
        console.log(this.pass)
    }
}

var u1 = new User('able2', '111')
u1.showName()
u1.showPass()

// 新版本继承
class VipUser extends User {
    constructor(name, pass, level) {	// 原来的属性加上个level
        super(name, pass)
        this.level = level
    }
    showLevel(){
        console.log(this.level)
    }
}
v1 = new VipUser('blue', '123', 3)
v1.showLevel()
```

# 八、json对象

- JSON 格式
  - JavaScript Object Notation 的缩写，是一种用于数据交换的文本格式
  - JSON 是 JS对象 的严格子集
  - JSON 的标准写法
  - **只能用双引号**
  - 所有的key都必须用双引号包起来
- JSON 对象
  - JSON 对象是 JavaScript 的原生对象，用来处理 JSON 格式数据，有两个静态方法
  - JSON.parse(string) ：接受一个 **JSON 字符串**并将其转换成一个 JavaScript **对象**。
  - JSON.stringify(obj) ：接受一个 JavaScript **对象**并将其转换为一个 **JSON 字符串**。

```js
var json = {a: 12, b: 5}	// 错误写法，不是json
var str = 'hi,' + JSON.stringify(json)
var url = 'http://www.xx.com/' + encodeURIComponent(JSON.stringify(json))
console.log(str)
console.log(url)
// 只能用双引号，即key和value都要用双引号，json字符串中都要用双引号
var str = '{"a": 12, "b": 4, "c": "abc"}'
var json = JSON.parse(str)
console.log(json)
hi,{"a":12,"b":5}
http://www.xx.com/%7B%22a%22%3A12%2C%22b%22%3A5%7D
{ a: 12, b: 4, c: 'abc' }
```

- 对象（object）
  - 是 JavaScript 语言的核心概念，也是最重要的数据类型
  - 对象就是一组“键值对”（key-value）的集合，是一种无序的复合数据集合
  - 对象的所有键名都是字符串, 所以加不加引号都可以
  - 如果键名是数值，会被自动转为字符串
  - 对象的每一个键名又称为“属性”（property），它的“键值”可以是任何数据类型
  - 如果一个属性的值为函数，通常把这个属性称为“方法”，它可以像函数那样调用
  - in 运算符用于检查对象是否包含某个属性（注意，检查的是键名，不是键值
  - for...in循环用来遍历一个对象的全部属性
- 对象 简写
  - key-value 的名字一样时可以简写，即key、value名字一样的时候，可以只写一个
  - 里面函数可以简写, 去掉

```js
var a = 12, b = 5
console.log({a:a, b:b})
console.log({a, b})
console.log({a, b, c:"c"})
console.log({ a, b, show(){ console.log('a') }})
{ a: 12, b: 5 }
{ a: 12, b: 5 }
{ a: 12, b: 5, c: 'c' }
{ a: 12, b: 5, show: [Function: show] }
```

# 九、Promise对象

​	所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

#### `Promise`对象有以下两个特点。

（1）对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

> 有了`Promise`对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise`对象提供统一的接口，使得控制异步操作更加容易。

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

> `Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
>
> `resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

> `then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。

- 异步和同步
  - 异步，操作之间没有关系，同时执行多个操作， 代码复杂
  - 同步，同时只能做一件事，代码简单
- Promise 对象
  - **用同步的方式来书写异步代码**
  - Promise 让异步操作写起来，像在写同步操作的流程，不必一层层地嵌套回调函数
  - 改善了可读性，对于多层嵌套的回调函数很方便
  - 充当异步操作与回调函数之间的中介，使得异步操作具备同步操作的接口
- Promise 也是一个构造函数
  - **接受一个回调函数f1作为参数，f1里面是异步操作的代码**
  - 返回的p1就是一个 Promise 实例
  - 所有异步任务都返回一个 Promise 实例
  - ==Promise 实例有一个then方法，用来指定下一步的回调函数==

```
function f1(resolve, reject) {
  // 异步代码...
}
var p1 = new Promise(f1);
p1.then(f2); // f1的异步操作执行完成，就会执行f2。
```

- Promise 使得异步流程可以写成同步流程

```js
// 传统写法
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // ...
      });
    });
  });
});

// Promise 的写法，先创建一个Promise对象，然后传入对应的函数
(new Promise(step1))
  .then(step2)
  .then(step3)
  .then(step4);
```

- Promise.all(promiseArray)方法
  - **将多个Promise对象实例包装，生成并返回一个新的Promise实例**
  - promise数组中所有的promise实例都变为resolve的时候，该方法才会返回
  - 并将所有结果传递results数组中
  - promise数组中任何一个promise为reject的话，则整个Promise.all调用会立即终止，并返回一个reject的新的promise对象

```js
var p1 = Promise.resolve(1),
    p2 = Promise.resolve(2),
    p3 = Promise.resolve(3);
//Promise.all里面放一个数组，然后执行后调用then里面的方法
Promise.all([p1, p2, p3]).then(function (results) {
    console.log(results);  // [1, 2, 3]
});
```

- Promise.race([p1, p2, p3])

  > 上面代码中，只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给`p`的回调函数。

  - Promse.race就是赛跑的意思
  - 哪个结果获得的快，就返回那个结果
  - 不管结果本身是成功状态还是失败状态

实例：

```js
<Script src="jquert.js" charset="utf-8"></Script>
<Script>
	Promise.all([	// 传入多个ajax请求
        $.ajax({url:'data/arr.txt',dataType:'json'}),
        $.ajax({url:'data/num.txt',dataType:'json'}),
        $.ajax({url:'data/json.txt',dataType:'json'})
]).then(results=>{
    let [arr,num,json]=results;		//解构赋值
    alert("成功了");
    console.log(arr,json,num);
	},err=>{
      alert('失败了');
});
</Script>
```

## 常见应用：

### 加载图片

我们可以将图片的加载写成一个`Promise`，一旦加载完成，`Promise`的状态就发生变化。

```javascript
const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    const image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
```

### Generator 函数与 Promise 的结合

使用 Generator 函数管理流程，遇到异步操作的时候，通常返回一个`Promise`对象。

```javascript
function getFoo () {
  return new Promise(function (resolve, reject){
    resolve('foo');
  });
}

const g = function* () {
  try {
    const foo = yield getFoo();
    console.log(foo);
  } catch (e) {
    console.log(e);
  }
};

function run (generator) {
  const it = generator();

  function go(result) {
    if (result.done) return result.value;

    return result.value.then(function (value) {
      return go(it.next(value));
    }, function (error) {
      return go(it.throw(error));
    });
  }

  go(it.next());
}

run(g);
```

上面代码的 Generator 函数`g`之中，有一个异步操作`getFoo`，它返回的就是一个`Promise`对象。函数`run`用来处理这个`Promise`对象，并调用下一个`next`方法。

## Promise.try()

实际开发中，经常遇到一种情况：不知道或者不想区分，函数`f`是同步函数还是异步操作，但是想用 Promise 来处理它。因为这样就可以不管`f`是否包含异步操作，都用`then`方法指定下一步流程，用`catch`方法处理`f`抛出的错误。一般就会采用下面的写法。

```javascript
Promise.resolve().then(f)
```

上面的写法有一个缺点，就是如果`f`是同步函数，那么它会在本轮事件循环的末尾执行。

```javascript
const f = () => console.log('now');
Promise.resolve().then(f);
console.log('next');
// next
// now
```

上面代码中，函数`f`是同步的，但是用 Promise 包装了以后，就变成异步执行了。

​	那么有没有一种方法，让同步函数同步执行，异步函数异步执行，并且让它们具有统一的 API 呢？回答是可以的，并且还有两种写法。第一种写法是用`async`函数来写。

```javascript
const f = () => console.log('now');
(async () => f())();
console.log('next');
// now
// next
```

上面代码中，第二行是一个立即执行的匿名函数，会立即执行里面的`async`函数，因此如果`f`是同步的，就会得到同步的结果；如果`f`是异步的，就可以用`then`指定下一步，就像下面的写法。

```javascript
(async () => f())()
.then(...)
```

需要注意的是，`async () => f()`会吃掉`f()`抛出的错误。所以，如果想捕获错误，要使用`promise.catch`方法。

```javascript
(async () => f())()
.then(...)
.catch(...)
```

第二种写法是使用`new Promise()`。

```javascript
const f = () => console.log('now');
(
  () => new Promise(
    resolve => resolve(f())
  )
)();
console.log('next');
// now
// next
```

上面代码也是使用立即执行的匿名函数，执行`new Promise()`。这种情况下，同步函数也是同步执行的。

鉴于这是一个很常见的需求，所以现在有一个[提案](https://github.com/ljharb/proposal-promise-try)，提供`Promise.try`方法替代上面的写法。

```javascript
const f = () => console.log('now');
Promise.try(f);
console.log('next');
// now
// next
```

事实上，`Promise.try`存在已久，Promise 库[`Bluebird`](http://bluebirdjs.com/docs/api/promise.try.html)、[`Q`](https://github.com/kriskowal/q/wiki/API-Reference#promisefcallargs)和[`when`](https://github.com/cujojs/when/blob/master/docs/api.md#whentry)，早就提供了这个方法。

由于`Promise.try`为所有操作提供了统一的处理机制，所以如果想用`then`方法管理流程，最好都用`Promise.try`包装一下。这样有[许多好处](http://cryto.net/~joepie91/blog/2016/05/11/what-is-promise-try-and-why-does-it-matter/)，其中一点就是可以更好地管理异常。

```javascript
function getUsername(userId) {
  return database.users.get({id: userId})
  .then(function(user) {
    return user.name;
  });
}
```

上面代码中，`database.users.get()`返回一个 Promise 对象，如果抛出异步错误，可以用`catch`方法捕获，就像下面这样写。

```javascript
database.users.get({id: userId})
.then(...)
.catch(...)
```

但是`database.users.get()`可能还会抛出同步错误（比如数据库连接错误，具体要看实现方法），这时你就不得不用`try...catch`去捕获。

```javascript
try {
  database.users.get({id: userId})
  .then(...)
  .catch(...)
} catch (e) {
  // ...
}
```

上面这样的写法就很笨拙了，这时就可以统一用`promise.catch()`捕获所有同步和异步的错误。

```javascript
Promise.try(() => database.users.get({id: userId}))
  .then(...)
  .catch(...)
```

事实上，`Promise.try`就是模拟`try`代码块，就像`promise.catch`模拟的是`catch`代码块。

# 十、generator

## 1.认识生成器函数

​	执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

​	形式上，Generator 函数是一个普通函数，但是有两个特征。**一是，`function`关键字与函数名之间有一个星号；二是，函数体内部使用`yield`表达式，定义不同的内部状态（`yield`在英语里的意思就是“产出”）。**

​	Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象（Iterator Object）。

​	**接着调用遍历器对象的`next`方法，使得指针移向下一个状态。**也就是说，每次调用`next`方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个`yield`表达式（或`return`语句）为止。换言之，Generator 函数是分段执行的，`yield`表达式是暂停执行的标记，而`next`方法可以恢复执行。

- generator 生成器函数
  - 普通函数是一路到底，即函数会一直执行到最后才结束
  - generator函数，在运行期间可以停，通过 yield 配合，交出执行权
    - yield 有 放弃、退让、退位的意思
  - 需要调用next()方法启动执行，需要遇到 yield 停, 踹一脚走一步
  - ==generator函数前面加一个 `*` 两边可以有空格，或靠近函数或`function`==
  - 背后实际生成多个小函数，实现走走停停

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

> 上面代码一共调用了四次`next`方法。
>
> 第一次调用，**Generator 函数开始执行，直到遇到第一个`yield`表达式为止。`next`方法返回一个对象，它的`value`属性就是当前`yield`表达式的值`hello`，==`done`属性的值`false`，表示遍历还没有结束==**。
>
> 第二次调用，Generator 函数从上次`yield`表达式停下的地方，一直执行到下一个`yield`表达式。`next`方法返回的对象的`value`属性就是当前`yield`表达式的值`world`，`done`属性的值`false`，表示遍历还没有结束。
>
> 第三次调用，**Generator 函数从上次`yield`表达式停下的地方，一直执行到`return`语句（如果没有`return`语句，就执行到函数结束）。**`next`方法返回的对象的`value`属性，就是紧跟在`return`语句后面的表达式的值（==如果没有`return`语句，则`value`属性的值为`undefined`==），`done`属性的值`true`，表示遍历已经结束。
>
> 第四次调用，此时 Generator 函数已经运行完毕，`next`方法返回对象的`value`属性为`undefined`，`done`属性为`true`。以后再调用`next`方法，返回的都是这个值。

​	总结一下，调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的`next`方法，就会返回一个有着`value`和`done`两个属性的对象。`value`属性表示当前的内部状态的值，是`yield`表达式后面那个表达式的值；`done`属性是一个布尔值，表示是否遍历结束。

## 2.generator-yield

​	`	yield`表达式本身没有返回值，或者说总是返回`undefined`。==`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值。==

- `yield`
  - 既可传参，又可以返回结果
  - ==第一个`next()`传参无效，只用来启动==
- 如果函数前漏掉 `*`
  - 就是普通函数
  - 如果有`yield`会报错， `ReferenceError: yield is not defined`
  - **yield 只能在Generator函数内部使用**

```js
function * show() {
    console.log('1')
    var a = yield
    console.log('2')
    console.log(a)
}
// yield 传参
var gen = show()
gen.next() // 1
gen.next() // 2 和 undefined 因为没有传参，yield没有返回值
var gen = show()
gen.next(10) // 1 第一次执行到yield，但没有执行赋值，只是用来启动
gen.next(20) // 2 和 20

function* show2() {
    console.log('1')
    yield 10
    console.log('2')
}
// yield 返回
var gen = show2()	//Generator对象
var res1 = gen.next()	
console.log(res1) // { value: 10, done: false }
var res2 = gen.next()
console.log(res2)
// { value: undefined, done: true }  最后的value需要return返回
```

#### 遍历器对象的`next`方法的运行逻辑如下。

（1）遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。

（2）下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式。

（3）如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。

（4）如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。

需要注意的是，`yield`表达式后面的表达式，只有当调用`next`方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。

```javascript
function* gen() {
  yield  123 + 456;
}
```

**上面代码中，`yield`后面的表达式`123 + 456`，不会立即求值，只会在`next`方法将指针移到这一句时，才会求值。**

> ```javascript
> function* foo(x) {
>   var y = 2 * (yield (x + 1));
>   var z = yield (y / 3);
>   return (x + y + z);
> }
> 
> var a = foo(5);
> a.next() // Object{value:6, done:false}
> a.next() // Object{value:NaN, done:false}
> a.next() // Object{value:NaN, done:true}
> 
> var b = foo(5);
> b.next() // { value:6, done:false }
> b.next(12) // { value:8, done:false }
> b.next(13) // { value:42, done:true }
> ```
>
> ​	上面代码中，第二次运行`next`方法的时候不带参数，导致 y 的值等于`2 * undefined`（即`NaN`），除以 3 以后还是`NaN`，因此返回对象的`value`属性也等于`NaN`。第三次运行`Next`方法的时候不带参数，所以`z`等于`undefined`，返回对象的`value`属性等于`5 + NaN + undefined`，即`NaN`。
>
> ​	如果向`next`方法提供参数，返回结果就完全不一样了。上面代码第一次调用`b`的`next`方法时，返回`x+1`的值`6`；**第二次调用`next`方法，将上一次`yield`表达式的值设为`12`，因此`y`等于`24`，返回`y / 3`的值`8`；第三次调用`next`方法，将上一次`yield`表达式的值设为`13`，因此`z`等于`13`，这时`x`等于`5`，`y`等于`24`，所以`return`语句的值等于`42`。**
>
> ​	注意，**由于`next`方法的参数表示上一个`yield`表达式的返回值，所以在第一次使用`next`方法时，传递参数是无效的。**V8 引擎直接忽略第一次使用`next`方法时的参数，只有从第二次使用`next`方法开始，参数才是有效的。从语义上讲，第一个`next`方法用来启动遍历器对象，所以不用带有参数。

## 3.generator-实例

- Promise 适合一次读一组
- generator 适合逻辑性的

```js
// 带逻辑-generator
runner(function * () {
    let userData = yield $.ajax({url: 'getUserData'})

    if (userData.type == 'VIP') {
        let items = yield $.ajax({url: 'getVIPItems'})
    } else {
        let items = yield $.ajax({url: 'getItems'})
    }
})
// yield 实例，用同步方式写异步
server.use(function * () {
    let data = yield db.query(`select * from user_table`)
    this.body = data
})
```

# 十二、总结

1. 变量 let const

| 声明方式 | 能否重复声明                                  | 作用域 | 类型 | 是否支持变量提升                  |
| -------- | --------------------------------------------- | ------ | ---- | --------------------------------- |
| var      | 能                                            | 函数级 | 变量 | 是,undefined                      |
| let      | 不能,不允许在相同作用域内，重复声明同一个变量 | 块级   | 变量 | 否,referrenceError:is not defined |
| const    | 不能                                          | 块级   | 常量 | 否                                |

> *暂时性死区*：在代码**块内**，使用let命令**声明**变量**之前**，该变量都是**不可用**的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）

1. 箭头函数
   - 方便
     - 如果只有一个参数，（）可以省略
     - 如果只有一个语句且为return,{}可以省略
   - 修正this
     - 只会从自己的作用域链的上一层继承this
     - 箭头函数没有自己的this指针，通过 call() 或 apply() 方法调用一个函数时，只能传递参数
     - 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。所以this对象的指向是可变的，但是在箭头函数中，它是固定的。
     - 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误
     - 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替
     - 不可以使用yield命令，因此箭头函数不能用作 Generator 函数
   - 不适用场合
     - 定义对象的方法，且该方法内部包括this。
     - 需要动态this的时候，也不应使用箭头函数

> *作用域* ：一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的。

1. 参数的扩展...rest
   - 收集
   - 扩展
   - 默认参数
2. 数组方法
   - map 映射
   - reduce 汇总，一堆 -> 一个
   - filter 过滤
   - forEach 循环
3. 字符串
   - startWith
   - endWith
   - 字符串模板 `${a}xxx${b}`，即拼接变量
4. Promise
   - 封装异步操作
   - Promise.all([]).then()
5. generator
   - 执行一半能暂停
   - yield
6. JSON
   - JSON.stringfy()
   - JSON.parse()
7. 面向对象
   - class Test{}
8. 解构赋值
   - 左右结构一样
   - 右边是合法事情
   - 声明赋值一次完成

# 十三、ES7 预览

- 数组
  - `arr.includes()` 数组是否包含某个东西
  - 数组的 arr.keys(), arr,entries()
  - for ... in 遍历数组 下标 key
  - for ... of 遍历数组 值 value, 不能用于json

```
let arr = ['a', 'b', 'c']
console.log(arr.includes(1))

for (let i in arr) {
    console.log(i) // 循环的时下标 key
}

for (let i of arr) {
    console.log(i) // 循环的是值 value
}
for (let i of arr.keys()) {
    console.log('>'+i)
}
for (let [key, value] of arr.entries()) {
    console.log('>' + key + value)
}

let json = { a: 12, b: 5, c: 7 }
for (let i in json) {
    console.log(i)
}
```

- 字符串
  - padStart()/padEnd() 指定宽度，不够就补空格或指定字符

```
console.log('=' + 'abcd'.padStart(6, '0') + '=')
console.log('=' + 'abcd'.padEnd(6, '0') + '=')
=00abcd=
=abcd00=
```

- 容忍度
  - [1, 2, 3,] 老版数组最后不能有逗号，新的可以有
  - 函数参数最后多的逗号也可以
- async await
  - 和 generator yield 类似
  - generator 不可以写成箭头函数， async 可以

```
async function show() {
    console.log(1)
    await
    console.log(2)
}
```



























参照：[ECMAScript 6 入门](http://es6.ruanyifeng.com/)

开课吧

<https://github.com/able8/hello-es6#1es6%E6%80%8E%E4%B9%88%E6%9D%A5%E7%9A%84>