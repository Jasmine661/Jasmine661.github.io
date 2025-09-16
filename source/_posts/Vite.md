---
title: Vite
description: Vite打包工具学习
top_img: /image/3993054a6603bde97c062e18480514bc.jpg
cover: /image/3993054a6603bde97c062e18480514bc.jpg
categories: 项目搭建
---
# 依赖预构建 

## 目的

​	1）将commonJS等模块规范转换为ESM
	2）优化性能：例如将lodash预构建成单独的模块，放在node_module/.vite文件夹中，减少浏览器发起的http请求次数
	3）缓存优化：避免每次启动开发服务器是重新分析依赖关系

## 实现方式

​	使用esbuild检测到lodash的commonjs模块规范，将其转换成ESM，之后打包到/ node_module / . vite / . lodash文件中，最后重写路径。在开发服务器启动时构建，会自动进行hash缓存。



# 环境变量

​    通过import.meta.env暴露环境变量，环境变量可以在 .env 文件中设置，通常以 VITE 开头(具体可以看官方文档，有详细说明文件名对应设置的环境变量内容)

​	在配置中加envPrefix,配置vite注入客户端环境变量校验的前缀，如" VITE_ " , " ENV_ "



# 对 css 的处理

## 步骤

​	1）vite读取到main.js中引用的index.js，直接去使用fs模块去读取index.css中的文件内容

​	2)  创建一个style标签，将index.css中文件内容直接copy进style标签中，将style标签插入head中

​	3）将该css文件中的内容直接替换为js脚本（方便热更新或者css模块化），同时设置Content-Type（这个是什么，浏览器就会以什么的脚本形式来执行该文件）为js，从而让浏览器以JS脚本形式来执行该css后缀的文件

## Css Module（css模块化）

### 场景

协同开发过程中，可能会出现定义了相同类名的情况

### 解决

将componentA.css文件名改为componentA.module.css，即以.module.css结尾，会被认为是Css module文件。

### 原理

​	1）.module.css( module是一种约定，表示需要进行css模块化 )，会给类名进行一定规则的替换( footer --> footer_i22st_1)，同时创建一个映射对象（footer："footer_i22st_1"）

​	2）将替换后的内容塞进style标签里面然后放入head标签中

​	3）将componentA.module.css内容进行全部抹除，替换成js脚本，将创建的映射对象在脚本中进行默认导出

### 配置

module配置最终会丢给postcss module

1.**localsConvention**：用于转换类名，value有

​	camelCase  同时保留原始类名和 camelCase 格式的类名

​	camelCaseOnly  只生成 camelCase 格式的类名 

​	dashes  同时保留原始类名和将中划线转为驼峰的类名（与 camelCase 类似）

​	dashesOnly  只保留中划线转驼峰后的类名

2.**scopeBehaviour**: 配置当前的模块化行为是模块化还是全局化(有hash就是开启了模块化的一个标志，因为他可以保证产生不同的hash值来控制我们的样式类名不被覆盖)，value有local和global

3.**generateScopedName**：生成的类名的规则，如"[name] _ [local] _ [hash:5]"，如果配置成函数，函数的返回值就是最终显示的类名（所有module都会以返回值为名）

4.**hashPrefix**：给hash配置前缀

5.**globalModulePaths**：value是数组，代表你不想生成css模块化的路径

## PostCss

### 概述

vite天生对postcss有良好的支持，postcss相当于饮用水过滤器，是为了保证css在执行起来的时候万无一失。

postcss是可以涵盖less和sass的，但是现在postcss社区对其涵盖的less和sass已经停止维护了。

### 作用

1.自动添加浏览器前缀

2.将现代 CSS 特性转换为兼容旧浏览器的写法

3.自定义语法扩展（比如嵌套）

4.压缩优化 CSS

### 配置

在postcss.config.js中配置

```javascript
module.exports = {
    plugin: [postcssPresetEnv(/* pluginOptions */)] 
    // postcssPresetEnv内置了很多的插件，装了postcssPresetEnv相当于是把最基础的插件给安装好了
}
```

在vite.config.js中

```javascript
export default defineConfig({
    ...
    css: {
      ...
      postcss: {
        plugin: [
            postcssPresetEnv({
        importFrom: path.resolve(__dirname,"/variables.css") 
            // 好比上postcss知道有全局变量需要记下来
          })
        ] // 或者直接创建postcss.config.js文件，会自动读取
      }
    }
})
```



# 静态资源

## 概念

除了动态API以外的99%的资源

在生产环境下的配置：

​	浏览器有一个缓存机制，静态资源只要名字不改变，浏览器就会直接读取缓存，即在刷新页面的时候，看请求的名字是不是同一个，是则读取缓存，hash算法可以帮我们避免名字一致，以更好的控制浏览器的缓存机制。

```javascript
...
build: {
    rollupOptions: { // 配置rollup的一些构建策略
      output: {
          assetFileNames: "[name].[hash].[ext]"
      },
      assetsInlineLimit: 4096， // 小于4kb的文件将会打包成base64格式
      outDir: "testDist",// 默认为dist
      assetsDir: "static", // 静态资源目录
    }
}
```



# 插件

vite已经将很多核心插件内置了，具体可以看官方文档（awesome-vite）

## vite-aliases

自动生成别名（鸡肋） 可用 **config** hook手写 （config通常用于配置配置文件的插件）

## vite-plugin-html 

动态控制html文件内容 用 **transformIndexHtml** hook 手写 

比如不想写死.html中的title标签的内容，涉及到EJS（服务端用的比较频繁）

## vite-plugin-mock

用于模拟数据   用 **configureServer** hook 手写

该插件的依赖项是mockjs  会自动去找mock文件夹(放在根目录下)

```javascript
// /mock/.index.js
import mockJS from "mockjs"

const userList = mockJS.mock({
    "data|100": [{
        name: "@cname",
        ename: "@name",
        "id|+1": 1,
        time: "@time",
        date: "@date",
    }] // 具体格式查看官方文档
})


// vite.config.js
...
plugins: [
  ...
  viteMockServer({
      // default
      mockPath: "mock",
      localEnable: command === 'serve' // 表示开发服务器的时候拦截api
  })
]
```

## vite特有钩子

**config**   通常用于配置配置文件的插件

**transformIndexHtml**  转换 index.html 的专用钩子。钩子接收当前的 HTML 字符串和转换上下文

**configureServer ** 可用于添加自定义中间件，本地mock开发

**configResolved**  读取和存储最终解析的配置

**configurePreviewServer**

# TypeScript

略，感觉基本是下载

# 性能优化

## 开发时的构建速度优化

yarn dev / yarn start 敲下瞬间到呈现结果要占用多长时间

## 页面性能指标

- 首屏渲染时长 fcp
  - 懒加载：需要写代码实现
  - http优化：协商缓存 强缓存
    - 强缓存：服务端给响应头追加一些字段（expires），客户端会记住这些字段，在expires（截止失效时间）没有到达之前，无论怎么刷新页面，浏览器都不会重新请求页面，而是从缓存里取
    - 协商缓存：是否使用缓存要跟后端商量，当服务端给我们打上协商缓存标记以后，客户端在下次刷新页面需要重新请求资源的时候会发送一个协商请求给服务端，服务端如果说需要变化，则会响应具体内容，如果服务端觉得没变化则会响应304
- 页面中最大元素的一个时长 lcp
- ......

## js逻辑

- 我们要注意副作用的清除 组件会频繁挂载和卸载：频繁点击计时器setTimeout，等于开多个线程计时器
- 我们在写法上的注意事项： requestAnimationFrame,requestIdleCallback（卡浏览器帧率） 
  - 浏览器帧率
- 防抖节流 ， lodash js 工具（原生js方法会不断地向上找原型链）

#### css

- 关注继承属性 能继承不要重复写
- 避免过深的css嵌套

## 构建优化 

vite(rollup) webpack

- 优化体积： 压缩，tree shaking , 图片资源压缩，cdn加载，分包...

### 分包策略

把不会常规更的文件单独打包处理

浏览器缓存策略

业务逻辑改变，lodash内容不变，修改业务逻辑需要重新请求

```javascript
// 在vite.config.js文件中
"rollupOptions": {
  "input": {
    // 多入口 (已默认优化)
    main: path.resolve(__dirname,"./index.html"),
    product: path.resolve(__dirname,"./product.html")
  }
  "output": {
    // 处理不会常规更新的文件，打包时会在vender文件中
    "manualChunks": (id: string) => {
      if (is.includes("node_modules")) {
        return "vender";
      }
    }
  }
}
```

### gzip压缩

将所有的静态文件进行压缩，以减少体积，使用 vite-plugin-compression 插件
流程： 服务端 -> 压缩文件   客户端 -> 解压缩

### 动态导入

是es6新特性  vite是按需加载 动态导入和按需加载异曲同工

  例1：

```javascript
import "./src/imageLoader" ---> 
   import("./src/imageLoader").then(data => {
    console.lod("data",data)
  })
```

  例2：路由中

```javascript
{
path: "/home",
component: import("./Home") // 函数返回一个Promise
}

function import(path) {
  return new Promise((resolve) => {
    // 阻止 promise 进入 fulfilled 状态
    // 进入对应路径就将 promise 状态设置为 fulfilled 调用 resolve，将 script 标签塞入 body 中去
    // e函数创造了一个 promise.all 创建一个 script 标签，引入 src 指向 home 文件 webpack 已经把 jsx 文件编译过了但是没有给浏览器
    webpack__require.e().then(() => {
      const result = await webpack__require(path)
    })
  })
}
```

### cdn加速 

​	我们的所有依赖以及文件打包以后会放到服务器上去

​	将我们依赖的第三方模块写成 cdn 的形式，保障自己代码的小体积（体积小服务器和客户端传输压力就没那么大）

**优点**

- 和 dns 一样

- 引用离你最近的 cdn 服务器


### 步骤

1.安装`vite-plugin-cdn-import`插件

```powershell
npm install vite-plugin-cdn-import --save-dev
```

2.配置

在plugin.viteCDNPlugin.module中配置要进行cdn优化的文件

再在build.rollupOptions中排除要进行cdn优化的文件的打包

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import viteCDNPlugin from 'vite-plugin-cdn-import'

export default defineConfig({
  plugins: [
    react(),
    viteCDNPlugin({
      modules: [
        {
          name: 'react', //  模块名，必须和你import的一致
          var: 'React', // 模块在全局 window 中暴露的变量名
          path: 'https://unpkg.com/react@18/umd/react.production.min.js', // CDN地址
        },
        {
          name: 'react-dom',
          var: 'ReactDOM',
          path: 'https://unpkg.com/react-dom@18/umd/react-dom.production.min.js',
        },
        {
          name: 'lodash',
          var: '_',
          path: 'https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js',
        },
      ],
    }),
  ],
  build: {
    rollupOptions: {
      external: ['react', 'react-dom', 'lodash'], 
    },
  },
})
```

注：webpack的CDN优化需要使用 htmlWebpackPlugin 插件，在public/index.html注入cdn资源url，而vite是自动注入的，所以不用担心





