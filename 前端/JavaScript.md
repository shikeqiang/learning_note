# 一、基础

## 1.alter()	

自带的弹出消息框的方法，可以在onclick中直接调用，如：onclick="alert('欢迎!')"；点击后就会弹出消息框；

## 2.匹配属性

​	如下，先获取myimage这个id，赋值给element，然后判断element的src属性中是否有bulboff这个字段，然后修改其src属性。

```js
<script>
function changeImage()
{
	element=document.getElementById('myimage')
	if (element.src.match("bulboff"))
	{
		element.src="/images/pic_bulbon.gif";
	}
	else
	{
		element.src="/images/pic_bulboff.gif";
	}
}
</script>
```

## 3.用法

​	通常的做法是把函数放入\<head> 部分中，或者放在页面底部。这样就可以把它们安置到同一处位置，不会干扰页面的内容。外部脚本不能包含 <script> 标签。

## 4.输出

JavaScript 可以通过不同的方式来输出数据：

- 使用 **window.alert()** 弹出警告框。

- 使用 **document.write()** 方法将内容写到 HTML 文档中。

  > ```HTML
  > <p>我的第一个段落。</p>
  > <script>
  > document.write(Date());
  > </script>
  > ```

  使用 document.write() 仅仅向文档输出写内容。如果在文档已完成加载后执行 document.write，整个 HTML 页面将被覆盖。

  如下，日期会覆盖之前的内容，因为点击事件是后面在页面加载完再去点击触发的：

  ```html
  <body>
  <h1>我的第一个 Web 页面</h1>
  <p>我的第一个段落。</p>
  <button onclick="myFunction()">点我</button>
  <script>
  function myFunction() {
     	document.write(Date());
  }
  </script>
  </body>
  ```

- 使用 **innerHTML** 写入到 HTML 元素。

  > ```HTML
  > <p id="demo">我的第一个段落</p>
  > <script>
  > document.getElementById("demo").innerHTML = "段落已修改。";
  > </script>
  > ```

  **document.getElementById("demo")** 是使用 id 属性来查找 HTML 元素的 JavaScript 代码 。

  **innerHTML = "段落已修改。"** 是用于修改元素的 HTML 内容(innerHTML)的 JavaScript 代码。

- 使用 **console.log()** 写入到浏览器的控制台。

## 5.语法

JavaScript 使用 Unicode 字符集。Unicode 覆盖了所有的字符，包含标点等字符。

## 6.数据类型

- JavaScript 拥有动态类型

  JavaScript 拥有动态类型。这意味着相同的变量可用作不同的类型：

  > 实例
  >
  > var x;               // x 为 undefined
  > var x = 5;           // 现在 x 为数字
  > var x = "John";      // 现在 x 为字符串

- 字符串

  字符串是存储字符（比如 "Bill Gates"）的变量。字符串可以是引号中的任意文本。您可以使用单引号或双引号。

- JavaScript 只有一种数字类型。数字可以带小数点，也可以不带；极大或极小的数字可以通过科学（指数）计数法来书写。

- 数组

  > var cars=new Array();
  > cars[0]="Saab";
  > cars[1]="Volvo";
  > cars[2]="BMW";
  >
  > 或：var cars=new Array("Saab","Volvo","BMW");
  >
  > 或：var cars=["Saab","Volvo","BMW"];

- 对象

  对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：

  var person={firstname:"John", lastname:"Doe", id:5566};

  - 对象属性有两种寻址方式：

    > name=person.lastname;
    > name=person["lastname"];

- 声明变量类型

  当您声明新变量时，可以使用关键词 "new" 来声明其类型：

  > var carname=new String;
  > var x=      new Number;
  > var y=      new Boolean;
  > var cars=   new Array;
  > var person= new Object;

- 数据类型

> 在 JavaScript 中有 5 种不同的数据类型：
>
> - string
> - number
> - boolean
> - object
> - function
>
> 3 种对象类型：
>
> - Object
> - Date
> - Array
>
> 2 个不包含任何值的数据类型：
>
> - null
> - undefined

- **constructor** 属性返回所有 JavaScript 变量的构造函数。返回String()等类型的构造函数
- 类型转换
  - 全局方法 **String()** 可以将数字转换为字符串；该方法可用于任何类型的数字，字母，变量，表达式；Number 方法 **toString()** 也是有同样的效果。String()还可以将布尔类型的转换为字符串

## 7.函数

​	可以用return返回值，当执行完return时，函数会停止执行，也可以当初使用return去结束函数；可以用变量去接收返回的值。

- 局部变量

  > 在 JavaScript 函数内部声明的变量（使用 var）是*局部*变量，所以只能在函数内部访问它。（该变量的作用域是局部的），可以在不同的函数中使用名称相同的局部变量，因为只有声明过该变量的函数才能识别出该变量。只要函数运行完毕，本地变量就会被删除。

  在函数外声明的变量是*全局*变量，网页上的所有脚本和函数都能访问它。

- 变量生存期

  > JavaScript 变量的生命期从它们被声明的时间开始；局部变量会在函数运行以后被删除；全局变量会在页面关闭后被删除。

- 未声明的变量

  如果把值赋给尚未声明的变量，该变量将被自动作为 window 的一个属性。

  如：var1="Volvo";   可以用 console.log(window.var1); 打印，可以用delete var1删除。

## 8.常见的HTML事件

| 事件        | 描述                         |
| ----------- | ---------------------------- |
| onchange    | HTML 元素改变                |
| onclick     | 用户点击 HTML 元素           |
| onmouseover | 用户在一个HTML元素上移动鼠标 |
| onmouseout  | 用户从一个HTML元素上移开鼠标 |
| onkeydown   | 用户按下键盘按键             |
| onload      | 浏览器已完成页面的加载       |

## 9.字符串

​	可以用单引号或者双引号表示字符串，可以使用索引位置来访问字符串中的每个字符。

常见的转义字符：

| 代码 | 输出        |
| ---- | ----------- |
| \'   | 单引号      |
| \"   | 双引号      |
| \\   | 反斜杠      |
| \n   | 换行        |
| \r   | 回车        |
| \t   | tab(制表符) |
| \b   | 退格符      |
| \f   | 换页符      |

通常， JavaScript 字符串是原始值，可以使用字符创建： **var firstName = "John"**

但我们也可以使用 new 关键字将字符串定义为一个对象： **var firstName = new String("John")**

```js
var x = "John";
var y = new String("John");
typeof x // 返回 String
typeof y // 返回 Object
```

> var x = "John";              
> var y = new String("John");
> (x === y) // 结果为 false，因为 x 是字符串，y 是对象
>
> === 为绝对相等，即数据类型与值都必须相等。

原始值字符串，如 "John", 没有属性和方法(因为他们不是对象)。

原始值可以使用 JavaScript 的属性和方法，因为 JavaScript 在执行方法和属性时可以把原始值当作对象。

## 10.break和continue标签

break和continue这两个关键字带或不带标签都可以跳出代码块。

所谓标签引用，即如下，labelname: break labelname

```js
<script>
cars=["BMW","Volvo","Saab","Ford","asfa"];
list:{
	document.write(cars[0] + "<br>"); 
	document.write(cars[1] + "<br>"); 
	document.write(cars[2] + "<br>"); 
	continue list;
	document.write(cars[3] + "<br>"); 
	document.write(cars[4] + "<br>"); 
	document.write(cars[5] + "<br>"); 
}
</script>
```

## 11.正则表达式

语法：/正则表达式主体/修饰符(可选)

| 修饰符 | 描述                                                     |
| ------ | -------------------------------------------------------- |
| i      | 执行对大小写不敏感的匹配。                               |
| g      | 执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。 |
| m      | 执行多行匹配。                                           |

在 JavaScript 中，正则表达式通常用于两个字符串方法 : search() 和 replace()。

**search() 方法** 用于检索字符串中指定的子字符串，或检索与正则表达式相匹配的子字符串，并返回子串的起始位置。

**replace() 方法** 用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

```js
var str = "Visit Runoob!"; 
var n = str.search(/Runoob/i);
//返回：6
var str = document.getElementById("demo").innerHTML; 
var txt = str.replace(/microsoft/i,"Runoob");
// 等同于var txt = str.replace("Microsoft","Runoob");
// 返回：Visit Runoob!
```

### 正则表达式模式

方括号用于查找某个范围内的字符：

| 表达式 | 描述                       |
| ------ | -------------------------- |
| [abc]  | 查找方括号之间的任何字符。 |
| [0-9]  | 查找任何从 0 至 9 的数字。 |
| (x\|y) | 查找任何以 \| 分隔的选项。 |

元字符是拥有特殊含义的字符：

| 元字符 | 描述                                        |
| ------ | ------------------------------------------- |
| \d     | 查找数字。                                  |
| \s     | 查找空白字符。                              |
| \b     | 匹配单词边界。                              |
| \uxxxx | 查找以十六进制数 xxxx 规定的 Unicode 字符。 |

量词:

| 量词 | 描述                                  |
| ---- | ------------------------------------- |
| n+   | 匹配任何包含至少一个 *n* 的字符串。   |
| n*   | 匹配任何包含零个或多个 *n* 的字符串。 |
| n?   | 匹配任何包含零个或一个 *n* 的字符串。 |

**在 JavaScript 中，RegExp 对象是一个预定义了属性和方法的正则表达式对象。**

## 12.表单

### HTML约束验证

HTML5 新增了 HTML 表单的验证方式：约束验证（constraint validation）。

约束验证是表单被提交时浏览器用来实现验证的一种算法。

HTML 约束验证基于：

- **HTML 输入属性**
- **CSS 伪类选择器**
- **DOM 属性和方法**

#### 约束验证 HTML 输入属性

| 属性     | 描述                     |
| -------- | ------------------------ |
| disabled | 规定输入的元素不可用     |
| max      | 规定输入元素的最大值     |
| min      | 规定输入元素的最小值     |
| pattern  | 规定输入元素值的模式     |
| required | 规定输入元素字段是必需的 |
| type     | 规定输入元素的类型       |

完整列表，请查看 [HTML 输入属性](https://www.runoob.com/html/html5-form-attributes.html)。

#### 约束验证 CSS 伪类选择器

| 选择器    | 描述                                    |
| --------- | --------------------------------------- |
| :disabled | 选取属性为 "disabled" 属性的 input 元素 |
| :invalid  | 选取无效的 input 元素                   |
| :optional | 选择没有"required"属性的 input 元素     |
| :required | 选择有"required"属性的 input 元素       |
| :valid    | 选取有效值的 input 元素                 |

完整列表，请查看 [CSS 伪类](https://www.runoob.com/css/css-pseudo-classes.html)。

### JavaScript 表单验证

​	==JavaScript 可用来在数据被送往服务器前对 HTML 表单中的这些输入数据进行验证。==

> 表单数据经常需要使用 JavaScript 来验证其正确性：
>
> - 验证表单数据是否为空？
> - 验证输入是否是一个正确的email地址？
> - 验证日期是否输入正确？
> - 验证表单输入内容是否为数字型？

下面的函数用来检查用户是否已填写表单中的必填（或必选）项目。假如必填或必选项为空，那么警告框会弹出，并且函数的返回值为 false，否则函数的返回值则为 true（意味着数据没有问题）：

```HTML
<script>
function validateForm(){
    // 获取myForm标签中的fname属性
var x=document.forms["myForm"]["fname"].value;
if (x==null || x==""){
  alert("姓必须填写");
  return false;
  }
}
</script>
</head>
<body>
	
<form name="myForm" action="demo-form.php" onsubmit="return validateForm()" method="post">
姓: <input type="text" name="fname">
<input type="submit" value="提交">
</form>
```

### E-mail 验证

下面的函数检查输入的数据是否符合电子邮件地址的基本语法。

意思就是说，输入的数据必须包含 @ 符号和点号(.)。同时，@ 不可以是邮件地址的首字符，并且 @ 之后需有至少一个点号：

```HTML
<script>
function validateForm(){
    // 获取myForm这个标签的email的value
	var x=document.forms["myForm"]["email"].value;
	var atpos=x.indexOf("@");
	var dotpos=x.lastIndexOf(".");
	if (atpos<1 || dotpos<atpos+2 || dotpos+2>=x.length){
		alert("不是一个有效的 e-mail 地址");
  		return false;
	}
}
</script>
</head>
<body>
	
<form name="myForm" action="demo-form.php" onsubmit="return validateForm();" method="post">
Email: <input type="text" name="email">
<input type="submit" value="提交">
</form>
```

### JavaScript 验证 API

#### 约束验证 DOM 方法

| Property            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| checkValidity()     | 如果 input 元素中的数据是合法的返回 true，否则返回 false。   |
| setCustomValidity() | 设置 input 元素的 validationMessage 属性，用于自定义错误提示信息的方法。使用 setCustomValidity 设置了自定义提示后，validity.customError 就会变成true，则 checkValidity 总是会返回false。如果要重新判断需要取消自定义提示，方式如下：`setCustomValidity('')  ，setCustomValidity(null)，  setCustomValidity(undefined)` |

```HTML
<body>
<p>输入数字并点击验证按钮:</p>
<input id="id1" type="number" min="100" max="300" required>
<button onclick="myFunction()">验证</button>
<p>如果输入的数字小于 100 或大于300，会提示错误信息。</p>
<p id="demo"></p>
<script>
function myFunction() {
    // 获取"id1"这个对象
    var inpObj = document.getElementById("id1");
    if (inpObj.checkValidity() == false) {
        document.getElementById("demo").innerHTML = inpObj.validationMessage;
    } else {
        document.getElementById("demo").innerHTML = "输入正确";
    }
}
</script>
</body>
```

### 约束验证 DOM 属性

| 属性              | 描述                                  |
| ----------------- | ------------------------------------- |
| validity          | 布尔属性值，返回 input 输入值是否合法 |
| validationMessage | 浏览器错误提示信息                    |
| willValidate      | 指定 input 是否需要验证               |

------

### Validity 属性

input 元素的 **validity 属性**包含一系列关于 validity 数据属性:

| 属性            | 描述                                                       |
| --------------- | ---------------------------------------------------------- |
| customError     | 设置为 true, 如果设置了自定义的 validity 信息。            |
| patternMismatch | 设置为 true, 如果元素的值不匹配它的模式属性。              |
| rangeOverflow   | 设置为 true, 如果元素的值大于设置的最大值。                |
| rangeUnderflow  | 设置为 true, 如果元素的值小于它的最小值。                  |
| stepMismatch    | 设置为 true, 如果元素的值不是按照规定的 step 属性设置。    |
| tooLong         | 设置为 true, 如果元素的值超过了 maxLength 属性设置的长度。 |
| typeMismatch    | 设置为 true, 如果元素的值不是预期相匹配的类型。            |
| valueMissing    | 设置为 true，如果元素 (required 属性) 没有值。             |
| valid           | 设置为 true，如果元素的值是合法的。                        |

```HTML
<p>输入数字并点击验证按钮:</p>
<input id="id1" type="number" max="100">
<button onclick="myFunction()">验证</button>
<p>如果输入的数字大于 100 ( input 的 max 属性), 会显示错误信息。</p>
<p id="demo"></p>
<script>
function myFunction() {
    var txt = "";
    if (document.getElementById("id1").validity.rangeOverflow) {
        txt = "输入的值太大了";
    } else {
        txt = "输入正确";
    }
    document.getElementById("demo").innerHTML = txt;
}
</script>
```

## 13.this关键字

面向对象语言中 this 表示当前对象的一个引用。

但在 JavaScript 中 this 不是固定不变的，它会随着执行环境的改变而改变。

- ==在方法中，this 表示该方法所属的对象==。

- 在函数中，this 表示全局对象。

- 在函数中，在严格模式下，this 是未定义的(undefined)。

- 在事件中，this 表示接收事件的元素。

  > ```
  > <h2>JavaScript <b>this</b> 关键字</h2>
  > //修改button的样式，点击后button会消失
  > <button onclick="this.style.display='none'">点我后我就消失了</button>
  > ```

- 类似 call() 和 apply() 方法可以将 this 引用到任何对象。

- 单独使用 this，则它指向全局(Global)对象。在浏览器中，window 就是该全局对象为 [**object Window**]。

```HTML
<h2>JavaScript <b>this</b> 关键字</h2>
<p>实例中，<b>this</b> 指向了 window 对象:</p>
<p id="demo"></p>
<script>
var x = this;
document.getElementById("demo").innerHTML = x;
</script>
//会输出
JavaScript this 关键字
实例中，this 指向了 window 对象:
[object Window]
```

- 显式函数绑定

  在 JavaScript 中函数也是对象，对象则有方法，apply 和 call 就是函数对象的方法。这两个方法异常强大，他们允许切换函数执行的上下文环境（context），即 this 绑定的对象。

  在下面实例中，==当我们使用 person2 作为参数来调用 person1.fullName 方法时, **this** 将指向 person2, 即便它是 person1 的方法==：

  ```HTML
  <h2>JavaScript this 关键字</h2>
  <p>实例中 <strong>this</strong> 指向了 person2，即便它是 person1 的方法:</p>
  <p id="demo"></p>
  <script>
  var person1 = {
    fullName: function() {
      return this.firstName + " " + this.lastName;
    }
  }
  var person2 = {
    firstName:"John",
    lastName: "Doe",
  }
  var x = person1.fullName.call(person2); 
  document.getElementById("demo").innerHTML = x; 
  </script>
  ```

## 14.let和const

let 声明的变量只在 let 命令所在的代码块内有效。

const 声明一个只读的常量，一旦声明，常量的值就不能改变。

使用 var 关键字声明的变量不具备块级作用域的特性，它在 {} 外依然能被访问到。

在 ES6 之前，是没有块级作用域的概念的，ES6 可以使用 let 关键字来实现块级作用域。

==let 声明的变量只在 let 命令所在的代码块 **{}** 内有效，在 **{}** 之外不能访问。==

```js
var x = 10;
// 这里输出 x 为 10
{ 
    let x = 2;
    // 这里输出 x 为 2
}
// 这里输出 x 为 10
```

在 JavaScript 中, 全局作用域是针对 JavaScript 环境。在 HTML 中, 全局作用域是针对 window 对象。

使用 **var** 关键字声明的全局作用域变量属于 window 对象，使用 **let** 关键字声明的全局作用域变量不属于 window 对象。

==const 用于声明一个或多个常量，声明时必须进行初始化，且初始化后值不可再修改。==

const定义常量与使用let 定义的变量相似：

- 二者都是块级作用域
- 都不能和它所在作用域内的其他变量或函数拥有相同的名称

两者还有以下两点区别：

- `const`声明的常量必须初始化，而`let`声明的变量不用
- const 定义常量的值不能通过再赋值修改，也不能再次声明。而 let 定义的变量值可以修改。

**const 的本质: const 定义的变量并非常量，并非不可变，它定义了一个常量引用一个值。使用 const 定义的对象或者数组，其实是可变的。**

## 15.JSON

JSON 是用于存储和传输数据的格式，它通常用于服务端向网页传递数据 。

> ==JSON 使用 JavaScript 语法，但是 JSON 格式仅仅是一个文本。文本可以被任何编程语言读取及作为数据格式传递。==

### 语法规则

- 数据为 键/值 对。
- 数据由逗号分隔。
- 大括号保存对象
- 方括号保存数组

### JSON 对象保存在大括号内。

就像在 JavaScript 中, 对象可以保存多个 键/值 对：

{"name":"Runoob", "url":"www.runoob.com"}

### JSON 数组保存在中括号内。

就像在 JavaScript 中, 数组可以包含对象：

"sites":[     {"name":"Runoob", "url":"www.runoob.com"},      {"name":"Google", "url":"www.google.com"},     {"name":"Taobao", "url":"www.taobao.com"} ]

```HTML
<h2>为 JSON 字符串创建对象</h2>
<p id="demo"></p>
<script>
var text = '{ "sites" : [' +
	'{ "name":"Runoob" , "url":"www.runoob.com" },' +
	'{ "name":"Google" , "url":"www.google.com" },' +
	'{ "name":"Taobao" , "url":"www.taobao.com" } ]}';
	
obj = JSON.parse(text);	// 转换为JS对象
document.getElementById("demo").innerHTML = obj.sites[2].name + " " + obj.sites[1].url;
</script>
```

| 函数                                                         | 描述                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| [JSON.parse()](https://www.runoob.com/js/javascript-json-parse.html) | 用于将一个 JSON 字符串转换为 JavaScript 对象。 |
| [JSON.stringify()](https://www.runoob.com/js/javascript-json-stringify.html) | 用于将 JavaScript 值转换为 JSON 字符串。       |

## 16.void

**href="#"与href="javascript:void(0)"的区别**

**#** 包含了一个位置信息，默认的锚是**#top** 也就是网页的上端；而javascript:void(0), 仅仅表示一个死链接。

在页面很长的时候会使用 **#** 来定位页面的具体位置，格式为：**# + id**。

如果你要定义一个死链接请使用 javascript:void(0) 。

# 二、HTML DOM

当网页被加载时，浏览器会创建页面的文档对象模型（Document Object Model）。

**HTML DOM** 模型被构造为**对象**的树：

![DOM HTML tree](/Users/jack/Desktop/md/images/pic_htmltree.gif)

通过可编程的对象模型，JavaScript 获得了足够的能力来创建动态的 HTML。

- JavaScript 能够改变页面中的所有 HTML 元素
- JavaScript 能够改变页面中的所有 HTML 属性
- JavaScript 能够改变页面中的所有 CSS 样式
- JavaScript 能够对页面中的所有事件做出反应

## 查找 HTML 元素

有三种查找HTML元素的方法：

- 通过 id 找到 HTML 元素
- 通过标签名找到 HTML 元素
- 通过类名找到 HTML 元素

### 通过 id 查找 HTML 元素

在 DOM 中查找 HTML 元素的最简单的方法，是通过使用元素的 id。

本例查找 id="intro" 元素：var x=document.getElementById("intro");

> 如果找到该元素，则该方法将以对象（在 x 中）的形式返回该元素。如果未找到该元素，则 x 将包含 null。

### 通过标签名查找 HTML 元素

本例查找 id="main" 的元素，然后查找 id="main" 元素中的所有\<p> 元素：

> var x=document.getElementById("main");
> var y=x.getElementsByTagName("p");

### 通过类名找到 HTML 元素

本例通过 **getElementsByClassName** 函数来查找 class="intro" 的元素：

var x=document.getElementsByClassName("intro");

## 改变HTML

### 改变HTML标签的内容：

document.getElementById(*id*Name).innerHTML=*新的 HTML*

### 改变HTML属性：

document.getElementById(*id*).*attribute=新属性值*

> 比如改变图片的属性：
>
> ```HTML
> <img id="image" src="smiley.gif">
> <script>
> document.getElementById("image").src="landscape.jpg";
> </script>
> ```

## 改变CSS

### 改变HTML样式：

document.getElementById(*id*).style.*property*=*新样式*

如下面改变p标签的颜色，字体和大小

```HTML
<body>
<p id="p1">Hello World!</p>
<p id="p2">Hello World!</p>
<script>
document.getElementById("p2").style.color="blue";
document.getElementById("p2").style.fontFamily="Arial";
document.getElementById("p2").style.fontSize="larger";
</script>
<p>以上段落通过脚本修改。</p>
</body>
```

## HTML DOM事件

在事件发生时执行 JavaScript，比如当用户在 HTML 元素上点击时。如需在用户点击某个元素时执行代码，请向一个 HTML 事件属性添加 JavaScript 代码：

onclick=*JavaScript*

HTML 事件的例子：

- 当用户点击鼠标时
- 当网页已加载时
- 当图像已加载时
- 当鼠标移动到元素上时
- 当输入字段被改变时
- 当提交 HTML 表单时
- 当用户触发按键时

如：\<h1 onclick="this.innerHTML='Ooops!'">点击文本!\</h1>     点击时文本会变成Ooops

## 各种事件

- **onload 和 onunload 事件，它们会在用户进入或离开==页面==时被触发。**

> onload 事件可用于检测访问者的浏览器类型和浏览器版本，并基于这些信息来加载网页的正确版本。

- onload 和 onunload 事件可用于处理 cookie。
- onchange 事件，常结合对输入字段的验证来使用。
- onmouseover 和 onmouseout 事件可用于在用户的鼠标移至 HTML 元素上方或移出元素时触发函数。
- **onmousedown, onmouseup 以及 onclick 构成了鼠标点击事件的所有部分。**首先当点击鼠标按钮时，会触发 onmousedown 事件，当释放鼠标按钮时，会触发 onmouseup 事件，最后，当完成鼠标点击时，会触发 onclick 事件。

### addEventListener() 方法

addEventListener() 方法用于向指定元素添加事件句柄，它添加的事件句柄不会覆盖已存在的事件句柄。

可以向一个元素添加多个事件句柄，也可以向同个元素添加多个同类型的事件句柄，如：两个 "click" 事件。

可以向任何 DOM 对象添加事件监听，不仅仅是 HTML 元素。如： window 对象。

addEventListener() 方法可以更简单的控制事件（冒泡与捕获）。

当你使用 addEventListener() 方法时, JavaScript 从 HTML 标记中分离开来，可读性更强， 在没有控制HTML标记时也可以添加事件监听。

可以使用 removeEventListener() 方法来移除事件的监听。

> 如在用户点击按钮时触发监听事件：
>
> document.getElementById("myBtn").addEventListener("click", displayDate);

#### 语法

==*element*.addEventListener(*event, function, useCapture*);==

> 第一个参数是事件的类型 (如 "click" 或 "mousedown").
>
> 第二个参数是事件触发后调用的函数。
>
> 第三个参数是个布尔值用于描述事件是冒泡还是捕获。该参数是可选的。
>
> **注意:不要使用 "on" 前缀。 例如，使用 "click" ,而不是使用 "onclick"。**

#### 向原元素添加事件句柄

可以引用外部函数名

```js
element.addEventListener("click", myFunction);
function myFunction() {
    alert ("Hello World!");
}
```

#### 向同一个元素中添加多个事件句柄

addEventListener() 方法允许向同一个元素添加多个事件，且不会覆盖已存在的事件：

```js
element.addEventListener("mouseover", myFunction);
element.addEventListener("click", mySecondFunction);
element.addEventListener("mouseout", myThirdFunction);
```

#### 向 Window 对象添加事件句柄

​	addEventListener() 方法允许你在 HTML DOM 对象添加事件监听， HTML DOM 对象如： HTML 元素, HTML 文档, window 对象。或者其他支出的事件对象如: xmlHttpRequest 对象。

```JS
//当用户重置窗口大小时添加事件监听：
window.addEventListener("resize", function(){
    document.getElementById("demo").innerHTML = sometext;
});
```

#### 传递参数

当传递参数值时，使用"匿名函数"调用带参数的函数：

```HTML
<button id="myBtn">点我</button>
<p id="demo"></p>
<script>
var p1 = 5;
var p2 = 7;
    //不能直接引用myFunction(p1, p2)
document.getElementById("myBtn").addEventListener("click", function() {
    myFunction(p1, p2);
});
function myFunction(a, b) {
    var result = a * b;
    document.getElementById("demo").innerHTML = result;
}
</script>
```

#### 冒泡与捕获

==事件传递有两种方式：冒泡与捕获。==

事件传递定义了元素事件触发的顺序。 如果你将\<p> 元素插入到 \<div> 元素中，用户点击 \<p> 元素, 哪个元素的 "click" 事件先被触发呢？

在 *冒泡* 中，内部元素的事件会先被触发，然后再触发外部元素，即： \<p> 元素的点击事件先触发，然后会触发 \<div> 元素的点击事件。

在 *捕获* 中，外部元素的事件会先被触发，然后才会触发内部元素的事件，即： \<div> 元素的点击事件先触发 ，然后再触发 \<p> 元素的点击事件。

addEventListener() 方法可以指定 "useCapture" 参数来设置传递类型：

```JS
addEventListener(event, function, useCapture);
```

默认值为 false, 即冒泡传递，当值为 true 时, 事件使用捕获传递。

#### removeEventListener() 方法

> removeEventListener() 方法移除由 addEventListener() 方法添加的事件句柄:
>
> *element*.removeEventListener("mousemove", myFunction);

#### 创建新的 HTML 元素 (节点) - appendChild()

要创建新的 HTML 元素 (节点)需要先创建一个元素，然后在已存在的元素中添加它。

```js
<script>
var para = document.createElement("p");		// 创建 <p> 元素	
var node = document.createTextNode("这是一个新的段落。");	//为 <p> 元素添加文本节点
para.appendChild(node);		// 将文本节点添加到 <p> 元素中,在一个已存在的元素中添加 p 元素。
 
var element = document.getElementById("div1");	//查找已存在的元素
element.appendChild(para);	//添加到已存在的元素中
</script>
```

#### 创建新的 HTML 元素 (节点) - insertBefore()

将新元素添加到开始位置，可以使用 **insertBefore()** 方法:

#### 移除已存在的元素

要移除一个元素，你需要知道该元素的父元素,removeChild()方法

#### 替换 HTML 元素 - replaceChild()

### DOM集合

getElementsByTagName() 方法返回 [HTMLCollection](https://www.runoob.com/jsref/dom-htmlcollection.html) 对象。

```js
//获取文档所有的 <p> 元素：
var x = document.getElementsByTagName("p");
//集合中的元素可以通过索引(以 0 为起始位置)来访问,访问第二个 <p> 元素:
y = x[1];
//HTMLCollection 对象的 length 属性定义了集合中元素的数量，如：x.length表示<p> 元素的数量
```

修改所有\<p>元素的背景颜色：

```js
var myCollection = document.getElementsByTagName("p");
var i;
for (i = 0; i < myCollection.length; i++) {
    myCollection[i].style.backgroundColor = "red";
}
```

# 二、高级













