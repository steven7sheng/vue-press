---
lastUpdated: false
---
# 学习webpack

[[toc]]

## Original
::: details 官方原文
### Concepts

webpack is a **static** module bundler for modern JavaScript application

it internally builds a [`dependency graph`] from one or more entry points
and then combines every module your project needs into one or more bundles,
which are static assets to serve your content from.

Core concepts:
* Entry
* Output
* Loaders
* Plugins
* Mode
* Browser Compatiblity

一期目标问题
* Manually Bundling an Application
* Live Coding a Simple Module Bundler
* Detailed Explaination of a Simple Module Bundler

### Entry
entry point = which module webpack begin building out its internal **dependency graph**.
figure out depends on(directly or indirectly).

`./src/index.js`

### Output
emit the bundles and specific the names of these files.
`./dist/main.js`
{ path, filename }

### Plugins
leveraged to perform a wider range of tasks like 
bundle optimization , asset management and injection of enviroment variables.
*extend capabilities*
* require it
* push to plugins array
* create instance with `new`

### Mode
development,`production`,none
built-in  optimization that correspond to each environment.

### Browser Compatibility
supports all "ES5-compliant" 
webpack needs Promise for `import()`,`require.ensure()`.
to support older browser you need to load polyfill.

> Webpack 5 runs on Node.js version 10.13.0+

### Scenarios
1. Seperate App and Vender Entries
```json
{
    entry: {
        main: 'xxx',
        vendor: 'yyy'
    },
    output: {
        filename: '[name].[contenthash].bundle.js', // prod
        filename: '[name].bundle.js' // dev
    },
}
```
`optimization.splitChunks` option takes care of seperating vendors and app modules.

### Multiple Entry Points
Output has a unique name.
can use substitutions    `[name]`
:::
## Webpack内容综述
::: tip 章节
* P1-P3 webpack 的基本概念和日常开发的使用技巧
* P4-P5 以工程化的方式组织webpack构建配置，和webpack打包优化
* P6-P7 详细剖析webpack打包原理和插件，以及loader的实现
* P8 从实际web商城项目出发，讲解webpack实际使用
::: 

### 01 webpack与构建发展简史
为什么需要构建工具
* 转换ES6语法
* 转换JSX
* CSS前缀补全/预处理器
* 压缩混淆
* 图片压缩
  
构建演变之路
  `ant+YUI tool --> grunt --> fis3/gulp --> rollup,webpack,parcel`

为什么选择webpack
* 社区生态丰富
* 配置灵活和插件化扩展
* 官方更新迭代速度快
  
初识webpack：配置文件名称

* webpack默认配置文件 `webpack.config.js`
* 可以通过 webpack --config 指定配置文件

初识webpack：配置组成
* entry // 打包的入口文件
* output // 打包的输出
* mode // 环境
* module.rules // Loader配置
* module.plugins // 插件配置

**零配置**webpack包含哪些内容
* 默认entry为 ./src/index.js
* 默认output为 ./dist/main.js
* loader raw-loader
* plugins HtmlwebpackPlugin template:./src/index.html

环境搭建 Node.js 和 NPM

安装webpack和 webpack-cli

webpack初体验

```js
const path=require('path');
module.exports={
    mode: 'production',
    entry: './src/index.js',
    output:{
        path: path.resolve(__dirname,'dist'),
        filename: 'bundle.js'
    }
};
```
--->构建结果
```html
<!doctype html>
<html>
    <head>
        <title>demo</title>
    </head>
    <body>
        <script src="dist/bundle.js"></script>
    </body>
</html>
```
> 通过npm script 运行`webpack`
> 
> **原理** 模块局部安装会在 node_modules/.bin目录创建软链接

### 02 webpack基础用法
Entry 用来指定webpack的打包入口

模块依赖图的入口是 entry
对于非代码比如图片、字体依赖也会不断加入到依赖图中

Entry的用法
* 单入口：entry是一个字符串
* 多入口：entry是一个对象

Output用来告诉webpack 如何将编译后的文件输出到磁盘

Output的用法：单入口配置
```js
module.exports={
    entry:'./path/to/my/entry/file.js'
    output:{
        filename: 'bundle.js'
        path: __dirname + '/dist'
        
    }
}
```
Output的用法：多入口配置
```js
module.exports={
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name].js',
        path: __dirname + '/dist'
    }
}
```
> 通过占位符确保文件名称的唯一

* 核心概念之Loaders
  
webpack开箱即用只支持js和json两种文件类型，通过Loaders去支持其他文件类型并且把它们转化成有效的模块，
并且可以添加到依赖图中。
本身是一个函数，接受源文件作为参数，返回转换的结果。

常见的Loaders
| 名称                    | 描述                     |
| ----------------------- | ------------------------ |
| babel-loader            | 转换ES6+等JS新特性语法   |
| css-loader              | 支持.css文件的加载和解析 |
| less-loader,sass-loader | ...                      |
| ts-loader               | 将ts转换成css            |
| file-loader             | 进行图片、字体等的打包   |
| raw-loader              | 将文件以字符串的形式导入 |
| thread-loader           | 多进程打包JS和CSS        |

Loaders 的用法
```js
const path=require('path');

module.exports={
    output:{
        filename: 'bundle.js'
    },
    module:{
        rules:[
            {test:/\.txt$/,use: 'raw-loader'}
        ]
    }
}
```
> `test` 指定匹配规则,`use` 指定使用的loader名称

* 核心概念之Plugins

插件用于bundle文件的优化，资源管理器和环境变量的注入
作用于整个构建过程

常见的Plugins有哪些?
| 名称                     | 描述                                       |
| ------------------------ | ------------------------------------------ |
| CommonsChunkPlugin       | 将chunks相同的模块代码提取成公共js         |
| CleanWebpackPlugin       | 清理构建目录                               |
| ExtractTextWebpackPlugin | 将CSS从bundle文件里提取成一个独立的CSS文件 |
| CopyWebpackPlugin        | 将文件或者文件夹拷贝到构建的输出目录       |
| HtmlWebpackPlugin        | 创建HTML文件去承载输出的bundle             |
| UglifyjsWebpackPlugin    | 压缩JS                                     |
| ZipWebpackPlugin         | 将打包出的资源生成一个zip包                |

Plugins的用法
```js
const path=require('path');

module.exports={
    output:{
        filename: 'bundle.js'
    },
    plugins:[
        new HtmlWebpackPlugin({template:'./src/index.html'})
    ]
}
```
> 放到plugins的数组里面

* 核心概念之Mode
Mode用来指定当前的构建环境是: production development还是none，
设置mode可以使用webpack内置的函数，默认值是production

Mode的内置函数功能
| 选项        | 描述                                                                                |
| ----------- | ----------------------------------------------------------------------------------- |
| development | 设process.env.NODE_ENV 的值为 development. 开启NamedChunksPlugin+NameModulesPlugin. |
| production  | 设process.env.NODE_ENV 的值为 production. 开启"一堆Plugin"                          |
| none        | 不开启任何优化选项                                                                  |

::: tip "一堆Plugin" 
* FlagDependencyUsagePlugin
* FlagIncludedChunksPlugin
* ModuleConcatentationPlugin
* NoEmitOnErrorsPlugin
* OccurenceOrderPlugin
* SideEffectsFlagPlugin
* TerserPlugin
:::

资源解析: 解析ES6
使用babel-loader babel的配置文件是.babelrc
```js
module.exports={
    module:{
        rules:[ {test:/\.js$/,use: 'babel-loader'} ]
    }
}
```

增加ES6的babel preset配置
```json
{
    "presets":["@babel/preset-env"],
    "plugins":["@babel/proposal-class-properties"]
}
```
> 增加ES6 preset配置

资源解析: 解析React JSX

```json
{
    "presets":["@babel/preset-env","@babel/preset-react"],
    "plugins":["@babel/proposal-class-properties"]
}
```
> 增加React的 babel-preset 配置

资源解析: 解析CSS
```js
module.exports={
    module:{
        rules:[
            {test:/\.css$/,use:["style-loader","css-loader"]}
        ]
    }
}
```
> css-loader用于加载.css文件，并且转换成commonjs对象
> style-loader 将样式通过`<style>`标签插入到head中

解析Less和SaSS
less-loader 用于将less转换成css
解析图片
file-loader 用于处理文件 `test: /\.(png|svg|jpg|gif)$/`
解析字体
file-loader 也可以用于处理字体 `test: /\.(woff|woff2|eot|ttf|otf)$/`
使用url-loader
url-loader 也可以处理图片和字体
可以设置较小的资源自动base64
 ```js
 module.exports={
     module:{rules:[
         {test:/\.(png|svg|jpg|git)$/,
         use:[{
             loader: 'url-loader',
             options: {limit: 10240}
         }]}

     ]}
 }
 ```
 webpack中的文件监听
 
文件监听是在发现源码发生变化时，自动重新构建出新的输出文件。

webpack 开启监听模式，有两种方式:
1. 启动 webpack命令时，带上--watch参数
2. 在配置webpack.config.js中设置 watch: true

唯一缺陷：每次需要手动 刷新浏览器

::: details 文件监听原理
轮询判断文件的最后编辑时间是否变化
某个文件发生了变化，并不会立即告诉监听者，而是先缓存起来，等aggregateTimeout

 ```js
 module.exports={
     watch: true, //默认false
     watchOptions:{
         ignored: /node_modules/, // 默认空，不监听的文件(夹),支持正则匹配
         aggregateTimeout: 300, // 监听到变化后会等300ms再去执行，默认300
         poll:1000 // 判断文件是否变化是通过轮询问系统指定文件有没变化实现的，默认每秒问1000次
     }
 }
 ```
:::

热更新 webpack-dev-server
* WDS 不刷新浏览器
* WDS 不输出文件，而是放在内存中
* 使用HotModuleReplacementPlugin插件
`webpack-dev-server --open`

热更新 使用webpack-dev-middleware
WDS将 webpack输出的文件传输给服务器
适用于灵活的定制场景
```js
const express= require('express');
const webpack= require('webpack');
const webpackDevMiddleware=require('webpack-dev-middleware');

const app=express();
const config=require('./webpack.config.js');
const compiler= webpack(config);

app.use(webpackDevMiddleware(compiler,{
    publicPath: config.output.publicPath
}));
app.listen(3000,function(){
    console.log('app listening on port 3000');
})
```

热更新的原理分析

* Webpack Compile: 将JS编译成 Bundle
* HMR Server: 将热更的文件输出给 HMR Runtime
* Bundle server: 提供文件在浏览器的访问
* HMR Runtime: 会被注入到浏览器，更新文件的变化
* bundle.js: 构建输出的文件

![HMR原理分析](./images/HMR-principal.png)

* 什么是文件指纹?
  打包后输出的文件名的后缀
hash串
文件指纹如何生成
* Hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的hash值就会更改
* Chunkhash：和webpack打包的chunk有关，不同的entry会生成不同的chunkhash值
* Contenthash: 根据文件内容来定义hash，文件内容不变，则contenthash不变

JS的文件指纹设置
设置output的filename，使用[chunkhash]
`output:{ filename: '[name][chunkhash:8].js' }`
CSS文件的指纹设置
设置MiniCssExtractPlugin的filename,使用[contenthash]
```js
new MiniCssExtractPlugin({ filename: `[name][contenthash:8].css`})
```

图片文件的指纹设置
设置file-loader的name，使用[hash]
`use: [{loader:'file-loader',options:{name:'img/[name][hash:8].[ext]'}}]`
| 占位符名称  | 含义                          |
| ----------- | ----------------------------- |
| ext         | 资源的后缀名                  |
| name        | 文件名称                      |
| path        | 文件的相对路径                |
| folder      | 文件所在的文件夹              |
| contenthash | 文件的内容hash，默认md5生成   |
| hash        | 文件内容的Hash,默认md5生成    |
| emoji       | 一个随机的指代文件内容的emoji |

代码压缩
HTML压缩 CSS压缩 JS压缩

JS文件的压缩 内置了uglifyjs-webpack-plugin
CSS文件的压缩 使用optimize-css-assets-webpack-plugin，同时使用 cssnano
```js
plugins:[
    new OptimizeCSSAssetsPlugin({
        assetsNameRegExp:/\.css$/g,
        cssProcessor: require('cssnano')
    })
]
```
html文件的压缩 修改html-webpack-plugin, 设置压缩参数
```js
new HtmlWebpackPlugin({
    template: path.join(__dirname,'src/search.html'),
    filename: 'search.html',
    chunks:['search'],
    inject: true,
    minify:{
        html5: true,
        collapseWhitespace: true,
        preserveLineBreaks: false,
        minifyCSS: true,
        minifyJS: true,
        removeComments: false 的定义: font-size of the root elementrem 和 px 对比
移动端CSS px自动转换成rem
使用px2rem-loader
    }
})
```



### 03 webpack进阶用法

1. 当前构建的问题
每次构建的时候不会清理目录，造成构建的输出目录output文件越来越多

通过 npm scripts 清理构建目录
`rm -rf ./dist && webpack`
`rimraf ./dist && webpack`

自动清理构建目录
避免构建前每次都需要手动删除dist
使用clean-webpack-plugin ,默认会删除output指定的输出目录

```js
const plugins=[ new CleanWebpackPlugin() ] 
```

2. CSS3属性为什么需要前缀?
* Trident(-ms)
* Geko(-moz)
* Webkit(-webkit)
* Presto(-o)

举个例子
```css
.box{
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px;
    -o-border-radius: 10px;
    border-radius: 10px;
}
```
PostCSS插件 autoprefixer 自动补齐CSS3前缀
使用 autoprefixer 插件
根据 CanIUse规则
```js
const module.rules=
{
    test: /\.less$/,
    use:[
        'style-loader', 'css-loader', 'less-loader',
        {
            loader: 'postcss-loader',
            options:{ plugins:()=>[
                    require('autoprefixer')({
                        browsers:[ "last 2 version",">1%","iOS 7" ]
                    })
                ]
            }
        }
    ]
}
```

3. 浏览器的分辨率
CSS 媒体查询实现响应式布局
```css
@media screen and (max-width: 980px){
    .header { width: 900px; }
}
@media screen and (max-width: 480px){
    .header { width: 400px; }
}
@media screen and (max-width: 350px){
    .header { width: 300px; }
}
```

4. rem 是什么
> W3C 对 `rem` 的定义: font-size of the root element

rem 和 px 对比
* rem 是相对单位
* px 是绝对单位

移动端CSS px自动转换成rem

使用`px2rem-loader`
页面渲染时计算根元素的font-size值
* 可以使用手淘的lib-flexible库
* https://github.com/amfe/lib-flexible

```js
const rules1.use=[{
    'style-loader','css-loader','less-loader',
    {
        loader: 'px2rem-loader',
        options:{
            remUnit: 75,
            remPecision: 8
        }
    }
}]
```

资源内联的意义
代码层面：
* 页面框架的初始化脚本
* 上报相关打点
* css内联避免页面闪动
请求层面: 减少HTTP网络请求数
* 小图片或者字体内联 url-loader

HTML 和 JS 内联

raw-loader 内联 html
```html
<script>${require('raw-loader!babel-loader!./meta.html')}</script>
```
raw-loader 内联 js
```html
<script>${require('raw-loader!babel-loader!../node_modules/lib-flexible')}</script>
```
css 内联
方案一 借助 style-loader
方案二 html-inline-css-webpack-plugin
```js
const rules1.use=[
    { 
        loader: 'style-loader',
        options:{
            insertAt: 'top',// 样式插入到<head>
            singleton: true,// 将所有的style标签合并成一个
        },
        'css-loader',
        'sass-loader'
    }
}]
```

多页面应用(PWA)概念
> 每一次页面跳转的时候，后台服务器都会给返回一个新的html文档，
> 
> 这种类型的网站也就是多页面网站，也叫做多页面应用

多页面打包基本思路

每个页面对应一个entry，一个html-webpack-plugin
缺点：每次新增或删除页面需要改webpack配置

多页面打包 通用方案

动态获取entry 和设置 html-webpack-plugin
利用 glob.sync
* entry: glob.sync(path.join(__dirname,'./src/*/index.js')),

使用 source map

作用：通过source map定位到源码
* source map 科普 阮一峰javascript_source_map

开发环境开启，线上环境关闭
* 线上排查问题的时候可以将sourcemap上传到错误监控系统

source map 关键字

eval: 使用eval包裹模块代码

source map： 产生.map文件

cheap： 不包含列信息

inline： 将.map作为DataURI嵌入，不单独产生.map文件

module：包含loader的sourcemap

source map 类型
* none =prod yes
* eval
* cheap-eval-source-map
* cheap-module-source-map
* eval-source-map
* cheap-source-map =prod yes
* cheap-module-source-map =prod yes
* inline-cheap-source-map
* inline-cheap-module-source-map
* source-map =prod yes
* inline-source-map
* hidden-source-map =prod yes

基础库分离
思路：将react、react-dom 基础包通过cdn引入，不打入bundle中
方法：使用html-webpack-externals-plugin
```js
cosnt plugins=[
    new HtmlWebpackExternalsPlugin({
        externals:[
            {module: 'react',entry:'//cdn/react.min.js?v=123'},
            {module: 'react-dom',entry:'//cdn/react-dom.min.js?v=123'}
        ]
    })
]
```
使用SplitChunksPlugin 进行公共脚本分离
webpack4 内置的，替代CommonsChunkPlugin 插件

chunks 参数说明：
* async 异步引入的库进行分离(默认)
* initial 同步引入的库进行分离
* all 所有引入的库进行分离(推荐)
```js
module.exports={
optimization: {
    splitChunks: {
        chunks: 'async',
        minSize: 30000,
        maxSize: 0,
        minChunks: 1,
        maxAsyncRequests: 5,
        maxInitialRequests: 3,
        automaticNameDelimiter: '~',
        name: true,
        cacheGroups: {
            vendors: {
            test: /[\\/]node_modules[\\/]/,
            priority: -10
        }
    }
}
```
利用SplitChunksPlugin 分离基础包

test: 匹配出需要分离的包
```js
optimization.splitChunks.cacheGroups={
    commons: {
        test:/(react|react-dom)/,
        name: 'vendors',
        chunks: 'all'
    }
}
```

利用SplitChunksPlugin 分离公共文件

minChunks: 设置最小引用次数为2次
minSize：分离的包体积的大小
```js
optimiaztion.splitChunks={
    minSize: 0,
    cacheGroups:{
        commons: {
            name: 'commons',
            chunks: 'all',
            minChunks: 2
        }
    }
}
```

Tree Shaking(摇树优化)
概念： 1个模块可能有多个方法，只要其中的某个方法使用到了，则整个文件都会被打到bundle里面去，tree-shaking就是只把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除掉。

使用：webpack默认支持，在.babelrc中里设置modules:false即可
* production mode的情况下默认开启
  
要求: 必须是ES6的语法,CJS的方式不支持

DCE(Dead code elimination)
代码不会被执行,不可到达
代码执行的结果不会被用到
代码只会影响死变量(只写不读)

Tree-Shanking原理
利用ES6 模块的特点:
* 只能作为模块顶层的语句出现(静态编译)
* import的模块名只能是字符串常量
* import binding 是 immutable的
代码擦除:uglify阶段删除无用代码

现象:构建后的代码存在大量闭包代码
```js
    //a.js
    export default 'xxxx';
    //b.js
    import index from './a';
    console.log(index);
```
--->
```js
function(module,__webpack_exports__,webpack_require__){
    "use strict";
    __webpack_require__.r(__webpack_exports__);
    var _js_index_WEBPACK_IMPORTED_MODULE_0__=__webpack_require__(
        console.log(__js_index_WEBPACK_IMPORTED_MODULE_0__["default"]);
    );
}
```

会导致什么问题?
大量作用域包裹代码,导致体积增大(模块越多越明显)
运行代码时创建的函数作用域变多,内存开销变大

模块转换分析
```js
import { helloworld } from './helloworld';
import '../../common';

document.write(helloworld());
```
--->
```js
/* harmony import */ var _common__WEBPACK_IMPORTED_MODULE_0__= __webpack_requrie__(1);
/* harmony import */ var _common__WEBPACK_IMPORTED_MODULE_1__= __webpack_requrie__(2);
```
结论:
* 被webpack转换后的模块会带上一层包裹
* import 会被转换成 __webpack_require

进一步分析webpack的模块机制
分析:
* 打包出来的是一个IIFE(匿名闭包)
* modules是一个数组,每一项是一个模块初始化函数
* __webpack_require 用来加载模块,返回 modules.exports
* 通过`WEBPACK_REQUIRE_METHOD(0)` 启动程序
  
scope hoisting原理
原理: 将所有模块的代码按照引用顺序放在一个函数作用域里,然后适当的重命名一些变量以防止变量名冲突
对比: 通过scope hoisting可以减少函数声明代码和内存开销
scope hoisting使用
webpack mode为production 默认开启
必须是ES6语法 CJS不支持

```js
    const plugins=[
        new webpack.optimize.ModuleConcatenationPlugin();
    ];
```
代码分割的意义

对于大的web应用来讲,将所有的代码都放在一个文件中显然是不够有效的,
特别是当你的某些代码是在某些特殊的时候才会被使用到.webapck有一个功能就是将你的
代码库分割成chunks(语块),当代码运行到需要它们的时候在进行加载.

适用的场景:
抽离相同代码到一个共享块
脚本懒加载,使得初始下载的代码更小

懒加载JS脚本的方式
CommonjS: require.ensure
ES6: 动态import(目前还没原生支持,需要babel转换)

如果使用动态 import?
安装babel插件 @babel/plugin-syntax-dynamic-import
```js
const plugins=['@babel/plugin-syntax-dynamic-import'];
```
代码分割效果
Assert Size Chunks
xxxx.js 176 bytes 2 [emmitted]


ESLint的必要性
案例: 某手机系统的webview而没有使用X5内核,解析JSON时遇到重复key报错,导致白屏
> 如何避免类似代码问题?

行业里优秀的 ESLint 规范实践
腾讯: 
* alloyteam 团队-> eslint-config-alloy
* ivweb 团队-> eslint-config-ivweb

制定团队的ESLint 规范
不重复早轮子,基于 eslint:recommend 配置并改进
能够帮助发现代码错误的规则,全部开启
帮助保持团队的代码风格统一,而不是限制开发体验

ESLint如何落地
和CI/CD系统集成,和webpack集成
方案一: webpack与CI/CD集成
**CI PIPELINE** 增加 lint pipeline
本地开发阶段增加pre commit钩子
1. 安装husky
2. 增加npm script -> precommit lint-staged
```json
{
    "lint-staged":{ linters:{"*.{js,scss}":["eslint --fix","git add"]}}
}
```
方案二: webpack与ESLint集成
使用eslint-loader,构建时检查JS规范
```js
module.rules[0].use=['babel-loader','eslint-loader'];
```

webpack打包库和组件
webpack除了可以用来打包应用,也可以用来打包js库
实现一个大整数加法的打包
* 需要打包压缩版和非压缩版本
* 支持AMD/CJS/CJS/ESM 模块引入
库的目录结构和打包要求
打包输出的库名称:
* 未压缩版 large-number.js
* 压缩版 large-number.min.js
```
|-/dist
|--large-number.js
|--large-number.min.js
|-webpack.config.js
|-package.json
|-/src
|--index.js
```
支持的使用方式
ESM: `import * as largeNumber from 'large-number'; largeNumber(a,b);`

CJS: `const largeNumber = require('large-number'); largeNumber(a,b);`

AMD: `require(['large-number'],function(largeNumber){ largeNumber(a,b);});`

可以直接通过script引入 
```html
<script src="https://unpkg.com/large-number"></script>
<script>
    // global variable
    largeNumber('999','1');
    // Property in the window object
    window.largeNumber('999','1');
</script>
```

如何将库暴露出去?
library: 指定库的全局变量
libraryTarget: 支持库的引入方式

```js
module.exports={
    mode: 'production',
    entry:{
        'large-number': './src/index.js',
        'large-number.min': './src/index.js'
    },
    output:{
        filename: '[name].js',
        library: 'largeNumber',
        libraryExport:'default',
        libraryTarget:'umd'
    }
}
```
如何只对 .min压缩
通过inclue设置 只压缩min.js 结尾的文件
```js
optimization={
    minimize: true,
    minmizer:[
        new TerserPlugin({
            inclue: /\.min\.js$/
        })
    ]
}
```

设置入口文件 package.json的main字段为 index.js
```js
 if(process.env.NODE_ENV === 'production') module.exports=require('./dist/large-number.min.js');
 else module.exports=require('./dist/large-number.js');
 ```

 页面打开过程
 服务端渲染(SSR)是什么
渲染:HTML+CSS+JS+Data -> 渲染后的HTML

服务端:
所有模板等资源都存在服务端
内网机器拉取数据更快
一个HTML返回所有数据

浏览器和服务器交互流程
请求开始-server-{html-template,data}-serverRender
-浏览器解析并渲染-加载并执行js和其他资源-页面完全可交互


客户端渲染vs服务端渲染
|          | 客户端渲染                                | 服务端渲染            |
| -------- | ----------------------------------------- | --------------------- |
| 请求     | 多个请求(HTML,数据等)                     | 1个请求               |
| 加载过程 | HTML&数据串行加载                         | 1个请求返回HTML和数据 |
| 渲染     | 前端渲染                                  | 服务端渲染            |
| 可交互   | 图片等静态资源加载完成,JS逻辑执行完可交互 |

总结: 服务端渲染(SSR)的核心是减少请求

SSR的优势
1. 减少白屏时间
2. 对于SEO友好

SSR代码实现思路
服务端
* 使用 react-dom/server的 renderToString 方法将React组件渲染成字符串
* 服务端路由返回对应的模板
客户端
* 打包出针对服务端的组件

Webapck ssr打包存在的问题
浏览器全局变量(Node.js中 没有 document,window);
* 组件适配: 将不兼容的组件根据打包环境进行适配
* 请求适配: 将fetch或者ajax发送请求的写法改成`isomorphic-fetch`或者`axios`
样式问题(Node.js无法解析样式css)
* 方案一: 服务端打包通过ignore-loader忽略掉css的解析
* 方案二: 将style-loader替换成isomorphic-style-loader
  
如何解决样式不显示的问题?

使用打包出来的浏览器端HTML为模板

设置占位符,动态插入组件
`<!--HTML_PLACEHOLDER-->`

首屏数据如何处理
服务端获取数据
替换占位符

当前构建时的日志显示
展示一大堆日志,很多并不是开发者关注
统计信息 stats
| preset      | alternative | description                    |
| ----------- | ----------- | ------------------------------ |
| errors-only | none        | 只在发生错误时输出             |
| minimal     | none        | 只在发生错误或有新的编译时输出 |
| none        | false       | 没有输出                       |
| normal      | true        | 标准输出                       |
| verbose     | none        | 全部输出                       |


如何优化命令行的构建日志
使用 friendly-errors-webpack-plugin
* success 构建成功的日志提示
* warning 构建警告时的日志提示
* error 构建报错时的日志提示

stats 设置成erros-only
```js
const plugins=[new FriendlyErrorsWebpackPlugin()];
const stats="errors-only";
```

如何判断构建是否成功?
 在CI/CD 的 pipeline 或者发布系统需要知道当前构建状态

 每次构建完成后输入 `echo $?` 获取错误码

构建异常和中断处理
webpack4之前的版本构建失败不会抛出错误码(error code)
Node.js  中的 prcess.exit 规范
* 0 表示成功完成,回调函数中,err 为null
* 非0 表示执行失败,回调函数中,error不为null,error.code 就是传给exit的数字

如何主动捕获并处理构建错误?
compiler在每次构建结束后出发 done这个 hook

process.exit 主动处理构建报错
```js
plugins:[
    function(){
        this.hook.done.tap('done',(stats)=>{
            if(stats.compilation.erros && stats.compilation.erros.length && process.argv.indexOf('--watch')===-1){
                console.log('build error');
                process.exit(1);
            }
        })
    }
]
```

### 04 编写可维护的webpack构建配置

构建配置抽离成npm包的意义

通用性
* 业务开发者无需关注构建配置
* 统一团队构建脚本

可维护性
* 构建配置合理的拆分
* README文档、ChangeLog文档

质量
* 冒烟测试、单元测试、测试覆盖率
* 持续集成 
  
构建配置管理的 可选方案
* 通过多个配置文件管理不同环境的构建， webpack --config参数进行控制
* 将构建配置设计成一个库，比如hjs-webpack，Neutino，webpack-blocks
* 抽成一个工具进行管理，比如create-react-app，kyt，nwb
* 将所有配置放在一个文件，通过 --env 参数控制分支选择

构建配置包设计
通过多个配置文件管理不同环境的webpack配置
* 基础配置 webpack.base.js
* 开发环境 dev
* 生产环境 prod
* ssr环境 ssr
  
抽离成一个npm包统一管理
* 规范： git commit日志，readme，eslint规范，semver规范
* 质量： 冒烟测试，单元测试，测试覆盖旅和CI

通过webpack-merge 组合配置
```js
const merge = require('webpack-merge');
module.exports=merge(baseConfig,devConfig);
```

功能模块设计
* 基础配置：base
  * 资源解析:ES6,React,CSS,Less,图片,字体
  * 样式增强:CSS前缀补齐，CSS px转换成rem
  * 目录清理
  * 多页面打包
  * 命令行信息显示优化
  * 错误捕获和处理
  * CSS提取成一个单独的文件
* 开发阶段配置：
  * 代码热更新：CSS热更新，JS热更新
  * sourcemap
* 生产阶段配置：
  * 代码压缩
  * 文件指纹
  * Tree Shaking
  * Scope Hoisting
  * 速度优化： 基础包CDN
  * 体积优化： 代码分割
* SSR配置：
  * output的libraryTarget设置
  * CSS解析的ignore

目录结构设计
lib 放置源代码
test 放置测试代码
使用eslint规范构建脚本
使用 eslint-config-airbnb-base
eslint --fix 可以自动处理空格
```js
module.exports={
    "parser": "babel-eslint",
    "extends": "eslint-config-airbnb-base",
    "env":{
        "browser": true, "node": true
    }
}
```

冒烟测试 smoke testing
冒烟测试是指对提交测试的软件在进行详细深入的测试之前而进行的预测试，
这种预测试的主要目的是暴露导致软件需重新发布的基本功能失效等严重问题

冒烟测试执行
构建是否成功
每次构建完成build目录是否有内容输出
* 是否有JS、CSS等静态资源文件
* 是否有HTML文件
  
判断是否构建成功
在示例项目里面运行构建，看看是否有报错
```js
const path = require('path');
const webpack = require('webpack');
const rimraf = require('rimraf');
const Mocha = require('mocha');

const mocha = new Mocha({ timeout: '10000ms' });
process.chdir(__dirname);

rimraf('./dist',()=>{
    const prodConfig= require('../../lib/webpack.prod');
    webpack(prodConfig,(err,stats)=>{
        if(err) console.error(err); return;
        console.log(stats.toString({
            colors: true,modules: false,children: false,chunks: false,chunkMoudles:false
        }))
    });
    console.log('\n'+'Complier success, begin mocha test');
});
```

判断基本功能是否正常
编写mocha测试用例 是否有JS，CSS静态资源，HTML文件;
```js
const glob =  require('glob-all');
describe('checking generated file exists',function(){
    it('should generate html files',function(done){
        const files=glob.sync(['./dist/index.html','./dist/search.html']);
        if(files.length>0) done();
        else throw new Error('No html files found');
    });

    it('should generate js & css files',function(){
        const files=glob.sync(['./dist/index_*.js','./dist/index_*.css']);
        if(files.length>0) done();
        else throw new Error('No files found');
    });
})
```

单元测试与测试覆盖率
单纯测试框架，需要断言库
* chai
* should.js
* expect
* better-assert

集成框架 开箱即用
* Jasmine
* Jest

极简API


编写单元测试用例
* 技术选型: Mocha + Chai
* 测试代码: describe + it + expect
* 测试命令: mocha add.test.js
```js
const expect = require('chai').expect;
const add = require('../src/add');
describe('use expect: src/add.js',()=>{
    it('add(1,2)===3',()=>{
        expect(add(1,2).to.equal(3));
    })
});
```
单元测试接入
1. 安装 mocha + chai
2. 新建test目录，并增加 [name].test.js测试文件
3. 在package.json中 scripts 字段增加 test命令 `node_modules/mocha/bin/_mocha`
4. 执行测试命令 `npm run test`

持续集成的作用
优点： 快速发现错误，放置分支大幅偏离主干

核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失效就不能集成。

Github最流行的CI （top10
Travis CI,Circle CI,Jenkins,AppVeyor,CodeShip,Drone,Semaphore CI,Buildkite,Wercker,TeamCity

接入Travis CI
1. https://travis-ci.org 使用github账号登录
2. 在/account/repositories 为项目开启
3. 项目根目录下新增 travis.yml

yml文件内容 install安装项目依赖 ，script运行测试用例
```yml
language: node_js

sudo: false

cache:
    apt: true
    directories:
        - node_modules

node_js: stable # 设置相应版本

install:
    - npm install -D # 安装构建依赖器
    - cd ./test/template-project
    - npm install -D # 安装构建依赖器

script: 
    - npm test
```

发布到npm
添加用户 `npm adduser`
升级版本
* 升级补丁版本号 npm version patch
* 升级小版本号 npm version minor
* 升级大版本号 npm version major
版本发布： npm publish

git规范和changelog生成
良好的 git commit规范优势：
* 加快code-review流程
* 根据git commit的元数据生成changelog
* 后续维护者可以知道feature被修改的原因
  
技术方案, 提交格式- 
统一团队commit标准 extends angular-gitcommit 
使用commitize工具，validate-commit-msg工具，gitlab serverhook，
使用conventional-changelog

提交格式要求
```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```
type代表某次提交的类型
* feat 新增feature
* fix 修复bug
* docs 仅仅修改了文档，比如README，CHANGELOG，CONTRIBUTE等等
* style 仅仅修改了空格，格式缩进，标点符号等，不改变代码逻辑
* refactor 代码重构，没有加新的功能或者修复bug
* perf 优化相关，比如提升性能、体验
* test 测试用例 包括单元测试 集成测试等
* chore 改变构建流程、或者增加依赖库 工具等
* rever 同GIT回滚

本地开发阶段增加preCommit钩子
安装husky 通过commitmsg钩子校验信息
```json
{
    "scripts": {
        "commitmsg": "validate-commit-msg",
        "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
    },
    "devDependencies":{
        "validate-commit-msg":"^2.11",
        "conventional-changelog-cli": "^1.2",
        "husky": "^0.13"
    }
}
```

开源项目版本信息案例
* 软件的版本通常由三位组成，形如X.Y.Z
* 版本是严格递增的
* 在发布重要版本时，可以发布alpha,rc等先行版本
* aplha，rc等修饰版本的关键字后面可以带上次数和meta信息

遵守semsemver 规范的优势
避免出现循环依赖
依赖冲突减少

语义化版本(Semantic Versioning)规范格式
主版本号： 当你做了不兼容的API修改
次版本号： 当你做了向下兼容的功能新增
修订号: 当你做了向下兼容的问题修正

先行版本号

先行版本号可以作为发布正式版之前的版本，格式是在修订版本号后面加上一个连接号(-),
再加上一连串以点(.)分割的标识符，标识符可以由英文数字和连接号[0-9A-Za-z-]组成
* alpha 是内部测试版，一般不向外发布，会有很多bug，一般只有测试人员使用。
* beta 也是测试版，这个阶段的版本不会一直加入新的功能，在Alpha版本之后推出
* rc Release Candidate 系统平台上就是发行候选版本。RC版本不会再加入新的功能了，主要着重于除错

### 05 Webpack构建速度和体积优化策略

初级分析：使用webpack内置的stats

stats：构建的统计信息
package.json 中使用 stats
`"build:stats":"webpack --env production --json > stats.json"`,

Node.js 使用
```js
const webpack =require('webpack');
const config = require('./webpack.config.js')('production');

webpack(config,(err,stats)=>{
    if(err) return console.error(err);
    if(stats.hasErrors()){
        return console.error(stats.toString('errors-only'));
    }
    console.log(stats);
})
```
** 颗粒度太粗，看不出问题所在

速度分析： 使用speed-measure-webpack-plugin
代码示例
```js
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const smp = new SpeedMeasurePlugin();
const webpackConfig = smp.wrap({
    plugins: [
        new MyPugin(),
        new MyOtherPlugin()
    ]
});
```
可以看到每个loader和插件执行耗时

速度分析插件作用
分析整个打包总耗时
每个插件和loader的耗时情况

webpack-bundle-analyzer 分析体积
代码示例
```js
const BunldeAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
module.exports = { plugins: [new BundleAnalyzerPlugin()]}
```
构建完成后会在8888端口展示大小
可以分析哪些问题
* 依赖的第三方模块文件大小
* 业务里面的组件代码大小

使用高版本的webpack和Node.js
构建时间降低了60%-98%；

使用webpack4 优化原因
* V8带来的优化（for of替代forEach Map Set替代Object includes替代indexOf）
* 默认使用更快的md4 hash算法
* webpack AST可以直接从loader传递给AST，减少解析时间
* 使用字符串方法替代正则表达式

多进程/多实例构建：资源并行解析可选方案
thread-loader 可选方案 parallel-webpack HappyPack
多进程/多实例：使用HappyPack解析资源
原理：每次webpack 解析一个模块，HappyPack会将它及它的依赖分配给worker线程中
代码示例
```js
exports.plugins=[
    new HappyPack({
        id: 'jsx',
        threads: 4,
        loaders:['babel-loader']
    }),
    new HappyPack({
        id: 'styles',
        threads: 2,
        loaders: ['style-loader','css-loader','less-loader']
    })
]
```
多进程/多实例：使用thread-loader解析资源
 原理：每次webpack解析一个模块 thread-loader会将它及它的依赖分配给worker线程中
 ```js
 const json={
     "module.rules":[
         {
             test:/.js$/,
             use: [
                 {
                     loader:'thread-loader',
                     options:{
                         workers: 3
                     }
                 },
                 'babel-loader',
                 // 'eslint-loader'
             ]
         }
     ]
 }
 ```

多进程/多实例：并行压缩
方法一：使用parallel-uglify-plugin插件
```js
const ParallelUglifyPlugin=require('webpack-parallel-uglify-plugin');
module.exports={
    plugins:[
        new ParallelUglifyPlugin({
            uglifyJS:{
                output: {
                    beautify: false,
                    comments: false
                },
                compress:{
                    warnings: false,
                    drop_console: true,
                    collapse_vars: true,
                    reduce_vars: true
                }
            }
        })
    ]
}
```

并行压缩
方法二：uglifyjs-webpack-plugin 开启parallel参数
```js
const UglifyJsPlugin=require('uglifyjs-webpack-plugin');
module.exports={
    plugins:[
        new UglifyJsPlugin({
            uglifyOptions:{
                warnings: true,
                parse:{},
                compress: {},
                mangle: true,
                output: null,
                toplevel: false,
                nameCache: null,
                ie8: false,
                keep_fnames: false
            },
            parallel: true
        })
    ]
}
```
并行压缩
方法三：terser-webpack-plugin 开启 parallel参数
```js
const TerserPlugin=require('terser-webpack-plugin');
module.exports={
    optimization:{
        minimizer:[
            new TerserPlugin({
                parallel: 4
            })
        ]
    }
}
```

分包：设置Externals
思路：将 react、react-dom基础包 通过cdn引入，不打入bundle中
方法: 使用html-webpack-externals-plugin
```js
new HtmlWebpackExternalsPlugin({
    externals:[
        {
            module: 'react',
            entry: '//11.url.cn/now/lib/15.1.0/react-with-addons.min.js?v=1',
            global: 'React'
        },
        {
            module: 'react-dom'
            entry: '//11.url.cn/now/lib/15.1.0/react-dom.min.js?v=1',
            global: 'ReactDom'
        }
    ]
})
```

进一步设置分包：预编译资源模块
思路： 将react、react-dom、redux、react-redux基础包和业务基础包打成一个文件
方法：使用DLLPlugin 进行分包，DllReferencePlugin 对manifest.json引用
```js
const path=require('path');
const webpack=require('webpack');
module.exports={
    context: process.cwd(),
    resolve:{
        extensions: ['.js','.jsx','.json','.less','.css'],
        modules:[__dirname,'node_modules']
    },
    entry:{
        library:['react','react-dom','redux','react-redux']
    },
    output:{
        filename: '[name].dll.js',
        path: path.resolve(__dirname,'./build/library'),
        library: '[name]'
    },
    plugins:[
        new webpack.DllPlugin({
            name: '[name]',
            path: './build/library/[name].json'
        })
    ]
}
```
在webpack.config.js引入
```js
module.exports={
    plugins:[
        new webpack.DllReferencePlugin({
            manifest: require('./build/library/manifest.json')
        })
    ]
}
```
引用效果
`<script src="/build/library/library.dll.js"></script>`

缓存
目的：提升二次构建速度
缓存思路：
babel-loader 开启缓存
terser-webpack-plugin 开启缓存
使用cache-loader或者 hard-source-webpack-plugin

缩小构建目标
目的:尽可能的少构建模块
比如babel-loader不解析node_modules
`loader:'happypack/loader',exclude:'node_modules'`

减少文件搜索范围
优化 resolve.modules设置（减少模块搜索层级）
优化resolve.mainFields 配置
优化 resolve.extensions 配置
合理使用alias
```js
module.exports={
    resolve:{
        alias:{
            react: path.resolve(__dirname,'./node_modules/react/dist/react.min.js'),
        },
        modules: [path.resolve(__dirname,'node_modules')],
        extensions: ['.js'],
        mainFields: ['main']
    }
}
```

图片压缩
要求：基于Node库的imagemin或者tinypng API
使用：配置image-webpack-loader
```js
return {
    test: /\.(png|svg|jpg|gif|blob)$/,
    use:[
        {loader: 'file-loader',options:{ name: `${filename}img/[name]${hash}.[ext]`}}
        {loader: 'image-webpack-loader',options:{
            mozjpeg:{progresive: true,quality: 65},
            optipng:{enabled: false},
            pngquant:{quality:'65-90',speed:4},
            gifsicle:{interlaced:false},
            webp:{quality:75}
        }}
    ]
}
```

Imagemin优点分析
 有很多定制选项
 可以引入更多第三方优化插件 例如pngquant
 可以处理多种图片格式
 压缩原理
* pngquant:  是一款png压缩器，通过将图像转为具有alpha通道的更高效的8为PNG格式，可显著减少文件大小
* pngcrush： 其主要目的是通常尝试不同的压缩级别和PNG过滤方法来降低PNG IDAT数据流的大小
* optipng: 其设计灵感来自于pngcrush。optipng可将图像文件重新压缩为更小尺寸，而不是会丢失任何信息
* tinypng：也是将24位png文件转化为更小有索引的8位图片，同时所有非必要的metadata也会剥离掉

#### Tree  Shanking（复习
概念： 1个模块可能有多个方法，只要其中的某个方法使用到了，则整个文件都会被打到
bundle 里面去，tressshaking就是把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除掉

使用webpack默认支持，在.babelrc里设置modules: false 即可
.production mode情况下默认开启
要求：必须是ES6语法，CJS的方式不支持

无用的CSS如何删除掉
PurifyCSS便利代码，识别已经用到的CSS class
uncss： HTML需要通过jsdom加载，所有的样式通过PostCSS解析，通过
document.querySelector来识别html文件里面不存在的选择器

在webpack中如何使用PurifyCSS
使用purgress-webpack-plugin 和mini-css-extract-plugin配合使用
和mini-css-extract-plugin
```js
module.exports={
    plugins:[
        new MiniCssExtractPlugin({
            filename:'[name].css'
        }),
        new PurgecssPlugin({
            paths: glob.sync(`${PATH.src}/**/*`,{nodir: true`})
        })
    ]
}
```
构建体积优化动态polyfill
babel-polyfill 打包后体积88.49K!!!
Promise的浏览器支持情况
| 方案                           | 优点                               | 缺点                   | 是否采用 |
| ------------------------------ | ---------------------------------- | ---------------------- | -------- |
| babel-polyfill                 | React官方推荐                      | 包体积大               | x        |
| babel-plugin-transform-runtime | 能只polyfill到方法                 | 不能polyfill原型上魔法 | x        |
| 自己map，set的polyfill         | 定制化高                           | 积小小                 | x        |
| polyfill-service               | 只给用户回需要的polyfill，社区维护 | 部分国内UA挂           | ☑️        |

Polyfill Service 原理
识别UserAgent 下发不同的Polyfill

构建体积优化：如何动态PolyfillService
polyfill.io 官方提供的服务
`<script src="https://cdn.polyfill.io/v2/polyfill.min.js"></script>`

基于官方自建的polyfill服务

体积优化策略总结
* Scope Hoisting
* Tree Shaking
* 公用资源分离
* 图片压缩
* 动态polyfill

### 06 通过源码掌握webpack打包原理
开始： 从webpack命令行说起
通过npm scripts运行 webpack
* 开发环境 npm run dev
* 生产环境 npm run build

通过webpack直接运行
* webpack entry.js bundle.js
> 这个过程发生了什么

查找webpack入口文件
在命令行运行以上命令后，npm会让命令行工具 进入`node_modules/.bin`
目录查找是否存在webpack.sh或者webpack.cmd文件，如果存在，就执行
不存在则抛出错误
实际入口文件是: node_modules/webpack/in/webpack.js

分析webpack的入口文件: webpack.js
```js
process.exitCode = 0；
const runCommand=(command,args)=>{...};
const isInstalled=packagName => {...};
const CLIs=[...]
// webpack-command
const installcides=CLI.filter(cli=>cli.installed);
if (installedClis.length===0){...}else if
(installedClis.length===1)
```

启动后的结果
webpack最终找到 webpack-cli(webpack-command)这个 npm包
并且执行cli

webpack-cli做的事情
引入yargs，对命令进行定制
分析命令行参数，对各个参数进行转换，组成编译配置项
引用webpack，根据配置项进行编译和构建

从NON_COMPILATION_CMD分析出不需要编译的命令
webpck-cli 处理不需要经过编译的命令
```js
const { NON_COMPILATION_ARGS} = require('./utils/constants');
const NON_COMPILATION_CMD= process.argv.find(arg=>{
    if(arg === 'serve'){
        global.precess.argv=global.process.argv.filter(a=>a!=='serve');
        process.argv=global.process.argv;
    }
    return NON_COMPILATION_ARGS.find(a=>a===arg);
})
if(NON_COMPILATION_CMD){
    return require('./utils/prompt-command')(NON_COMPILATION_CMD,...process.argv);
}
```

_ARGS 的内容
init // 创建一份webpack配置文件
migrate // 进行webpack版本迁移 
add // 往webpack配置文件中增加属性
remove // 删除属性
serve // 运行webpack-serve
generate-loader // 生成loader代码   
generate-plugin // 生成plugin代码
info 返回与本地环境相关的一些信息

命令行工具包yargs介绍

动态生成help 帮助信息

webpack-cli使用args分析
参数分组(config/config-args.js),将命令划分为9类

config options 配置相关的参数 文件名称，运行环境等
basic options 基础参数 entry设置，debug模式，watch，devtool设置等
module options 模块参数，给loader 设置扩展
output options 输出参数（输出路径，输出文件名称
advacnded options 高级用法，记录设置，缓存设置，监听频率，bail等
resloving options 解析参数 alias和解析文件后缀 设置
optmization options 优化参数
status options 统计参数
options 通用参数 帮助命令 版本信息等

webpack-cli 执行结果
webpack-cli 对配置文件和命令行参数进行转换最终生成配置选项参数options
最终会根据配置参数实例化webpack对象，然后执行构建流程
webpack本质
webpack可以将其理解是一种基于事件流的编程规范，一系列的插件运行
先看一段代码
核心对象Compiler继承Tapable
class Compiler extends Tapable{
    //
}
核心对象Compilation集成Tapable{
    //
}
Tapable是什么？
Tapable是一个类似于Node.js的EventEmitter的库,主要是控制钩子的发布与订阅，控制这个webpack的插件系统
Tapable库暴露了很多Hooks
...

#### 模拟Compiler.js
```js
module.exports=class Compiler{
    constructor(){
        this.hoos={
            accelerate: new SyncHook(['newspped']),
            brake: new SyncHook(),
            calculateRoutes:new AsyncSeriesHook(['source','target','routesList'])
        }
    }
    run(){
        this.accerate(10)
        this.break()
        this.calculateRoutes('Async','hook','demo')
    }
    accelerate(speed){
        this.hooks.accelerate.call(speed);
    }
    break(){
        this.hooks.brake.call();
    }
    calculateRoutes(){
        this.hooks.calculateRoutes.promise(...arguments).then(()=>{},err=>{console.lerror(err)});
    }
}
```

插件 my-plugins.js
```js
const Compiler=require('./Compiler');
class MyPlugin{
    constructor(){}
    apply(compiler){
        compiler.hooks.brake.tap('WarningLampPlugin',()=>console.log('WarningLampPlugin'));
        compiler.hooks.accelerate.tap('LoggerPlugin',newSpeed=>console.log(`Accelerating to ${newSpeed}`));
        compiler.hooks.calculateRoutes.tapPromise('calculateRoutes tapAsync',(source,target,routesList)=>{
            return new Promise((resolve,reject)=>{
                setTimeout(()=>{
                    console.log(`tapPromise to ${source}${target}${routesList}`);
                    reslove();
                },1000)
            })
        })
    }
}
```
模拟插件执行
```js
const myPlugin =new MyPlugin();
const options={plugins: [myPlugin]};
const compiler=new Compiler();
for(const plugin of options.plugins){
    if(typeof plugin==='function'){
        plugin.call(compiler,compiler);
    }else{
        plugin.apply(compiler);
    }
}
compiler.run();
```

#### Webppack 流程篇
webpack的编译都按照下面的钩子调用顺序执行
*entry-option -> *run -> *make -> *before-resolve -> *build-module
-> *normal-module-loader -> *program -> *seal -> *emit
1. 初始化 option 
2. 开始编译
3. 从entry开始递归的分析依赖，对每个依赖模块进行build
4. 对模块位置进行解析
5. 开始构建某个模块
6. 将loader加载完成的module进行编译，生成AST树
7. 遍历AST树，当遇到require等一些调用表达式时，收集依赖
8. 所有依赖build完成，开始优化
9. 输出到dist目录

WebpackOptionsApply
将所有的配置options参数转换成 webpack内部插件
使用默认插件列表
举例:
* outpu.library->LibraryTemplatePlugin
* externals->ExternalsPlugin
* devtool->EvalDevtoolModulePlugin,SourceMapDevToolPlugin
* AMDPlugin,CommonJsPlugin
* RemoveEmptyChunksPlugin

Compiler Hooks
流程相关：
* before-)run
* before-/after-)compile
* make
* after-)emit
* done
监听相关:
* watch-run
* watch-close

Compilation
Compiler调用Compilation生命周期方法
* addEntry -> addModuleChain
* finish(上报模块错误)
* seal

ModuleFactory
* NormalModuleFactory
* ContextModuleFactory
Module
* NormalModule
* ContextModule(./src/*)
* ExternalModule(module.exports=jQuery)
* DelegatedModule(manifest)

NormalModule
Build
* 使用loader-runner运行loaders
* 通过Parse解析(内部是acron)
* ParsePlugins添加依赖

Compilation hooks
模块相关
* build-module
* failed-module
* succeed-module
优化相关：
* after-)seal
* optimize
* optimize-modules(-basic/advanced)
* after-optimize-modules
* after-optimize-chunks
* after-optimize-tree
* optimize-chunk-modules(-basic/advanced)

资源生成相关：
* module-asset
* chunk-asset

Chunk 生成算法
1. webpack先将entry中对应的module都生成一个新的chunk
2. 遍历module的依赖列表，将依赖的module也加入到chunk中
3. 如果一个依赖module是动态引入的模块，那么就会根据这个module创建一个新的chunk，继续遍历依赖
4. 重复上面的过程直至得到所有的chunks
   
模块化：增加代码可读性和维护性
传统的网页开发转变成WebApps开发
代码复杂度在逐步增高
分离的JS文件/模块，便于后续代码的维护性
部署时希望把代码优化成几个HTTP请求

常见的集中模块化方式
ES module
CJS
AMD

AST 基础知识
抽象语法树(abstract syntax tree),或者语法树(syntax tree),是源代码的抽象语法结构的树状表现形式,这里特指编程语言的源代码，树上的每个节点都表示源代码中的一种结构

#### 复习一下webpack的模块机制
```js
(function(modules){
    var installedModules={};

    function __webpack_require__(moduleId){
        if(installedModules[moduleId]){
            return installedModules[moduleId].exports;
        }
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        modules[moduleId].call(module.exports,module,module.exports,__webpack_require__);
        module.l=true;
        return module.exports;
    }
    __webpack_require__(0);
})([
    /* 0 module */
    (function(module,__webpack_exports__,__webpack_require__){
        // ...
    },
    /* 1 module */

])
```
* 打包出来的是一个 IIFE（匿名闭包)
* modules 是一个数组，每一项是一个模块初始化函数
* __webpack_require用来加载模块，返回module.exports
* 通过WEBPACK_REQUIRE_METHOD(0) 启动程序

动手实现一个简易的webpack
1. 可以将ES6语法转换成ES5的语法
   * 通过babylon 生成AST
   * 通过babel-core将AST重新生成源码
2. 可以分析模块之间的依赖关系
   * 通过babel-traverse 的 ImportDeclaration方法获取依赖属性
3. 生成的JS文件可以在浏览器中运行


### 07 编写Loader插件
定义: loader只是一个导出为函数的javascript模块
`module.exports=function(source){ return source}`

多loader时的执行顺序
多个loader串行执行 顺序从后到前
```js
module.exports={
//...
    module:{
        rules:[
            {test: /\.less$/,use:['style-loader','css-loader','less-loader']}
        ]
    }
}
```

函数组合的两种情况
Unit中的pipline
Compose（webpack采取的是这种
`compose=(f,g)=>(...args)=>f(g(...args));`

通过一个例子验证 loader的执行顺序
a-loader.js
export=function(source){console.log('loader a is executed'); return source;}
b-loader.js
export=function(source){console.log('loader b is executed'); return source;}

7. loader-runner介绍
定义：loader-runner允许你在不安装webpack的情况下运行loaders
作用： 
* 作为webpack的依赖，webpack中使用它执行loader
* 进行loader的开发和调试

8. loader-runner的使用
```js
import {runLoaders} from 'loader-runner';
runLoaders({
    resource: '/abs/path/to/file.txt?query', // String: 资源的绝对路径 可加query
    loaders: ['/abs/path/to/loader.js?query'],// String[]:loader的绝对路径 可加query
    context: {minimize:true}, // 基础上下文之外额外的loader上下文
    readResource: fs.readFile.bind(fs)// 读取资源的函数
},function(err,result){
    // error: Error?
    // result.result: Buffer|String
})
```
9. 开发一个raw-loader
```js
// src/raw-loader.js
module.exports=function(source){
    const json=JSON.stringify(source)
    .replace(/\u2028/g,'\\u2028')
    .replace(/\u2029/g,'\\u2029') // 为了安全起见，ES模板字符串的问题
    return `export default ${json}`;
};

// src/demo.txt
foobar

```
10. 使用loader-runner调试loader
run-loader.js
```js
const fs=require('fs');
const path=require('path');
const {runLoaders}= require('loader-runner');
runLoaders({
    resource: './demo.txt',
    loaders: [path.resolve(__dirname,'./loaders/raw-loader')],
    readResource: fs.readFile.bind(fs)
},(err,result)=>(err?console.error(err):console.log(result))
);
```
查看运行结果： node run-loader.js

11. loader的参数获取
通过 loader-utils的getOptions方法获取
```js
const loaderUtils=require('loader-utils');
module.exports=function(content){
    const {name}=loaderUtils.getOptions(this);
}
```

12. loader的异常处理

loader内置直接通过throw抛出
通过this.callback传递错误
`this.callback(error: Error|null,content: string|Buffer,sourceMap?:SourceMap,meta?: any);`
13. loader的异步处理
通过this.async 来返回一个异步函数
* 第一个参数是Error，第二个参数是处理的结果
```js
module.exports=function(input){
    const callback=this.async();
    // No callback=> return synchronous results
    // if(callback) 
    callback(null,input+input);
}
```

13. 在loader中使用缓存
webpack中默认开启loader缓存
*  可以使用 this.cacheable(false) 关掉缓存
缓存条件: loader的结果在相同的输入下有确定的输出
* 有依赖的loader无法使用缓存
  
14. loader如何进行文件输出?

通过this.emitFile进行写入
```js
const loaderUtils=require('loader-utils');
module.exports=funciton(content){
    const url=loaderUtils.interpolateName(this,'[hash].[ext]',{content});
    this.emitFile(url,content);
    const path= `__webpack_public_path__+ ${JSON.stringify(url)};`;
    return `export default ${path}`;
}
```

#### 实战开发一个自动合成雪碧图的loader
支持的语法:
1. `background: url('a.png?__sprite');`
2. `background: url('b.png?__sprite');`
---> background: url('sprite.png');
准备知识: 如何将两张图片合成一张图片?
使用 spritesmith(https://www.npmjs.com/package/spritesmith)

spritesmith使用示例  
```js
const sprites=['./images/1.jpg','./images/2.jpg'];
Spritesmith.run({src: sprites},function handleResult(err,result){
    // result.image;
    // result.coordinates;
    // result.properties;
})
```

#### 插件的运行环境
18. 插件没有像loader那样的独立运行环境
只能在webpack里面运行

19. 插件的基本结构 