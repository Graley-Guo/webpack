# webpack基础详解
## 0、目的地

1. webpack的意义
2. webpack的安装和使用
3. webpack的配置
   - 入口
   - 输出
   - loader
   - plugin
   - resolve 代码解析
   - devServer
4. 提升webpack构建速度
5. webpack4.X性能优化
  


## 1、webpack的产生和意义

webpack 本质上是一个打包工具，它会根据代码的内容解析模块依赖，帮助我们把多个模块的代码打包。借用 webpack 官网的图片： ![](https://user-gold-cdn.xitu.io/2018/7/23/164c5106ffe2c72d?w=1280&h=506&f=webp&s=11674)
打包目的：减少代码体积，减少网络请求。

## 2、webpack 的安装和使用
### npm、yarn是什么
npm（node package manager）node的包管理工具。

举例来说： 如果我们在开发过程中使用jquery，那么是不是要引入jquery，你可能会下载这个jquery.js文件，然后在代码中<script src="jquery.js"></script>是吧；
 如果使用 `npm`，那么就方便了，
  
 直接在npm下使用命令：`$ npm install jquery`就自动下载了；

在远端有一个`npm`服务器，里面有很多别人写的代码，我们可以直接使用npm下载使用；同时你也可以把自己写的代码推送到npm 服务器，让别人使用；

Yarn 对代码来说是一个包管理器， 你可以通过它使用全世界开发者的代码，或者分享自己的代码。

### webpack 安装
我们使用 npm 或者 yarn 来安装webpack ，可以作为一个全局的命令来使用：

```shell
npm install webpack webpack-cli -g

# 或者
yarn global add webpack webpack-cli

# 然后就可以全局执行命令了
webpack --help
```

`webpack-cli` 是使用 webpack 的命令行工具，在 4.x 版本之后不再作为 webpack 的依赖了，我们使用时需要单独安装这个工具。

在项目中，我们更多地会把 webpack 作为项目的开发依赖来安装使用，这样可以指定项目中使用的 webpack 版本，更加方便多人协同开发：

> 确保你的项目中有 package.json 文件，如果没有可以使用 npm init 来创建。

```shell
npm install webpack -D //devDependencies用于本地环境开发时候，

# 或者
yarn add webpack -D
```

这样 webpack 会出现在 package.json 中，我们再添加一个 npm scripts：允许在 package.json 文件里面使用 scripts 字段定义任务：

```json
"scripts": {
    "build": "webpack --mode production"
  },
  "devDependencies": {
    "webpack": "^4.1.1",
    "webpack-cli": "^2.0.12",
  }
```

然后我们创建一个 `./src/index.js` 文件，可以写任意的 JS 代码。创建好了之后执行`npm run build` 或者`yarn build`命令，你就会发现新增了一个 dist 目录，里边存放的是 webpack 构建好的 `main.js` 文件。

`webpack 4.x` 的版本可以零配置就开始进行构建，但是个人觉得这个功能还不全面，缺少很多实际项目需要的功能，所以基本你还是需要一个配置文件。

## 3、一个简单的webpack配置
webpack 运行时默认读取项目下的 `webpack.config.js` 文件作为配置。
所以我们在项目中创建一个 `webpack.config.js` 文件：
【例子一】

创建了 webpack.config.js 后再执行 webpack 命令，webpack 就会使用这个配置文件的配置了。

## 4、入口

如上图所示，在多个代码模块中会有一个起始的 `.js` 文件，这个便是 webpack 构建的入口。webpack 会读取这个文件，并从它开始解析依赖，然后进行打包。如图，一开始我们使用 webpack 构建时，默认的入口文件就是 `./src/index.js`。

入口可以使用 `entry` 字段来进行配置，webpack 支持配置多个入口来进行构建：

```javascript
module.exports = {
  entry: './src/index.js'
  /*
  entry: {
    main: './src/index.js'
  }
  */
}

// 或者配置多个入口
module.exports = {
  entry: {
    foo: './src/page-foo.js',
    bar: './src/page-bar.js',
    // ...
  }
}

// 使用数组来对多个文件进行打包，可以理解为多个文件作为一个入口，
//webpack 会解析两个文件的依赖后进行打包
module.exports = {
  entry: {
    main: [
      './src/foo.js',
      './src/bar.js'
    ]
  }
}
```

## 5、输出

webpack 的输出即指 webpack 最终构建出来的静态文件，可以看看上面 webpack 官方图片右侧的那些文件。当然，构建结果的文件名、路径等都是可以配置的，使用 output 字段：

```javascript
module.exports = {
  // ...
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
}

// 或者多个入口生成不同文件
module.exports = {
  entry: {
    foo: './src/foo.js',
    bar: './src/bar.js',
  },
  output: {
    filename: '[hash].[name].[id].js',
    path: __dirname + '/dist',
  },
}

// 路径中使用 hash，每次构建时会有一个不同 hash 值，避免发布新版本时线上使用浏览器缓存
module.exports = {
  // ...
  output: {
    filename: '[name].js',
    path: __dirname + '/dist/[hash]',
  },
}
```

我们一开始直接使用 webpack 构建时，默认创建的输出内容就是 ./dist/main.js。

## 6、loader

webpack 中提供一种处理多种文件格式的机制，便是使用 loader。我们可以把 loader 理解为是一个转换器，负责把某种文件格式的内容转换成 webpack 可以支持打包的模块。把不同格式的文件都解析成 js 代码，以便打包后在浏览器中运行。
当我们需要使用不同的 loader 来解析处理不同类型的文件时，我们可以在 `module.rules` 字段下来配置相关的规则，
先来看一个基础的例子：

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.jsx?/, // 条件
        include: [
          path.resolve(__dirname, 'src'),
        ], // 条件
        use: 'babel-loader', // 规则应用结果
      }, // 一个 object 即一条规则
      // ...
    ],
  },
}
```

loader 的匹配规则中有两个最关键的因素：一个是匹配条件，一个是匹配规则后的应用。
### 	匹配条件
webpack 的规则提供了多种配置形式：

- { test: ... } 匹配特定条件
- { include: ... } 匹配特定路径
- { exclude: ... } 排除特定路径
- { and: [...] }必须匹配数组中所有条件
- { or: [...] } 匹配数组中任意一个条件
- { not: [...] } 排除匹配数组中所有条件

上述的所谓条件的值可以是：

- 字符串：必须以提供的字符串开始，所以是字符串的话，这里我们需要提供绝对路径
- 正则表达式：调用正则的 test 方法来判断匹配
- 函数：(path) => boolean，返回 true 表示匹配
- 数组：至少包含一个条件的数组
- 对象：匹配所有属性值的条件 通过例子来帮助理解：

```javascript
rules: [
  {
    test: /\.jsx?/, // 正则
    include: [
      path.resolve(__dirname, 'src'), // 字符串，注意是绝对路径
    ], // 数组
    // ...
  },
  {
    test: {
      js: /\.js/,
      jsx: /\.jsx/,
    }, // 对象，不建议使用
    not: [
      (value) => { /* ... */ return true; }, // 函数，通常需要高度自定义时才会使用
    ],
  },
],
```

上述多个配置形式结合起来就能够基本满足各种各样的构建场景了，通常我们会结合使用 `test/and` 和 `include&exclude` 来配置条件，如上述那个简单的例子。

### 匹配后的应用

```javascript
rules: [
  {
    test: /\.less/,
    use: [
      'style-loader', // 直接使用字符串表示 loader
      {
        loader: 'css-loader',
        options: {
          importLoaders: 1
        },
      }, // 用对象表示 loader，可以传递 loader 配置等
      {
        loader: 'less-loader',
        options: {
          noIeCompat: true
        }, // 传递 loader 配置
      },
    ],
  },
],
```

我们看下上述的例子，先忽略 loader 的使用情况，单纯看看如何配置。use 字段可以是一个数组，也可以是一个字符串或者表示 `loader` 的对象。
我们还可以使用 options 给对应的 `loader` 传递一些配置项，这里不再展开。当你使用一些 loader 时，loader 的说明一般都有相关配置的描述。


#### 构建 CSS

我们编写 CSS，并且希望使用 webpack 来进行构建，为此，需要在配置中引入 loader 来解析和处理 CSS 文件：

```javascript
module.exports = {
  module: {
    rules: [
      // ...
      {
        test: /\.css/,
        include: [
          path.resolve(__dirname, 'src'),
        ],
        use: [
          'style-loader',
          'css-loader',
        ],
      },
    ],
  }
}
```

> style-loader 和 css-loader 都是单独的 node package，需要安装。

我们创建一个 index.css 文件，并在 index.js 中引用它，然后进行构建。

```javascript
import "./index.css"
```

可以发现，构建出来的文件并没有 CSS，先来看一下新增两个 loader 的作用：

- css-loader 负责解析 CSS 代码，主要是为了处理 CSS 中的依赖，例如 @import 和 url() 等引用外部文件的声明；
- style-loader 会将 css-loader 解析的结果转变成 JS 代码，运行时动态插入 style 标签来让 CSS 代码生效。

经由上述两个 loader 的处理后，CSS 代码会转变为 JS，和 index.js 一起打包了。如果需要单独把 CSS 文件分离出来，我们需要使用 `extract-text-webpack-plugin` 插件。

`extract-text-webpack-plugin` 这个插件不支持 webpack 4.x 的正式版本，所以安装的时候需要指定使用它的 alpha 版本：`npm install extract-text-webpack-plugin@next -D` 或者 `yarn add extract-text-webpack-plugin@next -D`。如果你用的是 webpack 3.x 版本，直接用 `extract-text-webpack-plugin` 现有的版本即可。不过针对webpack4.x可以使用 `mini-css-extract-plugin`。

看一个简单的例子：

```javascript
const ExtractTextPlugin = require('extract-text-webpack-plugin')

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        // 因为这个插件需要干涉模块转换的内容，所以需要使用它对应的 loader
        use: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: 'css-loader',
        }),
      },
    ],
  },
  plugins: [
    // 引入插件，配置文件名，这里同样可以使用 [hash]
    new ExtractTextPlugin('index.css'),
  ],
}
```
#### 常用loader
![](https://raw.githubusercontent.com/Graley-Guo/webpack/master/img/%E5%B8%B8%E7%94%A8loader.png)


### loader 应用顺序

前面提到，一个匹配规则中可以配置使用多个 loader，即一个模块文件可以经过多个 loader 的转换处理，执行顺序是从最后配置的 loader 开始，一步步往前。例如，对于上面的 `less` 规则配置，一个 style.less 文件会途径 less-loader、css-loader、style-loader 处理，成为一个可以打包的模块。

上述从后到前的顺序是在同一个 rule 中进行的，那如果多个 rule 匹配了同一个模块文件，loader 的应用顺序又是怎样的呢？看一份这样的配置：

```javascript
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "eslint-loader",
  },
  {
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
  },
],
```

这样无法保证 eslint-loader 在 babel-loader 应用前执行。webpack 在 `rules` 中提供了一个 `enforce` 的字段来配置当前 rule 的 loader 类型，没配置的话是普通类型，我们可以配置 `pre` 或 `post`，分别对应前置类型或后置类型的 loader。

> eslint-loader 要检查的是人工编写的代码，如果在 babel-loader 之后使用，那么检查的是 Babel 转换后的代码，所以必须在 babel-loader 处理之前使用。

还有一种行内 loader，即我们在应用代码中引用依赖时直接声明使用的 loader，如 `const json = require('json-loader!./file.json')` 这种。不建议在应用开发中使用这种 loader，后续我们还会再提到。

顾名思义，所有的 loader 按照**前置 -> 行内 -> 普通 -> 后置**的顺序执行。所以当我们要确保 eslint-loader 在 babel-loader 之前执行时，可以如下添加 `enforce` 配置：

```javascript
rules: [
  {
    enforce: 'pre', // 指定为前置类型
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "eslint-loader",
  },
]
```

当项目文件类型和应用的 loader 不是特别复杂的时候，通常建议把要应用的同一类型 loader 都写在同一个匹配规则中，这样更好维护和控制。

## 7、plugin 

在 webpack 的构建流程中，plugin 用于处理更多其他的一些构建任务。可以这么理解，模块代码转换的工作由 loader 来处理，除此之外的其他任何工作都可以交由 plugin 来完成。它们在 webpack 中的配置都只是把插件实例添加到 plugins 字段的数组中。不过由于需要提供不同的功能，不同的插件本身的配置比较多样化。

下面通过介绍几个常用的插件来了解插件的使用方法。

### uglifyjs-webpack-plugin
要使用压缩 JS 代码的 uglifyjs-webpack-plugin 插件，只需在配置中通过 plugins 字段添加新的 plugin 即可：

```javascript
const UglifyPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  plugins: [
    new UglifyPlugin()
  ],
}
```
### 常用plugin
![](https://raw.githubusercontent.com/Graley-Guo/webpack/master/img/%E5%B8%B8%E7%94%A8plugin.png)

## 8、webpack 如何解析代码模块路径

在 webpack 支持的前端代码模块化中，我们可以使用类似 `import * as m from './index.js'` 来引用代码模块 `index.js`。

引用第三方类库则是像这样：`import React from 'react'`。webpack 构建的时候，会解析依赖后，然后再去加载依赖的模块文件，那么 webpack 如何将上述编写的 `./index.js` 或 `react` 解析成对应的模块文件路径呢

### 模块解析规则

我们简单整理一下基本的模块解析规则，以便更好地理解后续 webpack 的一些配置会产生的影响。

- 解析相对路径

  1、查找相对当前模块的路径下是否有对应文件或文件夹

  2、是文件则直接加载

  3、是文件夹则继续查找文件夹下的 package.json 文件

  4、有 package.json 文件则按照文件中 `main` 字段的文件名来查找文件

  5、无 package.json 或者无 `main` 字段则查找 `index.js` 文件

- 解析模块名

  查找当前文件目录下，父级目录及以上目录下的 `node_modules` 文件夹，看是否有对应名称的模块

- 解析绝对路径（不建议使用）

  直接查找对应路径的文件

在 webpack 配置中，和模块路径解析相关的配置都在 `resolve` 字段下：

```javascript
module.exports = {
  resolve: {
    // ...
  }
}
```

webpack 可以支持解析路径规则的自定义配置。

#### `resolve.alias`

假设我们有个 `utils` 模块极其常用，经常编写相对路径很麻烦，希望可以直接 `import 'utils'` 来引用，那么我们可以配置某个模块的别名，如：

```javascript
alias: {
  utils: path.resolve(__dirname, 'src/utils') // 这里使用 path.resolve 和 __dirname 来获取绝对路径
}
```

上述的配置是模糊匹配，意味着只要模块路径中携带了 `utils` 就可以被替换掉，如：

```javascript
import 'utils/query.js' // 等同于 import '[项目绝对路径]/src/utils/query.js'
```

如果需要进行精确匹配可以使用：

```javascript
alias: {
  utils$: path.resolve(__dirname, 'src/utils') // 只会匹配 import 'utils'
}
```

一般我们使用的时候，我们也不会去修改。更多配置可以参考官方文档 [Resolve](https://webpack.docschina.org/configuration/resolve)，这里不一一举例说明。


### 9、devServer

#### 为什么要使用devServer？
在开发模式下，DevServer 提供虚拟服务器，让我们进行开发和调试，而且提供实时重新加载。他没有进行写的过程，即没有重新生成dist文件。如果我们重新build，会执行写的过程，会特别慢。大大减少开发时间。DevServer 会把 Webpack 构建出的文件保存在内存中，在要访问输出的文件时，必须通过 HTTP 服务访问。

他的安装

```shell
npm i -D webpack-dev-server
```
安装成功后执行 webpack-dev-server 命令， DevServer 就启动了。

几个常用配置：
#### hot
`devServer. hot `配置是否启用模块热替换功能。 DevServer 默认的行为是在发现源代码被更新后会通过自动刷新`整个页面`来做到实时预览，开启模块热替换功能后将在不刷新整个页面的情况下通过用新模块替换老模块来做到实时预览。
#### port
`devServer.port`配置项用于配置 DevServer 服务监听的端口，默认使用 8080 端口。 如果 8080 端口已经被其它程序占有就使用 8081...
#### https
`DevServer` 默认使用 `HTTP 协议服务`，它也能通过 `HTTPS 协议服务`。 有些情况下你必须使用 HTTPS 服务，最简单的方式是：

```javascript
devServer:{
  https: true
}
```
#### compress
`devServer.compress` 配置是否启用 gzip 压缩。boolean 为类型，默认为 false.

想要了解更多的`devServer`的配置，可参照官网[devServer](https://webpack.docschina.org/configuration/dev-server/)

#### 开启静态服务

我们可以使用 `webpack-dev-server` 在本地开启一个简单的静态服务来进行开发。

在项目下安装 webpack-dev-server，然后添加启动命令到 package.json 中：

```json
"scripts": {
  "build": "webpack --mode production",
  "start": "webpack-dev-server --mode development"
}
```

尝试着运行 npm run start 或者 yarn start，然后就可以访问 <http://localhost:8080/> 来查看你的页面了。默认是访问 index.html，如果是其他页面要注意访问的 URL 是否正确。



## 10、提升 webpack 的构建速度

我们的前端项目随着时间推移和业务发展，页面可能会越来越多，或者功能和业务代码会越来越多，又或者依赖的外部类库会越来越多，这个时候原本不足为道的 webpack 构建时间消耗就会慢慢地进入我们的视野。

### 让 webpack 少干点活

提升 webpack 构建速度本质上就是想办法让 webpack 少干点活，活少了速度自然快了，尽量避免 webpack 去做一些不必要的事情。

#### `减少 resolve 的解析`

如果我们可以精简 `resolve` 配置，让 webpack 在查询模块路径时尽可能快速地定位到需要的模块，不做额外的查询工作，那么 webpack 的构建速度也会快一些，下面举个例子，介绍如何在 `resolve` 这一块做优化：

```javascript
resolve: {
  modules: [
    path.resolve(__dirname, 'node_modules'), // 使用绝对路径指定 node_modules，不做过多查询
  ],

  // 删除不必要的后缀自动补全，少了文件后缀的自动匹配，即减少了文件路径查询的工作
  // 其他文件可以在编码时指定后缀，如 import('./index.scss')
  extensions: [".js"],

  // 避免新增默认文件，编码时使用详细的文件路径，代码会更容易解读，也有益于提高构建速度
  mainFiles: ['index'],
},
```

上述是可以从配置 `resolve` 下手提升 webpack 构建速度的配置例子。

我们在编码时，如果是使用我们自己本地的代码模块，尽可能编写完整的路径，避免使用目录名，如：`import './lib/slider/index.js'`，这样的代码既清晰易懂，webpack 也不用去多次查询来确定使用哪个文件，一步到位。

#### `把 loader 应用的文件范围缩小`

我们在使用 loader 的时候，尽可能把 loader 应用的文件范围缩小，只在最少数必须的代码模块中去使用必要的 loader，例如 node_modules 目录下的其他依赖类库文件，基本就是直接编译好可用的代码，无须再经过 loader 处理了：

```javascript
rules: [
  {
    test: /\.jsx?/,
    include: [
      path.resolve(__dirname, 'src'),
      // 限定只在 src 目录下的 js/jsx 文件需要经 babel-loader 处理
      // 通常我们需要 loader 处理的文件都是存放在 src 目录
    ],
    use: 'babel-loader',
  },
  // ...
],
```

如上边这个例子，如果没有配置 `include`，所有的外部依赖模块都经过 Babel 处理的话，构建速度也是会收很大影响的。

#### `减少 plugin 的消耗`

webpack 的 plugin 会在构建的过程中加入其它的工作步骤，如果可以的话，适当地移除掉一些没有必要的 plugin。

这里再提一下 webpack 4.x 的 mode，区分 mode 会让 webpack 的构建更加有针对性，更加高效。例如当 mode 为 development 时，webpack 会避免使用一些提高应用代码加载性能的配置项，如 UglifyJsPlugin，ExtractTextPlugin 等，这样可以更快地启动开发环境的服务，而当 mode 为 production 时，webpack 会避免使用一些便于 debug 的配置，来提升构建时的速度，例如极其消耗性能的 Source Maps 支持。

#### `换种方式处理图片`

打包图片一般我们使用 webpack 的 `image-webpack-loader` 来压缩图片（`image-webpack-loader` 的压缩是使用 `imagemin` 提供的一系列图片压缩类库来处理的），在对 webpack 构建性能要求不高的时候，这样是一种很简便的处理方式，但是要考虑提高 webpack 构建速度时，这一块的处理就得重新考虑一下了，思考一下是否有必要在 webpack 每次构建时都处理一次图片压缩。

这里介绍一种解决思路，我们可以直接使用 `imagemin` 来做图片压缩，编写简单的命令即可。然后使用 `pre-commit` 这个类库来配置对应的命令，使其在 `git commit` 的时候触发，并且将要提交的文件替换为压缩后的文件。

这样提交到代码仓库的图片就已经是压缩好的了，以后在项目中再次使用到的这些图片就无需再进行压缩处理了，`image-webpack-loader` 也就没有必要了。

### webpack 4.x 的构建性能

从官方发布的 webpack 4.0 更新日志来看，webpack 4.0 版本做了很多关于提升构建性能的工作，我觉得比较重要的改进有这么几个：

- AST语法数可以直接从 `loader` 直接传递给 webpack，避免额外的解析。
- 使用速度更快的 md4 作为默认的 hash 方法，对于大型项目来说，文件一多，需要 hash 处理的内容就多，webpack 的 hash 处理优化对整体的构建速度提升应该还是有一定的效果的。
- Node 语言层面的优化，如用 `for of` 替换 `forEach`，用 `Map` 和 `Set` 替换普通的对象字面量等等。
- 默认开启 `uglifyjs-webpack-plugin` 的 `cache` 和 `parallel`，即缓存和并行处理，这样能大大提高 `production mode` 下压缩代码的速度。

除此之外，还有比较琐碎的一些内容，可以查阅：[webpack release 4.0](https://github.com/webpack/webpack/releases/tag/v4.0.0)，留意 performance 关键词。

很显然，webpack 的开发者们越来越关心 webpack 构建性能的问题，有一个关于 webpack 4.x 和 3.x 构建性能的简单对比：

> 6 entries, dev mode, source maps off, using a bunch of loaders and plugins. dat speed ⚡️

![](https://user-gold-cdn.xitu.io/2018/7/23/164c619b70ac83c7?w=498&h=164&f=webp&s=15598)

![](https://user-gold-cdn.xitu.io/2018/7/23/164c619d7d0b763e?w=440&h=154&f=webp&s=10450) 从这个对比的例子上看，4.x 的构建性能对比 3.x 是有很显著的提高，而 webpack 官方后续计划加入多核运算，持久化缓存等特性来进一步提升性能（可能要等到 5.x 版本了），所以，及时更新 webpack 版本，也是提升构建性能的一个有效方式。

### 换个角度
webpack 的确是一个好工具，但总归多多少少会有一些局限性，再怎么优化，不可能总能达到理想的效果，因为它确确实实完成那些构建任务就是需要这么一些时间。作为开发者，面对项目中各种各样的情况要随机应变，灵活处理，不能被好工具捆绑了思维模式，很多问题你不要过于依赖于 webpack，换个角度，可能可以找到更好的处理方式。
