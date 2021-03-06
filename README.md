## webpack 基本打包机制

webpack 是一个 js 静态模块的打包器（module bundler），当 webpack 处理应用程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或多个 bundle

打包过程可以分成四步
1. 利用 babel 完成代码转换，并生成单个文件的依赖
2. 从入口开始递归分析，并生成依赖图
3. 将各个引用模块打包成一个立即执行函数
4. 将最终的 bundle 文件写入 bundle.js 中

## os（操作系统）
os 模块提供了与操作系统相关的实用方法和属性，使用方法如下：

```js
const os = require('os')
```

## path (路径)
path 模块提供用于处理文件路径和目录路径的实用工具，使用方法如下：

```js
const path = require('path')
```

## happypack 提高构建速度
在使用 Webpack 对项目进行构建时，会对大量文件进行解析和处理。当文件数量变多之后，Webpack 构件速度就会变慢。由于运行在 Node.js 之上的 Webpack 是单线程模型的，所以 Webpack 需要处理的任务要一个一个进行操作

而 Happypack 的作用就是将文件解析任务分解成多个子进程并发执行。子进程处理完任务后再将结果发送给主进程。所以可以大大提升 Webpack 的项目构建速度

### Usage

```js
var os = require('os')
var path = requore('path')
var HappyPack = require('happypack')
var happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length })

module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.js$/,
        // use: ['babel-loader?cacheDirectory'] 之前是使用这种方式直接使用 loader
        // 现在用下面的方式替换成 happypack/loader，并使用 id 指定创建的 HappyPack 插件
        use: ['happypack/loader?id=babel'],
        exclude: /node_module/
      }
    ]
  },
  plugins: [
    ...,
    new HappyPack({
      // id 标识符，要和 rules 中指定的 id 对应起来
      id: 'babel',
      // 代表共享进程池，即多个 HappyPack 实例都使用同一个共享进程池中的子进程去处理任务，以防止资源占用过多
      threadPool: happyThreadPool,
      // 使用的loader
      use: [
        {
          loader: 'babel-loader?cacheDirectory'
        }
      ]
    })
  ]
}
```
