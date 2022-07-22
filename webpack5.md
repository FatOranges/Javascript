# Webpack5

## npm

```shell
#全局安装 webpack 5.72.1
npm i webpack -g
```

## create-react-app

### npm

```shell
#全局下载脚手架 5.0.1
npm i create-react-app -g

#初始化项目。注所需 npm 版本 6+
npm init react-app my-app
```

### package.json

```json
{
    "scripts": {
        "start": "react-scripts start",//开发环境
        "build": "react-scripts build",//生产环境
        "test": "react-scripts test",//测试环境
        "eject": "react-scripts eject"//将 webpack 配置暴露到外面。不可逆运行
      }
}
```

### paths.js

```js
const path = require('path');
const fs = require('fs');
const getPublicUrlOrPath = require('react-dev-utils/getPublicUrlOrPath');
// 项目根目录
const appDirectory = fs.realpathSync(process.cwd());
// 生成绝对路径的方法
const resolveApp = relativePath => path.resolve(appDirectory, relativePath);
```

#### 资源的公共访问路径

1. package.json ——> homepage
2. process.env.PUBLIC_URL

#### 不开启 sourcemap

##### npm

```shell
npm i cross-env -D
```

##### package.json

```json
{
    "scripts": {
        "start": "cross-env GENERATE_SOURCEMAP=false node scripts/start.js",
        "build": "node scripts/build.js",
        "test": "node scripts/test.js"
      }
}
```

### Error

==This git repository has untracked files or uncommitted changes:==

> 提交 git 即可解决

## @vue/cli

### npm

```shell
# 全局下载脚手架
npm i @vue/cli -g

npm i @vue/cli-service -g

# 初始化项目
vue create my-vue-project
```

### 输出相关配置文件

```shell
# 开发环境
vue inspect --mode==development > webpack.dev.js

# 生产环境
vue inspect --mode=production > webpack.prod.js
```

### 配置 loader 解析规则

```js
module.exports = {
    resolveLoader: {
        modules: [
            'node_modules',
            path.resolve(__dirname, 'loaders')
        ]
    }
}
```



