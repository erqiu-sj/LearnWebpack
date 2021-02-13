<!--
 * @Author: 邱狮杰
 * @Date: 2021-01-10 16:48:37
 * @LastEditTime: 2021-02-13 21:59:44
 * @FilePath: /webpack/webpackConfigDetails.md
 * @Description: 描述
-->

# webpack 详细配置

# entry 入口

- `string` 单入口 `entry file path`=>会形成一个 `chunk`,输出一个 `bundle(main.js)`

- `array` 多入口 `[filePath,filePath]`=>会形成一个 `chunk`,输出一个 `bundle(main.js)`只有在`HMR`中让`html`热更新生效

- `object` 多入口 `{index:filePath,add:filePath}`=>有几个入口文件就会形成几个`chunk`输出几个`bundle`文件，此时`chunk`文件名为`key`

# output 输出

- `filename`: 可以指定打包文件`[main].js`，也可以指定打包目录`js/[main].[contenthash:10].js`

- `path`:将来所有输出的公共目录,输出文件路径`path.resolve(__dirname,'build')`

- `publicPath`: 所有输出资源引入的公共路径=>路径前面的～，一般适用与生产环境，默认"/",也就是`img/img.png=>/img/img.png`

- `chunkFilename`:`[name]_chunk.js`// 非入口 chunk 的文件名
- `libary`:`[name]`,因为打包后的代码是在一个函数作用于下，外部想要应用是不行的，所以需要向外到处一个[name],默认为 `main`,通过 `var` 挂载调用

- `libaryTarget`:`window`指定挂载到哪，`node`下挂载到`global`,也可以是`commonjs`这表示我想用`commonjs`的语法去引用它

# module

## rules

```javascript
module: {
  rules: [
    {
      // 匹配规则
      test: /\.css$/,
      usr: ["style-loader", "css-loader"], // 多个loader用use
      loader: "", //单个loader用loader
      // 排除文件
      exculude: /node_modules/,
      // 只检查src下的文件
      include: resolve(__dirname, "src"),
      enforce: "pre", // pre=优先执行，post=延后执行，默认中间执行规则
      options: {
        //表示8kb以下的图片会被base64处理
        limit: 8 * 1024,
        //图片名太长了，会使用该配置，表示文件名取前10位保留图片后缀
        name: "[hash:10].[ext]",
        // 关闭es6模块化开启commonJs，一般在引入了html-loader后配置该属性
        esModule: false,
        // 指定匹配的文件输出到哪个目录
        outputPath: "imgs",
      },
    },
    {
      // 以下配置只会生效一个
      oneOf: [],
    },
  ];
}
```

# resolve

**_解析模块规则_**

```javascript
resolve: {
  // 解析路径别名
  alias: {
    aliasName: resolve(__dirname, path);
  },
  // 配置可忽略的文件后缀
  extensions:[".js",".json"]
  // 解析文件模块去找哪个目录，如果当前目录没找到会向上查找
  modules:["node_modeuls",resolve(path)]
}
```

# devServer

```javascript
devServer: {
  // 运行代码的目录
  contentBase: resolve(__dirname, "build"),
  // 监视运行代码的目录是否更新，更新就会重载
  watchContentBase:true
  // 忽略文件
  watchOptions:{
    ignored:"/node_modules/"
  }
  // 开启gzip压缩
  compress:true
  post:number
  host:string
  // 自动打开浏览器
  open:true
  // 开启HMR功能
  hot:true
  //除了基本的启动信息之外的信息都不要log
  quiet:true
  // 不要显示启动服务器日志信息
  clientLogLevel:true
  // 如果报错不要全屏提示
  overlay:false
}
```

# mode

- `production`
  > 能够让代码优化上线的配置
- `development`
  > 让代码本地调试运行的配置

# 构建环境配置

## 提取 css 成单独文件

```javascript
// npm i mini-css-extract-plugin -D
// 复习style-loader=创建style标签，将样式放入
// css-loader=将css文件整合在js中
// 如果需要提取成单独文件需要将style-loader替换成位MiniCssExtractPlugin.loader=>提取js中的css成为单独文件
plugin: [
  new MiniCssExtractPlugin({
    // 重命名
    // static/css/[name].[chunkhash:8].css
    filename: "css/build.css",
  }),
];
```

## css 兼容性处理

```javascript
//需要设置node环境变量
// npm i postcss-loader postcss-preset-env -D
process.env.NODE_ENV = "development";
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        MiniCssExtractPlugin.loader,
        "css-loader",
        {
          loader: "postcss-loader",
          options: {
            // 固定写法
            ident: "postcss",
            //帮postcss找到package.json中browserslist里的配置
            //通过配置加载css指定的兼容性样式
            // or [require('postcss-perset-env')()]
            plugin: () => [require("postcss-preset-env")()],
          },
        },
      ],
    },
  ];
}
```

```javascript
// in package.json
// 可在github上搜browserslist
// 默认生成环境需要设置nodejs环境变量
browserslist: {
  development: [
    // 兼容最近版本的chrmoe配置
    //兼容主要浏览器版本就足够
    "last 1 chrome version",
    "last 1 ie version",
  ];
  production: [
    // 大于百分之9.8的浏览器
    ">0.2%",
    // 不要已经死了的浏览器
    "not dead",
    // 不要open_mini 的浏览器
    "not op_mini all",
  ];
}
```

## 压缩 Css

```javascript
// npm i optimize-css-assets-webpack-plugin -D
plugin: [new OptimizeCssAssetsWebpackPlugin()];
```

## js 语法检查 eslint

```javascript
// npm i eslint-loader eslint
// 需要设置检查规则
module: {
  rules: [
    {
      test: /\.js$/,
      loader: "eslint-loader",
      exculude: /node_modules/,
      options: {
        //自动修复eslint错误
        fix: true,
      },
    },
  ];
}
```

```javascript
//  in package.json
// airbnb -> eslint-config-airbnb-base eslint eslint-plugin-import
// airbnb推荐的代码风格,在github上能搜索到
eslintConfig: {
  extends:"airbnb-base"
}
```

## js 兼容性处理 eslint

## 压缩 js and html

```javascript
// 压缩html
// npm i -D html-webpack-plugin
plugin: [
  new HtmlWebpackPlugin({
    template: "./index.html",
    minify: {
      // 移除空格
      collapseWhitespace: true,
      // 删除注释
      removeComments: true,
    },
  }),
];
// 压缩js
mode: "production";
```
