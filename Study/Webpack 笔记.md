# Webpack 笔记

> 从0开始配置webpack

[TOC]

## 初始化项目

```javascript
npm init
yarn add vue
yarn add webpack webpack-cli -D
```

大概初始化完项目，开始手写配置。

发现缺少vue-loader

```javascript
yarn add vue-loader -D
```

执行webpack

<img src="Webpack%20%E7%AC%94%E8%AE%B0.assets/image-20200616102219178.png" alt="image-20200616102219178" style="zoom:60%;" />

发现报错了。

1. 第一个错误：需要安装vue-template-compiler，来编译模板文件(版本号需要跟vue保持一致)。
2. 第二个错误：Vue-loader在15.*之后的版本都是 vue-loader的使用都是需要伴生 VueLoaderPlugin的.![image-20200616111902547](Webpack%20%E7%AC%94%E8%AE%B0.assets/image-20200616111902547.png)

再次执行webpack，发现编译成功了。

手动打开index.html，发现“hello”出现在page中。

## 完善编译结果

上述需要手动把打包编译后的js文件加入到index.html中，然后打开index.html，非常的不方便。

我希望能够一键将html和js都编译好，怎么做？

使用Webpack的一个Plugin: **HtmlWebpackPlugin**。

直接引入使用，会创建一个index.html，并生成在output的path指定的位置（如果没有指定，则默认是./dist目录）。

### 如果想要指定Html模板呢？

HtmlWebpackPlugin 提供了很多参数

> https://www.webpackjs.com/plugins/html-webpack-plugin/
>
> https://github.com/jantimon/html-webpack-plugin#configuration

```javascript
new htmlWebpackPlugin({
  title: 'webpack-practice',
  template: 'index.html',	// 指定根目录下的index.html为模板
})
```

修改index.html，将title改成

```html
<title><%= htmlWebpackPlugin.options.title %></title>
```

生成的html的title就会是htmlWebpackPlugin中配置的title了。

同时删掉`<script src="bounce.js" />`标签，因为插件已经帮我注入(inject)了。

> 其他参数请自行尝试

## 增加对Sass的支持

由前面对vue的解析，可以推断出，要对Sass进行解析，也需要对应的loader。

> https://www.webpackjs.com/loaders/sass-loader/

通过将 [style-loader](https://github.com/webpack-contrib/style-loader) 和 [css-loader](https://github.com/webpack-contrib/css-loader) 与 sass-loader 链式调用，可以立刻将样式作用在 DOM 元素。

```javascript
{
  test: /\.scss$/,
  use: [
    { loader: 'style-loader' }, // 将 JS 字符串生成为 style 节点
    { loader: 'css-loader' }, // 将 CSS 转化成 CommonJS 模块
    { loader: 'sass-loader' }, // 将 Sass 编译成 CSS
  ]
}
```

给index.vue增加scss的style。

```vue
<style lang="scss" scoped>
  .main {
    .title {
      font-weight: bold;
    }
  }
</style>
```

执行webpack后，马上能够看到效果。

### 使用SASS全局变量

> 目标：我们定义SCSS的全局变量，然后在任意vue component中直接使用(不需要import)。

> 参考sass-loader https://github.com/webpack-contrib/sass-loader

可以看到，loader能够传递参数，sass-loader有prependData参数，能够指定全局变量。

```javascript
{
  test: /\.s[ac]ss$/i,
    use: [
      { loader: 'style-loader' }, // 将 JS 字符串生成为 style 节点
      { loader: 'css-loader' }, // 将 CSS 转化成 CommonJS 模块
      { 
        loader: 'sass-loader', 
        options: {
          prependData: `$title-font-weight: bold;`
        } 
      }, // 将 Sass 编译成 CSS
    ]
}
```

同时把index.vue中的样式调整下：

```vue
<style lang="scss" scoped>
  .main {
    .title {
      font-weight: $title-font-weight;
    }
  }
</style>
```

> 提问：如果要做到，引入一个全局的SCSS文件，怎么做？



## 图片

图片也是一类文件格式，用到的是`file-loader`。

### 资源目录

直接打包后图片是在根路径下的，我需要让他待在资源目录下，方便分层。

可以使用`file-loader`的参数达到目的

> https://webpack.js.org/loaders/file-loader/

```javascript
{
  test: /\.(png|svg|jpe?g|gif)$/,
  use: {
    loader: 'file-loader',
    options: {
      name: '[hash].[ext]',
      outputPath: 'assets/images',
    }
  }
}
```

这样就把图片输出到`assets/images`目录下了



## 分割压缩包

> 目标：split-code，按vendor lib（第三方库）、runtime（vue&webpack等rt）、业务按需加载。

准备：添加lodash、dayjs、ant-design-vue、vuex、vue-router。并在page1-3分别使用。

这里要用到splitChunksPlugin来解决这个需求

> https://webpack.docschina.org/plugins/split-chunks-plugin/

Webpack 4 中有optimization属性，配置优化。

> 使用Plugin：webpack-bundle-analyzer来图形化分析打包后的大小

```javascript
optimization: {
  splitChunks: {
    chunks: 'all',
      cacheGroups: {
        // third party lib, such as lodash, dayjs, 
        lib: {
          test: /[\\/]node_modules[\\/](lodash|dayjs|moment|vue)/,
            priority: 5
        },
          // ant-design
          antd: {
            test: /[\\/]node_modules[\\/](ant-design-vue|@ant-design)/,
              priority: 10
          },
      }
  }
}
```

打包后

![image-20200623134208940](Webpack%20%E7%AC%94%E8%AE%B0.assets/image-20200623134208940.png)

可以看到，antd和指定的第三方库都分别打包进了antd和lib文件中。但是antd的文件太大，而且文件名很奇怪。我们解决这个问题。

![image-20200623134120833](Webpack%20%E7%AC%94%E8%AE%B0.assets/image-20200623134120833.png)







> splitChunks 解析 https://juejin.im/post/5af1677c6fb9a07ab508dabb



## 缓存

> 当我只改动了某一个业务模块时，编译出的文件也应该只有这一部分发生变化。在文件名上的体现就是，只有这个模块对应的文件的id发生变化，其他文件大小内容和文件名都没变。这样每次上线的时候，只需要重新请求改变的内容，而不是整体都重新请求。





## 区分环境 + HotDev



## babel



## TypeScript

1. ts file
2. ts in vue



## JSX

https://github.com/vuejs/babel-plugin-transform-vue-jsx

