# uniappH5生产环境去除console.log

## 方案一

> uniapp中如果开启了TreeShaking树摇会导致不生效

vue.config.js 

```js
module.exports = {
  chainWebpack: (config) => {
    // 发行或运行时启用了压缩时会生效
    config.optimization.minimizer('terser').tap((args) => {
      const compress = args[0].terserOptions.compress
      // 非 App 平台移除 console 代码(包含所有 console 方法，如 log,debug,info...)
      compress.drop_console = true
      compress.pure_funcs = [
        '__f__', // App 平台 vue 移除日志代码
        // 'console.debug' // 可移除指定的 console 方法
      ]
      return args
    })
  }
};
```

uniapp内置了`terser`，在`chainWebpack`中使用。



## 方案二

在`main.js`中插入以下代码

```js
if (process.env.NODE_ENV !== "development") {  
    console.log = () => {}  
}  
```

