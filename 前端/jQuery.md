# 一、简介

jQuery库包含以下功能：

- HTML 元素选取
- HTML 元素操作
- CSS 操作
- HTML 事件函数
- JavaScript 特效和动画
- HTML DOM 遍历和修改
- AJAX
- Utilities

**提示：** 除此之外，Jquery还提供了大量的插件。

## 1.语法

jQuery 语法是通过选取 HTML 元素，并对选取的元素执行某些操作。

基础语法： **$(selector).action()**

- 美元符号定义 jQuery
- 选择符（selector）"查询"和"查找" HTML 元素
- jQuery 的 action() 执行对元素的操作

实例:

- $(this).hide() - 隐藏当前元素
- $("p").hide() - 隐藏所有 <p> 元素
- $("p.test").hide() - 隐藏所有 class="test" 的 <p> 元素
- $("#test").hide() - 隐藏所有 id="test" 的元素

### 文档就绪事件

```js
$(document).ready(function(){
   // 开始写 jQuery 代码...
});
```

这是为了防止文档在完全加载（就绪）之前运行 jQuery 代码，即在 DOM 加载完成后才可以对 DOM 进行操作。如果在文档没有完全加载之前就运行函数，操作可能失败。

## 2.jQuery 选择器

jQuery 选择器可以对 HTML 元素组或单个元素进行操作。

**jQuery 中所有选择器都以美元符号开头：$()。**

#### 元素选择器

jQuery 元素选择器基于元素名选取元素。

在页面中选取所有 \<p> 元素:$("p")

#### #id 选择器

jQuery #id 选择器通过 HTML 元素的 id 属性选取指定的元素。

页面中元素的 id 应该是唯一的，所以您要在页面中选取唯一的元素需要通过 #id 选择器。

通过 id 选取元素语法如下：$("#test")

#### .class 选择器

jQuery 类选择器可以通过指定的 class 查找元素。

语法如下：$(".test")

#### 更多实例

| 语法                     | 描述                                                    | 实例                                                         |
| ------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| $("*")                   | 选取所有元素                                            | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_all2) |
| $(this)                  | 选取当前 HTML 元素                                      | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_this) |
| $("p.intro")             | 选取 class 为 intro 的 <p> 元素                         | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_pclass) |
| $("p:first")             | 选取第一个 <p> 元素                                     | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_pfirst) |
| $("ul li:first")         | 选取第一个 <ul> 元素的第一个 <li> 元素                  | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_ullifirst) |
| $("ul li:first-child")   | 选取每个 <ul> 元素的第一个 <li> 元素                    | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_ullifirstchild) |
| $("[href]")              | 选取带有 href 属性的元素                                | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_hrefattr) |
| $("a[target='_blank']")  | 选取所有 target 属性值等于 "_blank" 的 <a> 元素         | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_hrefattrblank) |
| $("a[target!='_blank']") | 选取所有 target 属性值不等于 "_blank" 的 <a> 元素       | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_hrefattrnotblank) |
| $(":button")             | 选取所有 type="button" 的 <input> 元素 和 <button> 元素 | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_button2) |
| $("tr:even")             | 选取偶数位置的 <tr> 元素                                | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_even) |
| $("tr:odd")              | 选取奇数位置的 <tr> 元素                                | [在线实例](https://www.runoob.com/try/try.php?filename=tryjquery_sel_odd) |

## 3.常用的 jQuery 事件方法

- $(document).ready()，在文档完全加载完后执行函数。
- click(),当按钮点击事件被触发时会调用一个函数。
- dblclick()，当双击元素时，会发生 dblclick 事件。
- mouseenter()，当鼠标指针穿过元素时，会发生 mouseenter 事件。
- mouseleave()，当鼠标指针离开元素时，会发生 mouseleave 事件。
- mousedown()，当鼠标指针移动到元素上方，并按下鼠标按键时，会发生 mousedown 事件。
- mouseup()，当在元素上松开鼠标按钮时，会发生 mouseup 事件。
- hover(),用于模拟光标悬停事件，当鼠标移动到元素上时，会触发指定的第一个函数(mouseenter);当鼠标移出这个元素时，会触发指定的第二个函数(mouseleave)。
- focus()，当元素获得焦点时，发生 focus 事件。当通过鼠标点击选中元素或通过 tab 键定位到元素时，该元素就会获得焦点。
- blur()，当元素失去焦点时，发生 blur 事件。

# 二、jQuery效果

## hide() 和 show()

用来隐藏和显示 HTML 元素，

> $(*selector*).hide(*speed,callback*);
>
> $(*selector*).show(*speed,callback*);
>
> 可选的 speed 参数规定隐藏/显示的速度，可以取以下值："slow"、"fast" 或毫秒。
>
> 可选的 callback 参数是隐藏或显示完成后所执行的函数名称。

## toggle()

用来切换 hide() 和 show() 方法，可以隐藏和显示。

> $(*selector*).toggle(*speed,callback*);
>
> 可选的 speed 参数规定隐藏/显示的速度，可以取以下值："slow"、"fast" 或毫秒。
>
> 可选的 callback 参数是隐藏或显示完成后所执行的函数名称。

## Fading 方法

fadeIn() 用于淡入已隐藏的元素。

> $(*selector*).fadeIn(*speed,callback*);
>
> 可选的 speed 参数规定效果的时长。它可以取以下值："slow"、"fast" 或毫秒。.
>
> 可选的 callback 参数是 fading 完成后所执行的函数名称。

fadeOut() 方法用于淡出可见元素。(语法同上面的)

fadeToggle() 方法可以在 fadeIn() 与 fadeOut() 方法之间进行切换。(语法同上面的)

> 如果元素已淡出，则 fadeToggle() 会向元素添加淡入效果。
>
> 如果元素已淡入，则 fadeToggle() 会向元素添加淡出效果。

fadeTo() 方法允许渐变为给定的不透明度（值介于 0 与 1 之间）。

> $(*selector*).fadeTo(*speed,opacity,callback*);
>
> 必需的 speed 参数规定效果的时长。它可以取以下值："slow"、"fast" 或毫秒。
>
> fadeTo() 方法中必需的 opacity 参数将淡入淡出效果设置为给定的不透明度（值介于 0 与 1 之间）。
>
> 可选的 callback 参数是该函数完成后所执行的函数名称。

## 滑动方法

 slideDown() 方法用于向下滑动元素。

> $(*selector*).slideDown(*speed,callback*);
>
> 可选的 speed 参数规定效果的时长。它可以取以下值："slow"、"fast" 或毫秒。
>
> 可选的 callback 参数是滑动完成后所执行的函数名称。

 slideUp() 方法用于向上滑动元素。

slideToggle() 方法可以在 slideDown() 与 slideUp() 方法之间进行切换。

> 如果元素向下滑动，则 slideToggle() 可向上滑动它们。
>
> 如果元素向上滑动，则 slideToggle() 可向下滑动它们。

## 动画

animate() 方法用于创建自定义动画。

> $(*selector*).animate({*params*}*,speed,callback*);
>
> 必需的 params 参数定义形成动画的 CSS 属性。
>
> 可选的 speed 参数规定效果的时长。它可以取以下值："slow"、"fast" 或毫秒。
>
> 可选的 callback 参数是动画完成后所执行的函数名称。

**注意：默认情况下，所有 HTML 元素都有一个静态位置，且无法移动。如需对位置进行操作，要记得首先把元素的 CSS position 属性设置为 relative、fixed 或 absolute！**

animate() 方法 操作多个属性

```js
$("button").click(function(){
  $("div").animate({
    left:'250px',
    opacity:'0.5',		// 透明度
    height:'150px',
    width:'150px'
  });
});
```

> **可以用 animate() 方法来操作所有 CSS 属性**，当使用 animate() 时，必须使用 Camel 标记法书写所有的属性名，比如，必须使用 paddingLeft 而不是 padding-left，使用 marginRight 而不是 margin-right，等等。
>
> 如果需要生成颜色动画，您需要从 [jquery.com](http://jquery.com/download/) 下载 [颜色动画](http://plugins.jquery.com/color/) 插件。

animate() - 使用相对值

```js
$("button").click(function(){
  $("div").animate({
    left:'250px',
    height:'+=150px',
    width:'+=150px'
  });
});
```

animate() - 使用预定义的值，可以把属性的动画值设置为 "show"、"hide" 或 "toggle"：

```js
$("button").click(function(){
  $("div").animate({
    height:'toggle'
  });
});
```

animate() - 使用队列功能

> 默认地，jQuery 提供针对动画的队列功能。
>
> 这意味着如果您在彼此之后编写多个 animate() 调用，jQuery 会创建包含这些方法调用的"内部"队列。然后逐一运行这些 animate 调用。

```js
$("button").click(function(){
  var div=$("div");
  div.animate({height:'300px',opacity:'0.4'},"slow");
  div.animate({width:'300px',opacity:'0.8'},"slow");
  div.animate({height:'100px',opacity:'0.4'},"slow");
  div.animate({width:'100px',opacity:'0.8'},"slow");
});
//把 <div> 元素往右边移动了 100 像素，然后增加文本的字号
$("button").click(function(){
  var div=$("div");
  div.animate({left:'100px'},"slow");
  div.animate({fontSize:'3em'},"slow");
});
```

 stop() 方法用于停止动画或效果，在它们完成之前。

> stop() 方法适用于所有 jQuery 效果函数，包括滑动、淡入淡出和自定义动画。
>
> $(*selector*).stop(*stopAll,goToEnd*);
>
> 可选的 stopAll 参数规定是否应该清除动画队列。默认是 false，即仅停止活动的动画，允许任何排入队列的动画向后执行。
>
> 可选的 goToEnd 参数规定是否立即完成当前动画。默认是 false。
>
> 默认地，stop() 会清除在被选元素上指定的当前动画。

Callback 回调函数在当前动画 100% 完成之后执行。

```js
//在隐藏效果完全实现后回调函数，即在hide之后又有个function
$("button").click(function(){
  $("p").hide("slow",function(){
    alert("段落现在被隐藏了");
  });
});
```

## 链(Chaining)

Chaining 允许我们在一条语句中运行多个 jQuery 方法（在相同的元素上）。链接（chaining）的技术，允许我们在相同的元素上运行多条 jQuery 命令，一条接着另一条。

**提示：** 这样的话，浏览器就不必多次查找相同的元素。

如需链接一个动作，您只需简单地把该动作追加到之前的动作上。

下面的例子把 css()、slideUp() 和 slideDown() 链接在一起。"p1" 元素首先会变为红色，然后向上滑动，再然后向下滑动：

```js
$("#p1").css("color","red")
  .slideUp(2000)
  .slideDown(2000);
```

# 三、jQuery HTML

## 获得/设置内容 - text()、html() 以及 val()

三个简单实用的用于 DOM 操作的 jQuery 方法：

- text() - 设置或返回所选元素的文本内容
- html() - 设置或返回所选元素的内容（包括 HTML 标记，会显示HTML相关标签）
- val() - 设置或返回表单字段的值

设置内容时，只要在函数里面填写对应的内容即可，注意html()函数要在里面填写对应的HTML标签。

## text()、html() 以及 val() 的回调函数

回调函数有两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

```js
$("#btn1").click(function(){
    $("#test1").text(function(i,origText){
        return "旧文本: " + origText + " 新文本: Hello world! (index: " + i + ")"; 
    });
});
 
$("#btn2").click(function(){
    $("#test2").html(function(i,origText){
        return "旧 html: " + origText + " 新 html: Hello <b>world!</b> (index: " + i + ")"; 
    });
});
```

## 获取/设置属性 - attr()

```html
<script>
$(document).ready(function(){
  $("button").click(function(){
    alert($("#runoob").attr("href"));	//获得链接中 href 属性的值
    $("#runoob").attr("href","http://www.runoob.com/jquery");	// 设置属性
  });
});
</script>

<body>
<p><a href="//www.runoob.com" id="runoob">菜鸟教程</a></p>
<button>显示 href 属性的值</button>
</body>
```

attr() 方法也允许您同时设置多个属性：

```js
$("button").click(function(){
    $("#runoob").attr({
        "href" : "http://www.runoob.com/jquery",
        "title" : "jQuery 教程"
    });
});
```

### attr() 的回调函数

回调函数有两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

```js
$("button").click(function(){
  $("#runoob").attr("href", function(i,origValue){
    return origValue + "/jquery"; 
  });
});
```

## 添加元素

### 添加新的 HTML 内容

我们将学习用于添加新内容的四个 jQuery 方法：

- append() - 在被选元素的结尾插入内容
- prepend() - 在被选元素的开头插入内容
- after() - 在被选元素之后插入内容
- before() - 在被选元素之前插入内容

```js
<script>
$(document).ready(function(){
	$("#btn1").click(function(){
		$("p").append(" <b>追加文本</b>。");
		$("p").prepend("<b>在开头追加文本</b>。 ");
	});
	$("#btn2").click(function(){
		$("ol").prepend("<li>在开头添加列表项</li>");
	});
});
</script>
```

多种方法追加元素：

```js
<script>
function appendText(){
	var txt1="<p>文本。</p>";              // 使用 HTML 标签创建文本
	var txt2=$("<p></p>").text("文本。");  // 使用 jQuery 创建文本
	var txt3=document.createElement("p");
	txt3.innerHTML="文本。";               // 使用 DOM 创建文本 text with DOM
	$("body").append(txt1,txt2,txt3);        // 追加新元素
}
</script>
```

## 删除元素

- remove() - 删除被选元素（及其子元素）
- empty() - 从被选元素中删除子元素

### 过滤被删除的元素

删除 class="italic" 的所有 \<p> 元素：$("p").remove(".italic");

## 获取并设置 CSS 类

- addClass() - 向被选元素添加一个或多个类
- removeClass() - 从被选元素删除一个或多个类
- toggleClass() - 对被选元素进行添加/删除类的切换操作
- css() - 设置或返回样式属性

```HTML
<script>
$(document).ready(function(){
  $("button").click(function(){
    $("h1,h2,p").addClass("blue");		// 添加相关的class属性
    $("div").addClass("important");
  });
  $("button").click(function(){
  $("body div:first").addClass("important blue");
	});
});
</script>
<style type="text/css">
.important
{
	font-weight:bold;
	font-size:xx-large;
}
.blue
{
	color:blue;
}
</style>
```

## css() 方法

css() 方法设置或返回被选元素的一个或多个样式属性。

css("*propertyname*");			返回指定的 CSS 属性的值

css("*propertyname*","*value*");	设置指定的 CSS 属性

css({"*propertyname*":"*value*","*propertyname*":"*value*",...});		设置多个指定的 CSS 属性

## 尺寸

![image-20190512121844324](/Users/jack/Desktop/md/images/image-20190512121844324.png)

jQuery 提供多个处理尺寸的重要方法：

- width()	设置或返回元素的宽度（不包括内边距、边框或外边距）。
- height()      设置或返回元素的高度（不包括内边距、边框或外边距）。
- innerWidth()              返回元素的宽度（包括内边距）。
- innerHeight()             返回元素的高度（包括内边距）。
- outerWidth()              返回元素的宽度（包括内边距和边框）。
- outerHeight()             返回元素的高度（包括内边距和边框）。

# 四、遍历

​	下图展示了一个家族树。通过 jQuery 遍历，您能够从被选（当前的）元素开始，轻松地在家族树中向上移动（祖先），向下移动（子孙），水平移动（同胞）。这种移动被称为对 DOM 进行遍历。

![jQuery Dimensions](/Users/jack/Desktop/md/images/img_travtree.png)

- <div> 元素是 <ul> 的父元素，同时是其中所有内容的祖先。
- <ul> 元素是 <li> 元素的父元素，同时是 <div> 的子元素
- 左边的 <li> 元素是 <span> 的父元素，<ul> 的子元素，同时是 <div> 的后代。
- <span> 元素是 <li> 的子元素，同时是 <ul> 和 <div> 的后代。
- 两个 <li> 元素是同胞（拥有相同的父元素）。
- 右边的 <li> 元素是 <b> 的父元素，<ul> 的子元素，同时是 <div> 的后代。
- <b> 元素是右边的 <li> 的子元素，同时是 <ul> 和 <div> 的后代。

> 祖先是父、祖父、曾祖父等等。后代是子、孙、曾孙等等。同胞拥有相同的父。

## 祖先

向上遍历 DOM 树

- parent()	返回被选元素的直接父元素，只会向**上一级**对 DOM 树进行遍历。

- parents()       返回被选元素的所有祖先元素，它一路向上直到文档的根元素 (\<html>)。也可以使用可选参数来过滤对祖先元素的搜索。

  > 如：返回所有 \<span> 元素的所有祖先，并且它是 \<ul> 元素：$("span").parents("ul");

- parentsUntil()        返回介于两个给定元素之间的所有祖先元素。

  > 如：返回介于 \<span> 与 \<div> 元素之间的所有祖先元素：
  >
  > $("span").parentsUntil("div");

## 后代

后代是子、孙、曾孙等等。

- children()	返回被选元素的**所有**直接子元素，只会向**下一级**对 DOM 树进行遍历。也可以使用可选参数来过滤对子元素的搜索。

  > 如：返回类名为 "1" 的所有 <p> 元素，并且它们是 <div> 的直接子元素：
  >
  >  $("div").children("p.1");

- find()               返回被选元素的后代元素，一路向下直到最后一个后代。

  > 返回属于 \<div> 后代的所有 \<span> 元素：$("div").find("span");
  >
  > 返回 \<div> 的所有后代：$("div").find("*");

## 同胞(siblings)

同胞拥有相同的父元素。

## 在 DOM 树中水平遍历

- siblings()		返回被选元素的所有同胞元素，也可以使用可选参数来过滤对同胞元素的搜索。
- next()                      返回被选元素的下一个同胞元素,只返回一个元素,即返回同级元素中的下一个。
- nextAll()                  返回被选元素的所有跟随的同胞元素，即被选元素下面所有同胞。
- nextUntil()              介于两个给定参数之间的**所有跟随的同胞元素**。
- prev()
- prevAll()
- prevUntil()

> 后面这三个prev方法与上面效果一样，只是方向相反。

## 过滤

三个最基本的过滤方法是：**first(), last() 和 eq()**，它们允许您基于其在一组元素中的位置来选择一个特定的元素。

其他过滤方法，比如 **filter() 和 not()** 允许您选取匹配或不匹配某项指定标准的元素。

- first() 方法返回被选元素的首个元素。

  > 选取首个 \<div> 元素内部的第一个 \<p> 元素：$("div p").first();

- last() 方法返回被选元素的最后一个元素。

- eq() 方法返回被选元素中带有指定索引号的元素。

  > 索引号从 0 开始，因此首个元素的索引号是 0 而不是 1。下面的例子选取第二个 \<p> 元素（索引号 1）：$("p").eq(1);

- filter() 方法允许规定一个标准，不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回。

  > 返回带有类名 "url" 的所有 \<p> 元素：$("p").filter(".url");

- not() 方法返回不匹配标准的所有元素,not() 方法与 filter() 相反。

# 五、AJAX

## load() 

从服务器加载数据，并把返回的数据放入被选元素中。

语法：$(selector).load(URL,data,callback);

必需的 *URL* 参数规定您希望加载的 URL。

可选的 *data* 参数规定与请求一同发送的查询字符串键/值对集合。

可选的 *callback* 参数是 load() 方法完成后所执行的函数名称。

> 实例：
>
> 文件**"demo_test.txt"**的内容：
>
> ```HTML
> <h2>jQuery AJAX 是个非常棒的功能！</h2>
> <p id="p1">这是段落的一些文本。</p>
> ```
>
> 把文件 "demo_test.txt" 的内容加载到指定的 \<div> 元素中：$("#div1").load("demo_test.txt");
>
> 可以把 jQuery 选择器添加到 URL 参数，把 "demo_test.txt" 文件中 id="p1" 的元素的内容，加载到指定的 \<div> 元素中：$("#div1").load("demo_test.txt #p1");

可选的 callback 参数规定当 load() 方法完成后所要允许的回调函数。回调函数可以设置不同的参数：

- *responseTxt* - 包含调用成功时的结果内容
- *statusTXT* - 包含调用的状态
- *xhr* - 包含 XMLHttpRequest 对象

在 load() 方法完成后显示一个提示框。如果 load() 方法已成功，则显示"外部内容加载成功！"，而如果失败，则显示错误消息：

```js
// 先加载，如果加载成功就弹出消息框，然后再更改对应的HTML内容
$("button").click(function(){
  $("#div1").load("demo_test.txt",function(responseTxt,statusTxt,xhr){
    if(statusTxt=="success")
      alert("外部内容加载成功!");
    if(statusTxt=="error")
      alert("Error: "+xhr.status+": "+xhr.statusText);
  });
});
```

## get() 和 post() 方法

​	**get() 和 post() 方法用于通过 HTTP GET 或 POST 请求从服务器请求数据。**

$.get() 方法通过 HTTP GET 请求从服务器上请求数据。

> $.get(*URL*,*callback*);
>
> 必需的 *URL* 参数规定您希望请求的 URL。
>
> 可选的 *callback* 参数是请求成功后所执行的函数名。
>
> 下面的例子使用 $.get() 方法从服务器上的一个文件中取回数据：
>
> ```js
> $("button").click(function(){
>   $.get("demo_test.php",function(data,status){
>     alert("数据: " + data + "\n状态: " + status);
>   });
> });
> ```
>
> $.get() 的第一个参数是我们希望请求的 URL（"demo_test.php"）。
>
> 第二个参数是回调函数。第一个回调参数存有被请求页面的内容，第二个回调参数存有请求的状态。

$.post() 方法通过 HTTP POST 请求向服务器提交数据。

> $.post(*URL,data,callback*);
>
> 必需的 *URL* 参数规定您希望请求的 URL。
>
> 可选的 *data* 参数规定连同请求发送的数据。
>
> 可选的 *callback* 参数是请求成功后所执行的函数名。
>
> 使用 $.post() 连同请求一起发送数据：
>
> ```js
> $("button").click(function(){
>     $.post("/try/ajax/demo_test_post.php",
>     {
>         name:"菜鸟教程",
>         url:"http://www.runoob.com"
>     },
>         function(data,status){
>         alert("数据: \n" + data + "\n状态: " + status);
>     });
> });
> ```
>
> $.post() 的第一个参数是我们希望请求的 URL ("demo_test_post.php")，接着连同请求（name 和 url）一起发送数据。"demo_test_post.php" 中的 PHP 脚本读取这些参数，对它们进行处理，然后返回结果。
>
> 第三个参数是回调函数。第一个回调参数存有被请求页面的内容，而第二个参数存有请求的状态。

# 六、noConflict() 方法

noConflict() 方法会释放对 $ 标识符的控制，这样其他脚本就可以使用它了。

> 通过全名替代简写的方式来使用 jQuery：
>
> ```js
> $.noConflict();
> jQuery(document).ready(function(){
>   jQuery("button").click(function(){
>     jQuery("p").text("jQuery 仍然在工作!");
>   });
> });
> ```
>
> 以创建自己的简写。noConflict() 可返回对 jQuery 的引用，您可以把它存入变量，以供稍后使用:
>
> ```js
> var jq = $.noConflict();
> jq(document).ready(function(){
>   jq("button").click(function(){
>     jq("p").text("jQuery 仍然在工作!");
>   });
> });
> ```
>
> 如果你的 jQuery 代码块使用 $ 简写，并且您不愿意改变这个快捷方式，那么您**可以把 $ 符号作为变量传递给 ready 方法。**这样就可以在函数内使用 $ 符号了 - 而在函数外，依旧不得不使用 "jQuery"：
>
> ```js
> $.noConflict();
> jQuery(document).ready(function($){
>   $("button").click(function(){
>     $("p").text("jQuery 仍然在工作!");
>   });
> });
> ```

# 七、jQuery 选择器

| 选择器                                                       | 实例                          | 选取                                                         |
| ------------------------------------------------------------ | ----------------------------- | ------------------------------------------------------------ |
| [*](https://www.runoob.com/jquery/jq-sel-all.html)           | $("*")                        | 所有元素                                                     |
| [#*id*](https://www.runoob.com/jquery/jq-sel-id.html)        | $("#lastname")                | id="lastname" 的元素                                         |
| [.*class*](https://www.runoob.com/jquery/jq-sel-class.html)  | $(".intro")                   | class="intro" 的所有元素                                     |
| [.*class,*.*class*](https://www.runoob.com/jquery/sel-multiple-classes.html) | $(".intro,.demo")             | ==class 为 "intro" **或** "demo" 的所有元素==                |
| [*element*](https://www.runoob.com/jquery/jq-sel-element.html) | $("p")                        | 所有 \<p> 元素                                               |
| [*el1*,*el2*,*el3*](https://www.runoob.com/jquery/sel-multiple-elements.html) | $("h1,div,p")                 | 所有 \<h1>、\<div> 和 \<p> 元                                |
| [:first](https://www.runoob.com/jquery/sel-first.html)       | $("p:first")                  | 第一个 <p> 元素                                              |
| [:last](https://www.runoob.com/jquery/sel-last.html)         | $("p:last")                   | 最后一个 <p> 元素                                            |
| [:even](https://www.runoob.com/jquery/sel-even.html)         | $("tr:even")                  | **:even 选择器**选取带有偶数索引号的每个元素（比如：0、2、4 等等）。所有偶数 \<tr> 元素，索引值从 0 开始，第一个元素是偶数 (0)，第二个元素是奇数 (1)，以此类推。 |
| [:odd](https://www.runoob.com/jquery/sel-odd.html)           | $("tr:odd")                   | **:odd 选择器**选取带有奇数索引号的每个元素（比如：1、3、5 等等）。所有奇数 \<tr> 元素，索引值从 0 开始，第一个元素是偶数 (0)，第二个元素是奇数 (1)，以此类推。 |
| [:first-child](https://www.runoob.com/jquery/jq-sel-firstchild.html) | $("p:first-child")            | 属于其父元素的第一个子元素的所有 \<p> 元素                   |
| [:first-of-type](https://www.runoob.com/jquery/sel-firstoftype.html) | $("p:first-of-type")          | 选取属于其父元素的特定类型的第一个子元素的所有元素。属于其父元素的第一个 \<p> 元素的所有 \<p> 元素；该选择器与 :nth-of-type(1) 相同。 |
| [:last-child](https://www.runoob.com/jquery/sel-lastchild.html) | $("p:last-child")             | 属于其父元素的最后一个子元素的所有 \<p> 元素                 |
| [:last-of-type](https://www.runoob.com/jquery/sel-lastoftype.html) | $("p:last-of-type")           | 属于其父元素的最后一个 \<p> 元素的所有 \<p> 元素             |
| [:nth-child(*n*)](https://www.runoob.com/jquery/sel-nthchild.html) | $("p:nth-child(2)")           | 属于其父元素的第二个子元素的所有 \<p> 元素                   |
| [:nth-last-child(*n*)](https://www.runoob.com/jquery/sel-nthlastchild.html) | $("p:nth-last-child(2)")      | 属于其父元素的第二个子元素的所有 \<p> 元素，从最后一个子元素开始计数 |
| [:nth-of-type(*n*)](https://www.runoob.com/jquery/sel-nthoftype.html) | $("p:nth-of-type(2)")         | 属于其父元素的第二个 \<p> 元素的所有 \<p> 元素               |
| [:nth-last-of-type(*n*)](https://www.runoob.com/jquery/sel-nthlastoftype.html) | $("p:nth-last-of-type(2)")    | 属于其父元素的第二个 \<p> 元素的所有 \<p> 元素，从最后一个子元素开始计数 |
| [:only-child](https://www.runoob.com/jquery/sel-onlychild.html) | $("p:only-child")             | 属于其父元素的唯一子元素的所有 \<p> 元素                     |
| [:only-of-type](https://www.runoob.com/jquery/sel-onlyoftype.html) | $("p:only-of-type")           | 属于其父元素的特定类型的唯一子元素的所有 \<p> 元素           |
| [parent > child](https://www.runoob.com/jquery/sel-parent-child.html) | $("div > p")                  | \<div> 元素的直接子元素的所有 \<p> 元素                      |
| [parent descendant](https://www.runoob.com/jquery/sel-parent-descendant.html) | $("div p")                    | \<div> 元素的后代的所有 \<p> 元素                            |
| [element + next](https://www.runoob.com/jquery/sel-previous-next.html) | $("div + p")                  | 每个 \<div> 元素相邻的下一个 \<p> 元素                       |
| [element ~ siblings](https://www.runoob.com/jquery/sel-previous-siblings.html) | $("div ~ p")                  | \<div> 元素同级的所有 \<p> 元素                              |
| [:eq(*index*)](https://www.runoob.com/jquery/sel-eq.html)    | $("ul li:eq(3)")              | 列表中的第四个元素（index 值从 0 开始）                      |
| [:gt(*no*)](https://www.runoob.com/jquery/sel-gt.html)       | $("ul li:gt(3)")              | 列举 index 大于 3 的元素                                     |
| [:lt(*no*)](https://www.runoob.com/jquery/sel-lt.html)       | $("ul li:lt(3)")              | 列举 index 小于 3 的元素                                     |
| [:not(*selector*)](https://www.runoob.com/jquery/jq-sel-not.html) | $("input:not(:empty)")        | 所有不为空的输入元素                                         |
| [:header](https://www.runoob.com/jquery/sel-header.html)     | $(":header")                  | 所有标题元素 \<h1>, \<h2> ...                                |
| [:animated](https://www.runoob.com/jquery/sel-animated.html) | $(":animated")                | 所有动画元素                                                 |
| [:focus](https://www.runoob.com/jquery/jq-sel-focus.html)    | $(":focus")                   | 当前具有焦点的元素                                           |
| [:contains(*text*)](https://www.runoob.com/jquery/sel-contains.html) | $(":contains('Hello')")       | 所有包含文本 "Hello" 的元素                                  |
| [:has(*selector*)](https://www.runoob.com/jquery/sel-has.html) | $("div:has(p)")               | 所有包含有 \<p> 元素在其内的 \<div> 元素                     |
| [:empty](https://www.runoob.com/jquery/jq-sel-empty.html)    | $(":empty")                   | 所有空元素                                                   |
| [:parent](https://www.runoob.com/jquery/sel-parent.html)     | $(":parent")                  | 匹配所有含有子元素或者文本的父元素。                         |
| [:hidden](https://www.runoob.com/jquery/sel-hidden.html)     | $("p:hidden")                 | 所有隐藏的 \<p> 元素                                         |
| [:visible](https://www.runoob.com/jquery/sel-visible.html)   | $("table:visible")            | 所有可见的表格                                               |
| [:root](https://www.runoob.com/jquery/jq-sel-root.html)      | $(":root")                    | 文档的根元素                                                 |
| [:lang(*language*)](https://www.runoob.com/jquery/jq-sel-lang.html) | $("p:lang(de)")               | 所有 lang 属性值为 "de" 的 \<p> 元素                         |
| [[*attribute*\]](https://www.runoob.com/jquery/jq-sel-attribute.html) | $("[href]")                   | 所有带有 href 属性的元素                                     |
| [[*attribute*=*value*\]](https://www.runoob.com/jquery/sel-attribute-equal-value.html) | $("[href='default.htm']")     | 所有带有 href 属性且值等于 "default.htm" 的元素              |
| [[*attribute*!=*value*\]](https://www.runoob.com/jquery/sel-attribute-notequal-value.html) | $("[href!='default.htm']")    | 所有带有 href 属性且值不等于 "default.htm" 的元素            |
| [[*attribute*$=*value*\]](https://www.runoob.com/jquery/sel-attribute-end-value.html) | $("[href$='.jpg']")           | 所有带有 href 属性且值以 ".jpg" 结尾的元素                   |
| [[*attribute*\|=*value*\]](https://www.runoob.com/jquery/sel-attribute-prefix-value.html) | $("[title\|='Tomorrow']")     | 所有带有 title 属性且值等于 'Tomorrow' 或者以 'Tomorrow' 后跟连接符作为开头的字符串 |
| [[*attribute*^=*value*\]](https://www.runoob.com/jquery/sel-attribute-beginning-value.html) | $("[title^='Tom']")           | 所有带有 title 属性且值以 "Tom" 开头的元素                   |
| [[*attribute*~=*value*\]](https://www.runoob.com/jquery/sel-attribute-contains-value.html) | $("[title~='hello']")         | 所有带有 title 属性且值包含单词 "hello" 的元素               |
| [[*attribute**=*value*\]](https://www.runoob.com/jquery/sel-attribute-contains-string-value.html) | $("[title*='hello']")         | 所有带有 title 属性且值包含字符串 "hello" 的元素             |
| [[*name*=*value*\][*name2*=*value2*]](https://www.runoob.com/jquery/sel-multipleattribute-equal-value.html) | $( "input[id][name$='man']" ) | 带有 id 属性，并且 name 属性以 man 结尾的输入框              |
| [:input](https://www.runoob.com/jquery/sel-input.html)       | $(":input")                   | 所有 input 元素                                              |
| [:text](https://www.runoob.com/jquery/sel-input-text.html)   | $(":text")                    | 所有带有 type="text" 的 input 元素                           |
| [:password](https://www.runoob.com/jquery/sel-input-password.html) | $(":password")                | 所有带有 type="password" 的 input 元素                       |
| [:radio](https://www.runoob.com/jquery/sel-input-radio.html) | $(":radio")                   | 所有带有 type="radio" 的 input 元素                          |
| [:checkbox](https://www.runoob.com/jquery/sel-input-checkbox.html) | $(":checkbox")                | 所有带有 type="checkbox" 的 input 元素                       |
| [:submit](https://www.runoob.com/jquery/sel-input-submit.html) | $(":submit")                  | 所有带有 type="submit" 的 input 元素                         |
| [:reset](https://www.runoob.com/jquery/sel-input-reset.html) | $(":reset")                   | 所有带有 type="reset" 的 input 元素                          |
| [:button](https://www.runoob.com/jquery/sel-input-button.html) | $(":button")                  | 所有带有 type="button" 的 input 元素                         |
| [:image](https://www.runoob.com/jquery/sel-input-image.html) | $(":image")                   | 所有带有 type="image" 的 input 元素                          |
| [:file](https://www.runoob.com/jquery/sel-input-file.html)   | $(":file")                    | 所有带有 type="file" 的 input 元素                           |
| [:enabled](https://www.runoob.com/jquery/sel-input-enabled.html) | $(":enabled")                 | 所有启用的元素                                               |
| [:disabled](https://www.runoob.com/jquery/sel-input-disabled.html) | $(":disabled")                | 所有禁用的元素                                               |
| [:selected](https://www.runoob.com/jquery/sel-input-selected.html) | $(":selected")                | 所有选定的下拉列表元素                                       |
| [:checked](https://www.runoob.com/jquery/sel-input-checked.html) | $(":checked")                 | 所有选中的复选框选项                                         |
| .selector                                                    | $(selector).selector          | 在jQuery 1.7中已经不被赞成使用。返回传给jQuery()的原始选择器 |
| [:target](https://www.runoob.com/jquery/jq-sel-target.html)  | $( "p:target" )               | 选择器将选中ID和URI中一个格式化的标识符相匹配的<p>元素       |

# 八、插件

菜鸟教程\<https://www.runoob.com/jquery/jquery-plugin-validate.html>



参照：菜鸟教程
