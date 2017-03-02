---
title: webpack传统后端渲染的项目前端配置
date: 2017-03-01 21:53:41
tags: [面试,基础,javascript]
type: "categories"
categories: 文章
---

对于比较传统的后端渲染项目,前端一直处于比较尴尬的位置,很多时候前端就是写一些样式一些动效.然后直接把写好的html文件扔给后端(jsp之类的木板)这就让很多后端工程师工作繁重了不少.但是由于很多后端的js不是那么的熟练.导致了两个问题:1. 事件绑定直接在dom上进行.后期改动代码变得很复杂. 2. 很多后端逻辑直接写到了模板里. 导致前后端耦合极深,对于后端的重构也不是什么好事.

现在前端的趋势是大前端,然而大部分前端还处于几年前的状态,仍然是后端主导.但是既然拿钱干活了我们就要把事情做好.也为了自己的话语权(方便自己嘛).啰嗦了这么多,下面就是正文部分.

# 做什么,实现什么样的效果

对于大部分前端来说, 我们要怎么才方便呢:热加载!主流前端框架通过webpack进行一定配置都可以实现热加载.那么后端渲染的项目可不可以呢,当然也是可以的.我们想想前段时间很火的[browser-sync](https://browsersync.io/),这货无论是作为一个命令行还是gulp的插件,用起来都及其爽.找了一下,我发现webpack同样有browse-sync的插件:`browser-sync-webpack-plugin`.

代码的热更新,这个1同样没有问题,我们都知道`webpack-dev-server`,这货就是实现这个功能的,webpack也有相关配置

除了热加载还有什么呢,es6?sass?postcss?ts? 都是可以的,因为都是代码编译的,所以这部分跟普通框架的配置并没有什么不一样.但是需要注意的就是如果你使用了jquery插件又使用了babel-preset-es2015 可能会发生错误我看了一下是因为preset默认`use strict`,但是某些插件有些冲突(会报变量undefined),但是我又懒得改.所以没加上.bablerc. 使用`babel-preset-es2015-without-strict`同样没用.会报另外的错误,当然这是遗留问题,如果你不使用插件(也不推荐使用)就没啥问题了.

总结一下就是两件事:

1. 热更新减少`ctrl + r`的使用率,如果有两个屏幕的话那就爽了(我并没有)

2. 追新哈哈,其实也是为了开发方便,jquery一时是仍不掉了,但是我可以优化开发过程 

# 为什么要这么做

其实并没有什么理由,你说开发方便吧,但是配置麻烦啊.各有得失吧,这样做是把工作量往自己身上揽,毕竟如果类似那些外包的做法.前端会省很多力气(毕竟很多js都是后端来写).对于活动页面就没必要这么折腾了.毕竟只用那么几次,不需要考虑后续重构之类的问题.

# 怎么实现

首先考虑如果是vue的单页应用我们改怎么实现打包后的资源生成与插入.如果是前端渲染,不管是单页还是多页我们都可以通过`html-webpack-plugin`插件通过模板文件生成需要的html资源,但是对于后端渲染的页面就不能这么做了,其实非要做也是可以的,但是可能每个页面都需要引入不同的资源文件,而且后端模板分的比较开,比如nunjucks直接引入文件还是不行的,必须引入到block中,这样一来就复杂了.也没太必要.

还有一个更好的做法,webpack打包的时候生成资源文件索引manifest,然后后端通过索引在需要的地方插入资源文件.这里同样的有个插件`webpack-manifest-plugin`,这个是在输出目录里生成资源文件列表,如果配合`webpac-dev-server`,可能并存在输出目录,文件都在内存中,但是应该可以配置`writeToFileEmit: true`生成实体文件让后端读取.为什么事应该呢,因为我没用这个插件,插件是后来才发现的,之前是这么写的:

```js
plugin: [
  ...
  function () {
    this.plugin('done', function(stats){
        require('fs').writeFileSync(path.join(config.mapPath, "map.json"), JSON.stringify(stats.toJson().assetsByChunkName, null, 4));
    })
  }
]
```

其实这也是简单的webpack插件的写法,再深层次的我也不是很清楚了.

生成资源文件索引以后,后端可以配置cdn的路径(一般来说应该是可配置的吧),由于`webpack-dev-server`是另外起一个服务器,所以就当是cdn了,当然如果你后端是nodejs可以使用`webpack-dev-middleware`代替`webpack-dev-server` 这个配置就不多说了.配置webpack中的devServer:

```js
devServer: {
    contentBase: path.join(__dirname, 'dist'),
    publicPath: '/assets',
    host: "0.0.0.0",
    compress: true,
    hot: true
}
```

具体路径可以根据自己的需要来修改,这么一来我们的webpack就可以进行动态编译了,不需要改动一点点就全部编译.一般来说文件都会加上hash,但是没有影响因为索引也是有hash的,后端直接引用就可以了.

下面就进行热重载的配置:

```js
plugins: [
    new BrowserSyncPlugin({
        // browse to http://localhost:3000/ during development,
        // ./public directory is being served
        host: 'localhost',
        port: 3000,
        proxy: 'localhost:8765'
    })
]
```

跟一般的前端重载不一样,这里我们需要设置代理,也就是proxy,因为页面是后端渲染的,所以访问链接也是后端提供的.代理好了后配合`webpack-dev-server`,一旦编译好了浏览器就会自动刷新.但是需要注意,修改后端模板文件浏览器不会刷新需要手动,毕竟是后端渲染的.

这样一来我们上面的两个需要算是基本实现了,还有其他的基础功能这里就不啰嗦了,什么配置es6,sass之类的,都是通过一些loader进行配置,对了,这里使用的是webpack2.

最后放上完整的配置:
```js
const webpack = require('webpack')
const CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
const CleanPlugin = require('clean-webpack-plugin');
const BrowserSyncPlugin = require('browser-sync-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const path = require('path')
const glob = require('glob')
const config = require('./config.build')
// isProduct 判断是否是生产环境
const isProduct = (process.env.NODE_ENV === 'production')
console.log(process.env.NODE_ENV)
const entryPath = path.resolve(config.projectPath, config.entryPathName)
const baseURL = !isProduct ? 'http://localhost:8080/assets/' : '/site/static/'
const commonModulePath = path.resolve('./assets/src/modules')
const commonPluginPath = path.resolve('./assets/src/plugins')
const getEntry = entries => {
    const entry = {}
    const srcDirName = entries + '/**/*.js'
    glob.sync(srcDirName).forEach(function (filepath) {
        const name = filepath.slice(filepath.lastIndexOf(config.entryPathName) + config.entryPathName.length + 1, -3);
        entry[name] = filepath;
    })
    return entry
}
module.exports = {
    entry: getEntry(entryPath),
    output: {
        path: config.buildPath,
        filename: 'js/[name].[chunkhash:6].js',
        publicPath: baseURL
    },
    module: {
        rules: [
            {
                test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader'
            },
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract({
                    fallback: 'style-loader',
                    use: 'css-loader?sourceMap&minimize!postcss-loader?sourceMap'
                })
            },
            {
                test: /\.scss$/,
                loader: ExtractTextPlugin.extract({
                    fallback: 'style-loader',
                    use: 'css-loader?sourceMap&minimize!sass-loader?sourceMap'
                })
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                loader: 'url-loader',
                query: {
                    limit: 10000,
                    name: 'images/[name].[hash:7].[ext]'
                }
            },
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                loader: 'url-loader',
                query: {
                    limit: 10000,
                    name: 'fonts/[name].[hash:7].[ext]'
                }
            }
        ]
    },
    resolve: {
        modules: [
            path.join(config.projectPath, './src/js'),
            'node_modules'
        ],
        extensions: ['.js', '.json', '.scss'],
        alias: {
            commonModule: commonModulePath,
            css: path.resolve(__dirname, 'plugins/Site/webroot/css'),
            commonPlugin: commonPluginPath,
            module: path.resolve(config.modulePath),
            plugin: path.resolve(config.pluginPath),
            layer: 'commonPlugin/layer/layer.js',
            lazyload: 'commonPlugin/jquery.lazyload.min.js',
            cookie: 'commonPlugin/js.cookie.js',
            dmuploader:  'commonPlugin/uploader/src/dmuploader.min.js',
            tmpl:  'commonPlugin/wu.tmpl.js/wu.tmpl.js',
            prettySocial: 'commonPlugin/prettySocial/jquery.prettySocial.js',
            "jquery.rating": 'commonPlugin/jquery-star-rating/src/rating.js',
            "jquery.rating.css": 'commonPlugin/jquery-star-rating/src/rating.css',
            "jquery.validate": "jquery-validation",
        }
    },
    externals: {
        'jquery': 'window.$',
        'lodash': 'window._'
    },
    devtool: 'source-map',
    plugins: [
        new BrowserSyncPlugin({
            // browse to http://localhost:3000/ during development,
            // ./public directory is being served
            host: 'localhost',
            port: 3000,
            proxy: 'localhost:8765'
        }),
        new CleanPlugin(['*'], {
            root: path.resolve(config.buildPath)
        }),
        new ExtractTextPlugin({ filename: "css/[name]-[chunkhash:6].css", allChunks: true }),
        new webpack.ProvidePlugin({
            $: "jquery",
            jQuery: "jquery",
            "window.jQuery": "jquery",
            "_": "lodash"
        }),
        new CommonsChunkPlugin({
            name: "common",
            filename: 'common-[chunkhash:6].js',
            minChunks: 3
        }),
        // new webpack.optimize.UglifyJsPlugin({
        //     minimize: true,
        //     compress: {
        //         warnings: false
        //     },
        //     comments: false
        // }),
        //生成编译之后的文件映射
        function () {
            this.plugin('done', function(stats){
                require('fs').writeFileSync(path.join(config.mapPath, "map.json"), JSON.stringify(stats.toJson().assetsByChunkName, null, 4));
            })
        }
    ],
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        publicPath: '/assets',
        host: "0.0.0.0",
        compress: true,
        hot: true
    }
}
```

原文地址: [webpack传统后端渲染的项目前端配置](https://mp.weixin.qq.com/s?__biz=MzI1MDYzNjk2Ng==&mid=2247483652&idx=1&sn=11bff26f0f4e3f8ebcc714a9a3f3076d&chksm=e9fe7ecdde89f7db4a86b69929e880af38083eb6db6de1f4ed1d41b2c6cfa6927a8ef08127a2#rd)
惯例放上二维码:
![image](https://cloud.githubusercontent.com/assets/8351437/23328040/d9cd734e-fb53-11e6-986b-fa3ba48eaa2b.png)

来自南京前端群(240344448)投递，我github：[https://github.com/xiadd](https://github.com/xiadd)，我们的[南京前端官网](http://nanjingjs.org)
