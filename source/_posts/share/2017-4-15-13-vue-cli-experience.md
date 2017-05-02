---
title: 【群内分享】- vue-cli 填坑记录(上)-北京-float
date: 2017-04-15 11:12:41
type: "categories"
categories: 文章
---

大家好，我是float，一个前端自学者。
今天和大家分享主要内容是：vue-cli中常用插件的填坑，在分享的过程中，我会提出一个个问题，围绕这些问题来做分析。
在分享之前和大家先聊两个有趣的问题：
* 为什么我和网上那些博客中的代码一摸一样，还是频频报错？
    - 在学习过程中网上的博客，专栏之类可以提供诸多便利，但在看之前一定要抱着一个审视的态度，因为每个人的侧重点，代码风格都有所不同，甚至博客中写的可能是一个“伪代码”。这里我建议从官方github clone案例来学习，官方的文档会随着版本迭代而同步更新，相对于博客之类也更加全面，避免旧版本带来的一些坑。
* 看到vue-cli那么多文件，该如何下手？
    - 从main.js 和 App.vue中的export default xx 到 import xx from "xxx"，一点一点来看，这个过程是在熟悉vue的require机制，这对了解这个项目结构起着至关重要的作用。

切入正题：
聊一聊，vue-router：
* “前后端分离”的概念：
    - 我粗浅理解为：共用一个url，多页面改变的是hash值；后端负责数据层，前端负责view层；在Node／HTTP或其他层面分离；前后端分离具体体现-路由建立的单页面应用
* vue-router 2.x的变化
    - router变更为一个数组对象
* 如何理解vue-router ？
    - 关系网：谁是父组件，谁是子组件，父组件通过路由映射出子组件
    - 嵌套路由：在理解关系网的前提下，在子组件中再次创建路由映射，实现嵌套路由，需要注意的就是路径的书写
    - 重定向：重定向的组件效果是什么样？ 如果重定向了A组件，观察url发现加载后的hash是“#/A”

聊一聊，axios：
* axios，vue 2.0后停止维护vue-resource，推荐使用axios来做ajax请求，axios基于Promise的HTTP请求客户端。
* axios在vue中如何使用？vue的插件可以用vue.use来调用，但是axios并不是基于vue开发的插件，所以只能在vue实例的原型上添加方法，当然也可以在vuex的actions内封装一下。

聊一聊， vuex
* vuex 是一个专为vue.js应用程序开发的状态管理模式，为所有组件提供数据和方法支持
* 一个有趣的例子：“在北京的一个城中村中，村长在用广播宣布一件事：“乡亲们，咱们村要拆迁了！”，村民们兴奋异常，奔走相告，炫耀着自己能分到几套房子。有些村民不相信，向村长再次求证拆迁信息，一户村民家里外出打工的儿子回来后，向父亲询问消息，知道了拆迁的事情。”
* “城中村”是一个父组件（App.vue）；各户村民是子组件；“拆迁信息” 则是vuex.store对象中的state(数据)；“求证” 则是mutations，同步函数；“兴奋异常，奔走相告” 则是actions，可为异步函数；”儿子向父亲询问消息“ 则是在actions中发送一个ajax，”从父亲那知道了拆迁消息“ 则是getters。
* 在开发中，vuex可以为所有组件提供一个特殊的“数据库”，这个数据库里不单单有数据，还有数据交互的方法（同步和异步分开），和获取数据状态的方法。这些数据／方法在vue实例下挂载的组件都可访问，可使用。

上述的这些问题，只是为了更好的理解这些插件做了一个简单的概括，更加详细的分析、demo和源码解析在我在github中有详细的记录：[a summary of vue-cli](https://github.com/Yinlongcoding/about_webpack.github.io)

下面是我的一些填坑记录，顺便一块分享了。

## vue-cli 填坑记录(上)

目录：
1. Hello VueJs
2. 准备工作
3. 热身组件
4. 常用插件

### Hello Vue
在填坑之前先铺垫一点基础知识
* [VueJs](https://cn.vuejs.org/) 是一个渐进式JavaScript 框架
* Vue 与 jQueryJs
    - vue
        + 操作DOM元素对象
        + 以数据为驱动，简化DOM操作，提高DOM复用率
        + 利于后期维护
    - jQuery
        + 语义化明显
        + 易于理解
        + 可扩展插件众多
    - 选择
        + DOM操作频繁，无需复杂动画效果 - Vue (为什么不自己写呢？)
        + DOM操作不频繁，需要复杂动画效果 - jQuery
* Vue 与 [AngularJS](https://angularjs.org/) & [ReactJs](https://facebook.github.io/react/)
    - 优势：
        + 易用： Vue 与其他两者很像，有些语法也相同
        + 轻灵： 相对于其他两者而言，Vue吸取了两者的缺陷，api更简洁，代码更简洁
        + 学习： 相比较其他两者陡峭的学习曲线，vue的学习曲线很平缓
        + 文档： 官方中文文档，你值得拥有（但这不是你不学英语的理由）
        + 插件： vue拥有不输于其他两者的插件资源
    - 劣势：
        + 语法： 需熟悉ES6语法
        + 高阶： vue 不缺入门教程，但是高阶教程和文档欠缺
        + 风格： 实现一个需求，vue可以有多种写法，但是在团队协作时，就显得有些难受了
        + 兼容： 不支持IE8及以下版本
        + 调试： 在components 内debugger委实难受
        + 其他： 待补充

### 准备工作
需要用到NodeJs来做一些准备工作
npm 安装相应的依赖库：
    Node.js && npm ？ continue ：[download Node.js](https://nodejs.org/en/)
```
    npm install webpack vue-cli -g
```
* [npm](https://www.npmjs.com/) 是什么？
    - npm，是JavaScript的包管理器和世界上最大的软件注册表。提供可复用代码的软件包，并以强大的新方式组合它们
* [cnpm](http://npm.taobao.org/) 是什么？
    - cnpm 是一个完整`npmjs`镜像，用此代替官方版本(只读)
* 两者有何区别？
    - 同：为项目所依赖的模块提供的一个统一的管理方案
    - 异：
        + npm 下载依赖模块时，速度偏慢(可翻墙提升下载速度)，对模块的依赖加载很完善
        + cnmp 看起来和npm 一样，下载速度很快，但是在下载模块时可能存在模块依赖加载不完善的问题，在开发中存在致命的隐患
        + 推荐使用官方化的 npm 下载依赖，避免项目出现不必要的问题

### 构建环境
在这之前需要创建一个文件夹用以存放你项目:  [注] * 为特殊说明点
```js
  vue init webpack [name]   *

? Generate project in current directory? Yes
  This will install Vue 2.x version of the template.
  For Vue 1.x use: vue init webpack#1.0

? Project name dictionary               // 项目名称
? Project description A Vue.js project  // 项目介绍
? Author Sliver <wnhoper@gmail.com>     // 作者
? Vue build standalone                  // 打包方式
? Install vue-router? No                // 路由
? Use ESLint to lint your code? No      // ESlint 书写标准  *
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? No  // 以下两项为测试依赖
? Setup e2e tests with Nightwatch? No

   vue-cli · Generated "app".

   To get started:  *

     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack

    npm install
    npm run dev
```
* vue init webpack [name] name为可选项，可在根目录下创建以name为命名的项目文件夹
* vue-router  这里我选择关闭，初次尝试还是单纯点好，多了反而不利于理解
* [ESLint](http://eslint.org/docs/rules/)
  - 代码书写标准，利于团队协作，bug排查，代码风格比较严谨
  - 对于tab,space,indent之类的有很细致的要求
  - 可以在`.eslintconfig->rules` 选项中添加/修改
* npm install  vue-cli 所依赖的模块还是比较多的，时间会久一点
* npm run dev  自动打开浏览器，  http://localhost:8080
  - 端口占用： config 文件夹下```index.js -> port```可以修改端口

![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/cli.png?raw=true)
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/floder.png?raw=true)
#### 在目录中需要注意一下几点
* App.vue 为所有组件`注册`和`使用`的入口文件，其自身也是一个组件
* main.js 为所有的js入口文件
* 所有的组件必须先注册，才能使用，否则在终端中会报错（提示信息完善）

### 热身组件
在项目开始前，先写一个打招呼的组件热热身吧，熟悉一下玩法 `[注] *为特殊说明`

`src` 目录下 `components` 下创建一个`header.vue` & `login.vue`

header.vue
```html
<template>
  <header class="header">
    <h1 class="logo">{{ nameLogo }}</h1>
    <ul>
      <li v-for="nav in navs">{{ nav.li }}</li>
    </ul>
  </header>
</template>

<script>
  export default {         *
    name:'headiv',         *
    props: ['nameLogo'],   *
    data () {              *
      return {
        navs: [
          {name: '主页'},
          {name: '单词'},
          {name: '音标'},
          {name: '例句'},
          {name: '发音'}
        ]
      }
    }
  }
</script>

<style>                   *
  .header {
    width: 100%;
    display: flex;
    justify-content: space-between;
  }
  .header h1 {
    font-size: 1.5em;
    color: #eac;
  }
  .header ul {
    list-style: none;
    display: flex;
  }
  .header ul li {
    padding-left: 1em;
  }
</style>
```

login.vue
```html
<template>
    <div class="login">
        <input v-model="userName" @change="setName">    *
    </div>
</template>

<script>
  export default {
    name: 'login',
    data () {
      return {
        userName: ''
      }
    },
    methods: {
      setName: function () {                             *
        this.$emit('changeUser', this.userName)
      }
    }
  }
</script>
```

App.vue
```html
<template>
  <div id="app">
    <headiv :nameLogo="user"></headiv>
    <login @changeUser="getUser"></login>
    <p>用户名：{{ user }}</p>
  </div>
</template>

<script>
import headiv from './components/header'     *
import login from './components/login.vue'

export default {
  name: 'app',
  data () {
    return {
      user: ''
    }
  },
  components: {
    headiv,
    login
  },
  methods: {
    getUser(msg) {
      this.user = msg                        *
    }
  }
}
</script>
```


* export default ES6语法，用于导出变量，函数，module ...
* name 可选，定义模块的标签名，[注]避免使用HTML原生标签名以及一些保留词
* [props](https://cn.vuejs.org/v2/guide/components.html#Prop)父组件向子组件传值属性
* data  vue组件中的data是一个`函数`，所有的值皆为返回值
* v-model  双向绑定，input触发change事件时，将新的user通过双向绑定传给$data.user，随之p标签内容发生改变
* \$emit 触发事件 详情参考官方文档[事件触发](https://cn.vuejs.org/v2/api/#vm-emit)&[自定义事件](https://cn.vuejs.org/v2/guide/components.html#camelCase-vs-kebab-case)



### 常用插件

#### UI依赖
比较了很多 Ui 组件，最终选择[element.ui](http://element.eleme.io/#/zh-CN)   原因：提升效率，对vue^2.x比较友好

```
npm i element-ui -S
```

引入：main.js
```js
import ElementUl from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
```
初步学习时，为防止一些不必要的麻烦，选择全部加载，待熟练之后可选择按需加载

#### vue-router
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/router.gif?raw=true)
### [vue-router](https://router.vuejs.org/zh-cn)
* 遇到过这么一个概念，“前后端分离”，我粗浅的理解：
  - 共用一个url，改变的是 hash值
  - 后端负责数据层，前端负责view层，在HTTP／Node／其他层面分离
  - 后端提供接口，前端调用接口
* 传统的a 标签`href`跳转显然并不太适用于vue且vue的组件是相对独立的
* vue-router 则像是一条纽带，将父子组件链接起来，形成因果(这词很玄乎)
* 安装: `npm install vue-router -D`
* 练习vue-router 最直观的莫过写一个导航栏
  - 在`src`下的`components`下建立`Home.vue` & `News.vue`
  - 在`src`下建立`router`文件夹下有`router.js`
  - 关系网：
    + App.vue 下的`router-view`是其下 Home &  News 的路由出口
    + Home.vue 是其下主页的路由，同时也是其下`Login`&`Reg`的路由出口
    + News.vue 是其下新闻的路由，同时也是`new`的路由出口
* 一个经典的嵌套路由

router.js
```js
import Home from '../components/Home'
import News from '../components/News'
import Login from '../components/children/Login'
import Reg from '../components/children/Reg'
import New from '../components/children/New.vue'

const routes = [              *
  {
    path: '/',
    redirect: '/home'
  },{
    path: '/home',
    component: Home,          *
    children: [               *
      {
        path: '/home/login',
        component: Login
      },{
        path: '/home/reg',
        component: Reg
      }
    ]
  },{
    path: '/news',
    component: News,
    children: [
      {
        path: '/news/new',
        component: New
      }
    ]
  }
]

export default routes
```
* routes 在 vue-router 2.0变更为一个数组对象
* 嵌套路由，一如开始分析的关系网一样
* '／' 表示：路径指向根路径
* redirect：`componentName` 重定向，一开始的指针指向指定组件
* component, 路由到的子组件
* children，子路由，弄清楚关系网，就可以在理论上实现无限的嵌套路由

main.js
```js
import Vue from 'vue'
import App from './App'
import VueRouter from 'vue-router'    *
import routes from './router/router'  *

Vue.use(VueRouter)

const router = new VueRouter({
  routes                              *
})

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  render: h=>h(App)
})

```
* 引入vue-router，并创建实例router对象
* router
  - mode: 路由模式，默认是更改hash 值，会带一个`#`,也可选择HTML5 History模式，需有服务端支持
  - routes：路由配置器

App.vue
```html
  <template>
  <div id="app">
    <div>
      <router-link to="/home">首页</router-link>
      <router-link to="/news">新闻</router-link>
      <div class="main">
        <router-view></router-view>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'app'
}
</script>
<style> </style>
```

Home.vue
```
<template>
  <header class="header">
    <router-link to="/home/login">登陆</router-link>
    <router-link to="/home/reg">注册</router-link>
    <div class="content">
      <router-view></router-view>
    </div>
  </header>
</template>
<script>
  export default {
    name: 'home'
  }
</script>
```

News.vue
```
<template>
  <div class="news">
    <h2>{{ msg }}</h2>
    <router-link to="/news/new">点击查看</router-link>
    <div>
      <router-view></router-view>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'news',
    data () {
      return {
        msg: '奇葩新闻报道'
      }
    }
  }
</script>
```

Login.vue
```
<template>
  <div class="login">
    <h3>{{ msg }}</h3>
    <label>
      <span>账号：</span><input type="text"><br>
      <span>密码：</span><input type="text">
    </label>
    <div class="buts">
      <button>登陆</button>
      <button>忘记密码</button>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'login',
    data () {
      return {
        msg: '登陆界面'
      }
    }
  }
</script>
```

其他的子组件就暂时省略了，详细代码可以在[这里](https://github.com/Yinlongcoding/about_webpack.github.io/tree/master/test-code/v-router/src)查看

#### axios
题记:
* vue.1.x版本官方推荐 Ajax请求插件是`vue-resource`
* 2.x推荐  [axios](https://www.npmjs.com/package/axios)
* axios是一个轻量级的Ajax请求插件，内部封装了ES6 Promise对象，更加直观的异步请求
* 综合 `vue-router` & `axios` 做一个小练习
* 安装:`npm i axios`
* `src-components-children`下添加`search.vue`

![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/axios.gif?raw=true)


main.js
```
import Axios from 'axios'
Vue.prototype.$http = axios
```
* axios 不同于 vue-resource 不能用 `use`方法
* 将axios添加在vue实例的原型上，在任意组件内部都可使用

search.vue
```
<template>
    <div class="search">
        <input type="text" @change="search" v-model="word">
        <button @click="search">search</button>
        <div v-for="word in results">
            <p>{{ word.headword }}</p>
            <div v-for="sense in word.senses">
                <p v-for="def in sense.defs">{{ def.defCn }}</p>
            </div>
        </div>
    </div>
</template>
<script>
    export default {
        name: 'search',
        data () {
            return {
                word: '',
                results: ''
            }
        },
        methods: {
            search () {
                let self = this
                let word = this.word.toLowerCase()
                this.$http.get('http://damiao.io:5000/word/'+word)
                     .then(function(res) {
                        self.results = res.data
                     })
                     .catch(function(error){
                        console.log(error)
                     })
            }
        }
    }
</script>
```
* this.$http.get(url) 请求JSON数据
* axios 封装了promess，调用方法
    +  .then(callback) 异步请求
    +  .catch(callback) catch 抛出错误信息
    +  关于self，在内部也是可以使用箭头函数，防止this指向混乱，强制将指针指向vue实例
* 更多的API还是去axios的官网查询

#### Vuex
* WHAT [vuex](https://vuex.vuejs.org/zh-cn/)？
    - 思想继承自[React](https://facebook.github.io/react/)-[Redux](http://redux.js.org/)
    - 前面提到过`vue-router`为组件提供了因果，但这只是父与子组件之间的
    - Vuex可以为平行组件之间的数据交互提供一条便捷的道路
* WHY Vuex？
  - 广泛性：状态管理器，所有的数据和方法全部放在vuex的对象里，所有组件皆可调用
  - 代码解耦：HTML和JavaScript低耦合，提升性能
  - 面向对象：储存数据，封装方法，提供接口，全局调用
* HOW Vuex?
  - 数据层次过深，可在Store内做预处理
  - mutations 封装大部分数据交互的方法，自给自足的提供接口
  - actions 异步提交mutations，交互变的轻而易举
  - getters 获取数据派生状态摆脱暴力 `v-for * n`
  - 使用接口，将需要的数据展现在view层中
* 多说无益，手下见真章.

从一个简单的例子开始吧！(本例来源: Vuex文档 [Counters](https://vuex.vuejs.org/zh-cn/getting-started.html))
下载：`npm install vuex`，接下来，一张关系图，描述vuex这个对象。
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/vuex.png?raw=true)
* 多个视图依赖于同一状态
* 来自不同视图的行为需要变更同一状态”
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/counter.gif?raw=true)

`src`目录下`vuex`文件夹下`store.js`
store.js

```js
import Vue form 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const state = {
    count: 0
}

const mutations = {
    increment (state) {
     state.count++
    },
    decrement (state) {
     state.count--
    }
}

const actions = {
    increment: ({commit}) => commit('increment'),
    decrement: ({commit}) => commit('decrement'),
    incrementIfOdd ({commit, state}) {
        if((state.count + 1) % 2 === 0) {
            commit('increment')
        }
    },
    incrementIfAsync ({commit}) {
        if((state.count + 1) % 2) {
            commit('increment')
        }
    }
}

const getter = {
    evenOrOdd: state => state.count % 2 === 0 ? 'even' : 'odd'
}

export default new Vuex.Store({
    state,
    getters,
    actions,
    mutations
})

```

* 代码中不难看出，Store就是那颗状态管理树，它包含了
    - 数据库 [state]
        + 所有的数据以及数据状态都讲存放在这里，所有的组件都可访问到数据
    - 变更方案 [mutations]
        + mutaitons 和事件极其类似，但是它`必须是同步函数`
        + [vue‘s devtools](https://github.com/vuejs/vue-devtools) 在mutations 触发时都会记录变化前后的状态，存在回调函数的时候，devtools并不知道回调函数是否执行，这给纪录状态带来不必要的麻烦
    - 执行方案 [actions]
        + 所有的异步执行事件全部放在actions里，就避免devtools懵逼的情况
        + 在actions里，需要显式的去`commit`一次mutaitions
    - 获取状态 [getters]
        + 数据可以保存，当然也是可以获取的
        + getters 可以在所有的组件提供获取数据及数据派生状态的服务

main.js
```js
import Vuex from 'vuex'
import store from './vuex/store'

new Vue({
    el: '#app',
    store,
    render: h=>h(App)
})
```

App.vue
```html
<template>
  <div id="app">
    <el-row type="flex" justify="center">
      <Count></Count>
    </el-row>
  </div>
</template>

<script>
import Count from './components/Count.vue'

export default {
    name: 'app',
    components: {
      Count
    }
}
</script>
```

Count.vue
```html
<template>
    <el-col>
        <p>Click: {{ $store.state.count }}</p>
        <p>Count is {{ evenOrOdd }}</p>
        <el-button type="primary" @click="increment">Increment</el-button>
        <el-button type="primary" @click="decrement">Decrement</el-button>
        <el-button type="primary" @click="incrementIfOdd">Odd</el-button>
        <el-button type="primary" @click="incrementIfAsync">Async</el-button>
    </el-col>
</template>
<script>
import {mapGetters, mapActions} from 'vuex'


export default {
    name: 'Count',
    computed: mapGetters([
        'evenOrOdd'
    ]),
    methods: mapActions([
        'increment',
        'decrement',
        'incrementIfOdd',
        'incrementIfAsync'
    ])
}
</script>
```
* UI方面由element.ui 支持
* 关于`\$`，不同于jQuery，这个`\$`是可以获取到挂载在vue实例上的对象如`\$data \$store`
* mapGetters, mapActions, mapSate 为对应对象的辅助函数

映射关系：
```js
computed: mapGetters([
    'evenOrOdd'
])
// the same as
computed: {
    eventOrOdd () {
        return this.\$store.getters.evenOrOdd
    }
}
```
接来下，制作一个todoMvc 文档案例
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/todo.gif?raw=true)
分析：
* 需要复用的模块只有中间的 todo-lists，这是唯一的一个模块
* 在整个事件流中，基本不存在异步事件，所以可以很愉快的使用mutations
* 分析整个todomvc的事件流，对vuex 进一步了解

store.js
```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const state = {
    todos: []
}

const mutations = {
    addTodo (state, {text}) {
        state.todos.push({
            text: text,
            done: false
        })
    },
    deleteTodo (state, {todo}) {
        state.todos.splice(state.todos.indexOf(todo), 1)
    },
    toggleTodo (state, {todo}) {
        todo.done = !todo.done
    },
    editTodo (state, {todo, value}) {
        todo.text = value
    },
    toggleAll (state, { done }) {
        if(state.todos.some( todo => todo.done === false)) {
            state.todos.forEach((todo=> todo.done = true))
        } else {
            state.todos.forEach((todo=> todo.done = false))
        }
    },
    clearCompleted (state) {
        state.todos = state.todos.filter(todo => !todo.done)
    }
}

export default new Vuex.Store({
    state,
    mutations
})
```
* state 如上面的例子一样，数组对象
* 关于传值，JavaScript 接受的参数类型具有多元化
* ...[[Object,string],[Object,array],[Object,function],[Object, object],...]
* mutations 下的commitFn接受state为第一个参数，({object}) 这种参数为一个对象
* 例如：{text} -> {text:'输入内容', done: boolean } 代表输入内容和完成状态
* {todo, value} -> {todo: target.todoList, value} 当前更改的 list 对象，以及更改内容

App.vue
```html
<template>
<div id="app">
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
        <input type="checkbox"
        id="toggle-all"
        v-show="todos.length"
        @change="toggleAll">
        <label for="toggle-all"></label>
      <input class="new-todo"
        autofocus
        autocomplete="off"
        placeholder="What needs to be done?"
        @keyup.enter="addTodo">
    </header>

    <section class="main" v-show="todos.length">
      <ul class="todo-list">
        <todoList v-for="(todo,index) in filteredTodos"
          :todo="todo" :key="index"></todoList>                   *
      </ul>
    </section>

    <section class="footer">
      <span class="todo-count">
        <strong>{{ remaining }}</strong>
        {{ remaining | pluralize('item') }} left
      </span>
      <ul class="filters">
        <li v-for="(val, key) in filters">
          <a href="#"
            :class="{selected: visibility === key}"
            @click="visibility = key">{{ key | capitalize }}</a>
        </li>
      </ul>
      <button class="clear-completed"
        v-show="todos.length > remaining"
        @click="clearCompleted">Clear completed</button>
    </section>
  </section>
</div>
</template>

<script>
import todoList from './components/todoList.vue'
import { mapMutations } from 'vuex'

const filters = {                                           *
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.done),
  completed: todos => todos.filter(todo => todo.done)
}

export default {
    name: 'app',
    components: {
      todoList
    },
    data () {
      return {
        visibility: 'all',                                 *
        filters: filters
      }
    },
    computed: {                                            *
      todos () {
        return this.$store.state.todos
      },
      filteredTodos () {
        return filters[this.visibility](this.todos)        *
      },
      remaining () {
        return this.todos.filter((todo) => !todo.done ).length
      }
    },
    methods: {
      addTodo (el) {
        let text = el.target.value
        if(text.trim()) {
          this.$store.commit('addTodo', { text })
        }
          el.target.value = ''
      },
    ...mapMutations([                                     *
      'toggleAll',
      'clearCompleted'
    ])
    },
    filters: {
      pluralize: (number, word) => number <= 1 ? word : (word + 's'),
      capitalize: key => key.charAt(0).toUpperCase() + key.slice(1)
    }
}
</script>
```
* 疑问：为什么数据交互不用actions ？
  - 代码，不难发现，改变的只有数据的状态，在整个事件中不存在异步
* 为什么不用getters 来获取数据状态？
  - 这只是一个选择问题，当然可以用，但是在这里有更好的选择-> filter([vue过滤器](http://es6.ruanyifeng.com/#docs/array))在语义化与性能方面一样很出色
* filters对象是在对数据状态做一个预处理，映射给visibility
* computed {} 前面说过，vuex可以在为所有组件提供数据库，但是却需要组件自己去接收
* `return filters[visibility](todo)`  这看起来很复杂，细致分析一下
  - fiters对象的keys和 visibility的value 是一种映射关系
  - 意思：告诉`filters`以当前`visibility`的值来对数据状态做预处理
* ...mapMuations [ES6 数组解构](http://es6.ruanyifeng.com/#docs/array)
* `:key="index"` v-for默认使用`就地复用策略`，添加／删除组件时vue时如果做到组件顺序和状态不会混乱的？当然这是比较深层的问题了，为每个v-for产生的组件提供一个`唯一的key,以便它能跟踪每个节点的身份，从而重用和重新排序现有元素`，详情参见官方文档 [key](https://cn.vuejs.org/v2/guide/list.html#key)

todoList.vue
```html
<template>
  <li class="todo" :class="{ completed: todo.done, editing: editing }">
    <div class="view">
      <input type="checkbox"
        class="toggle"
        :checked="todo.done"
        @change="toggleTodo({ todo: todo })">
      <label v-text="todo.text"
        @dblclick="editing = true"></label>
      <button class="destroy"
        @click="deleteTodo({ todo: todo })"></button>  *
    </div>
    <input class="edit"
      v-show="editing"
      v-focus="editing"
      :value="todo.text"
      @keyup.enter="doneEdit"
      @keyup.esc="canceEdit"
      @blur="doneEdit">
  </li>
</template>

<script>
import { mapMutations } from 'vuex'

  export default {
    name: 'todoList',
    props: ['todo'],
    data () {
      return {
        editing: false
      }
    },
    directives: {                                       *
      focus (el, { value }, { context }) {
        if(value) {
          context.$nextTick(()=>{
            el.focus()
          })
        }
      }
    },
    methods: {
      ...mapMutations([
        'editTodo',
        'toggleTodo',
        'deleteTodo'
      ]),
      doneEdit (el) {
        let value = el.target.value.trim()
        let { todo } = this
        if(!value) {
          this.deleteTodo({todo})
        } else if (this.editing) {
          this.editTodo({todo, value})
          this.editing = false
        }
      },
      canceEdit (el) {
        el.target.value = this.todo.text
        this.editing = false
      }
    }
  }
</script>
```
* 前言中，父子组件之间的因果就是[prop对象](https://cn.vuejs.org/v2/guide/components.html#Prop)，父组件将值传递给子组件就是依靠prop来传递
* `{(todo: todo)}` 又是一种头疼的传参方式，在这之前回到store.muataions看看这个参数所属的方案需要的形参是什么
  - `deleteTodo (state, {todo})` 这下就很明确了
  - 把当前子组件的数据状态显式传递{todo}这个对象
* [directives](https://cn.vuejs.org/v2/api/#Vue-directive) vue的自定义指令
* [$nextTick](https://cn.vuejs.org/v2/api/#Vue-nextTick)`在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM`

### 问题来袭：HTTP请求获取的数据怎么保存进state里？
![img](https://github.com/Yinlongcoding/about_webpack.github.io/blob/master/images/vuexHttp.gif?raw=true)

store.js
```js
import Vue from 'vue'
import Vuex from 'vuex'
import axios from 'axios'

Vue.use(Vuex)

const state = {
  defCn: ''
}

const mutations = {
  getWord: function (state, data) {
    state.defCn = data[0].senses[0].defs
  }
}

const actions = {
  search ({commit}, text) {
    let word = text.target.value
    axios.get('http://damiao.io:5000/word/'+word)
      .then(function(res) {
        let data = res.data
        commit('getWord', data)
      })
  }
}

export default new Vuex.Store({
  state,
  mutations,
  actions,
})
```
* ajax 为异步请求，那必然是写在actions
* 通过传参的方式，将response 传入 mutations, 对获取到的数据进行预处理，存放在state内

App.vue
```html
<template>
  <div id="app">
    <el-row type="flex" justify="center">
      <el-col :span="15">
        <el-input icon="search"
          placeholder="请输入单词"
          @keyup.enter.native="search"></el-input>
      </el-col>
    </el-row>

    <el-row type="flex" justify="center">
      <el-col :span="15">
        <el-collapse>
          <el-collapse-item v-for="(word, index) in defCn"
          :title="word.defCn"
          :key="index">
          <div v-for="(example, index) in word.examples" :key="index">
            <p>{{ example.en }}</p>
            <p>{{ example.cn }}</p>
          </div>
          </el-collapse-item>
        </el-collapse>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'

export default {
  name:'App',
  computed: mapState([
      'defCn'
  ]),
  methods: mapActions([
    'search'
    ])
}
</script>
```


来自北京前端交流群(578874693)【(北京-float)[https://github.com/Yinlongcoding/about_webpack.github.io]】投递，我们的[北京前端交流官网](http://beijingjs.org/)