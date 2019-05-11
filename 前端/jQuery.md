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





















