# 一、基础

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

### 双大括号表达式 

1)  语法: {{exp}} 

2)  功能: 向页面输出数据 

3)  可以调用对象的方法 

### 指令一**:** 强制数据绑定 

1)  功能: 指定变化的属性值 

2)  完整写法: v-bind:xxx='yyy' //yyy 会作为表达式解析执行 

3)  简洁写法: :xxx='yyy' 

### 指令二**:** 绑定事件监听 

1)  功能: 绑定指定事件名的回调函数 ，(xxx表示回调函数的名字)

2)  完整写法: 

v-on:keyup='xxx'	 v-on:keyup='xxx(参数)' 	v-on:keyup.enter='xxx' 

3)  简洁写法: @keyup='xxx' 		@keyup.enter='xxx' 

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

## 3.计算属性和监视

### 1.计算属性

1) 在 computed 属性对象中定义计算属性的方法 

2) 在页面中使用{{方法名}}来显示计算的结果

### 2.监视属性 

1)  通过通过 vm 对象的$watch()或 watch 配置来监视指定的属性 

2)  当属性变化时, 回调函数自动调用, 在函数内部进行计算 

### 3.计算属性高级 

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

































参照：尚硅谷