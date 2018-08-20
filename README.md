# webpack
课件

# webpack基础详解

## 1、webpack 的安装和使用

我们使用 npm 或者 yarn 来安装 webpack，可以作为一个全局的命令来使用：

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
npm install webpack -D

# 或者
yarn add webpack -D
```

这样 webpack 会出现在 package.json 中，我们再添加一个 npm scripts：

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

因为是作为项目依赖进行安装，所以不会有全局的命令，npm/yarn 会帮助我们在当前项目依赖中寻找对应的命令执行，如果是全局安装的 webpack，直接执行 `webpack --mode production` 就可以。

`webpack 4.x` 的版本可以零配置就开始进行构建，但是个人觉得这个功能还不全面，缺少很多实际项目需要的功能，所以基本你还是需要一个配置文件。

## 2、webpack 的基本概念

webpack 本质上是一个打包工具，它会根据代码的内容解析模块依赖，帮助我们把多个模块的代码打包。借用 webpack 官网的图片： ![](https://user-gold-cdn.xitu.io/2018/7/23/164c5106ffe2c72d?w=1280&h=506&f=webp&s=11674)

### 入口

如上图，webpack 会把我们项目中使用到的多个代码模块（可以是不同文件类型），打包构建成项目运行仅需要的几个静态文件。webpack 有着十分丰富的配置项，提供了十分强大的扩展能力，可以在打包构建的过程中做很多事情。我们先来看一下 webpack 中的几个基本概念。

如上图所示，在多个代码模块中会有一个起始的 `.js` 文件，这个便是 webpack 构建的入口。webpack 会读取这个文件，并从它开始解析依赖，然后进行打包。如图，一开始我们使用 webpack 构建时，默认的入口文件就是 `./src/index.js`。

我们常见的项目中，如果是单页面应用，那么可能入口只有一个；如果是多个页面的项目，那么经常是一个页面会对应一个构建入口。

入口可以使用 `entry` 字段来进行配置，webpack 支持配置多个入口来进行构建：

```javascript
module.exports = {
  entry: './src/index.js'
}

// 上述配置等同于
module.exports = {
  entry: {
    main: './src/index.js'
  }
}

// 或者配置多个入口
module.exports = {
  entry: {
    foo: './src/page-foo.js',
    bar: './src/page-bar.js',
    // ...
  }
}

// 使用数组来对多个文件进行打包
module.exports = {
  entry: {
    main: [
      './src/foo.js',
      './src/bar.js'
    ]
  }
}
```

最后的例子，可以理解为多个文件作为一个入口，webpack 会解析两个文件的依赖后进行打包。

### loader

webpack 中提供一种处理多种文件格式的机制，便是使用 loader。我们可以把 loader 理解为是一个转换器，负责把某种文件格式的内容转换成 webpack 可以支持打包的模块。

举个例子，在没有添加额外插件的情况下，webpack 会默认把所有依赖打包成 js 文件，如果入口文件依赖一个 .hbs 的模板文件以及一个 .css 的样式文件，那么我们需要 handlebars-loader 来处理 .hbs 文件，需要 css-loader 来处理 .css 文件（这里其实还需要 style-loader，后续详解），最终把不同格式的文件都解析成 js 代码，以便打包后在浏览器中运行。

当我们需要使用不同的 loader 来解析处理不同类型的文件时，我们可以在 `module.rules` 字段下来配置相关的规则，例如使用 Babel 来处理 .js 文件：

```javascript
module: {
  // ...
  rules: [
    {
      test: /\.jsx?/, // 匹配文件路径的正则表达式，通常我们都是匹配文件类型后缀
      include: [
        path.resolve(__dirname, 'src') // 指定哪些路径下的文件需要经过 loader 处理
      ],
      use: 'babel-loader', // 指定使用的 loader
    },
  ],
}
```

loader 是 webpack 中比较复杂的一块内容，它支撑着 webpack 来处理文件的多样性。后续我们还会介绍如何更好地使用 loader 以及如何开发 loader。

### plugin

在 webpack 的构建流程中，plugin 用于处理更多其他的一些构建任务。可以这么理解，模块代码转换的工作由 loader 来处理，除此之外的其他任何工作都可以交由 plugin 来完成。通过添加我们需要的 plugin，可以满足更多构建中特殊的需求。例如，要使用压缩 JS 代码的 uglifyjs-webpack-plugin 插件，只需在配置中通过 plugins 字段添加新的 plugin 即可：

```javascript
const UglifyPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  plugins: [
    new UglifyPlugin()
  ],
}
```

除了压缩 JS 代码的 `uglifyjs-webpack-plugin`，常用的还有定义环境变量的 `DefinePlugin`，生成 CSS 文件的 `ExtractTextWebpackPlugin` 等。

plugin 理论上可以干涉 webpack 整个构建流程，可以在流程的每一个步骤中定制自己的构建需求。在必要时，也可以在 webpack 的基础上开发 plugin 来应对一些项目的特殊构建需求。

### 输出

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
    filename: '[name].js',
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

### devServer

实际开发中我们会需要做到以下几点：

1. 提供 HTTP 服务而不是使用本地文件预览；
2. 监听文件的变化并自动刷新网页，做到实时预览；
3. 支持 Source Map，以方便调试。

对于这些， Webpack 都为你考虑好了。Webpack 原生支持上述第2、3点内容，再结合官方提供的开发工具 `DevServer` 也可以很方便地做到第1点。 `DevServer` 会启动一个 HTTP 服务器用于服务网页请求，同时会帮助启动 Webpack ，并接收 Webpack 发出的文件更变信号，通过 WebSocket 协议自动刷新网页做到实时预览。
他的安装

```shell
	npm i -D webpack-dev-server
```
安装成功后执行 webpack-dev-server 命令， DevServer 就启动了。
`DevServer` 启动后会一直驻留在后台保持运行，访问这个网址你就能获取项目根目录下的 index.html。 同时你会发现并没有文件输出到 dist 目录，原因是 DevServer 会把 Webpack 构建出的文件保存在内存中，在要访问输出的文件时，必须通过 HTTP 服务访问。
几个常用配置：

#### port
`devServer.port`配置项用于配置 DevServer 服务监听的端口，默认使用 8080 端口。 如果 8080 端口已经被其它程序占有就使用 8081，如果 8081 还是被占用就使用 8082，
#### https
`DevServer` 默认使用 `HTTP 协议服务`，它也能通过 `HTTPS 协议服务`。 有些情况下你必须使用 HTTPS，例如 `HTTP2` 和 `Service Worker` 就必须运行在 HTTPS 之上。 要切换成 HTTPS 服务，最简单的方式是：

```javascript
devServer:{
  https: true
}
```
`DevServer` 会自动的为你生成一份 HTTPS 证书。

如果你想用自己的证书可以这样配置：

```javascript
devServer:{
  https: {
    key: fs.readFileSync('path/to/server.key'),
    cert: fs.readFileSync('path/to/server.crt'),
    ca: fs.readFileSync('path/to/ca.pem')
  }
}
```
#### compress
`devServer.compress` 配置是否启用 gzip 压缩。boolean 为类型，默认为 false.


### 一个简单的 webpack 配置

我们把上述涉及的几部分配置内容合到一起，就可以创建一个简单的 webpack 配置了，webpack 运行时默认读取项目下的 `webpack.config.js` 文件作为配置。

所以我们在项目中创建一个 `webpack.config.js` 文件：

```javascript
const path = require('path')
const UglifyPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  entry: './src/index.js',

  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },

  module: {
    rules: [
      {
        test: /\.jsx?/,
        include: [
          path.resolve(__dirname, 'src')
        ],
        use: 'babel-loader',
      },
    ],
  },

  // 代码模块路径解析的配置
  resolve: {
    modules: [
      "node_modules",
      path.resolve(__dirname, 'src')
    ],
	 //文件名省略的话，默认寻找循序
    extensions: [".wasm", ".mjs", ".js", ".json", ".jsx"],
   
  },

  plugins: [
    new UglifyPlugin(),
    // 使用 uglifyjs-webpack-plugin 来压缩 JS 代码
    // 如果你留意了我们一开始直接使用 webpack 构建的结果，你会发现默认已经使用了 JS 代码压缩的插件
    // 这其实也是我们命令中的 --mode production 的效果，后续会介绍 webpack 的 mode 参数
  ],
}
```

webpack 的配置其实是一个 Node.js 的脚本，这个脚本对外暴露一个配置对象，webpack 通过这个对象来读取相关的一些配置。因为是 Node.js 脚本，所以可玩性非常高，你可以使用任何的 Node.js 模块，如上述用到的 `path` 模块，当然第三方的模块也可以。

创建了 webpack.config.js 后再执行 webpack 命令，webpack 就会使用这个配置文件的配置了。

有的时候我们开始一个新的前端项目，并不需要从零开始配置 webpack，而可以使用一些工具来帮助快速生成 webpack 配置。

## 3、搭建基本的前端开发环境

### 关联 HTML

webpack 默认从作为入口的 .js 文件进行构建，但通常一个前端项目都是从一个页面（即 HTML）出发的，最简单的方法是，创建一个 HTML 文件，使用 script 标签直接引用构建好的 JS 文件，如：

```html
<script src="./dist/bundle.js"></script>
```

但是，如果我们的文件名或者路径会变化，例如使用 `[hash]` 来进行命名，那么最好是将 HTML 引用路径和我们的构建结果关联起来，这个时候我们可以使用 `html-webpack-plugin`。

html-webpack-plugin 是一个独立的 node package，所以在使用之前我们需要先安装它，把它安装到项目的开发依赖中：

```shell
npm install html-webpack-plugin -D

# 或者
yarn add html-webpack-plugin -D
```

然后在 webpack 配置中，将 html-webpack-plugin 添加到 plugins 列表中：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin(),
  ],
}
```

这样配置好之后，构建时 html-webpack-plugin 会为我们创建一个 HTML 文件，其中会引用构建出来的 JS 文件。实际项目中，默认创建的 HTML 文件并没有什么用，我们需要自己来写 HTML 文件，可以通过 html-webpack-plugin 的配置，传递一个写好的 HTML 模板：

```javascript
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // 配置输出文件名和路径
      template: 'assets/index.html', // 配置文件模板
    }),
  ],
}
```

这样，通过 html-webpack-plugin 就可以将我们的页面和构建 JS 关联起来，回归日常，从页面开始开发。如果需要添加多个页面关联，那么实例化多个 html-webpack-plugin， 并将它们都放到 `plugins` 字段数组中就可以了。

### 构建 CSS

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

### CSS 预处理器

在上述使用 CSS 的基础上，通常我们会使用 Less/Sass 等 CSS 预处理器，webpack 可以通过添加对应的 loader 来支持，以使用 Less 为例，我们可以在官方文档中找到对应的 `loader`。

我们需要在上面的 webpack 配置中，添加一个配置来支持解析后缀为 `.less` 的文件：

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.less$/,
        // 因为这个插件需要干涉模块转换的内容，所以需要使用它对应的 loader
        use: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: [
            'css-loader',
            'less-loader',
          ],
        }),
      },
    ],
  },
  // ...
}
```

### 处理图片文件

在前端项目的样式中总会使用到图片，虽然我们已经提到 css-loader 会解析样式中用 `url()` 引用的文件路径，但是图片对应的 jpg/png/gif 等文件格式，webpack 处理不了。是的，我们只要添加一个处理图片的 loader 配置就可以了，现有的 file-loader 就是个不错的选择。

file-loader 可以用于处理很多类型的文件，它的主要作用是直接输出文件，把构建后的文件路径返回。配置很简单，在 `rules`中添加一个字段，增加图片类型文件的解析配置：

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {},
          },
        ],
      },
    ],
  },
}
```

### 使用 Babel

`Babel` 是一个让我们能够使用 ES 新特性的 JS 编译工具，我们可以在 webpack 中配置 Babel，以便使用 ES6、ES7 标准来编写 JS 代码。

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.jsx?/, // 支持 js 和 jsx
        include: [
          path.resolve(__dirname, 'src'), // src 目录下的才需要经过 babel-loader 处理
        ],
        loader: 'babel-loader',
      },
    ],
  },
}
```

Babel 的相关配置可以在目录下使用 .babelrc 文件来处理，详细参考 Babel 官方文档 [.babelrc](http://babeljs.io/docs/en/babelrc/)。

### 启动静态服务

至此，我们完成了处理多种文件类型的 webpack 配置。我们可以使用 `webpack-dev-server` 在本地开启一个简单的静态服务来进行开发。

在项目下安装 webpack-dev-server，然后添加启动命令到 package.json 中：

```json
"scripts": {
  "build": "webpack --mode production",
  "start": "webpack-dev-server --mode development"
}
```

> 也可以全局安装 webpack-dev-server，但通常建议以项目开发依赖的方式进行安装，然后在 npm package 中添加启动脚本。

尝试着运行 npm run start 或者 yarn start，然后就可以访问 <http://localhost:8080/> 来查看你的页面了。默认是访问 index.html，如果是其他页面要注意访问的 URL 是否正确。

## 4、webpack 如何解析代码模块路径

在 webpack 支持的前端代码模块化中，我们可以使用类似 `import * as m from './index.js'` 来引用代码模块 `index.js`。

引用第三方类库则是像这样：`import React from 'react'`。webpack 构建的时候，会解析依赖后，然后再去加载依赖的模块文件，那么 webpack 如何将上述编写的 `./index.js` 或 `react` 解析成对应的模块文件路径呢？

> 在 JavaScript 中尽量使用 ECMAScript 2015 Modules 语法来引用依赖。

webpack 中有一个很关键的模块 `enhanced-resolve` 就是处理依赖模块路径的解析的，这个模块可以说是 Node.js 那一套模块路径解析的增强版本，有很多可以自定义的解析配置。

> 不熟悉 Node.js 模块路径解析机制的同学可以参考这篇文章：深入 [Node.js 的模块机制](http://www.infoq.com/cn/articles/nodejs-module-mechanism)。

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

接下来的内容会省略上述代码，直接描述 `resolve` 字段中的内容。

### 常用的一些配置

我们先从一些简单的需求来阐述 webpack 可以支持哪些解析路径规则的自定义配置。

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

更多匹配相关的写法可以参考官方文档 [Resolve Alias](https://webpack.docschina.org/configuration/resolve/#resolve-alias)，这里不一一举例说明。

#### `resolve.extensions`

在前面看 webpack 配置时，你可能留意到了这么一行：

```javascript
extensions: ['.wasm', '.mjs', '.js', '.json', '.jsx'],
// 这里的顺序代表匹配后缀的优先级，例如对于 index.js 和 index.jsx，会优先选择 index.js
```

看到数组中配置的字符串大概就可以猜到，这个配置的作用是和文件后缀名有关的。是的，这个配置可以定义在进行模块路径解析时，webpack 会尝试帮你补全那些后缀名来进行查找，例如有了上述的配置，当你在 src/utils/ 目录下有一个 common.js 文件时，就可以这样来引用：

```javascript
import * as common from './src/utils/common'
```

webpack 会尝试给你依赖的路径添加上 `extensions` 字段所配置的后缀，然后进行依赖路径查找，所以可以命中 `src/utils/common.js` 文件。

但如果你是引用 `src/styles` 目录下的 `common.css` 文件时，如 `import './src/styles/common'`，webpack 构建时则会报无法解析模块的错误。

你可以在引用时添加后缀，`import './src/styles/common.css'` 来解决，或者在 `extensions` 添加一个 `.css` 的配置：

```javascript
extensions: ['.wasm', '.mjs', '.js', '.json', '.jsx', '.css'],
```

#### `resolve.modules`

前面的内容有提到，对于直接声明依赖名的模块`（如 react`），webpack 会类似 Node.js 一样进行路径搜索，搜索 node_modules 目录，这个目录就是使用 `resolve.modules` 字段进行配置的，默认就是：

```javascript
resolve: {
  modules: ['node_modules'],
},
```

通常情况下，我们不会调整这个配置，但是如果可以确定项目内所有的第三方依赖模块都是在项目根目录下的 node_modules 中的话，那么可以在 node_modules 之前配置一个确定的绝对路径：

```javascript
resolve: {
  modules: [
    path.resolve(__dirname, 'node_modules'), // 指定当前目录下的 node_modules 优先查找
    'node_modules', // 如果有一些类库是放在一些奇怪的地方的，你可以添加自定义的路径或者目录
  ],
},
```

这样配置在某种程度上可以简化模块的查找，提升构建速度。

#### `resolve.mainFields`

有一些第三方模块会针对不同环境提供几分代码。 例如分别提供采用 ES5 和 ES6 的2份代码，这2份代码的位置写在 package.json 文件里，如下：

```javascript
{
  "jsnext:main": "es/index.js",// 采用 ES6 语法的代码入口文件
  "main": "lib/index.js" // 采用 ES5 语法的代码入口文件
}
```

Webpack 会根据 mainFields 的配置去决定优先采用那份代码，mainFields 默认如下：

```javascript
mainFields: ['browser', 'main']
```

Webpack 会按照数组里的顺序去package.json 文件里寻找，只会使用找到的第一个。

假如你想优先采用 ES6 的那份代码，可以这样配置：

```javascript 
mainFields: ['jsnext:main', 'browser', 'main']
```
####  `resolve.enforceExtension`

`resolve.enforceExtension` 如果配置为 `true` 所有导入语句都必须要带文件后缀， 例如开启前 `import './foo'` 能正常工作，开启后就必须写成 `import './foo.js'`。

#### `resolve.enforceModuleExtension`
`enforceModuleExtension` 和 `enforceExtension` 作用类似，但 `enforceModuleExtension` 只对 `node_modules` 下的模块生效。 `enforceModuleExtension` 通常搭配 `enforceExtension` 使用，在 `enforceExtension:true` 时，因为安装的第三方模块中大多数导入语句没带文件后缀， 所以这时通过配置 `enforceModuleExtension:false` 来兼容第三方模块。

## 5、配置 loader

webpack 的 loader 用于处理不同的文件类型，在日常的项目中使用 loader 时，可能会遇到比较复杂的情况，这里深入探讨 loader 的一些配置细节。

### loader 匹配规则

当我们需要配置 loader 时，都是在 `module.rules` 中添加新的配置项，在该字段中，每一项被视为一条匹配使用 loader 的规则。

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

匹配条件通常都使用请求资源文件的绝对路径来进行匹配，在官方文档中称为 resource，除此之外还有比较少用到的 `issuer`，则是声明依赖请求的源文件的绝对路径。举个例子：在 /path/to/app.js 中声明引入 `import './src/style.scss'`，`resource` 是 /path/to/src/style.scss，`issuer` 是 `/path/to/app.js`，规则条件会对这两个值来尝试匹配。

上述代码中的 `test` 和 `include` 都用于匹配 `resource` 路径，是 `resource.test` 和 `resource.include` 的简写，你也可以这么配置：

```javascript
module.exports = {
  // ...
  rules: [
      {
        resource: { // resource 的匹配条件
          test: /\.jsx?/,
          include: [
            path.resolve(__dirname, 'src'),
          ],
        },
        // 如果要使用 issuer 匹配，便是 issuer: { test: ... }
        use: 'babel-loader',
      },
      // ...
    ],
}
```

> issuer 规则匹配的场景比较少见，你可以用它来尝试约束某些类型的文件中只能引用某些类型的文件。

当规则的条件匹配时，便会使用对应的 loader 配置，如上述例子中的 `babel-loader`。关于 loader 配置后面再详细介绍，这里先来看看如何配置更加复杂的规则匹配条件。

### 规则条件配置

大多数情况下，配置 loader 的匹配条件时，只要使用 test 字段就好了，很多时候都只需要匹配文件后缀名来决定使用什么 loader，但也不排除在某些特殊场景下，我们需要配置比较复杂的匹配条件。webpack 的规则提供了多种配置形式：

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

### module type

webpack 4.x 版本强化了 module type，即模块类型的概念，不同的模块类型类似于配置了不同的 loader，webpack 会有针对性地进行处理，现阶段实现了以下 5 种模块类型。

- javascript/auto：即 webpack 3 默认的类型，支持现有的各种 JS 代码模块类型 ---- CommonJS、AMD、ESM
- javascript/esm：ECMAScript modules，其他模块系统，例如 CommonJS 或者 AMD 等不支持，是 .mjs 文件的默认类型
- javascript/dynamic：CommonJS 和 AMD，排除 ESM
- javascript/json：JSON 格式数据，require 或者 import 都可以引入，是 .json 文件的默认类型
- webassembly/experimental：WebAssembly modules，当前还处于试验阶段，是 .wasm 文件的默认类型

如果不希望使用默认的类型的话，在确定好匹配规则条件时，我们可以使用 type 字段来指定模块类型，例如把所有的 JS 代码文件都设置为强制使用 ESM 类型：

```javascript
{
  test: /\.js/,
  include: [
    path.resolve(__dirname, 'src'),
  ],
  type: 'javascript/esm', // 这里指定模块类型
},
```

上述做法是可以帮助你规范整个项目的模块系统，但是如果遗留太多不同类型的模块代码时，建议还是直接使用默认的 `javascript/auto`。

### 使用 loader 配置

当然，在当前版本的 webpack 中，`module.rules` 的匹配规则最重要的还是用于配置 loader，我们可以使用 `use` 字段：

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

我们看下上述的例子，先忽略 loader 的使用情况，单纯看看如何配置。use 字段可以是一个数组，也可以是一个字符串或者表示 `loader` 的对象。如果只需要一个 loader，也可以这样：`use: { loader: 'babel-loader', options: { ... } }`。

我们还可以使用 options 给对应的 `loader` 传递一些配置项，这里不再展开。当你使用一些 loader 时，loader 的说明一般都有相关配置的描述。

### loader 应用顺序

前面提到，一个匹配规则中可以配置使用多个 loader，即一个模块文件可以经过多个 loader 的转换处理，执行顺序是从最后配置的 loader 开始，一步步往前。例如，对于上面的 `less` 规则配置，一个 style.less 文件会途径 less-loader、css-loader、style-loader 处理，成为一个可以打包的模块。

loader 的应用顺序在配置多个 loader 一起工作时很重要，通常会使用在 CSS 配置上，除了 style-loader 和 css-loader，你可能还要配置 less-loader 然后再加个 postcss 的 autoprefixer 等。

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

### 使用 `noParse`

在 webpack 中，我们需要使用的 loader 是在 `module.rules` 下配置的，webpack 配置中的 module 用于控制如何处理项目中不同类型的模块。

除了 `module.rules` 字段用于配置 loader 之外，还有一个 `module.noParse` 字段，可以用于配置哪些模块文件的内容不需要进行解析。对于**一些不需要解析依赖（即无依赖）** 的第三方大型类库等，可以通过这个字段来配置，以提高整体的构建速度。

```javascript
module.exports = {
  // ...
  module: {
    noParse: /jquery|lodash/, // 正则表达式

    // 或者使用 function
    noParse(content) {
      return /jquery|lodash/.test(content)
    },
  }
}
```

`noParse` 从某种程度上说是个优化配置项，日常也可以不去使用。

## 6、使用 plugin

webpack 中的 plugin 大多都提供额外的能力，它们在 webpack 中的配置都只是把插件实例添加到 plugins 字段的数组中。不过由于需要提供不同的功能，不同的插件本身的配置比较多样化。

社区中有很多 webpack 插件可供使用，而优秀的插件基本上都提供了详细的使用说明文档。更多的插件可以在这里查找：[plugins in awesome-webpack。]()

下面通过介绍几个常用的插件来了解插件的使用方法。

### DefinePlugin

DefinePlugin 是 webpack 内置的插件，可以使用`webpack.DefinePlugin` 直接获取。

这个插件用于创建一些在编译时可以配置的全局常量，这些常量的值我们可以在 webpack 的配置中去指定，例如：

```javascript
module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      PRODUCTION: JSON.stringify(true), // const PRODUCTION = true
      VERSION: JSON.stringify('5fa3b9'), // const VERSION = '5fa3b9'
      BROWSER_SUPPORTS_HTML5: true, // const BROWSER_SUPPORTS_HTML5 = 'true'
      TWO: '1+1', // const TWO = 1 + 1,
      CONSTANTS: {
        APP_VERSION: JSON.stringify('1.1.2') // const CONSTANTS = { APP_VERSION: '1.1.2' }
      }
    }),
  ],
}
```

有了上面的配置，就可以在应用代码文件中，访问配置好的变量了，如：

```javascript
console.log("Running App version " + VERSION);

if(!BROWSER_SUPPORTS_HTML5) require("html5shiv");
```

上面配置的注释已经简单说明了这些配置的效果，这里再简述一下整个配置规则。

- 如果配置的值是字符串，那么整个字符串会被当成代码片段来执行，其结果作为最终变量的值，如上面的 "1+1"，最后的结果是 2
- 如果配置的值不是字符串，也不是一个对象字面量，那么该值会被转为一个字符串，如 true，最后的结果是 'true'
- 如果配置的是一个对象字面量，那么该对象的所有 key 会以同样的方式去定义 这样我们就可以理解为什么要使用 `JSON.stringify()` 了，因为 `JSON.stringify(true)` 的结果是 `'true'`，`JSON.stringify("5fa3b9")` 的结果是 `"5fa3b9"`。

社区中关于 `DefinePlugin` 使用得最多的方式是定义环境变量，例如 `PRODUCTION = true` 或者 `__DEV__ = true` 等。部分类库在开发环境时依赖这样的环境变量来给予开发者更多的开发调试反馈，例如 react 等。

> 建议使用 process.env.NODE_ENV: ... 的方式来定义 process.env.NODE_ENV，而不是使用 process: { env: { NODE_ENV: ... } } 的方式，因为这样会覆盖掉 process 这个对象，可能会对其他代码造成影响。

### copy-webpack-plugin

这个插件看名字就知道它有什么作用，没错，就是用来复制文件的。

我们一般会把开发的所有源码和资源文件放在 src/ 目录下，构建的时候产出一个 build/ 目录，通常会直接拿 build 中的所有文件来发布。有些文件没经过 webpack 处理，但是我们希望它们也能出现在 build 目录下，这时就可以使用 CopyWebpackPlugin 来处理了。

我们来看下如何配置这个插件：

```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new CopyWebpackPlugin([
      { from: 'src/file.txt', to: 'build/file.txt', }, // 顾名思义，from 配置来源，to 配置目标路径
      { from: 'src/*.ico', to: 'build/*.ico' }, // 配置项可以使用 glob
      // 可以配置很多项复制规则
    ]),
  ],
}
```

### extract-text-webpack-plugin

extract-text-webpack-plugin 之前有简单介绍过，我们用它来把依赖的 CSS 分离出来成为单独的文件。这里再看一下使用 extract-text-webpack-plugin 的配置：

const ExtractTextPlugin = require('extract-text-webpack-plugin')

```javascript
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

在上述的配置中，我们使用了 index.css 作为单独分离出来的文件名，但有的时候构建入口不止一个，extract-text-webpack-plugin 会为每一个入口创建单独分离的文件，因此最好这样配置：

```javascript
plugins: [
  new ExtractTextPlugin('[name].css'),
],
```

这样确保在使用多个构建入口时，生成不同名称的文件。

这里再次提及 extract-text-webpack-plugin，一个原因是它是一个蛮常用的插件，另一个原因是它的使用方式比较特别，除了在 `plugins` 字段添加插件实例之外，还需要调整 loader 对应的配置。

在这里要强调的是，在 webpack 中，loader 和 plugin 的区分是很清楚的，针对文件模块转换要做的使用 loader，而其他干涉构建内容的可以使用 plugin。 ExtractTextWebpackPlugin 既提供了 plugin，也提供了 extract 方法来获取对应需要的 loader。

### ProvidePlugin

ProvidePlugin 也是一个 webpack 内置的插件，我们可以直接使用 `webpack.ProvidePlugin` 来获取。

该组件用于引用某些模块作为应用运行时的变量，从而不必每次都用 `require` 或者 `import`，其用法相对简单：

```javascript
new webpack.ProvidePlugin({
  identifier: 'module',
  // ...
})

// 或者
new webpack.ProvidePlugin({
  identifier: ['module', 'property'], // 即引用 module 下的 property，类似 import { property } from 'module'
  // ...
})
```

在你的代码中，当 identifier 被当作未赋值的变量时，module 就会被自动加载了，而 identifier 这个变量即 module 对外暴露的内容。

注意，如果是 ES 的 `default export`，那么你需要指定模块的 `default` 属性：`identifier: ['module', 'default'],`。

### IgnorePlugin

IgnorePlugin 和 ProvidePlugin 一样，也是一个 webpack 内置的插件，可以直接使用 `webpack.IgnorePlugin` 来获取。

这个插件用于忽略某些特定的模块，让 webpack 不把这些指定的模块打包进去。例如我们使用 [moment.js](http://momentjs.cn/docs/)，直接引用后，里边有大量的 i18n 的代码，导致最后打包出来的文件比较大，而实际场景并不需要这些 i18n 的代码，这时我们可以使用 IgnorePlugin 来忽略掉这些代码文件，配置如下：

```javascript
module.exports = {
  // ...
  plugins: [
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
  ]
}
```

IgnorePlugin 配置的参数有两个，第一个是匹配引入模块路径的正则表达式，第二个是匹配模块的对应上下文，即所在目录名。防止在 import 或 require 调用时，生成的本地文件夹下忽略moment文件夹。


## 7、提升 webpack 的构建速度

我们的前端项目随着时间推移和业务发展，页面可能会越来越多，或者功能和业务代码会越来越多，又或者依赖的外部类库会越来越多，这个时候原本不足为道的 webpack 构建时间消耗就会慢慢地进入我们的视野。

### 让 webpack 少干点活

提升 webpack 构建速度本质上就是想办法让 webpack 少干点活，活少了速度自然快了，尽量避免 webpack 去做一些不必要的事情。

#### `减少 resolve 的解析`

在前边详细介绍了 webpack 的 `resolve` 配置，如果我们可以精简 `resolve` 配置，让 webpack 在查询模块路径时尽可能快速地定位到需要的模块，不做额外的查询工作，那么 webpack 的构建速度也会快一些，下面举个例子，介绍如何在 `resolve` 这一块做优化：

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

打包图片可以使用 webpack 的 `image-webpack-loader` 来压缩图片，在对 webpack 构建性能要求不高的时候，这样是一种很简便的处理方式，但是要考虑提高 webpack 构建速度时，这一块的处理就得重新考虑一下了，思考一下是否有必要在 webpack 每次构建时都处理一次图片压缩。

这里介绍一种解决思路，我们可以直接使用 `imagemin` 来做图片压缩，编写简单的命令即可。然后使用 `pre-commit` 这个类库来配置对应的命令，使其在 `git commit` 的时候触发，并且将要提交的文件替换为压缩后的文件。

这样提交到代码仓库的图片就已经是压缩好的了，以后在项目中再次使用到的这些图片就无需再进行压缩处理了，`image-webpack-loader` 也就没有必要了。

### webpack 4.x 的构建性能

从官方发布的 webpack 4.0 更新日志来看，webpack 4.0 版本做了很多关于提升构建性能的工作，我觉得比较重要的改进有这么几个：

- AST可以直接从 `loader` 直接传递给 webpack，避免额外的解析。
- 使用速度更快的 md4 作为默认的 hash 方法，对于大型项目来说，文件一多，需要 hash 处理的内容就多，webpack 的 hash 处理优化对整体的构建速度提升应该还是有一定的效果的。
- Node 语言层面的优化，如用 `for of` 替换 `forEach`，用 `Map` 和 `Set` 替换普通的对象字面量等等，这一部分就不展开讲了，有兴趣的同学可以去 webpack 的 PRs 寻找更多的内容。
- 默认开启 `uglifyjs-webpack-plugin` 的 `cache` 和 `parallel`，即缓存和并行处理，这样能大大提高 `production mode` 下压缩代码的速度。

除此之外，还有比较琐碎的一些内容，可以查阅：[webpack release 4.0](https://github.com/webpack/webpack/releases/tag/v4.0.0)，留意 performance 关键词。

很显然，webpack 的开发者们越来越关心 webpack 构建性能的问题，有一个关于 webpack 4.x 和 3.x 构建性能的简单对比：

> 6 entries, dev mode, source maps off, using a bunch of loaders and plugins. dat speed ⚡️

![](https://user-gold-cdn.xitu.io/2018/7/23/164c619b70ac83c7?w=498&h=164&f=webp&s=15598)

![](https://user-gold-cdn.xitu.io/2018/7/23/164c619d7d0b763e?w=440&h=154&f=webp&s=10450) 从这个对比的例子上看，4.x 的构建性能对比 3.x 是有很显著的提高，而 webpack 官方后续计划加入多核运算，持久化缓存等特性来进一步提升性能（可能要等到 5.x 版本了），所以，及时更新 webpack 版本，也是提升构建性能的一个有效方式。

### 换个角度

webpack 的构建性能优化是比较琐碎的工作，当我们需要去考虑 webpack 的构建性能问题时，往往面对的是项目过大，涉及的代码模块过多的情况。在这种场景下你单独做某一个点的优化其实很难看出效果，你可能需要从我们上述提到的多个方面入手，逐一处理，验证，有些时候你甚至会觉得吃力不讨好，投入产出比太低了，这个时候我们可以考虑换一个角度来思考我们遇到的问题。

例如，拆分项目的代码，根据一定的粒度，把不同的业务代码拆分到不同的代码库去维护和管理，这样子单一业务下的代码变更就无须整个项目跟着去做构建，这样也是解决因项目过大导致的构建速度慢的一种思路，并且如果处理妥当，从工程角度上可能会给你带来其他的一些好处，例如发布异常时的局部代码回滚相对方便等等。

这可能有点跑题，但是不得不说，webpack 的确是一个好工具，但总归多多少少会有一些局限性，再怎么优化，不可能总能达到理想的效果，因为它确确实实完成那些构建任务就是需要这么一些时间。作为开发者，面对项目中各种各样的情况要随机应变，灵活处理，不能被好工具捆绑了思维模式，很多问题你不要过于依赖于 webpack，换个角度，可能可以找到更好的处理方式。
