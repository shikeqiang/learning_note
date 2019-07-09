# 一、Vue基础语法

## 1.扩展插件及MVVM

1)  vue-cli: vue 脚手架 

2)  vue-resource(axios): ajax 请求 

3)  vue-router: 路由 

4)  vuex: 状态管理 

5)  vue-lazyload: 图片懒加载 

6)  vue-scroller: 页面滑动相关 

7)  mint-ui: 基于 vue 的 UI 组件库(移动端) 

8)  element-ui: 基于 vue 的 UI 组件库(PC 端) 

### mvvm

![image-20190614183759665](/Users/jack/Desktop/md/images/image-20190614183759665.png)

VM：主要是DOM监听和数据绑定实现。

Model：数据对象data，就是业务逻辑相关的数据对象，通常从数据库映射而来。

View：模板页面，就是展现出来的用户界面。

ViewModel：视图模型(即Vue实例)，就是与界面(view)对应的Model。**因为数据库结构往往是不能直接跟界面控件一一对应上的，所以，需要再定义一个数据对象专门对应view上的控件。**而ViewModel的职责就是把model对象封装成可以显示和接受输入的界面数据对象。

　　至于viewmodel的数据随着view自动刷新，并且同步到model里去，这部分代码可以写成公用的框架，不用程序员自己操心了。

　　简单的说，ViewModel就是View与Model的连接器，View与Model通过ViewModel实现双向绑定。

![img](/Users/jack/Desktop/md/images/1050920-20161207111940601-1823413171.png)

参照：[VUE的MVVM框架解析](https://www.cnblogs.com/carr-css/p/6140450.html)

```html
<body>
<!--
  1. 引入Vue.js
  2. 创建Vue对象
    el : 指定根element(选择器)
    data : 初始化数据(页面可以访问)
  3. 双向数据绑定 : v-model
  4. 显示数据 : {{xxx}}
-->
<!--模板,iview-->
<div id="test">
  <input type="text" v-model="msg"><br> <!--指令-->
  <input type="text" v-model="msg"> <!--指令-->
  <p>{{msg}}</p>  <!--大括号表达式-->
</div>

<!--引入vue-->
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
 //创建vue实例,const声明一个只读的常量。一旦声明，常量的值就不能改变。vm是视图模型
  const vm = new Vue({ // 配置对象 options
    // 配置选项(option)
    el: '#test',  // element: 指定用vue来管理页面中的哪个标签区域,这里表示管理id为test的标签区域
    data: {   // 数据(model)
      msg: 'hello vue'
    }
  })
</script>
</body>
```

## 2.模板语法

### 模板的理解 

1)  动态的 html 页面 

2)  包含了一些 JS 语法代码 

​	a. 双大括号表达式 

​	b. 指令(以 v-开头的自定义标签属性) 

指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

**“” 双引号表示的是变量，'' 单引号表示的是属性，在data里面，属性都要用单引号**

> ```html
> <p v-if="seen">现在你看到我了</p>
> ```
>
> 这里，`v-if` 指令将根据表达式 `seen` 的值的真假来**插入/移除 `<p>` 元素。**
>
> [参数](https://cn.vuejs.org/v2/guide/syntax.html#%E5%8F%82%E6%95%B0)
>
> 一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML 特性：
>
> ```HTML
> <a v-bind:href="url">...</a>
> ```
>
> 在这里 `href` 是参数，告知 `v-bind` 指令将该元素的 `href` 特性与表达式 `url` 的值绑定。
>
> 另一个例子是 `v-on` 指令，它用于监听 DOM 事件：
>
> ```HTML
> <a v-on:click="doSomething">...</a>
> ```

### 双大括号表达式 

1)  语法: {{exp}} 

2)  功能: 向页面输出数据 

3)  可以调用对象的方法 

### 指令一**:** 强制数据绑定 

1)  功能: 指定变化的属性值 

2)  完整写法: 	v-bind:xxx='yyy'    //yyy 会作为表达式解析执行 

3)  简洁写法: 	:xxx='yyy' 

### 指令二**:** 绑定事件监听 

1)  功能: 绑定指定事件名的回调函数 ，(xxx表示回调函数的名字)

2)  完整写法: 

v-on:keyup='xxx'	 v-on:keyup='xxx(参数)' 	v-on:keyup.enter='xxx' 

3)  简洁写法: 	@keyup='xxx' 		@keyup.enter='xxx' 

```html
<div id="app">
  <h2>1. 双大括号表达式</h2>
  <p>{{content}}</p>
  <p>{{content.toUpperCase()}}</p>

  <h2>2. 指令一: 强制数据绑定</h2>
  <a href="url">访问指定站点</a><br>  <!--无法访问,在HTML页面无法读取url的内容-->
  <a v-bind:href="url">访问指定站点2</a><br>
  <a :href="url">访问指定站点2</a><br>

  <h2>3. 指令二: 绑定事件监听</h2>
  <button v-on:click="test">点我</button>
  <button @click="test2(content)">点我</button>
  <button @click="test2('你好')">点我</button>
</div>


<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#app',
   // “” 双引号表示的是变量，'' 单引号表示的是属性，在data里面，属性都要用单引号
    data: { // data 的所有属性都会称为 vm 对象的属性, 而模板页面中可以直接访问
      content: 'NBA I Love This Game',
      url: 'https://www.baidu.com/'
    },
    //事件回调函数，括号中也可以带参数
    methods: {
      test () {
        alert('好啊!!!')
      },
      test2 (abc) {
        alert(abc)
      }
    }
  })
</script>
```

## 3.基础demo

### 1.计算属性和监视

#### 计算属性

1) 在 computed 属性对象中定义计算属性的方法 

2) 在页面中使用{{方法名}}来显示计算的结果

> 注意：computed属性对象中定义的方法返回的还是一个属性，所以可以直接用{{方法名}}绑定数据和显示。

#### 监视属性 

1)  通过通过 vm 对象的 $watch()或 watch 配置来监视指定的属性 

2)  当属性变化时, 回调函数自动调用, 在函数内部进行计算 

#### 计算属性高级 

1)  通过 getter/setter 实现对属性数据的显示和监视 

2)  计算属性存在缓存, 多次读取只执行一次 getter 计算 

```html
<div id="demo">
    姓: <input type="text" placeholder="First Name" v-model="firstName"><br>
    名: <input type="text" placeholder="Last Name" v-model="lastName"><br>
    <!--fullName1是根据fistName和lastName计算(拼接)产生-->
    姓名1(单向): <input type="text" placeholder="Full Name1" v-model="fullName1"><br>
    姓名2(单向): <input type="text" placeholder="Full Name2" v-model="fullName2"><br>
    姓名3(双向): <input type="text" placeholder="Full Name3" v-model="fullName3"><br>
    <p>{{fullName1}}</p>
    <p>{{fullName1}}</p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    const vm = new Vue({
        el: '#demo',
        data: {
            firstName: 'A',
            lastName: 'B',
            fullName2: 'A-B'
        },
        // 计算属性配置: 值为对象，即拼接属性值
        computed: {
            fullName1() { // 属性的get()
                console.log('fullName1()', this)
                return this.firstName + '-' + this.lastName   // 拼接A-B
            },
            fullName3: {
    // 回调函数：1.自定义的，2.没有调用，3.最终执行了
    // 当获取当前属性值时自动调用, 将返回值(根据相关的其它属性数据)作为属性值
                get() {
                    console.log('fullName3 get()')
                    return this.firstName + '-' + this.lastName
                },
   // 当属性值发生了改变时自动调用, 监视当前属性值变化, 同步更新相关的其它属性值
                set(value) { // fullName3的最新value值  A-B23
                    console.log('fullName3 set()', value)
                    // 更新firstName和lastName
                    const names = value.split('-')
                    this.firstName = names[0]
                    this.lastName = names[1]
                }
            }
        },
        watch: {  // 当监听的值发生变化时会执行
            // 监视firstName
            firstName: function (value) { // 相当于属性的set
                console.log('watch firstName', value)
                // 更新fullName2
                this.fullName2 = value + '-' + this.lastName
            }
        }
    })
    // 监视lastName属性
    vm.$watch('lastName', function (value) {
        console.log('$watch lastName', value)
        // 更新fullName2
        this.fullName2 = this.firstName + '-' + value
    })
</script>
```

### 2.**class**与**style**绑定

#### **class**绑定 

1)  :class='xxx' 

2)  表达式是字符串: 'classA' 

3)  表达式是对象: {classA:isA, classB: isB} 

4)  表达式是数组: ['classA', 'classB'] 

> 绑定的class属性不一定需要在模板里，也可以在Vue实例的data中，此外，还可以绑定计算属性
>
> ```HTML
> <div v-bind:class="classObject"></div>
> // js
> data: {
>   isActive: true,
>   error: null
> },
> computed: {
>   classObject: function () {
>     return {
>       active: this.isActive && !this.error,
>       'text-danger': this.error && this.error.type === 'fatal'
>     }
>   }
> }
> ```
>
> 

#### **style**绑定 

1)  :style="{ color: activeColor, fontSize: fontSize + 'px' }" 

2)  其中 activeColor/fontSize 是 data 属性 

```html
<head>
    <meta charset="UTF-8">
    <title>04_class与style绑定</title>
    <style>
        .classA {
            color: red;
        }
        .classB {
            background: blue;
        }
        .classC {
            font-size: 30px;
        }
    </style>
</head>
<body>
<!--
1. 理解
  在应用界面中, 某个(些)元素的样式是变化的，class/style绑定就是专门用来实现动态样式效果的技术
2. class绑定: :class='xxx'，注意要用单引号
  xxx是字符串
  xxx是对象,根据对象class的值判断class生效,即key是类名，value是布尔类型
  xxx是数组,数组中写class样式类名
3. style绑定,style是key-value形式
  :style="{ color: activeColor, fontSize: fontSize + 'px' }"
  其中activeColor/fontSize是data属性
-->
<div id="demo">
    <h2>1. class绑定: :class='xxx'</h2>
    <p :class="myClass">xxx是字符串</p>
    <!--绑定两个class样式，根据对象的属性布尔值判断样式生效-->
    <p :class="{classA: hasClassA, classB: hasClassB}">xxx是对象</p>
    <p :class="['classA', 'classB']">xxx是数组</p>
    <h2>2. style绑定</h2>
    <p :style="{color:activeColor, fontSize}">:style="{ color: activeColor, fontSize: fontSize + 'px' }"</p>
    <button @click="update">更新</button>     <!--调用更新方法-->
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#demo',
        data: {
            myClass: 'classA classC',   /*绑定两个class样式*/
            hasClassA: true,
            hasClassB: false,
            activeColor: 'red',
            fontSize: '40px'
        },
        methods: {
            update() {
                this.myClass = 'classB'     // 将myClass指定为classB
                this.hasClassA = !this.hasClassA
                this.hasClassB = !this.hasClassB
                this.activeColor = 'green'
                this.fontSize = '30px'
            }
        }
    })
</script>
```

### 3.条件渲染

1)  v-if 与 v-else 

2)  v-show 

> 如果需要频繁切换 v-show 较好 
>
> 当条件不成立时, v-if 的所有子节点不会解析(项目中使用) 
>
> 带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS 属性 `display`。
>
> ==注意，`v-show` 不支持 `<template>` 元素，也不支持 `v-else`。==

```html
<!--1. 条件渲染指令
  v-if，v-else,只写属性名表示属性为true，如：默认显示“表白失败”		v-show
2. 比较v-if与v-show,如果需要频繁切换 v-show 较好,v-if隐藏时会移除对象,即这里会移除p标签-->
<div id="demo">
  <p v-if="res">表白成功</p>
  <p v-else>表白失败</p>
  <hr>
  <p v-show="res">求婚成功</p>
  <p v-show="!res">求婚失败</p>
  <button @click="res=!res">切换</button>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      res: false
    }
  })
</script>
```

> ​	==一般来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。==

### 4.列表渲染

#### 1.列表显示指令 

数组: v-for / index 

对象: v-for / key 

> 注意遍历的时候，数组的索引或对象的key都是在后面，如：v-for="(p, index) in persons或v-for="(value, key) in persons[1]"

#### 2.列表的更新显示 

删除/替换 item 

```html
<div id="demo">
    <h2>测试: v-for 遍历数组</h2>
    <ul>
        <!--遍历persons，p是对象名字，index是索引,:key数据绑定，会作为表达式解析执行 -->
        <li v-for="(p, index) in persons" :key="index">
            {{index}}--{{p.name}}--{{p.age}}
            - -
            <button @click="deleteP(index)">删除</button>
            - -
            <button @click="updateP(index, {name:'更新的数据', age: 16})">更新</button>
        </li>
    </ul>
    <button @click="addP({name: 'baichen', age: 18})">添加</button>
    <h2>测试: v-for 遍历对象</h2>
    <ul>
        <!--遍历列表persons中的第二个元素,指定它的key为属性名,要保证每个key的值都不同-->
        <li v-for="(value, key) in persons[1]" :key="key">{{key}}={{value}}</li>
    </ul>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#demo',
        data: {
            persons: [
// 定义一个数组列表，数组元素是对象类型,vue本身只简史persons的改变，没有监视数组内部数据的改变
// Vue重写了数组中的一系列改变数组内部数据的方法(先调用原生，更新界面)，数据内部改变，界面自动变化,即数据绑定
                {name: 'Tom', age: 18},
                {name: 'Jack', age: 17},
                {name: 'Bob', age: 19},
                {name: 'Mary', age: 16}
            ]
        },
        methods: {    // 自定义一些方法
            deleteP(index) {   // 删除persons中指定index的p,删除1个元素
                this.persons.splice(index, 1) // 参照Vue文档，发现调用了不是原生数组的splice(), 而是一个变异(重写)方法
                // 1. 调用原生的数组的对应方法
                // 2. 更新界面
            },
            updateP(index, newP) {
                console.log('updateP', index, newP)
// this.persons[index] = newP ,如果这样写的话没有改变persons本身，只是改变persons指向的对象，
// 数据内部发生了变化，但并没有调用变异方法，vue根本就不知道，所以不会更新界面
                this.persons.splice(index, 1, newP)
                // this.persons = []    ,有改变persons本身，并进行数据绑定
            },
            addP(newP) {
                this.persons.push(newP)
            }
        }
    })
</script>
```

#### 3.列表的高级处理 

​	列表过滤和列表排序 

```html
<div id="demo">
    <!--v-model 双向绑定数据-->
    <input type="text" v-model="searchName">
    <ul>
        <!--filterPersons是过滤后的数组，是filterPersons()方法返回后的数组-->
        <li v-for="(p, index) in filterPersons" :key="index">
            {{index}}--{{p.name}}--{{p.age}}
        </li>
    </ul>
    <div>
        <button @click="setOrderType(2)">年龄升序</button>
        <button @click="setOrderType(1)">年龄降序</button>
        <button @click="setOrderType(0)">原本顺序</button>
    </div>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#demo',
        data: {
            searchName: '',
            orderType: 0, // 0代表不排序, 1代表降序, 2代表升序
            persons: [
                {name: 'Tom', age: 18},
                {name: 'Jack', age: 17},
                {name: 'Bob', age: 19},
                {name: 'Mary', age: 16}
            ]
        },
        computed: {  // 计算属性,filterPersons通过计算searchName和persons得出返回的数组
            filterPersons() {
                // 取出相关数据,定义三个对象
                const {searchName, persons, orderType} = this
                let arr = [...persons]      // 要返回的数组
                // 对persons数组进行过滤
                if (searchName.trim()) {
                    // 判断p.name中是否包含searchName
                    arr = persons.filter(p => p.name.indexOf(searchName) !== -1)
                }
                // 按年龄进行排序
                if (orderType) {
                    arr.sort(function (p1, p2) { //如果返回负数，p1在前，返回整数，p2在前
                        if (orderType === 1) { // 降序
                            return p2.age - p1.age
                        } else { // 升序
                            return p1.age - p2.age
                        }

                    })
                }
                return arr
            }
        },
        methods: {
            setOrderType(orderType) {   // 绑定页面用户点击的按钮
                this.orderType = orderType
            }
        }
    })
</script>
```

### 5.事件处理

#### 绑定监听:

v-on:xxx="fun"
  @xxx="fun"
  @xxx="fun(参数)"
  默认事件形参: event，方法内 *`this`* 指向 *vm*， `event`是原生 *DOM* 事件
  隐含属性对象: $event

==在内联语句处理器中访问原始的 DOM 事件。可以用特殊变量 `$event`==

#### 事件修饰符:

> ==修饰符是由点开头的指令后缀来表示的。==

.prevent : 阻止事件的默认行为，即event.preventDefault(),比如：点击链接之后，不会跳转，而是做出其他定义的行为
  .stop : 停止事件冒泡，即event.stopPropagation()，即一个div内，点击外层按钮后，不会继续响应里层按钮，如下面的test6

> ```HTML
> <!-- 阻止单击事件继续传播 -->
> <a v-on:click.stop="doThis"></a>
> 
> <!-- 提交事件不再重载页面 -->
> <form v-on:submit.prevent="onSubmit"></form>
> 
> <!-- 修饰符可以串联 -->
> <a v-on:click.stop.prevent="doThat"></a>
> 
> <!-- 只有修饰符 -->
> <form v-on:submit.prevent></form>
> 
> <!-- 添加事件监听器时使用事件捕获模式 -->
> <!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
> <div v-on:click.capture="doThis">...</div>
> 
> <!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
> <!-- 即事件不是从内部元素触发的 -->
> <div v-on:click.self="doThat">...</div>
> ```
>
> 注意：
>
> 使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止**所有的点击**，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。

#### 按键修饰符

 .keycode : 操作的是某个keycode值的健，即键盘上按键对应的数值
 .enter : 操作的是enter键

> 在监听键盘事件时，我们经常需要检查详细的按键。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符：
>
> ```HTML
> <!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
> <input v-on:keyup.enter="submit">
> ```
>
> 你可以直接将 [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) 暴露的任意有效按键名转换为 kebab-case 来作为修饰符。
>
> ```HTML
> <input v-on:keyup.page-down="onPageDown">
> ```
>
> 在上述示例中，处理函数只会在 `$event.key` 等于 `PageDown` 时被调用。

```html
<div id="example">
    <h2>1. 绑定监听</h2>
    <button @click="test1()">test1</button>
    <button @click="test2('abc')">test2</button>
    <!--$event代表事件对象，传入参数和事件对象，获取当前属性值test3-->
    <button @click="test3('abcd', $event)">test3</button>
    <h2>2. 事件修饰符</h2>
    <a href="http://www.baidu.com" @click.prevent="test4">百度一下</a>
    <div style="width: 200px;height: 200px;background: red" @click="test5">
        <div style="width: 100px;height: 100px;background: blue" @click.stop="test6"></div>
    </div>
    <h2>3. 按键修饰符</h2>
    <input type="text" @keyup.13="test7">
    <input type="text" @keyup.enter="test7">
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#example',
        data: {},
        methods: {
            test1(event,msg) {  // 不传参数时，调用test1可以直接用 @click="test1"
                alert(event.target.innerHTML)
            },
            test2(msg) {
                alert(msg)
            },
            test3(msg, event) { // 传入参数和得到当前属性值test3
                alert(msg + '---' + event.target.textContent)
            },
            test4() {
                alert('点击了链接')
            },
            test5() {
                alert('out')
            },
            test6() {
                alert('inner')
            },
            test7(event) {
                console.log(event.keyCode)
                alert(event.target.value)
            }
        }
    })
</script>
```

### 6.表单输入绑定

​	使用v-model(双向数据绑定)自动收集数据

```html
<div id="demo">
    <!--调用handleSubmit方法，.prevent : 阻止事件的默认行为，这里表示不提交表单-->
    <form action="/xxx" @submit.prevent="handleSubmit">
        <span>用户名: </span>
        <input type="text" v-model="username"><br>
        <span>密码: </span>
        <input type="password" v-model="pwd"><br>
        <span>性别: </span>
        <input type="radio" id="female" value="女" v-model="sex">
        <label for="female">女</label>
        <input type="radio" id="male" value="男" v-model="sex">
        <label for="male">男</label><br>

        <span>爱好: </span>
        <input type="checkbox" id="basket" value="basket" v-model="likes">
        <label for="basket">篮球</label>
        <input type="checkbox" id="foot" value="foot" v-model="likes">
        <label for="foot">足球</label>
        <input type="checkbox" id="pingpang" value="pingpang" v-model="likes">
        <label for="pingpang">乒乓</label><br>

        <span>城市: </span>
        <select v-model="cityId">
            <option value="">未选择</option>
            <!--遍历选项,:value会转变成表达式-->
            <option :value="city.id" v-for="(city, index) in allCitys" :key="city.id">{{city.name}}</option>
        </select><br>
        <span>介绍: </span>
        <textarea rows="10" v-model="info"></textarea><br><br>
        <input type="submit" value="注册">
    </form>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#demo',
        data: { // 这里是对表单数据进行赋值
            username: '',
            pwd: '',
            sex: '男',   // 默认勾选为 男
            likes: ['foot'],    // 默认选中 足球，foot是value对应的值
            allCitys: [{id: 1, name: 'BJ'}, {id: 2, name: 'SS'}, {id: 3, name: 'SZ'}],
            cityId: '2',    // 绑定城市ID，默认为第二个城市
            info: ''
        },
        methods: {
            handleSubmit() {
                console.log(this.username, this.pwd, this.sex, this.likes, this.cityId, this.info)
                alert('提交注册的ajax请求')
            }
        }
    })
</script>
```

### 7.Vue实例的生命周期

![lifecycle](/Users/jack/Desktop/md/images/lifecycle.png)

#### vue对象的生命周期

 1). 初始化显示

beforeCreate()

created()

beforeMount()

==mounted()==	挂载

2). 更新状态:this.xxx=value

beforeUpdate()

updated()
3). 销毁vue实例: vm.$destroy()

==beforeDestory()==

destroyed()

#### 常用的生命周期方法

created()/mounted(): 	发送ajax请求, 启动定时器等**异步任务**
 beforeDestroy(): 	做收尾工作, 如: 清除定时器

```html
<div id="test">
    <button @click="destroyVue">destroy vm</button>
    <p v-if="isShow">间隔显示/隐藏文本</p>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#test',
        data: {
            isShow: true
        },
        beforeCreate() {
            console.log('beforeCreate()')
        },
        created() {
            console.log('created()')
        },
        beforeMount() {
            console.log('beforeMount()')
        },
        mounted() {     // 初始化显示之后立即调用,调用一次
            console.log('mounted()')
            // 执行异步任务
            this.intervalId = setInterval(() => {
                console.log('-----')
                this.isShow = !this.isShow
            }, 1000)    // 指定变化时间为一秒钟
        },
        beforeUpdate() {
            console.log('beforeUpdate()')
        },
        updated () {
            console.log('updated()')
        },
        destroyed() {
            console.log('destroyed()')
        },
        beforeDestroy() {   // 实例死亡之前调用，只调用一次
            console.log('beforeDestroy()')
            // 执行收尾的工作,清除定时器
            clearInterval(this.intervalId)
        },
        methods: {
            destroyVue() {
                this.$destroy()
            }
        }
    })
</script>
```

> 不要在选项属性或回调上使用[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`，`this`会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。

### 8.过渡和动画效果(渐变和移动)

#### vue动画的理解

操作css的transition或animation
==vue会给目标元素添加/移除特定的class==

#### 基本过渡动画的编码

1). 在目标元素外包裹\<transition name="xxx">
2). 定义class样式
	1>. 指定过渡样式: transition
	2>. 指定隐藏时的样式: opacity/其它

#### 过渡的类名

 xxx-enter-active: 指定显示的transition，即过渡的效果，有move，fade等等，或者自定义，样式中定义过渡效果，并且要与transition中的name的值一致，如：\<transition name="xxx">
 xxx-leave-active: 指定隐藏的transition
 xxx-enter: 指定隐藏时的样式

![transition](/Users/jack/Desktop/md/images/transition.png)

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>10_过渡&动画1</title>
    <style>
        /*指定过渡样式*/
        .xxx-enter-active, .xxx-leave-active {
            transition: opacity 1s
        }
        /*指定隐藏时的样式*/
        .xxx-enter, .xxx-leave-to {
            opacity: 0;
        }
        /*显示的过渡效果*/
        .move-enter-active {
            transition: all 1s
        }
        /*隐藏的过滤效果*/
        .move-leave-active {
            transition: all 3s
        }
        /*指定隐藏时的样式*/
        .move-enter, .move-leave-to {
            opacity: 0;
            transform: translateX(20px)     /*在水平方向上移动*/
        }
    </style>
</head>
<body>
<div id="demo">
    <button @click="isShow = !isShow">Toggle</button>
    <transition name="xxx">
        <p v-show="isShow">hello</p>
    </transition>
</div>
<hr>
<div id="demo2">
    <button @click="isShow = !isShow">Toggle2</button>
    <transition name="move">
        <p v-show="isShow">hello</p>
    </transition>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    // 两个Vue实例，因为el标签不同
    new Vue({
        el: '#demo,#demo2',
        data: {
            isShow: true
        }
    })
    new Vue({
        el: '#demo2',
        data: {
            isShow: true
        }
    })
</script>
</body>
</html>
```

放大后隐藏

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>10_过渡&动画2</title>
    <style>
        .bounce-enter-active {      /*进入的动画*/
            animation: bounce-in .5s;
        }
        .bounce-leave-active {      /*离开的动画，加了reverse，与进入的时候相反*/
            animation: bounce-in .5s reverse;
        }
        @keyframes bounce-in {	
            0% {
                transform: scale(0);
            }
            50% {
                transform: scale(1.5);
            }
            100% {
                transform: scale(1);
            }
        }
    </style>
</head>
<body>
<div id="example-2">
    <button @click="show = !show">Toggle show</button>
    <br>
    <transition name="bounce">
        <p v-if="show" style="display: inline-block;background: red">放大/缩小</p>
    </transition>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script>
    new Vue({
        el: '#example-2',
        data: {
            show: true
        }
    })
</script>
</body>
</html>
```

### 9.过滤器

  功能: 对要显示的数据进行特定格式化后再显示
  注意: 并没有改变原本的数据, 可是产生新的对应的数据

#### 编码

1). 定义过滤器

```html
Vue.filter(filterName, function(value[,arg1,arg2,...]){
  // 进行一定的数据处理
  return newValue
})
```

  2). 使用过滤器

```html
<div>{{myData | filterName}}</div>
<div>{{myData | filterName(arg)}}</div>
```

对当前时间进行指定格式显示:

```html
<div id="test">
    <h2>显示格式化的日期时间</h2>
    <p>{{time}}</p>
    <p>最完整的: {{time | dateString}}</p>
    <p>年月日: {{time | dateString('YYYY-MM-DD')}}</p>
    <p>时分秒: {{time | dateString('HH:mm:ss')}}</p>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript" src="https://cdn.bootcss.com/moment.js/2.22.1/moment.js"></script>
<script>
    // 自定义过滤器，函数对象，指定过滤器名字dateString，将要被格式化的参数value
    Vue.filter('dateString', function (value, format) {
        // moment JavaScript 日期处理类库,如果有格式参数，就按照新格式显示，如果没有就按照原来的格式显示
        return moment(value).format(format||'YYYY-MM-DD HH:mm:ss');
    })
    new Vue({
        el: '#test',
        data: {
            time: new Date()    // Date对象time,time可以随意起名
        },
        mounted() {     // 实时按秒更新时间
            setInterval(() => {
                this.time = new Date()
            }, 1000)
        }
    })
</script>
```

### 10.指令

​	指令是为了操作HTML标签的。

#### 常用内置指令

- v:text : 更新元素的 textContent

- v-html : 更新元素的 innerHTML

- v-if : 如果为true, 当前标签才会输出到页面

- v-else: 如果为false, 当前标签才会输出到页面

- v-show : **通过控制display样式来控制显示/隐藏**

- v-for : 遍历数组/对象

- v-on : 绑定事件监听, 一般简写为@，如：@click=hint  ,hint为方法名

- v-bind : 强制绑定解析表达式, 可以省略v-bind，即 :key=key 

- v-model : 双向数据绑定

- ref : 为某个元素注册一个唯一标识, vue对象通过$refs属性访问这个元素对象

- **v-cloak : 使用它防止闪现表达式,即页面加载时还没读取到数据，而先显示了表达式, 与css配合: [v-cloak] { display: none }**

  > [v-cloak] 是属性选择器，找到v-cloak这个属性名所属的标签p

```html
<style>
        [v-cloak] {     /*属性选择器，找到v-cloak这个属性名所属的标签p*/
            display: none
        }
    </style>
<div id="example">
    <p v-cloak>{{content}}</p>
    <p v-text="content"></p>   <!--p.textContent = content-->
    <p v-html="content"></p>  <!--p.innerHTML = content-->
    <p ref="msg">在当前页面跳转到百度首页</p>
    <button @click="hint">提示</button>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    new Vue({
        el: '#example',
        data: {
            content: '<a href="http://www.baidu.com">百度一下</a>'
        },
        methods: {
            hint() {
                alert(this.$refs.msg.innerHTML)     // 显示msg的具体内容
            }
        }
    })
</script>
```

#### 自定义指令

##### 注册全局指令

```js
// my-directive是指令名，通过v-my-directive 可以调用指令；
// el: 指令所在的标签对象
// binding: 包含指令相关数据的容器对象
Vue.directive('my-directive', function(el, binding){
el.innerHTML = binding.value.toupperCase()
  })
```

##### 注册局部指令

```js
// my-directive是指令名，通过v-my-directive 可以调用指令
directives : {
'my-directive' : {
    bind (el, binding) {
      el.innerHTML = binding.value.toupperCase()
    	}
	}
  }
```

##### 使用指令

```js
v-my-directive='xxx'		//xxx 是data中的key，即数据
```

```html
<!--
需求: 自定义2个指令
  1. 功能类型于v-text, 但转换为全大写，自定义的指令：v-upper-text，指令名为upper-text
  2. 功能类型于v-text, 但转换为全小写，自定义的指令：v-lower-text，指令名为lower-text
-->
<div id="test">
    <p v-upper-text="msg"></p>
    <p v-lower-text="msg"></p>
</div>
<div id="test2">
    <p v-upper-text="msg"></p>
    <p v-lower-text="msg"></p>      <!--不在局部指令的vm内，无效-->
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
    // 定义全局指令
    // upper-text为指令名
    // el: 指令属性所在的标签对象
    // binding: 包含指令相关数据的容器对象
    Vue.directive('upper-text', function (el, binding) {
        console.log(el, binding)
        // textContent 属性设置或者返回指定节点的文本内容。
        el.textContent = binding.value.toUpperCase()   // 指定文本内容，这里转换为大写
    })
    new Vue({
        el: '#test',
        data: {
            msg: "I Like You"
        },
        directives: {   // 注册局部指令,只在当前vm(Vue实例)管理范围内有效
// 下面这句相当于：'lower-text':function(el, binding) {...}，属性如果有特殊符号的话要用''括起来
            'lower-text'(el, binding) {
                console.log(el, binding)
                el.textContent = binding.value.toLowerCase()
            }
        }
    })
    new Vue({
        el: '#test2',
        data: {
            msg: "I Like You Too"
        }
    })
</script>
```

### 11.自定义插件

```js
// 自定义插件,Vue的插件库,通过匿名函数(function (window) {...}调用
(function (window) {
    const MyPlugin = {}     /*先定义一个对象类型的变量*/
    MyPlugin.install = function (Vue, options) {    // 插件对象必须有一个install方法
        // 1. 添加全局方法或属性
        Vue.myGlobalMethod = function () {
            console.log('Vue函数对象的myGlobalMethod()')
        }

        // 2. 添加全局资源，自定义指令
        Vue.directive('my-directive', function (el, binding) {
            el.textContent = '自定义插件加上绑定属性的值-----' + binding.value
        })

        // 4. 添加实例方法，实例方法要放在原型上面
        Vue.prototype.$myMethod = function () {
            console.log('Vue实例对象的方法$myMethod()')
        }
    }
    window.MyPlugin = MyPlugin
})(window)
```

```html
<div id="test">
    <p v-my-directive="msg"></p>
</div>
<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript" src="vue-myPlugin.js"></script>      <!--引入自定义的插件，要在引入Vue下面引入-->
<script type="text/javascript">
    // 声明使用插件(安装插件: 调用插件的install())
    Vue.use(MyPlugin)   // 内部会调用插件对象的install()
    const vm = new Vue({
        el: '#test',
        data: {
            msg: '绑定的属性'
        }
    })
    Vue.myGlobalMethod()    //  插件定义的全局方法
    vm.$myMethod()          // 插件定义的Vue实例对象方法
    new Object()
</script>
```

# 二、Vue组件化编码

## 1、使用vue-cli 创建模板项目

### **1.使用 **vue-cli创建模板项目 

- 1)  vue-cli 是 vue 官方提供的脚手架工具 
- 2)  github: https://github.com/vuejs/vue-cli 
- 3)  作用: 从 https://github.com/vuejs-templates 下载模板项目 

### 2.创建 **vue** 项目 

- npm install -g vue-cli

- vue init webpack vue_demo cd vue_demo

  > 这里的webpack 是模板的意思，vue-cli官方提供了多个模板

- npm install

- npm run dev

- 访问: http://localhost:8080/ 

### 3.模板项目的结构

![image-20190620110616919](/Users/jack/Desktop/md/images/image-20190620110616919.png)

## 2、项目的打包和发布

### 打包：npm run build 

### 发布 **1:** 使用静态服务器工具包 

npm install -g serve
serve dist
 访问: http://localhost:5000 

### 发布 **2:** 使用动态 **web** 服务器**(tomcat)** 

修改配置: webpack.prod.conf.js 

​	output: { 

​		publicPath: '/xxx/' //打包文件夹的名称 

​		} 

重新打包:
 npm run build 

修改 dist 文件夹为项目名称: xxx
 将 xxx 拷贝到运行的 tomcat 的 webapps 目录下 ，访问: http://localhost:8080/xxx 

## 3、**eslint**

### **3.1.** 说明、提供的支持及校验

#### 说明：

1)  ESLint 是一个代码规范检查工具 

2)  它定义了很多特定的规则, 一旦你的代码违背了某一规则, eslint 会作出非常有用的提示 

3)  官网: http://eslint.org/ 

4)  基本已替代以前的 JSLint 

#### 提供以下支持：

1)  ES 

2)  JSX 

3)  style 检查 

4)  自定义错误和提示 

#### 提供的校验：

1)  语法错误校验 

2)  不重要或丢失的标点符号，如分号 

3)  没法运行到的代码块(使用过 WebStorm 的童鞋应该了解) 

4)  未被使用的参数提醒 

5)  确保样式的统一规则，如 sass 或者 less 

6)  检查变量的命名 

### 3.2 规则的错误等级有三种 

1)  0:关闭规则。 

2)  1:打开规则，并且作为一个警告(信息打印黄色字体) 

3)  2:打开规则，并且作为一个错误(信息打印红色字体) 

### 3.3 相关配置文件

1)  .eslintrc.js : 全局规则配置文件 

```
'rules': { 
	'no-new': 1 
} 
```

2)  在 js/vue 文件中修改局部规则 

```
/* eslint-disable no-new */ 
new Vue({ 
	el: 'body', 
	components: { App } 
}) 
```

3)  .eslintignore: 指令检查忽略的文件 

​	*.js 	*.vue 

## 4、组件定义与使用

​	组件是一个局部的功能模块，包括JS、HTML和CSS等，组件系统是 Vue 的另一个重要概念，**因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。仔细想想，几乎任意类型的应用界面都可以抽象为一个组件树：**

![Component Tree](/Users/jack/Desktop/md/images/components.png)

在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。在 Vue 中注册组件很简单：

```js
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是个待办项</li>'
})
```

现在你可以用它构建另一个组件模板：

```html
<ol>
  <!-- 创建一个 todo-item 组件的实例 -->
  <todo-item></todo-item>
</ol>
```

​	但是这样会为每个待办项渲染同样的文本，**应该能从父作用域将数据传到子组件才对。**修改一下组件的定义，使之能够接受一个 [prop](https://cn.vuejs.org/v2/guide/components.html#%E9%80%9A%E8%BF%87-Prop-%E5%90%91%E5%AD%90%E7%BB%84%E4%BB%B6%E4%BC%A0%E9%80%92%E6%95%B0%E6%8D%AE)：

```js
Vue.component('todo-item', {
  // todo-item 组件现在接受一个"prop"，类似于一个自定义特性,这个 prop 名为 todo。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```

接下来使用 `v-bind` 指令将待办项传到循环输出的每个组件中：

```html
<div id="app-7">
  <ol>
<!--现在为每个todo-item提供todo对象,todo对象是变量，即其内容可以是动态的。也需要为每个组件提供一个“key”。-->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})
```

> 输出：
>
> 1. 蔬菜
> 2. 奶酪
> 3. 随便其它什么人吃的东西

​	上面代码设法将应用分割成了两个更小的单元。**子单元通过 prop 接口与父单元进行了良好的解耦。**现在可以进一步改进 `<todo-item>` 组件，提供更为复杂的模板和逻辑，而不会影响到父单元。

​	在一个大型应用中，有必要将整个应用程序划分为组件，以使开发更易管理。

```vue
<!--组件是一个局部的功能模块，包括JS、HTML和CSS等，下面即HTML+JS+CSS
    这是一个根组件-->
<template>
  <div>
    <div class="row">
      <div class="col-xs-offset-2 col-xs-8">
        <div class="page-header"><h2>Router Test</h2></div>
      </div>
    </div>

    <div class="row">
      <div class="col-xs-2 col-xs-offset-2">
        <div class="list-group">
          <!--生成路由链接-->
          <router-link to="/about" class="list-group-item">About</router-link>
          <router-link to="/home" class="list-group-item">Home</router-link>
        </div>
      </div>
      <div class="col-xs-6">
        <div class="panel">
          <div class="panel-body">
            <!--显示当前组件-->
            <keep-alive>
              <router-view msg="abc"></router-view>
            </keep-alive>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {}		// 向外默认暴露一个配置对象（与Vue一致），里面必须写函数
</script>

<style>
</style>
```































































































































































































参照：尚硅谷

Vue官方文档