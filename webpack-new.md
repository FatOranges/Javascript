# Webpack

## 核心

### Entry

#### 概述

入口（entry）指示 webpack 以哪个文件为入口起点开始打包，分析构建内部依赖图。

### Output

#### 概述

输出（output）指示 webpack 打包后的资源 bundles 输出到哪里去，以及如何命名。

### Loader

#### 概述

loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只能理解 JavaScript）。

### Plugins

#### 概述

插件（plugins）可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等。

### mode

#### 概述

模式（mode）指示 webpack 使用相应模式的配置。

#### 模式选项

| 选项        | 描述                                                         | 特点                         |
| ----------- | ------------------------------------------------------------ | ---------------------------- |
| development | 会将 process.env.NODE_ENV 的值设为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin。 | 能让代码本地调试运行的环境。 |
| production  | 会将 process.env.NODE_ENV 的值设为 production。启用 FlagDependencyUsagePlugin，FlagIncludedChunksPlugin，ModuleConcatenationPlugin，NoEmitOnErrorsPlugin，OccurrenceOrderPlugin，SideEffectsFlagPlugin 和 UglifJsPlugin。 | 能让代码优化上线运行环境。   |

### webpack 版本

```shell
npm i webpack@4.41.6 webpack-cli@3.3.11 -g
```

### 初体验

```shell
#初始化项目
npm init

#安装依赖（默认全局已安装过 webpack）
npm i webpack@4.41.6 webpack-cli@3.3.11 -D

#开发环境打包指定文件。
webpack ./src/index.js -o ./build/build.js --mode-development

#生产环境打包指定文件。
webpack ./src/index.js -o ./build/build.js --mode-production
```

### webpack.config.js

#### 打包样式资源

```js
/**
构建工具都是基于 nodejs 平台运行的，模块化默认采用 commonjs。
*/
const { resolve } = require('path');

module.exports = {
    //入口文件
    entry: './src/index.js',
    //出口配置
    output: {
        //输出文件名
    	filename: 'build.js',
        //输出路径
        path: resolve(__dirname, 'build')
    },
    //loader 配置
    module: {
        rules: [
            //详细的 loader 配置。
            {
                //匹配哪些文件。
                test: /\.css$/,
                //使用哪些 loader 进行处理。
                use: [
                    //use 数组中 loader 执行顺序：从右到左，从上到下依次执行。
                    //创建 style 标签，将 js 中的样式资源插入进行，添加到 head 中生效。
                    'style-loader',
                    //将 css 文件变成 commonjs 模块加载 js 中，里面内容是样式字符串。
                    'css-loader'
                ]
            },
            {
                //匹配哪些文件。
                test: /\.less$/,
                //使用哪些 loader 进行处理。
                use: [
                    //use 数组中 loader 执行顺序：从右到左，从上到下依次执行。
                    //创建 style 标签，将 js 中的样式资源插入进行，添加到 head 中生效。
                    'style-loader',
                    //将 css 文件变成 commonjs 模块加载 js 中，里面内容是样式字符串。
                    'css-loader',
                    //将 less 文件编译成 css 文件。
                    'less-loader'
                ]
            }
        ]
    },
    //plugin 配置
    plugins: [
        //详细 plugin 的配置
    ],
    //模式 配置
    mode: 'development',
    //mode: 'production'
}
```

#### 打包 HTML 文件

```js
const { resolve } = requrie('path');
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'build.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            
        ]
    },
    plugins: [
        //功能：默认会创建一个空的 HTML，自动引入打包输出的所有资源（js/css）。
        new HtmlWebpackPlugin({
            //复制 './src/index.html' 文件，并自动引入打包输出的所有资源（js/css）。
            template: './src/index.html'
        })
    ],
    mode: 'development'
}
```

#### 打包图片资源

```js
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: '',
    output: {
        filename: '',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            {
                test: /\.less$/,
                //使用多个 loader 时，用 use 属性即可。
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            },
            {
                //默认处理不了 html 中的 img 图片。
                test: /\.(jpg|png|gif)$/,
                //使用单个 loader 时，用 loader 属性即可。
                loader: 'url-loader',
                options: {
                    //图片大小小于 8kb，就会被 base64 处理。
                    //优点：减少请求数量（减轻服务器压力）
                    //缺点：图片体积会更大（文件请求速度更慢）
                    limit: 8 * 1024,
                    //关闭 url-loader 的 es6 模块化，使用 commonjs 解析。为解决打包 html 中 img 图片不成功的问题。[object Module]
                    esModule: false,
                    //取图片的 hash 的前 10 位，并取原来文件的扩展名。
                    name: '[hash:10].[ext]'
                }
            },
            {
                test: /\.html$/,
                //处理 html 文件的 img 图片（负责引入 img，从而能被 url-loader 进行处理）。
                loader: 'html-loader'
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    mode: 'development'
}
```

#### 打包其他资源

```js
const { resolve } = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'build.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            {
                //排除 css、js、html、less 文件。
                exclude: /\.(css|js|html|less)$/,
                loader: 'file-loader',
                options: {
                    name: '[hash:10].[ext]'
                }
            }
        ]
    },
    plugins: [
        
    ],
    mode: 'development'
}
```

#### devServer

```shell
npm i webpack-dev-server@3.10.3 -D
```

```js
const { resolve } = require('path');

module.exports = {
    entry: '',
    output: {
        filename: '',
        path: ''
    },
    module: {
        rules: [
            {
                test: '',
                use: [
                    ''
                ]
            }
        ]
    },
    plugins: [
        
    ],
    mode: '',
    //开发服务器 devServer：用来自动化（自动编译，自动打开浏览器，自动刷新浏览器）。
    //特点：只会在内存中编译打包，不会有任何输出。
    //启动 devServer 指令为：npx webpack-dev-server.
    devServer: {
        contentBase: resolve(__dirname, 'build'),
        //启动gzip压缩
        compress: true,
        //端口号
        port: 8088,
        //自动打开浏览器
        open: true,
    }
}
```

### 开发环境配置

```js
const {resolve} = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'build.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    mode: 'development',
    devServer: {
        contentBase: resolve(__dirname, 'build'),
        compress: true,
        port: 8088,
        open: true
    }
}
```

### 构建环境介绍

#### 提取 CSS 成单独文件

```js
const {resolve} = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    entry: "./src/js/index.js",
    output: {
        filename: "js/build.js",
        path: resolve(__dirname, "build")
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    //创建 style 标签，然后插入头部
                    // 'style-loader',
                    //取代 style-loader，作用：提取 js 中的 css 成单独文件。
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            //指定生成模板文件
            template: './src/index.html'
        }),
        new MiniCssExtractPlugin({
            //对输出的文件重命名，默认 main.css
            filename: 'css/build.css'
        })
    ],
    mode: "development"
}
```

#### CSS 兼容处理

##### package.json

```json
{
    "browserslist": {
        "development": [
          "last 1 chrome version",
          "last 1 firefox version",
          "last 1 safari version"
        ],
        "production": [
          ">0.2%",
          "not dead",
          "not op_mini all"
        ]
      }
}
```

##### webpack.config.js

```json
const {resolve} = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

//设置 nodejs 环境变量
process.env.NODE_ENV = 'development';

module.exports = {
    entry: "./src/js/index.js",
    output: {
        filename: "js/build.js",
        path: resolve(__dirname, "build")
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    //创建 style 标签，然后插入头部
                    // 'style-loader',
                    //取代 style-loader，作用：提取 js 中的 css 成单独文件。
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    //css 兼容性处理，postcss：postcss-loader postcss-preset-env
                    //browserslist 默认为生产环境，开发环境设置：process.env.NODE_ENV = 'development';
                    {
                        loader: "postcss-loader",
                        options: {
                            ident: 'postcss',
                            plugins: () => {
                                //postcss 插件
                                require('postcss-preset-env')()
                            }
                        }
                    }
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            //指定生成模板文件
            template: './src/index.html'
        }),
        new MiniCssExtractPlugin({
            //对输出的文件重命名，默认 main.css
            filename: 'css/build.css'
        })
    ],
    mode: "development"
}
```

#### JS 语法检查 eslint

##### 需要准备的东西

1. eslint-config-airbnb
2. eslint
3. eslint-plugin-import

##### 其他小技巧

==JS 中==

1. 下一行不进行 eslint 检查——//eslint-disable-next-line

##### package.json

```json
{
    "eslintConfig": {
        "extends": "airbnb-base",
    }
}
```

##### webpack.config.js

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'eslint-loader',
                options: {
                    //自动修复 eslint 的错误
                    fix: true
                }
            }
        ]
    }
}
```

#### JS 兼容性处理

##### webpack.config.js

```js
module.exports = {
    module: {
        rules: [
            {
                //js 兼容性处理：@babel/core babel-loader @babel/preset-env
                //基本兼容性处理：@babel/preset-env，但只能转换基本语法，如 promise 不能转换。
                //全部 js 处理：@babel/polyfill
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    //预设：指示 babel 做怎样的兼容性处理
                    presets: [
                        [
                            '@babel/preset-env',
                            {
                                //按需加载
                                useBuiltIns: 'usage',
                                //指定 core-js 版本
                                corejs: {
                                    version: '3'
                                },
                                //指定兼容性做到哪个版本的浏览器
                                targets: {
                                    chrome: '60',
                                    firefox: '60',
                                    ie: '9',
                                    safari: '10',
                                    edge: '17'
                                }
                            }
                        ]
                    ]
                }
            }
        ]
    }
}
```

##### javascript

```js
//只在全部 js 处理时使用
import '@babel/polyfill';
```

#### HTML 文件压缩

```js
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
                //移除空格
                collapseWhitespace: true,
                //移除注释
                removeComments: true
            }
        })
    ]
}
```

### 生产环境配置

#### webpack.config.js

```js
const {resolve} = require('path');
//打包成单个 css 文件。
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
//压缩 css 文件
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
//打包 html 文件
const HtmlWebpackPlugin = require('html-webpack-plugin');

//定义 nodejs 环境变量，决定使用 browserslist 的哪个环境。
process.env.NODE_ENV = 'production';

//抽取 css 公共配置。
const commonCssLoader = [
    MiniCssExtractPlugin.loader,
    'css-loader'
    {
    	//需要在 package.json 中定义 browserslist
        loader: 'postcss-loader',
    	options: {
    		idents: 'postcss',
    		plugins: () => [require('postcss-preset-env')()]
        }
    }
];

module.exports = {
	entry: './src/js/index.js',
    output: {
        filename: 'js/built.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [...commonCssLoader]
            },
            {
                test: /\.less$/,
                use: [...commonCssLoader, 'less-loader']
            },
            {
                //在 package.json 中 eslintConfig ——> airbnb
                test: /\.js$/,
                exclude: /node_modules/,
                //优先执行
                enforce: 'pre',
                loader: 'eslint-loader',
                options: {
                    //自动修复错误
                    fix: true
                }
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    presets: [
                        [
                            '@babel/preset-env',
                            {
                                useBuiltIns: 'usage',
                                corejs: {
                                    version: 3
                                },
                                targets: [
                                    chrome: '60',
                                    firefox: '50'
                                ]
                            }
                        ]
                    ]
                }
            },
            //打包图片
            {
                test: /\.(jpg|png|gif)$/,
                loader: 'url-loader',
                options: {
                    limit: 8 * 1024,
                    name: '[hash:10].[ext]',
                    outputPath: 'images'
                },
                //使用 html-loader 必须关闭 esModule
                esModule: false
            },
            //打包 html 中的图片资源
            {
                test: /\.html$/,
                loader: 'html-loader'
            },
            {
                exclude: '/\.(js|css|less|html|jpg|png|gif)$/',
                loader: 'file-loader',
                options: {
                    outputPath: 'media'
                }
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/built.css'
        }),
        new OptimizeCssAssetsWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
                collapseWhitespace: true,
                removeComments: true
            }
        })
    ],
    mode: "production"
}
```

#### package.json

```json
{
 	"browserslist": {
        "development": [
            "last 1 chrome version",
            "last 1 firefox version",
            "last 1 safari version"
        ],
        "production": [
            ">0.2%",
            "not dead",
            "not op_mini all"
        ]
    },
    "eslintConfig": {
        "extends": "airbnb-base"
    }
}
```

### loader & plugin

```shell
# css-loader style-loader 需要配套使用，前者解析 css 文件，后者将解析好的 css 文件添加至 head 中。
npm i css-loader@3.4.2 style-loader@1.1.3 -D

# less 和 less-loader 需要同时下载配套使用。
npm i less@3.11.1 less-loader@5.0.0 -D

# 打包 html 文件
npm i html-webpack-plugin@3.2.0 -D

# 打包图片文件 url-loader 和 file-loader 配套使用
npm i url-loader@3.0.0 file-loader@5.0.2 html-loader@0.5.5 -D

# 热部署
npm i webpack-dev-server@3.10.3 -D

# 提取 CSS 成单独文件
npm i mini-css-extract-plugin@0.9.0 -D

# css 兼容性处理
npm i postcss-loader@3.0.0 postcss-preset-env@6.7.0 -D

# 压缩 css
npm i optimize-css-assets-webpack-plugin@5.0.3 -D

# eslint
npm i eslint@6.8.0 eslint-loader@3.0.3 eslint-config-airbnb-base eslint-plugin-import -D
npm i eslint eslint-loader eslint-config-airbnb-base eslint-plugin-import -D

# js 兼容性处理——基本兼容性处理
npm i @babel/core@7.8.4 babel-loader@8.0.6 @babel/preset-env@7.8.4 -D

# js 兼容性处理——全部兼容性处理
npm i @babel/polyfill@7.8.3 -D

# js 兼容性处理——按需加载
npm i core-js@3.6.4 -D
```

