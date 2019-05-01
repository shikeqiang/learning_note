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

















