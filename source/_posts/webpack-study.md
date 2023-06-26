---
title: webpack学习
date: 2023-4-3 15:18
categories: 打包
---
# webpack

先去[node官网](https://nodejs.org/en)下载最新版的nodeLTS，下载完之后在终端执行`node -v`，之后安装webpack。

## webpack安装

- 全局安装

  ```bash
  npm install webpack webpack-cli --global
  ```

- 本地安装

  ```bash
  npm init -y
  ```

  ```bash
  npm install webpack webpack-cli --save-dev
  ```

直接在当前目录库运行webpack命令就会生成一个dist文件，完成一个简易打包。

## webpack的自定义配置

- 在命令行输入`webpack --help`可以看到很多自定义配置的指令
- 在目录下创建`webpack.config.js`注意此文件名是不可修改的，因为webpack会自动读取此文件。

注意：

```js
module.exports = {
    // 入口文件设置
    entry: './src/index.js',
    // 出口位置
    output: {
        filename: 'bundle.js',
        path: './dist'
    }
}
```

如此配置打包时会出现错误，因为出口的path规定要用绝对路径所以应做如下修改

```js
const path = require('path')

module.exports = {
    // 入口文件设置
    entry: './src/index.js',
    // 出口位置
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist')
    }
}
```

如此设置即可

## 自动引入资源

### plugins(插件) 

在webpack编译的过程中完成一些辅助性的操作

#### 使用 HtmlWebpackPlugin

安装命令

```bash
npm install html-webpack-plugin -D
```

```js
const HTMLWebpackPlugin = require('html-webpack-plugin')
module.exports = {
	// 插件
    plugins: [
        new HTMLWebpackPlugin({
        	// 套用的模板
            template: './index.html',
            // 生成的文件名
            filename: 'app.html',
            // script标签的位置
            inject: "body"
        })
    ]
}
```

#### 清理掉上次遗留的打包文件

在output配置项中添加`clean: true`，即可清理。

### 如何在生成的bundle.js中显示可读性强的代码

精准定位代码的行数

```js
devtool: 'inline-source-map'
```

在webpack.config.js中加入此配置项即可

### 不再多次执行打包命令并且页面在数据更改时自动刷新的方法

- 可以在文件修改后自动执行打包命令

  ```bash
  npx webpack --watch
  ```

- 代码修改时页面实时更新(也可以自动执行打包)

  - 安装插件

    ```bash
    npm install webpack-dev-server
    ```

  - 在webpack.config.js中添加如下配置

    ```js
    devServer: {
    	static: './dist'
    }
    ```

  - 在命令行执行如下命令

    ```bash
    npx webpack-dev-server
    ```

  此插件就是将我们打包好的文件放在内存中，在代码运行期间删除代码，也依然可以显示出原来的结果。

## 资源模块

### resource

可以发送一个单独的文件并导出URL

在webpack.config.js添加如下属性

```js
module: {
        rules: [
            {
                test: /\.png$/,
                type: 'asset/resource'
            }
        ]
    }
```

然后再入口文件中引入png类型图片，再执行打包命令，就有一个编译器生成的名字的图片，想要修改路径和文件名，可以加入如下配置

- 再output中添加`assetModuleFilename: 'image/[contenthash][ext]'`

- 在module里面的rules内部添加

  ```js
  generator:{
  	filename： 'images/test.png'
  } 
  ```

以上两者同时执行，第二个优先级更高一点

### inline

用于导出一个资源的data URL(base 64)，不会生成新的文件

```js
module: {
        rules: [
            {
                test:/\.jpg$/,
                type: 'asset/inline',
            }
        ]
    }
```

### source

用于导出资源的源代码

```js
module: {
        rules: [
            {
                test:/\.jpg$/,
                type: 'asset/source',
            }
        ]
    }
```

### 通用资源类型assets

会在inline和resource中自动选择

默认情况下，文件大于8k此类型就会默认选择resource

```js
module: {
        rules: [
            {
                test: /\.jpg$/,
                type: 'asset',
                parser: {
                    dataUrlCondition: {
                    	// 设置临界值 改为4M
                        maxSize: 4 * 1024 * 1024
                    }
                }
            }
        ]
    }
```

## 管理资源

### loader

webpack默认只识别js和json类型文件，通过loader可以使webpack识别其他类型文件

#### 加载css类型文件

先下载css-loader依赖

```bash
npm install css-loader -D
```

在webpack.confog.js中添加如下配置

```js
module: {
        rules: [
            {
                test: /\.css$/,
                use: 'css-loader',
            }
        ]
    }
```

到此再用webpack打包就不会出现错误了，但是还不能将样式挂载到页面元素上，所以还需要安装一个loader

```bash
npm install style-loader -D
```

webpack.config.js中做如下更改

```js
module: {
        rules: [
            {
                test: /\.css$/,
                // 这里的顺序不要更改，因为需要先用css-loader打包css文件确定没问题之后再用style-loader把样式挂载到页面上
                use: ['style-loader', 'css-loader'],
            }
        ]
    }
```

除此之外我们想引入less，scss等这种类型的css文件也是可以的，我们拿less举例

先安装less-loader和less

```bash
npm install less-loader less -D 
```

然后在webpack.config.js中做如下更改

```js
module: {
        rules: [
            {
                test: /\.(css|less)$/,
                // 顺序同样不能修改，因为需要将less文件转换问css文件，再去确定css文件没问题之后再挂载到页面上
                use: ['style-loader', 'css-loader', 'less-loader'],
            }
        ]
    }
```

### 抽离和压缩css

#### 抽离

当前的css是在style标签中存在的，我们想把它变为link引入需要安装以下插件

```
npm install mini-css-extract-plugin -D
```

在webpack.config.js中加入如下配置

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            // 决定打包后的出口位置
            filename: 'styles/[contenthash].css'
        })
    ],
    module: {
        rules: [
            {
                test: /\.(css|less)$/,
            	// 这里使用的是当前插件的loader来挂载样式
                use: [MiniCssExtractPlugin.loader , 'css-loader', 'less-loader'],
            }
        ]
    }
}
```

#### 压缩

安装此代码

```bash
npm install css-minimizer-webpack-plugin -D
```

```
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
module.exports = {
    mode: 'production',
    optimization: {
        minimizer: [
            new CssMinimizerPlugin()
        ]
    }
}
```

### 加载images图片资源  

直接在css文件中添加类，里面写入`background-image: url();`在引入到页面上即可实现

### 加载fonts字体

在webpack.config.js中添加如下代码即可

```js
module: {
        rules: [
            // 加载fonts字体
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/,
                type: 'asset/resource'
            }
        ]
    },
```

### 加载数据

安装以下依赖

```bash
npm install csv-loader xml-loader -D
```

在webpack.config.js中加入如下配置即可使用

```js
module: {
        rules: [
            {
                test: /\.(csv|tsv)$/,
                use: 'csv-loader'
            },
            {
                test: /\.xml$/,
                use: 'xml-loader'
            }
        ]
    },
```

### 自定义JSON模块parser

首先安装以下依赖

```bash
npm install toml yaml json5 -D
```

在webpack.config.js中添加如下代码

```js
const toml = require('toml');
const yaml = require('yaml');
const json5 = require('json5')
module.exports = {
    module: {
        rules: [
            {
                test: /\.toml$/,
                type: 'json',
                parser: {
                    parse: toml.parse,
                }
            },
            {
                test: /\.yaml$/,
                type: 'json',
                parser: {
                    parse: yaml.parse
                }
            },
            {
                test: /\.json5$/,
                type: 'json',
                parser: {
                    parse: json5.parse
                }
            }
        ]
    },
}
```

## 使用babel-loader

需要安装三个插件

- babel-loader: 在webpack里应用babel解析ES6的桥梁
- @babel/core: babel核心模块
- @babel/preset-env: babel预设，一组babel插件的集合

安装指令如下

```bash
npm install babel-loader @babel/core @babel/preset-env -D
```

在webpack.config.js中添加以下配置

```js
module: {
        rules: [
            {
                test: /\.js$/,
                // 不对依赖中的js文件进行处理
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
        ]
    },
```

虽然webpack可以完成对js模块的打包，但是并没法完成对js代码的转换，比如一些浏览器只支持ES5语法，需要把我们写的代码中ES6的部分语句转换成ES5，这时babel-loader就很重要了

## 代码分离

### 入口起点

修改webpack.config.js中的一些属性

```js
const path = require('path')
module.exports = {
    // 入口文件设置
    entry: {
        // 配置多入口
        index: './src/index.js',
        another: './src/another-module.js'
    },
    // 出口位置
    output: {
        // 多入口文件打包生成的文件名做区分
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, './dist'),
        clean: true,
        assetModuleFilename: 'image/[contenthash][ext]'
    },
}
```

如此即可实现，但是如果两个入口文件含有相同的依赖项时，打包的体积会很大。以下方法可处理这个问题

- 使用import&&dependOn

  ```js
  module.exports = {
      // 入口文件设置
      entry: {
          index: {
              import: './src/index.js',
              // 抽离代码中相同部分
              dependOn: 'shared'
          },
          another: {
              import: './src/another-module.js',
              dependOn: 'shared'
          },
          shared: 'loadsh'
      },
  }
  ```

- 使用webpack内置插件split-chunks-Plugin

  ```js
  module.exports = {
      // 入口文件设置
      entry: {
          index: './src/index.js',
          another: './src/another-module.js'
      },
      optimization: {
          splitChunks: {
              chunks: 'all'
          }
      }
  }
  ```

- 动态导入

  使用import()方法

  ```js
  function getComponent() {
      return import('loadsh').then(({default: _}) => {
          const element = document.createElement('div');
          element.innerHTML = _.join(['Hello', 'webpack', ' '])
          return element
      })
  }
  
  getComponent().then((element) => {
      document.body.appendChild(element)
  })
  ```

  再配合webpack.config.js里面的配置即可完成简单动态导入

### 懒加载

在一个js文件中加入以下代码，点击时才会引入文件，即需要的时候再引入，会使代码首屏加载更加迅速

```js
const btn = document.createElement('button');
btn.textContent = '点击执行加法运算'
btn.addEventListener('click', () => {
    // 路径前代码是设置生成打包文件的名称
    import(/* webpackChunkName: 'math' */'./math.js').then(({ add }) => {
        console.log(add(4,5));
    })
})
document.body.appendChild(btn)
```

### 预获取/预加载模块

在声明import时，使用下面指令可以让webpack输出"resource hint(资源提示)"

- prefetch(预获取)：将来某些导航下可能需要的资源

  ```js
  btn.addEventListener('click', () => {
      import(/* webpackPrefetch: true */'./math.js').then(({ add }) => {
          console.log(add(4,5));
      })
  })
  ```

  这样可以让浏览器在加载完页面之后，空闲时间去预获取此文件，比懒加载还要优秀

- preload(预加载)：当前导航下可能需要资源

  ```js
  btn.addEventListener('click', () => {
      import(/* webpackPreload: true */'./math.js').then(({ add }) => {
          console.log(add(4,5));
      })
  })
  ```

  和懒加载有点相似，所以最推荐还是prefetch

## 缓存

我们打包输出的文件，如果名字不会修改，浏览器会默认缓存，也就是认为我们没有修改此文件，所以为了解决此问题，进行如下更改即可。

```js
output: {
        filename: '[name].[contenthash].js',
    },
```

每次打包都会更新新的哈希字符串作为文件名，就不会存在缓存的问题

### 缓存第三方库

由于抽离出来的第三方库代码内容不会改变，所以要进行缓存，可以进行如下更改

```js
optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                }
            }
        }
    }
```

### 将所有的js文件放到一个单独的文件夹中

```js
output: {
        filename: 'scripts/[name].[contenthash].js',
    },
```

## 拆分开发环境和生产环境的配置

### 公共路径

在webpack.config.js中添加如下配置

```js
module.exports = {
	// 出口位置
    output: {
        publicPath: 'http://localhost:8080/'
    },
}
```

### 环境变量

去灵活改变当前代码是在开发环境还是生产环境

将module.exports写成一个函数，并且会接受一个参数env

```js
mode: env.production?'production':'development',
```

将里面的mode改成这样可以动态改变生产环境和开发环境，执行代码后，发现文件成功打包但是没有压缩，这是由于webpack有内置压缩js代码的方法，但是我们配置了压缩css文件的插件，所以需要重新配置一下js压缩

```bash
npm install terser-webpack-plugin -D
```

下载完成要在配置文件中添加如下属性

```js
const TerserPlugin = require('terser-webpack-plugin')
module.exports = (env) => {
    console.log(env);
    return {
        optimization: {
            minimizer: [
                new TerserPlugin()
            ],
        }
    }
}
```

### 拆分配置文件

创建`webpack.config.dev.js`和`webpack.config.prod.js`，做一些相应的修改即可

### npm脚本

在package.json添加如下代码

```json
"scripts": {
        "start": "webpack serve -c ./config/webpack.config.dev.js",
        "build": "webpack -c ./config/webpack.config.prod.js"
    },
```

即可用npm run start或者npm run build来执行开发环境和生产环境打包。

生产环境下会出现文件过大的警告，可以添加如下代码解决，但是文件体积过大时也要注意

```js
performance: {
	hint: false
}
```

### 提取公共配置

安装一个依赖 webpack-merge

```
npm install webpack-merge -D
```

安装完成之后，在config文件夹中添加四个文件

- webpack.config.common.js

  公共代码部分

- webpack.config.dev.js

  开发环境部分

- webpack.config.prod.js

  生产环境部分

- webpack.config.js

  合并部分，代码如下

  ```js
  const { merge } = require('webpack-merge');
  
  const commonConfig = require('./webpack.config.common');
  const productionConfig = require('./webpack.config.prod');
  const developmentConfig = require('./webpack.config.dev');
  
  module.exports = (env) => {
      switch(true) {
          case env.development:
              return merge(commonConfig, developmentConfig)
  
          case env.production:
              return merge(commonConfig, productionConfig)
  
          default:
              return new Error('No matching configuration was found')
      }
  }
  ```


## loader和plugin的区别

### loader

loader只关注转化文件这一个领域，完成压缩、打包、语言翻译，**仅仅是为了打包**

举例：

- css-loader和style-loader模块是为了打包css
- babel-loader和babel-core模块是为了把ES6代码转成ES5
- url-loader和file-loader是把图片进行打包

### plugin

plugin也是为了扩展webpack的功能，但是plugin是作用在webpack本身上的，他才可以处理各种各样的任务，在优化中经常会用到

### 运行时机角度区分

- loader运行在打包文件之前
- plugins在整个编译周期都起作用

## webpack的打包过程

- 初始化参数

  从配置文件中和shell语句中读取与合并参数，得到最终参数

- 开始编译

  用上一步得到的参数初始化Compiler对象，加载所有配置的插件，执行对象的run方法开始执行编译

- 确定入口

  根据配置中的entry找出所有的入口文件

- 编译模块

  从入口文件触发，调用所有配置的loader对模块进行编译，再找出该模块依赖的模块，再递归本步骤，直到所有入口依赖的文件都经过本步骤的处理

- 完成模块编译

  在经过第四个步骤使用loader编译完所有模块后，得到每个模块和编译后的最终输出内容以及它们之间的依赖关系

- 输出资源

  根据入口和模块之间的依赖关系，组装成一个个包含多个模块的Chunk，再把每个Chunk转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会

- 输出完成

  再确认好输出内容后，根据配置确定输出的路径和文件名，把文件内容写进到文件系统中

## 如何提升webpack的构建速度

- 优化loader配置

  再使用loader时，可以配置include、exclude、test属性配置文件

- 合理使用resolve.extensions

  通过此API解析到文件时自动添加扩展名

- 优化resolve.modules

  用于配置让webpack更快的寻找到存放第三方模块的文件夹

- 优化resolve.alias

  此API用来给常用的路径起一个别名，例如最常用的@

- DLLPlugin插件

  DLL全称为动态链接库，就是可以实现多项目共享一部分代码

- 使用cache-loader

  一些性能开销较大的loader之前添加cache-loader，将结果缓存到磁盘里，显著提升二次构建速度
