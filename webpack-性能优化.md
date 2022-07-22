# Webpack 性能优化

## 开发环境性能优化

### 优化打包构建速度

#### HMR

> HMR：hot module replacement 热模块替换。
>
> 作用：一个模块发生变化，只会重新打包这一模块，而不是打包所有模块，极大的提升了构建速度。
>
> 样式文件：可以使用 HMR 功能，因为 style-loader 内部实现了。
>
> js 文件：默认不能使用 HMR 功能。HMR 功能只能处理非入口 js 文件的其他文件。
>
> html 文件：默认不能使用 HMR 功能，同时会导致 html 文件不能热更新了。

##### webpack.config.js

```js

module.exports = {
    //解决 html 文件不能热更新问题。
    entry: ['./src/js/index.js', './src/index.html'],
    
    devServer: {
        contentBase: resolve(__dirname, 'build'),
        compress: true,
        port: 3000,
        open: true,
        //开启 HMR 功能
        hot: true
    }
}
```

##### js 文件热更新

```js
import print from './print.js';

print();

if (module.hot) {
    module.hot.accept('./print.js', function () {
        //监听 print.js 文件的变化，一旦发生变化，其他模块不会重新打包构建。会执行后面的回调函数。
        print();
    })
}
```

### 优化代码调试

#### source-map

> 一种提供源代码到构建后代码映射技术。如果构建后代码出错了，通过映射可以追踪源代码错误。

##### webpack.config.js

```js
module.exports = {
    /*
    可能值：
    [inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map
    source-map：外部。能够提示错误代码的准确信息和源代码的错误位置。
    inline-source-map：内联。只生成一个内联 source-map。与 source-map 效果一致。
    hidden-source-map：外部。只能显示错误代码错误原因，但是没有错误位置。不能追踪源代码错误，只能提示到构建后代码的错误位置。(只隐藏源代码，会提示构建后代码的错误信息)
    eval-source-map：内联。每个文件都生成对应的 source-map，都在 eval 中。与 source-map 效果一致，展示时多了文件的 hash 值。
    nosources-source-map：外部。只能展示错误信息，没有任何源代码信息。(全部隐藏)
    cheap-source-map：外部。能够提示错误代码的准确信息和源代码的错误位置。但只能精确到行，无法精确到列。
    cheap-module-source-map：外部。能够提示错误代码的准确信息和源代码的错误位置。module 会将 loader 的 source-map 加入。
    
    速度：eval-cheap > eval > inline > cheap
    调试：source-map > cheap-module-source-map > cheap-source-map
    */
    //开发环境
    devtool: "eval-source-map"
    //生产环境
    //devtool: "source-map"
}
```

## 生产环境性能优化

### 优化打包构建速度

#### oneOf

##### webpack.config.js

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
                //以下 loader 只会匹配一个，不能有两个配置处理同一种类型的文件。
                oneOf: [
                    {
                        test: /\.css$/,
                        use: [...commonCssLoader]
                    },
                    {
                        test: /\.less$/,
                        use: [...commonCssLoader, 'less-loader']
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

#### 缓存

1. 对 babel 进行缓存

   cacheDirectory: true

2. 对静态资源进行缓存

   - hash
   - chunkhash
   - ==contenthash==

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
                                useBuiltIns: 'usage',
                                corejs: {
                                    version: '3'
                                },
                                targets: {
                                    chrome: '60',
                                    firefox: '60',
                                    ie: '9',
                                    safari: '10',
                                    edge: '17'
                                }
                            }
                        ]
                    ],
                    cacheDirectory: true
                }
            }
        ]
    }
}
```

#### 多线程打包

##### npm

```shell
npm i thread-loader@2.1.3 -D
```

##### webpack.config.js

```js
const {resolve} = require('path');

//定义 nodejs 环境变量，决定使用 browserslist 的哪个环境。
process.env.NODE_ENV = 'production';

module.exports = {
	entry: './src/js/index.js',
    output: {
        filename: 'js/built.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: 'thread-loader',
                        options: {
                            workers: 2
                        }
                    },
                    {
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
                    }
                ]
            }
        ]
    },
    plugins: [],
    mode: "production"
}
```

#### externals

##### webpack.config.js

```js
const {resolve} = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/js/index.js',
    output: {
        filename: 'built.js',
        path: resolve(__dirname, 'build')
    },
    module: {
        
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    mode: 'production',
    externals: {
        //拒绝 jQuery 被打包。换为手动引用的方式。
        jquery: 'jQuery'
    }
}
```

#### dll

> 运行命令：webpack --config webpack.dll.js

##### npm

```shell
npm i add-asset-html-webpack-plugin@3.1.3 -D
```

##### webpack.dll.js

```js
/*
使用 dll 技术，对某些库（第三方库：jQuery、Vue、react...）进行单独打包。
*/
const {resolve} = require('path');
const webpack = require('webpack');

module.exports = {
    entry: {
        //最终打包生成的 [name]——>jquery
        //['jquery']——>要打包的库是 jquery
        jquery: ['jquery']
    },
    output: {
        filename: '[name].js',
        path: resolve(__dirname, 'dll'),
        library: '[name]_[hash]'//打包的库里面向外暴露出去的内容叫什么名字。
    },
    plugins: [
        //打包生成一个 manifest.json 提供和 jquery 映射。
        new webpack.DllPlugin({
            name: '[name]_[hash]',//映射库的暴露的名称。
            path: resolve(__dirname, 'dll/manifest.json')//输出文件路径
        })
    ],
    mode: 'production'
}
```

##### webpack.config.js

```js
const {resolve} = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

module.exports = {
    entry: './src/js/index.js',
    output: {
        filename: 'built.js',
        path: resolve(__dirname, 'build')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        }),
        //告诉 webpack 哪些库不参与打包，同时使用的名称也得变。
        new webpack.DllReferencePlugin({
            manifest: resolve(__dirname, 'dll/manifest.json')
        }),
        //将某个文件打包输出去，并在 html 中自动引入该资源。
        new AddAssetHtmlWebpackPlugin({
            filepath: resolve(__dirname, 'dll/jquery.js')
        })
    ],
    mode: 'production'
}
```

### 优化代码运行的性能

#### tree shaking

> 去除无用代码，减少代码体积。
>
> 前提：
>
> - 必须使用 ES6 模块化
> - 开启 production 环境
>
> 在 package.json 中配置，"sideEffects": false 表示所有代码都没有副作用（都可以进行 tree shaking）
>
> 可能会引发的问题：可能会把 css / @babel/polyfill 文件给干掉
>
> 解决："sideEffects": ["\*.css", "\*.less"]

#### code split

> 代码分隔

##### entry 多入口

###### webpack.config.js

```js
const {resolve} = require('path');
//打包 html 文件
const HtmlWebpackPlugin = require('html-webpack-plugin');


module.exports = {
    //单入口
	//entry: './src/js/index.js',
    //多入口
    entry: {
      	main: './src/js/index.js',
        test: './src/js/test.js'
    },
    output: {
        //[name]：取文件名，与 entry 中的配置有关（key）
        filename: 'js/[name][contenthash:10].js',
        path: resolve(__dirname, 'build')
    },
    plugins: [
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

##### optimization.splitChunks

###### webpack.config.js

```js
const {resolve} = require('path');
//打包 html 文件
const HtmlWebpackPlugin = require('html-webpack-plugin');


module.exports = {
    //单入口
	//entry: './src/js/index.js',
    //多入口
    entry: {
      	index: './src/js/index.js',
        test: './src/js/test.js'
    },
    output: {
        //[name]：取文件名，与 entry 中的配置有关（key）
        filename: 'js/[name][contenthash:10].js',
        path: resolve(__dirname, 'build')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: {
                collapseWhitespace: true,
                removeComments: true
            }
        })
    ],
    /*
    可以将 node_modules 中代码单独打包成一个 chunk 最终输出。
    自动分析多入口 chunk 中，有没有公共的文件。如果有会打包成单独的一个 chunk。
    */
    optimization: {
        splitChunks: {
            chunks: all
        }
    }
    mode: "production"
}
```

##### import

###### javascript

```js
/*
通过 js 代码，让某个文件被单独打包成一个 chunk。
import 动态导入语法：能将某个文件单独打包。
*/
//webpackChunkName 重命名加载文件名称。
import(/*webpackChunkName: 'test'*/'./test.js')
    .then(({mul, count}) => {
        //eslint-disable-next-line
    	console.log(mul(1, 2, 3, 4));
    })
	.catch(() => {
		//eslint-disable-next-line
    	console.log('加载失败')
    })
```

#### 懒加载 lazy loading

##### import

###### javascript

```js
//index.js
document.getElementById("btn").onclick = function () {
    //懒加载
    import(/*webpackChunkName: 'test'*/'./test.js').then(({ mul }) => {
        console.log(mul(4, 5));
    })
    //预加载
    import(/*webpackChunkName: 'test', webpackPrefetch: true*/'./test.js').then(({ mul }) => {
        console.log(mul(4, 5));
    })
}
```

#### PWA

##### npm

```shell
npm i workbox-webpack-plugin@5.0.0 -D
```

##### webpack.config.js

```js
const {resolve} = require('path');
const WorkboxWebpackPlugin = require('workbox-webpack-plugin');

module.exports = {
    entry: '',
    output: {
        filename: '',
        path: ''
    },
    module: {
        
    },
    plugins: [
        new WorkboxWebpackPlugin.GenerateSW({
            //1.帮助 serviceworker 快速启动
            //2.删除旧的 serviceworker 
            clientsClaim: true,
            skipWaiting: true
        })
    ],
    mode: 'production'
}
```

##### javascript

```js
//注册 serviceWorker
//处理兼容性问题
if('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWork.register('/service-worker.js')
        	.then(() => {
            	console.log("sw 注册成功");
            })
        	.catch(() => {
            	console.log("sw 注册失败");
        	})
    })
}
```

##### package.json

```json
/**
1.eslint 不认识 window、navigator 全部变量。
2.sw 代码必须运行在服务器上。
	npm i serve -g
	serve -s build 启动服务器，将 build 目录下所有资源作为静态资源暴露出去。
*/
{
    "eslintConfig": {
        "extends": "airbnb-base",
        "env": {
            "browser" : true
        }
    }
}
```