# multipages

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).


<!-- 多页面配置应用参考https://segmentfault.com/a/1190000016758185 -->

1. 新建 vue 项目

vue init webpack vue_multiple_test
cd vue_multiple_test
npm install

2. 安装 glob

npm i glob --save-dev

glob 模块用于查找符合要求的文件


3. 目标结构目录

.
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── logo.png
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── generatePage.sh
├── index.html
├── package-lock.json
├── package.json
├── src
│   ├── assets
│   │   └── logo.png
│   └── pages
│       ├── page1
│       │   ├── App.vue
│       │   ├── index.html
│       │   └── index.js
│       └── page2
│           ├── App.vue
│           ├── index.html
│           └── index.js
└── static


其中，pages文件夹用于放置页面。 page1 和 page2用于分别放置不同页面，且默认均包含三个文档: App.vue, index.html, index.js, 这样在多人协作时，可以更为清晰地明确每个文件的含义。除此之外，此文件也可配置路由。


4. utils 增加下述代码:

const glob = require('glob')
const PAGE_PATH = path.resolve(__dirname, '../src/pages')
const HtmlWebpackPlugin = require('html-webpack-plugin')

其中：PAGE_PATH 为所有页面所在的文件夹路径，指向 pages文件夹。

exports.entries = function () {
    /*用于匹配 pages 下一级文件夹中的 index.js 文件 */
    var entryFiles = glob.sync(PAGE_PATH + '/*/index.js')
    var map = {}
    entryFiles.forEach((filePath) => {
        /* 下述两句代码用于取出 pages 下一级文件夹的名称 */
        var entryPath = path.dirname(filePath)
        var filename = entryPath.substring(entryPath.lastIndexOf('\/') + 1)
        /* 生成对应的键值对 */
        map[filename] = filePath
    })
    return map
}

该方法用于生成多页面的入口对象，例如本例，获得的入口对象如下：

{ 
    page1: '/Users/work/learn/vue/vue_multiple_test/src/pages/page1/index.js',
    page2: '/Users/work/learn/vue/vue_multiple_test/src/pages/page2/index.js',
 }

其中：key 为当前页面的文件夹名称， value 为当前页面的入口文件名称


exports.htmlPlugin = function () {
    let entryHtml = glob.sync(PAGE_PATH + '/*/index.html')
    let arr = []
    entryHtml.forEach((filePath) => {
        var entryPath = path.dirname(filePath)
        var filename = entryPath.substring(entryPath.lastIndexOf('\/') + 1)
        let conf = {
            template: filePath,
            filename: filename + `/index.html`,
            chunks: ['manifest', 'vendor', filename],
            inject: true
        }
        if (process.env.NODE_ENV === 'production') {
            let productionConfig = {
                minify: {
                  removeComments: true,         // 移除注释
                  collapseWhitespace: true,     // 删除空白符和换行符
                  removeAttributeQuotes: true   // 移除属性引号 
                },
                chunksSortMode: 'dependency'    // 对引入的chunk模块进行排序
            }
            conf = {...conf, ...productionConfig} //合并基础配置和生产环境专属配置
        }
        arr.push(new HtmlWebpackPlugin(conf))
    })
    return arr
}


5. webpack.base.conf.js修改入口如下：

entry: utils.entries()


6. webpack.dev.conf.js
在 devWebpackConfig 中的 plugins数组后面拼接上上面新写的htmlPlugin:

plugins: [
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin(), // HMR shows correct file names in console on update.
    new webpack.NoEmitOnErrorsPlugin(),
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.dev.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ].concat(utils.htmlPlugin())

  并删除下述代码:

  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true
})

7. webpack.prod.conf.js

做 webpack.dev.conf.js 中的类似处理:

plugins: [
    new webpack.DefinePlugin({
      'process.env': env
    }),
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      parallel: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      allChunks: true,
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    new webpack.HashedModuleIdsPlugin(),
    new webpack.optimize.ModuleConcatenationPlugin(),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ].concat(utils.htmlPlugin())

  并删除下述代码:

  new HtmlWebpackPlugin({
    filename: config.build.index,
    template: 'index.html',
    inject: true,
    minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
    },
    chunksSortMode: 'dependency'
})

8. 构建结果

【dev】开发环境下，执行 npm run dev 访问:

 http://localhost:8080/page1/index.html
 http://localhost:8080/page2/index.html

 即为访问不同的页面


【production】生产环境下，执行 npm run build, 生成的文件目录如下所示:

│   ├── dist
│   ├── page1
│   │   └── index.html
│   ├── page2
│   │   └── index.html
│   └── static
│       ├── css
│       │   ├── page1.86a4513a3e04c0dcb73e6d6aea4580e4.css
│       │   ├── page1.86a4513a3e04c0dcb73e6d6aea4580e4.css.map
│       │   ├── page2.86a4513a3e04c0dcb73e6d6aea4580e4.css
│       │   └── page2.86a4513a3e04c0dcb73e6d6aea4580e4.css.map
│       └── js
│           ├── manifest.0c1cd46d93b12dcd0191.js
│           ├── manifest.0c1cd46d93b12dcd0191.js.map
│           ├── page1.e2997955f3b0f2090b7a.js
│           ├── page1.e2997955f3b0f2090b7a.js.map
│           ├── page2.4d41f3b684a56847f057.js
│           ├── page2.4d41f3b684a56847f057.js.map
│           ├── vendor.bb335a033c3b9e5d296a.js
│           └── vendor.bb335a033c3b9e5d296a.js.map






总结：自己在根据以上配置多页面应用，生成生产环境文件时出现html中访问js和css的路径错误问题，然后通过修改config/index.js

build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist/index.html'),

    // Paths
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '../',

    /**
     * Source Maps
     */

    productionSourceMap: true,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }

  中 assetsPublicPath: '../',根据生成的dist目录来修改assetsPublicPath的配置路径；
  以上只是个人的一些修改，因为对webpack只有一个初步的了解，此处这样修改后续是否会出现错误尚不得知，仍在测试实验中，同时开始对webpack的详细学习尽快认识和发现webpack中各种配置的作用。
  123