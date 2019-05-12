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



