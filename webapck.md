### Webpack

#### 官方文档地址

https://www.webpackjs.com

#### 局部安装 webpack

```shell
#进入项目根目录下
npm init	#初始化项目
npm i webpack@4.20.2 --save-dev
npm i webpack-cli@3.1.2 --save-dev
npm i webpack-dev-server@3.1.9 --save-dev
```

#### 打包单个文件

##### 打包单个文件报错的参考地址

https://blog.csdn.net/weixin_42243479/article/details/101100756

```shell
webpack hello.js -o hello.bundle.js	# -o 为高版本语法
```

<img src="D:\typora-img\webpack\image-20220429235045315.png" alt="image-20220429235045315" style="zoom:50%;" />

##### 打包信息

1. Hash：打包后的文件的 hash 值。
2. Version： webpack 版本。
3. Time：打包所花费的时间。

- Asset：本次打包生成的文件。
- Size：本次打包生成的文件的大小。
- Chunks：本次打包的分块。
- Chunk Names：本次打包的名称。

##### 打包警告

缺少模式设置，由于 webpack4 引入模式，分为以下三种模式。

1. 无
2. development
3. production

##### 如何去除黄色警告——指定打包模式。

```shell
webpack hello.js -o hello.bundle.js --mode development# -o 为高版本语法，并指定打包的模式
```

<img src="D:\typora-img\webpack\image-20220430113025493.png" alt="image-20220430113025493" style="zoom:50%;" />

#### 打包默认目录下的文件

默认打包 src/index.js 文件，打包后默认生成至 dist/main.js 文件。

```shell
npm run dev	#开发者模式
```

==注：输入上述命令前请在 package.json 中完成以下配置==

```json
{
    "scripts": {
        "dev": "webpack --mode development",
        "build": "webpack --mode production"
     },
}
```

#### 文件引入方式

1. require()
2. import ... from ...

```js
//hello.js
require('./world.js');
require('./helllo.css');

function main(param) {
    console.log(param);
}

main("test");
```



#### 初始化 webpack 项目

```shell
npm i webpack --save-dev
```

