# Webpack 详解

## entry

### 语法

1. string：entry: string（单入口）
   - 打包形成一个 chunk。输出一个 bundle 文件。
   - chunk 的默认名称 main。
2. array：entry: [string]（多入口）
   - 所有文件最终只会形成一个 chunk，输出出去只有一个 bundle 文件。
   - 只有在 HMR 功能中让 html 热更新生效。
3. object：entry: {}（多入口）
   - 有几个入口文件就生成几个 chunk，输出几个 bundle 文件。
   - 此时 chunk 的名称是 key。
   - 特殊用法：object + array

```js
module.exports = {
    entry: {
        //所有入口文件最终形成一个 chunk，输出出去只有一个 bundle 文件。
        index: ['./src/js/index.js', './src/js/add.js'],
        count: './src/js/count.js'
    }
}
```

### 可配置的属性—object

- `dependOn`: 当前入口所依赖的入口。它们必须在该入口被加载前被加载。
- `filename`: 指定要输出的文件名称。
- `import`: 启动时需加载的模块。
- `library`: 指定 library 选项，为当前 entry 构建一个 library。
- `runtime`: 运行时 chunk 的名字。如果设置了，就会创建一个新的运行时 chunk。在 webpack 5.43.0 之后可将其设为 `false` 以避免一个新的运行时 chunk。
- `publicPath`: 当该入口的输出文件在浏览器中被引用时，为它们指定一个公共 URL 地址。

## output

## module

## plugins

## mode