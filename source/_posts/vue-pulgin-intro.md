---
title: Vue插件介绍
date: 2017-03-06 09:53:41
tags: [vue]
type: "categories"
categories: 文章
---

## Vue插件

<h3 style="color:red">1、概述</h3>

简单来说，插件就是指对Vue的功能的增强或补充。

比如说，让你在每个单页面的组件里，都可以调用某个方法，或者共享使用某个变量，或者在某个方法之前执行一段代码等

<h3 style="color:red">2、使用方法</h3>

总体流程应该是：

【声明插件】——【写插件】——【注册插件】——【使用插件】

写插件和声明插件是同步的，然后注册到Vue对象中（不用担心重复注册），最后在写Vue组件的时候使用写的插件

#### **声明插件**

先写一个js文件，这个js文件就是插件文件，里面的基本内容如下：

```
/*   说明：
 *   插件文件：service.js
 *   作者：王冬   QQ：20004604
 * */
export default {
    install: function (Vue, options) {
        // 添加的内容写在这个函数里面
    }
};
```

其中install的第一个参数Vue表示的是Vue的实例，第二个参数表示的是一些设置选项。

Vue实例好理解，就是Vue对象。

而options设置选项就是指，在调用这个插件时，可以传一个对象。

例如这个对象有一个属性float，然后在写插件的一个方法/变量时，我需要输出一个数字，然后写一个if判断语句，

假如options.float为true时，输出浮点数；

假如为false或undefined（即没传参）时，输出为整数。

具体怎么添加，之后再说。

#### **注册插件**

如果使用过Vue-router，就很好理解，通过import引入后，然后通过 ``Vue.use(插件名)`` 注册插件；

例如，我们通常在main.js里引入各种东西，并且组件的根实例也在这里

```
//main.js
import Vue from 'vue'
import App from './App.vue'

//关键是这两行
import service from './service.js'
Vue.use(service)

new Vue({
    el: '#app',
    render: (h) => h(App)
})
```

如代码中注释所说，关键是通过import导入service文件，然后在创建根组件之前，让Vue对象通过use方法来注册插件service。

通过这样简单的两步，就可以使用插件了。

<h3 style="color:red">3、写插件、使用插件</h3>

按照官方文档，写插件有四种方法，先给出官方的代码：

```
//以下内容都是添加到上面install的函数里面的

// 1. 添加全局方法或属性
Vue.myGlobalMethod = function () {
    // 逻辑...
}
// 2. 添加全局资源
Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
        // 逻辑...
    }
    ...
})
// 3. 注入组件
Vue.mixin({
    created: function () {
        // 逻辑...
    }
    ...
})
// 4. 添加实例方法
Vue.prototype.$myMethod = function (options) {
    // 逻辑...
}
```

先给出最常用的：【4. 添加实例方法】的写法和使用方法

#### **3.1【添加实例方法或属性】**

1、核心思想：

    通过prototype来添加方法和属性。

2、写：

    //让输出的数字翻倍，如果不是数字或者不能隐式转换为数字，则输出null
    Vue.prototype.doubleNumber = function (val) {
        if (typeof val === 'number') {
            return val * 2;
        } else if (!isNaN(Number(val))) {
            return Number(val) * 2;
        } else {
            return null
        }
    }

3、用：

假设有这样一个组件：

```
<template>
    <div>
        {{num}}
        <button @click="double">点击后让左边的数字翻倍</button>
    </div>
</template>
<script>
    export default{
        data(){
            return {
                num: 1
            }
        },
        methods: {
            double: function () {
                //这里的this.doubleNumber()方法就是上面写的组件里的方法
                this.num = this.doubleNumber(this.num);
            }
        }
    }
</script>
```

我们便可以通过点击button按钮，让num的值，在每次点击都翻倍了。

4、假如添加的是属性：

例如：

    Vue.prototype.number = 1;
    
会发生什么事情呢？

    1、不管是【按值传递类型】还是【按引用传递类型】，该变量都不会被不同组件所共享，更准确的说，假如有A、B两个组件。A组件里的number数值改变，B组件里的number数值是不会跟着改变的。因此不要想着引用这样一个变量，然后修改了A中的值，B里也自动跟着改变了；
    2、当组件里没有该属性时，调用时，显示的是通过插件获取的值；
       当组件里有该属性时，调用时，显示的是组件里该属性的值；
       由此而推，函数也是这样的，组件里的同名函数总是会覆盖插件提供的函数。
       
也就是说，当插件提供一个属性时，组件里没这个属性，就用插件的属性；组件有，就用组件自己的。


#### **3.2【添加全局方法或属性】**

1、核心思想：

就是给Vue对象添加一个属性。

初次接触很容易和上面3.1弄混，实际上，3.1是给组件里使用的，而3.2是给Vue对象使用的。

例如，假如添加一个方法test()，那么：

    通过3.1添加，是在组件里，通过this.test()来调用
    
    通过3.2添加，是在外面，通过Vue实例，如Vue.test()来调用
    
2、写：

```
//放在哪里参考上面
Vue.test = function () {
    alert("123")
}
```

3、用：

```
//注意先导入Vue对象才能使用
Vue.test()
```

使用时会执行对应的方法，比如这里就是alert弹窗

4、其他：

别问我如果和Vue本身属性同名会发生什么事情，我没试过=.=


#### **3.3【注入组件】**

1、核心思想：

就像写Vue组件时，那样写，方法名保持一致，其会在执行组件对应的方法名之前执行。


2、写：

例如：

```
Vue.mixin({
    created: function () {
        console.log("组件开始加载")
    }
})
```

然后这里的代码会在每个组件（包括根组件）的created执行之前执行。

可以自行在每个组件的created方法里写一段console.log来查看测试

可以和【实例属性】配合使用，用于调试或者控制某些功能

```
// 注入组件
Vue.mixin({
    created: function () {
        if (this.NOTICE)
            console.log("组件开始加载")
    }
})
// 添加注入组件时，是否利用console.log来通知的判断条件
Vue.prototype.NOTICE = false;
```

【注入给非Vue实例本身就有的方法】：

假如是写给例如methods属性的某个方法，例如以下注入：

```
Vue.mixin({
    methods: {
        test: function () {
            console.log("mixin test");
        }
    }
})
```

那么，组件里若本身有test方法，并 **不会** 先执行插件的test方法，再执行组件的test方法。

而是只执行其中一个，并且优先执行组件本身的同名方法。这点需要注意




3、用：

不需要手动调用，在执行对应的方法时会被自动调用的（并且先调用插件里的，再调用组件本身的）


4、其他：

    1、如果同时有多个插件注入一个方法（例如created，那么会先执行先注入的那个方法，再依次执行后注入的，最后执行组件本身的）
    2、注意，像methods属性下的方法，并不会在组件注入后每个都执行，而是只执行一个，并且优先执行组件本身的。
    

#### **3.4【添加全局资源】**

1、核心思想：

添加方法和正常添加方法类似，甚至几乎一样。

可以添加【自定义指令】、【过滤器】、【过渡等】，这里以【过滤器】为例

2、写：

例如：

```
//时间格式化过滤器，输入内容是number或者Date对象，输出是YYYY-MM-DD HH-MM-SS
Vue.filter('formatTime', function (value) {
    Date.prototype.Format = function (fmt) { //author: meizz
        var o = {
            "M+": this.getMonth() + 1, //月份
            "d+": this.getDate(), //日
            "h+": this.getHours(), //小时
            "m+": this.getMinutes(), //分
            "s+": this.getSeconds(), //秒
            "q+": Math.floor((this.getMonth() + 3) / 3), //季度
            "S": this.getMilliseconds() //毫秒
        };
        if (/(y+)/.test(fmt))
            fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
        for (var k in o)
            if (new RegExp("(" + k + ")").test(fmt))
                fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
        return fmt;
    }
    return new Date(value).Format("yyyy-MM-dd hh:mm:ss");
})
```

3、用：

和正常使用一样用就行了，so easy。例如：

```
{{num|formatTime}}
```

4、其他：

可以用这个找各种有意思的功能，作为插件写好，然后需要的地方导入就行，超级方便！



<h3 style="color:red">4、示例demo</h3>

附一个有简单功能的示例demo，提供参考使用

```
/*  说明：
 *  插件demo，供学习使用
 *  本页面用于提供各种处理服务
 *  作者：王冬   QQ：20004604
 *  功能有：
 *  1、插件created执行时提示；
 *  2、
 * */
export default {
    install: function (Vue, options) {
        // 1. 添加全局方法或属性
        // 略

        // 2. 添加全局资源
        //  时间格式化过滤器，输入内容是number或者Date对象，输出是YYYY-MM-DD HH-MM-SS
        Vue.filter('formatTime', function (value) {
            Date.prototype.Format = function (fmt) { //author: meizz
                var o = {
                    "M+": this.getMonth() + 1, //月份
                    "d+": this.getDate(), //日
                    "h+": this.getHours(), //小时
                    "m+": this.getMinutes(), //分
                    "s+": this.getSeconds(), //秒
                    "q+": Math.floor((this.getMonth() + 3) / 3), //季度
                    "S": this.getMilliseconds() //毫秒
                };
                if (/(y+)/.test(fmt))
                    fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
                for (var k in o)
                    if (new RegExp("(" + k + ")").test(fmt))
                        fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
                return fmt;
            }
            return new Date(value).Format("yyyy-MM-dd hh:mm:ss");
        })

        // 2. 添加全局资源
        //  添加注入组件时，是否利用console.log来通知的判断条件，也是组件实例属性
        Vue.prototype.NOTICE = true;


        // 3. 注入组件
        //  注入组件，插件加载开始前提示
        Vue.mixin({
            created: function () {
                if (this.NOTICE)
                    console.log("组件开始加载")
            },
            methods: {
                test: function () {
                    console.log("mixin test");
                }
            }
        })


        // 4. 添加实例方法
        //  返回数字是输入数字的两倍，如果不是数字或者不能隐式转换为数字，则输出null
        //  组件实例方法
        Vue.prototype.doubleNumber = function (val) {
            if (typeof val === 'number') {
                return val * 2;
            } else if (!isNaN(Number(val))) {
                return Number(val) * 2;
            } else {
                return null
            }
        }

        // 4. 添加实例方法
        //  服务组，将实例方法整合到$service中，避免命名冲突
        Vue.prototype.$service = {
            //电话号码合法性检查
            telNumberCheck: function (tel) {
                var pattern = /(^(([0\+]\d{2,3}-)?(0\d{2,3})-)(\d{7,8})(-(\d{3,}))?$)|(^0{0,1}1[3|4|5|6|7|8|9][0-9]{9}$)/;
                return pattern.test(tel)
            }
        }
    }
};

```

来自上海前端群(426462717)【`上海-零零水`】投递，他的github：https://github.com/qq20004604 ，我们的[上海前端交流官网](http://shanghaijs.org/)