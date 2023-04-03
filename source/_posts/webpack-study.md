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

