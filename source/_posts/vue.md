---
title: " Vue-Learn"
tags:
  - vue
comments: false
categories:
  - front
date: 2018-10-19 10:18:32
---
[官网教程](https://cn.vuejs.org/v2/guide/)

---
## 基础
挂载成功后通过$el来获取挂载的元素，

```javascript
var app = new Vue({
    el:'#app',
    data:{}
})
```
数据为双向绑定，即绑定的双方数据任何一方数据发生变化，另外一边会发生同样的变化
`{% raw %}{{ }} {% endraw %}` 为最基本的文本插值方式，可以使用元运算，做简单的运算
如：`{% raw %}{{ '123,123'.split(',') }} {% endraw %}`，`{% raw %}{{ isSuccess?'成功':'失败' }} {% endraw %}`

```
vue只支持单个表达式，不支持语句和流控制，另外，在表达式中，不能使用用户自定义的全局变量，只能使用vue白名单的全局变量，如Math和Date
```
### 过滤器
可以在{% raw %}`{{ }} `{% endraw %}加入管道对数据进行过滤(filters,methods)

```javascript
{{select}}
el:'#element',
data:{},
filters{
    select:function(value){.....}
}
```
过滤器可以串联也可以接收参数
串联 `{% raw %}{{message |filterA|filterB}}{% endraw %}`
接收参数 `{% raw %}{{message |filterA('arg1','arg2')}} {% endraw %}`

### 事件绑定
使用`v-on:绑定的事件`，如`v-on:click`，可以简写为语法糖`@click`

```html
<div id="app">
    <p v-if="show">这是一段文字</p>
    <button @click="toggle">点击隐藏</button>
</div>
```

```javascript
new Vue({
    el:'#app',
    data:{
        show:true
    },
    methods:{
        toggle:function () {
            this.show = !this.show;
        }
        //methods中的方法可以互相调用，使用this.方法名称
    }
})
```
### 计算属性
需要对{% raw %}{{ }} {% endraw %}中的数据进行复杂处理的可以使用计算属性(computed)

```html
<p>{{split1}}</p>
```

```javascript
new Vue({
    el:'element',
    data:{},
    computed:{
        split1:function(){....}
    }
})
```
两个vue示例的计算实例可以互相调用，使用实例名称.计算属性
#### 计算属性缓存

    计算属性是基于它的依赖缓存的，一个计算属性所依赖的数据发生变化时，它才会重新取值，所以只要text不改变，计算属性就不更新
### 其他
vue在渲染元素的时候，处于效率考虑，会尽可能的复用已有的元素而非从新渲染。如果要重新渲染，加上属性`key=""`

## 生命周期
- create : 实例创建完成后调用，此阶段完成了数据的观测，但尚未挂载，$el还不可用，一般用于初始化数据
- mounted : el挂载到实例上后调用，一般业务逻辑从这开始
- beforeDestroy : 实例销毁之前调用，主要解绑使用addEventListerner监听的事件
## 内置语法

```
v-html 插入html代码，而不是当做文本处理
v-pre 如果想要显示{{}}使用这个标签跳过编译
v-if 
v-bind 语法糖 :class 绑定元素的属性
    例：<div :class="{'active':isActive}"></div>
    当isActive为true时,这个div拥有类名active，反之没有，也可以
    一般绑定一个计算属性或者写在data中，也可以绑定一个Object类型的数据，或者methods
    需要多个class的时候可以使用数组语法，还可以使用三元表达式切换
    例：<div :class"[arg1,arg2]"></div>,data中写arg1和arg2的值
    一般也应该使用计算属性，methods，等
    :style 使用方法和:class类似，CSS属性名称使用驼峰命名或者短横分隔命名
    使用:style时，vue会自动给特殊的CSS属性名称增加前缀，如transform
v-cloak 不需要表达式，会在结束编译时从绑定的html上移除，常和CSS的display:none使用，用得较少
v-once 不需要表达式，定义它的元素或组件只渲染一次，包括元素或组件的所有子节点，首次渲染后，不再随数据的变化重新渲染，一般很少用到
v-if,v-if-else,v-else,根据表达式的值在DOM中渲染或销毁元素/组件
    v-if-else要紧跟v-if,v-else要紧跟v-else-if或v-if,表达式的值为真时，当前元素/组件及所有子节点将被渲染，为假时被移除
    例：
    <p v-if="status==1">这是第一行</p>
    <p v-else-if="status==2">这是第二行</p>
    <p v-else="">这是第三行</p>
    var app = new Vue({
    el:'#app',
    data:{
        status:2,
    }})//这里会显示第二行
    v-if可以配合<template></template>使用
v-show 用法与v-if基本一致，v-show是改变CSS属性display,v-show值为false时，元素会隐藏，v-show不能在template上使用
v-for 需要循环显示的时候用到v-for，需结合in/of使用
    例：<temple v-for="item in list">{{item}}</temple>
        还可以带一个参数表示元素位置
        <template v-for="(item,index) in list">{{index}}-{{item}}</template>
    对象的属性也是可以遍历的，有两个可选参数key,value,如：(value,key,index) in user
    也可以用来迭代整数，item in 10
    修改数组时，渲染的视图也会更新（双向绑定）
        数组的一些操作方法：
            push()
            pop()
            shift()
            unshift()
            splice()
            sort()
            reverse()
        以上方法会改变原数组,以下方法不会
            filter()
            concat()
            slice()
    以下变动的数组中，vue是不能检测到的，也不会更新视图
    - 通过索引直接设置项，如app.books[3]={..}
    - 修改数组长度，如app.books.length=1
    第一个可以使用set方法，Vue.set(app.books,3,{....}),在webpack里写为this.$set(app.books,3,{...}),另一个方法，app.books.splice(3,1,{...})，第二个问题也可以用splice
    如果不想改变原数组，如过滤和排序，可以使用计算属性来返回过滤或排序后的数组
    例：<template v-for="item in filterSort"></template>//这里的filterSort表示计算属性
v-on 语法糖 @click 绑定元素的事件
    例：<a href="..." @click="login()">
    click后的方法可以不写()，vue会按照默认的JavaScript处理，vue还提供了一个特殊的变量$event，用于访问原生的DOM事件
    例：<a href="..." @click="login('登录成功',$event)">可以在方法中调用事件如：preventDefult()
    可以使用修饰符来指定事件,vue支持一下修饰符
    - .stop
    - .prevent
    - .capture
    - .self
    - .once
    如：<a href="..." @click.stop="">
    如果是键盘事件时，vue支持按键修饰符，和一些快捷名称如.enter .tab可以组合使用
```

### 其他
- v-if和v-show的选择
  具有相似的功能，但是v-if才是真的条件渲染，它会根据表达式适当的销毁或重建元素及绑定的事件或子组件，若表达式初始值为false,则元素一开始不会渲染，只有当条件第一次变为真的时候才会开始编译
  而v-show只是简单的CSS属性切换，无论条件是否是真都会被编译，相比之下，v-if适合条件经常不变的场景，v-show适合频繁切换的条件


  ​      
