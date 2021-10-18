# 20210924-使用dev-server&proxy代理解决本地跨域问题

## http-proxy-middleware 代理中间件

> https://github.com/chimurai/http-proxy-middleware

创建一个文件命名为proxy.js，写入以下内容

```js
//proxy.js
// include dependencies
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

// proxy middleware options
const options = {
  target: 'https://jizhikongqi.fenaor.com', // target host
  changeOrigin: true, // needed for virtual hosted sites
  ws: true, // proxy websockets
  pathRewrite: {
    '^/myapi': '', // rewrite path
  },
};

// create the proxy (without context)
const exampleProxy = createProxyMiddleware(options);

// mount `exampleProxy` in web server
const app = express();
app.use('/myapi', exampleProxy);
app.listen(3000);
```

```bash
npm i express http-proxy-middleware

node proxy.js
```

服务启动后，所有请求到 `http://localhost:3000/myapi`的请求，都会转发到`target`设置的地址中，`pathRewrite`可以把请求中的`/myapi`替换成设置的值，也就是说`http://localhost:3000/myapi/user`会变成`http://localhost:3000/user`

http-proxy-middleware如果单独使用的话，因为前端项目运行的端口比如是8080，http-proxy-middleware启动的端口必须也是8080，所以会和本地项目运行的端口冲突。所以要配合webpack-dev-server一起使用，将整个项目运行在dev-server中，http-proxy-middleware作为中间件处理请求代理。

单独使用时，使用postman等工具请求是可以正常使用的：

![image-20210924120041426](https://i.loli.net/2021/09/24/Z4mhxkF9NdpXBJz.png)



## webpack-dev-server

> https://github.com/webpack/webpack-dev-server
>
> https://webpack.docschina.org/configuration/dev-server/#devserverport

```js
//vue.config.js
module.exports = {
  //...
  devServer: {
    proxy: {
      '/api': {
        target: 'http://www.baidu.com/',
        pathRewrite: {'^/api' : ''},
        changeOrigin: true,     // target是域名的话，需要这个参数，
        secure: false,          // 设置支持https协议的代理
      },
      '/api2': {
          .....
      }
    }
  }
};
```

uniapp中的manifest.json配置dev-server

```json
"h5": {
    "devServer": {
      "port": 8081,
      "disableHostCheck": true,
      "proxy": {
        "/api": {
          "target": "http://jizhikongqi.fenaor.com",
          "changeOrigin": true,
          "ws": true,
          "secure": false,
          "pathRewrite": {
            "^/api": ""
          }
        }
      }
    },
    "optimization": {
      "treeShaking": {
        "enable": true //启用摇树优化  
      }
    }
  }
```

## 2. 配置中主要的参数说明

### 2.1 '/api'

捕获API的标志，如果API中有这个字符串，那么就开始匹配代理，
比如API请求`/api/users`, 会被代理到请求 [http://www.baidu.com/api/users](https://link.segmentfault.com/?url=http%3A%2F%2Fwww.baidu.com%2Fapi%2Fusers) 。

### 2.2 target

代理的API地址，就是需要跨域的API地址。
地址可以是域名,如：`http://www.baidu.com`
也可以是IP地址：`http://127.0.0.1:3000`
如果是域名需要额外添加一个参数`changeOrigin: true`，否则会代理失败。

### 2.3 pathRewrite

路径重写，也就是说会修改最终请求的API路径。
比如访问的API路径：`/api/users`,
设置`pathRewrite: {'^/api' : ''},`后，
最终代理访问的路径：`http://www.baidu.com/users`，
这个参数的目的是给代理命名后，在访问时把命名删除掉。

### 2.4 changeOrigin

这个参数可以让target参数是域名。

### 2.5 secure

`secure: false`,不检查安全问题。
设置后，可以接受运行在 HTTPS 上，可以使用无效证书的后端服务器

### 2.6 其他参数配置查看http-proxy-middleware文档

其他的配置参数等信息，可以查看这里：[https://github.com/chimurai/h...](https://link.segmentfault.com/?url=https%3A%2F%2Fgithub.com%2Fchimurai%2Fhttp-proxy-middleware)

## 3. 参考资料

webpack-dev-server： [https://github.com/webpack/we...](https://link.segmentfault.com/?url=https%3A%2F%2Fgithub.com%2Fwebpack%2Fwebpack-dev-server)
http-proxy-middleware： [https://github.com/chimurai/h...](https://link.segmentfault.com/?url=https%3A%2F%2Fgithub.com%2Fchimurai%2Fhttp-proxy-middleware)
[webpack官网 devServer.proxy配置](https://link.segmentfault.com/?url=https%3A%2F%2Fwebpack.docschina.org%2Fconfiguration%2Fdev-server%2F%23devserver-proxy)