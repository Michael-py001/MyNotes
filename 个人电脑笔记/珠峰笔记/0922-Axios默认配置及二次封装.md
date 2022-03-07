[TOC]

### axios的配置项

> 带星号 *的，是当前项目中绝大部分请求都公用的信息，偶尔有极个别的请求配置信息和它不一样，这类配置信息，我们应该统一设定，所有请求都用这些配置信息 => axios的二次封装/配置

- url
- method
- baseURL *
- transformRequest *
- transformResponse *
- headers *
- params
- data
- timeout *
- withCredentials *
- responseType *
- validateStatus *



### axios的默认配置

#### validateStatus

```js
axios.defaults.validateStatus = status => {
	//status是从服务器获取的HTTP状态码
	return status >= 200 && status < 300;
}
```

#### transformRequest

> 真实项目中，大部分post请求，基于请求主体传递给服务器的格式，不期望是默认的json格式字符串，而是需要改为服务器要求的格式，例如：x-www-form-urlencoded，则需要我们统一基于transformRequest处理，Qs有stringify和parse方法

```js
axios.defaults.transformRequest = [data => {
    // data是基于请求主体传递给服务器的信息，transformRequest只对post系列请求有用
    // 返回的是啥，最后基于请求主体传递给服务器的就是啥
    return Qs.stringify(data);
}];
```

#### transformResponse

> 从服务器返回的数据中，先经过这里处理，以后.then获取的data是这里return的数据

```js
axios.defaults.transformResponse = [data => {
    // data是从服务器获取的响应主体信息，并且根据responseType的值，格式化处理过了
    // 返回的是啥，以后基于.then获取的response.data就是啥
    try {
        data = JSON.parse(data);
    } catch (err) {}
    return data;
}];
```

#### baseURL

> 请求地址不变的部分，后续用get,post就直接省略这一部分

```js
axios.defaults.baseURL = 'http://127.0.0.1:8888';
```

#### withCredentials

> 是否允许携带cookies等认证信息

```js
axios.defaults.withCredentials = true;
```

#### headers

> 自定义请求头

```js
 axios.defaults.headers['Content-Type'] = 'application/x-www-form-urlencoded';
```

分别在指定的请求下才添加headers，common是不管什么请求都加

```
axios.defaults.headers.common/post/get/delete/...['Content-Type']=application/x-www-form-urlencoded'
```

### axios配置项信息的三大部分及优先级

#### 第一部分

> axios库源码中默认给的配置项-->在 `lib/defaults.js` 找到的库的默认值

**优先级最低**

```js
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```

#### 第二部分

> 自己提取出来的公共默认配置信息

**优先级第二**

```
axios.defaults.baseURL='http://127.0.0.1:8888'
...
```

#### 第三部分

> 发送具体请求时传递的参数

**优先级最高**

```js
axios.post('user/update',[date],{
  headers: {
    xxx:'xxx'
  }
})
```

#### 配置的优先顺序

配置会以一个优先顺序进行合并。这个顺序是：在 `lib/defaults.js` 找到的库的默认值，然后是实例的 `defaults` 属性，最后是请求的 `config` 参数。后者将优先于前者。这里是一个例子：

```
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();

// 覆写库的超时默认值
// 现在，在超时前，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;

// 为已知需要花费很长时间的请求覆写超时设置
instance.get('/longRequest', {
  timeout: 5000
});
```

经过这三部分的配置汇总出最后的配置信息，并且根据配置信息向服务器发送请求。

注意：如果遇到个别请求的配置信息和"公共设定的默认配置信息不符合"，则需要在发送请求的时候，单独配置一下，以此覆盖默认配置的公共配置信息。

### 自定义实例默认值

#### 常态开发&特殊开发

- 常态开发是指：小型项目、接口请求单一的情况，根据以上描述的配置即可。
- 特殊开发：大型项目、接口请求很丰富的情况。

![image-20200922170307447](https://i.loli.net/2020/09/22/i46V9qDSIcRaLnJ.png)

#### 自定义实例的配置信息

```js
// Set config defaults when creating the instance
const instance = axios.create({
  baseURL: 'https://api.example.com'
});

// Alter defaults after instance has been created
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

在框架开发中，不同的接口自己定义不同的axios实例

![image-20200922170747876](https://i.loli.net/2020/09/22/GgTc1KtYxNPfBWb.png)

- `http_flie.js`文件下：

  ```js
  import axios from './axios';
  
  let instance = axios.create();
  instance.defaults.baseURL = 'http://127.0.0.1:9999';
  // ...
  
  export default instance;
  ```

- `http.js`文件下：

  ```js
  import axios from './axios';
  axios.defaults.baseURL = 'http://127.0.0.1:8888';
  // ...
  export default axios;
  ```

不同的接口，可以在不同的文件下用`let instance = axios.create();`创建不同的实例。

- App.vue下

  ```vue
  <template>
    <div></div>
  </template>
  
  <script>
  import axios from "../api/http";
  
  export default {
    mounted() {
      axios.get("/xxx/xxx");
    },
  };
  </script>
  ```

- File.vue下

  ```vue
  <template>
    <div></div>
  </template>
  
  <script>
  import instance from "../api/http_file";
  
  export default {
    mounted() {
      instance.get("/xxx/xxx");
    },
  };
  </script>
  ```

  

### 关于baseURL根据环境变量区分不同的接口

> 关于baseURL在真实项目中的一些处理 "根据环境变量，区分不同的接口"「需要webpack的支持」

- 开发环境  请求的是测试服务器接口  http://168.1.1.12:8081
- 测试环境  请求的是另外服务器的接口地址(模拟的是实际上线的数据)  http://168.1.1.0:9000
- 生产环境  部署到真是的服务器上  http://api.zhufengpeixun.cn

我们期望：以后只需要在webpack或者其他地方指定好环境变量，项目打包或者运行的时候，自动切换接口地址。

```js
let env = process.env.NODE_ENV; //webpack或者node中配置出来的
switch (env) {
    case 'development':
        axios.defaults.baseURL = 'http://168.1.1.12:8081';
        break;
    case 'test':
        axios.defaults.baseURL = 'http://168.1.1.0:9000';
        break;
    case 'production':
        axios.defaults.baseURL = 'http://api.zhufengpeixun.cn';
        break;
}
```

### 拦截器：axios.interceptors

在请求或响应被 `then` 或 `catch` 处理前拦截它们。

```js
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```

如果你想在稍后移除拦截器，可以这样：

```js
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

可以为自定义 axios 实例添加拦截器

```js
const instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

#### 请求拦截器：axios.interceptors.request

> 请求拦截器：axios.interceptors.request  发生在axios内部帮我们整理好配置项，在发送给服务器之前，请求拦截器一般是对配置项的处理

```js
axios.interceptors.request.use(config => {
    // config通过上面三部分整理好的配置项，拦截这个配置项，返回全新的config
    // 实战场景，例如：每一次向服务器发送请求的时候，可能需要传递一个Token来校验身份(放在请求头中)
    let token = localStorage.getItem('token');
    if (token) {
        // X-Token 服务器要求的名字 ，还可能会叫做Authorization...
        config.headers['X-Token'] = token;
    }
    return config;
});
```

#### 响应拦截器:axios.interceptors.response

> 响应拦截器：从服务器获取到信息后（或者知道结果，哪怕是失败），到我们自己执行.then/.catch之间触发的

```js
axios.interceptors.response.use(response => {
    // 成功:根据validateStatus处理的
    // 实际开发中，response包含的信息太多了，但是到业务逻辑层，往往只需要响应主体的信息
    return response.data;
}, reason => {
    // 失败:获取数据了,但是状态码不对，或者是没有获取任何的数据...
    // 实际开发中，不论哪一个请求失败，基本上的提示信息或者处理方案是一致的，此时我们在响应拦截器中对错误进行统一的处理
    let response = reason.response;
    if (response) {
        // 从服务器获取到数据了，只是状态码不对，根据不同的状态码，做不同的提示即可
        switch (response.status) {
            case 400:
                break;
            case 401:
                break;
            case 404:
                break;
        }
    } else {
        // 数据都没有获取到
        if (!navigator.onLine) {
            // 断网了
        }
    }
    return Promise.reject(reason);
});
```

