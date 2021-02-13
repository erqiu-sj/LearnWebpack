<!--
 * @Author: 邱狮杰
 * @Date: 2021-01-16 20:20:57
 * @LastEditTime: 2021-02-13 22:59:46
 * @FilePath: /webpack/optimizationConfig.md
 * @Description: 描述
-->

# 性能优化

# HMR

> 作用：一个模块修改了只重新打包一个模块，必须要将所有模块重新打包，极大提升构建速度
> 注意，入口文件是不能做 HRM 功能的，因为他会从入口出发寻找依赖，一旦入口文件发生变化，会重新寻找依赖文件，所以建议在非入口文件做 HMR

```javascript
// 样式可以HMR 由style-loader实现
// js文件默认没有HMR
// html不能实现HMR，则会导致HTML不能热更新了
// 解决html不能热更新,将entry改为数组，将html引入
// html不需要做HMR功能，热更新就够了，因为通常HTML文件只有一个
entry:['./index.js',"./index.html"],
devServer: {
  // 开启HMR
  hot: true;
}
```

```javascript
// in index.js
// webpack会全局找module上是否由hot属性
if (module.hot) {
  // 有hot属性，意味着开启了HMR功能，让HMR功能代码生效
  module.hot.accept("./parint.js", () => {
    // 监听parint文件，一旦发生变化会执行这段回调函数
    // 一旦发生变化其他文件不会重新打包。
    // 以此类推,需要自己定义监听
  });
}
```

# source-map

> 提供了一种源代码到构建后代码的映射技术（如果构建代码出错，会通过映射关键追踪到源代码的错误）

> option:`[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map`
> inline-xx 内联：构建速度更快，将 map 文件生成在 js 内,只生成一个内联 map，能找到错误位置，也能知道原因

> hidden 外链：在外部生成 map 文件不会在 js 内部，能找到错误原因，不能找到错误位置

> eval 内联，生成在 buildjs 内部，每段 js 都会生成一段 map 被 eval 包裹，而 inline 只生成一段的，原因位置都能找到

> 剩余选项都为外部生成 map 文件

> nosources 能找到源代码错误信息，位置，但是不能点击跳转到对应位置

> hidden-nosources 都是为了隐藏源代码避免一些被人发现攻击

> cheap 错误位置都能在到，但是会提示整行报错，soucres-map 会提示关键地方报错

> cheap-module 同上

```javascript
devtool: "source-map";
```

开发环境：速度快，调试友好

> 速度快（eval>inline>cheap...）`eval-cheap-sources-map`,`eval-sources-map`

> 调试友好`sources-map`,`cheap-module-sources-map`,`cheap-sources-map`

> 推荐：`eval-sources-map`|`eval-cheap-module-sources-map`

生产环境：源代码隐藏？调试友好吗？思考题

# oneOf

> 正常来说一个文件只能被一个 `loader` 处理，因为每个文件都会被多个 `loader` 过一遍,但匹配规则过多，则会降低匹配性能，所以建议使用`oneOf`提高效率

```javascript
module: {
  rules: [
    {
      oneOf: [
        // 在这写下规则，则会执行一下一个loader
        // 注意有两项配置处理同一个文件，如果非得执行，需要将对应的loader提取到oneOf外部
        {
          test: /\.css$/,
        },
      ],
    },
  ];
}
```

# tree shaking

> 去除无用的 js 代码 orcss 代码，**_前提_**：必须使用 es6 模块化，开启`production`自动开启`tree shaking`

```javascript
// in packge.json
sideEffects: false; //  表示所有代码都无副作用(都可以进行tree shaking),问题是可能把css\babal(文件)忽略打包
sideEffects: ["*.css"]; // 标记不要进行tree shaking 的资源
```

# code split

```javascript
/**
 可以将node_modules的代码打包成一个chunk最终输出
 import $ from 'jquery'
 console.log($)
 // 该配置这样会将jquery单独打包称为一个chunk，如果没有该配置则会讲jquery和自己的js代码打包称为一个chunk，从而没有实现代码分割的效果
 当多个文件引用同一个文件时也会只引用一个文件chunk


  如果使用单文件入口也想使用代码分割，则需要改写引入依赖的写法
  import(／* webpackChunkName：'test'    *／"filename").then().catch() // 即可分割
*/

optimization: {
  splitChunks: {
    chunks: "all";
  }
}
```

# 懒加载和预加载

```javascript
function fn() {
  // 懒加载,并不会重新加载，第二次加载时则会读取缓存，上来不会加载也不会执行，并行加载
  // 预加载prefetch,一上来文件会被加载，但不会被执行，等其他资源加载完毕浏览器空闲后在加载
  import(/* webpackPrefetch:true  */ "./test.js").then().catch();
}
```

# 多进程打包

```javascript
// npm i thread-loader -D
/**
有利有弊，进程启动大概都是600ms,一般只在大工程中，工作消耗时间较长的情况下使用,一般使用在babel上
use:[
  "thread-loader",
  {
    loader:"babel-loader"
  }
]
*/
```

# externals

```javascript
//当我们忽略打包的时候需要手动cdn引入进来
externals: {
  //忽略包名 ---- npm下的包名
  jquery: "jQuery";
}
```

# dll
