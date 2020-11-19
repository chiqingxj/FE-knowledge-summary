## webpack 相关概念
**配置文件**

默认为 webpack.config.js，可以通过 webpack --config 指定配置文件

**配置文件构成**

+ 打包的入口：Entry
    + 单入口：entry 是一个字符串
    ```js
    entry: './src/index.js'
    ```
    + 多入口：entry 是一个对象
    ```js
    entry: {
        app: './src/app.js',
        about: './src/about.js'
    }
    ```

+ 打包的输出：Output
    + 单入口配置
    ```js
    output: {
        filename: 'bundle.js',
        path: __dirname + '/dist'
    }
    ```
    + 多入口配置
    ```js
    output: {
        filename: '[name].js', // 通过占位符确保文件名唯一
        path: __dirname + '/dist'
    }
    ```

+ 环境：Mode

    Mode ⽤来指定当前的构建环境是：production、development 还是 none。

    可以通过设置 mode 使用 webpack 内置的函数，默认值为 production。

    内置函数功能：
    | 选项 | 描述 |
    |-----|------|
    | development | 设置 process.env.NODE_ENV 的值为 development，开启 NamedChunksPlugin、NamedModulesPlugin |
    | production | 设置 process.env.NODE_ENV 的值为 production，开启 FlagDependencyUsagePlugin、FlagIncludeChunksPlugin、ModuleConcatenationPlugin、NoEmitOnErrorsPlugin、OccurrenceOrderPlugin、SideEffectsFlagPlugin、TerserPlugin |
    | none | 不开启任何优化选项 |

+ Loader 配置

    webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其它文件类型并且把它们转化成有效的模块，并且可以添加到依赖图中。

    loader 本身是一个函数，接受源文件作为参数，返回转换的结果。

+ Plugin 配置

    plugin ⽤于 bundle 文件的优化，资源管理和环境变量注入，作用于整个构建过程。

    用法：每个 plugin 都放入到定义的 plugins 数组⾥
    ```js
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/template/index.html'
        })
    ]
    ```


## webpack 的文件监听
⽂件监听是在发现源码发生变化时，⾃动重新构建出新的输出文件。

webpack **开启监听模式**，有两种方式：
+ 启动 webpack 命令时，带上--watch 参数
+ 在配置 webpack.config.js 中设置 watch: true

```js
// webpack.config.js
module.export = {
    watch: true,
    watchOptions: {
        // 不监听的文件或者文件夹，支持正则匹配，默认为空
        ignored: /node_modules/,
        // 监听到变化发生后会等 300ms 再去执行，默认 300ms
        aggregateTimeout: 300,
        // 判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒 1000 次
        poll: 1000
    }
}

// package.json
"scripts": {
    "watch": "webpack --watch"
}
```

文件监听的**原理**：轮询判断文件的最后编辑时间是否发生了变化，当某个文件发生了变化，并不会立即告诉监听者，而是先缓存起来，等待 aggregateTimeout。

文件监听的**缺点**：每次更新后都需要手动刷新浏览器才能获取到更新的内容。


## webpack 热更新
+ webpack-dev-server

    WDS 不刷新浏览器

    WDS 不输出文件，而是放在内存中

    使⽤ HotModuleReplacementPlugin 插件

    ```js
    // package.json
    "scripts": {
        "dev": "webpack-dev-server --open"
    }
    ```

+ webpack-dev-middleware

    WDM 将webpack 输出的文件传输给服务器，适用于灵活的定制场景。

    ```js
    const express = require('express');
    const webpack = require('webpack');
    const webpackDevMiddleware = require('webpack-devmiddleware');
    const config = require('./webpack.config.js');

    const app = express();
    const compiler = webpack(config);

    app.use(webpackDevMiddleware(compiler, {
        publicPath: config.output.publicPath
    }));

    app.listen(3000, function () {
        console.log('Example app listening on port 3000!\n');
    });
    ```

热更新原理分析：

Webpack Compile: 将 JS 编译成 Bundle

HMR Server: 将热更新的⽂文件输出给 HMR Rumtime

Bundle server: 提供文件在浏览器的访问

HMR Rumtime: 会被注入到浏览器，

更新⽂文件的变化

bundle.js: 构建输出的⽂文件


## 文件指纹
文件指纹：打包后输出的文件名的后缀。

生成文件指纹：
+ Hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改。
+ Chunkhash：和 webpack 打包的 chunk 有关，不同的 entry 会生成不同的 chunkhash 值。
+ Contenthash：根据文件内容来定义 hash ，文件内容不变，则 contenthash 不变。

文件指纹设置：
```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        // 设置 output 的 filename，使用 [chunkhash]
        filename: '[name][chunkhash:8].js',
        path: __dirname + '/dist'
    },
    module: {
        rules: [
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: [{
                    loader: 'file-loader',
                    options: {
                        // 设置 file-loader 的 name，使用 [hash]
                        name: 'img/[name][hash:8].[ext] '
                    }
                }]
            }
        ]
    },
    plugins: [
        // 设置 MiniCssExtractPlugin 的 filename，使用 [contenthash]
        new MiniCssExtractPlugin({
            filename: `[name][contenthash:8].css`
        });
    ]
}
```


## webpack 代码压缩
+ **JavaScript** 文件的压缩

    webpack 内置了 uglifyjs-webpack-plugin 实现 js 代码的压缩。
    
+ **CSS** 文件的压缩

    使用 optimize-css-assets-webpack-plugin 和 cssnano 进行 CSS 代码的压缩。

    ```js
    plugins: [
        new OptimizeCSSAssetsPlugin({
            assetNameRegExp: /\.css$/g,
            cssProcessor: require('cssnano’)
        })
    ]
    ```

+ **Html** 文件的压缩
  
    使用 html-webpack-plugin，并且设置压缩参数。

    ```js
    plugins: [
        new HtmlWebpackPlugin({
            template: path.join(__dirname, 'src/search.html'),
            filename: 'search.html',
            chunks: ['search'],
            inject: true,
            minify: {
                html5: true,
                collapseWhitespace: true,
                preserveLineBreaks: false,
                minifyCSS: true,
                minifyJS: true,
                removeComments: false
            }
        })
    ]
    ```


## webpack 构建自动清理目录
+ npm scripts

    ```linux
    rm -rf ./dsit && webpack
    rimraf ./dsit && webpack
    ```

+ clean-webpack-plugin

    默认会删除 output 指定的输出目录，避免构建前每次都需要手动删除 dist。

    ```js
    plugins: [
        new CleanWebpackPlugin(),
    ]
    ```


## webpack CSS3 自动补齐前缀
使⽤ autoprefixer 插件，根据 Can I Use 规则。

```js
module.exports = {
    module: {
        rules: [{
            test: /\.less$/,
            use: [
                'style-loader',
                'css-loader',
                'less-loader',
                {
                    loader: 'postcss-loader',
                    options: {
                        plugins: () => [
                            require('autoprefixer')({
                                browsers: ["last 2 version", "> 1%", "iOS 7"]
                            })
                        ]
                    },
                }
            ]
        }]
    }
};
```


## webpack CSS3 响应式布局
+ CSS3 媒体查询
  
    缺点：需要写多套适配样式的代码。
    
+ rem
  
    **px2rem-loader**：实现移动端 CSS px 自动转换成 rem。

    ⻚面渲染时计算根元素的 font-size 值：可以使⽤手淘的 **lib-flexible** 库。
    
    ```js
    module.exports = {
        module: {
            rules: [{
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader',
                    {
                        loader: "px2rem-loader",
                        options: {
                            remUnit: 75,
                            remPrecision: 8,
                        }
                    }
                ]
            }]
        }
    };
    ```


## webpack 实现资源内联
资源内联的意义：
+ 代码层面
    + 页面框架的初始化脚本
    + 上报相关埋点
    + CSS 内联避免页面闪动
+ 请求层面：减少 HTTP 请求数
    + ⼩图片或者字体内联(url-loader)

Html 和 JS 内联：raw-loader
```html
<!-- 内联 html -->
<script>${require('raw-loader!babel-loader!. /meta.html')}</script>

<!-- 内联 js -->
<script>${require('raw-loader!babel-loader!../node_modules/lib-flexible')}</script>
```

CSS 内联：style-loader / html-inline-css-webpack-plugin
```js
// style-loader
rules: [{
    test: /\.scss$/,
    use: [
        {
            loader: 'style-loader',
            options: {
                insertAt: 'top', // 样式插入到 <head>
                singleton: true, //将所有的 style 标签合并成一个
            }
        },
        "css-loader",
        "sass-loader"
    ],
}]

// html-inline-css-webpack-plugin

```


## webpack MPA(多页应用)打包
**MPA**：每一次页面跳转的时候，后台服务器都会给返回一个新的 html 文档，这种类型的网站也就是多页网站，也叫做多页应用。

基本**打包思路**：每个页面对应一个 entry，一个 html-webpack-plugin。

**通用方案**：利用 glob.sync 动态获取 entry 和设置 html-webpack-plugin 数量。

```js
const glob = require('glob');
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const setMPA = () => {
    const entry = {};
    const htmlWebpackPlugins = [];
    const entryFiles = glob.sync(path.join(__dirname, './src/*/index.js'));

    Object.keys(entryFiles)
        .map((index) => {
            const entryFile = entryFiles[index];
            const match = entryFile.match(/src\/(.*)\/index\.js/);
            const pageName = match && match[1];

            entry[pageName] = entryFile;
            htmlWebpackPlugins.push(
                new HtmlWebpackPlugin({
                    inlineSource: '.css$',
                    template: path.join(__dirname, `src/${pageName}/index.html`),
                    filename: `${pageName}.html`,
                    chunks: ['vendors', pageName],
                    inject: true,
                    minify: {
                        html5: true,
                        collapseWhitespace: true,
                        preserveLineBreaks: false,
                        minifyCSS: true,
                        minifyJS: true,
                        removeComments: false
                    }
                })
            );
        });

    return {
        entry,
        htmlWebpackPlugins
    }
}

const { entry, htmlWebpackPlugins } = setMPA();

module.exports = {
    entry: entry,
    output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name]_[chunkhash:8].js'
    },
    plugins: [].concat(htmlWebpackPlugins)
}
```


## webpack 使用 source-map
作用：通过 source map 定位到源代码，开发环境开启，线上环境关闭，线上排查问题的时候可以将 source map 上传到错误监控系统。

**source map 关键字**：

+ eval：使用 eval 包裹模块代码
+ source map：产生 .map 文件
+ cheap：不含列信息
+ inline：将 .map 作为 DataURI 嵌⼊，不单独生成 .map 文件
+ module：包含 loader 的 source map


## webpack 实现基础库分离
思路：将 react、react-dom 基础包通过 cdn 引入，不打入到 bundle 中。

方法：使用 html-webpackexternals-plugin。

```js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

module.export = {
    plugins: [
        new HtmlWebpackExternalsPlugin({
            externals: [
                {
                    module: 'react',
                    entry: 'https://11.url.cn/now/lib/16.2.0/react.min.js',
                    global: 'React',
                },
                {
                    module: 'react-dom',
                    entry: 'https://11.url.cn/now/lib/16.2.0/react-dom.min.js',
                    global: 'ReactDOM',
                },
            ]
        }),
    ]
}
```




## 常用 Loader
**babel-loader**

babel-loader 的作用是解析 ES6、React JSX。

babel 使用 .babelrc 文件进行配置。

```js
// webpack
module: {
    rules: [
        {
            test: /\.js$/,
            use: 'babel-loader'
        }
    ]
}

// .babelrc
{
    'presets': [
        '@babel/preset-env',
        '@babel/preset-react'
    ],
    'plugins': [
        '@babel/proposal-class-properties'
    ]
}
```

**style-loader/css-loader/less-loader**

style-loader：用于将样式通过 <style> 标签插入到 head 中。

css-loader：用于加载 .css 文件，并且转换成 cjs 对象。

less-loader：用于将 less 文件转换成为 css 文件。

```js
module: {
    rules: [
        {
            test: /\.less$/,
            use: [
                'style-loader',
                'css-loader',
                'less-loader'
            ]
        }
    ]
}
```

**注意**：loader 的解析顺序是**从右到左**的。

**file-loader**

file-loader：用于解析文件资源，包括图片、字体等。

```js
module: {
    rules: [
        {
            test: /\.(png|svg|jpg|gif)$/,
            use: [
                'file-loader'
            ]
        }
    ]
}
```

**url-loader**

url-loader：也可以用于处理图片和字体，可以设置较小资源自动进行 base64 转换。

```js
module: {
    rules: [
        {
            test: /\.(png|svg|jpg|gif)$/,
            use: [
                loader: 'url-loader',
                options： {
                    limit: 10240
                }
            ]
        }
    ]
}
```


## 常用 Plugin


## 常见问题
1. npm run 的原理是什么？
模块局部安装会在 node_modules/.bin 文件夹下创建软链接

