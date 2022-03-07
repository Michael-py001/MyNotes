### 前后端数据通信方案

- ajax XMLHttpRequest：同源 & 跨域「cors & proxy」
  - JQ $.ajax
  - axios
- Fetch：同源 & 跨域「cors & proxy」
- webscoket
- jsonp
- postMessage
- iframe + document.domain/location.hash...

### 前后端数据通信的数据格式

> POST->请求主体  GET->URL问号传递参数

- form-data  MIME:multipart/form-data

  - 表单提交

  - 文件上传:文件流信息放置在formData中

    ```js
    let formData = new FormData();
    formData.append('name', 'li');
    formData.append('age', 11);
    console.log(formData);//输出是一个formData对象，里面有很多方法，用里面的方法可以提取出数据
    ```

    ![image-20200921003258160](https://i.loli.net/2020/09/21/YQJAwNfuhrMvePR.png)

- x-www-form-urlencoded  MIME(媒体类型):application/x-www-form-urlencoded
  - 普通数据的传输一般都用这种方式（这样GET和POST传递给服务器的数据格式统一了）
  - 'xxx=xxx&xxx=xxx'
  - GET系列请求，URL传递的参数信息其实就是这种格式

- raw 原始格式

  - json字符串  MIME:application/json  服务器返回给客户端的数据一般也都是json格式字符串

  -  text普通字符串  MIME:text/plain
  - xml字符串 MIME:application/xml

- binary 文件流
  - 根据上传的文件不同，传递的MIME也是不一样的  例如：image/jpeg 或者 application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

- GraphQL

## axios的应用

> http://www.axios-js.com/zh-cn/docs/ 
>
> 基于Promise封装和管理的ajax库(核心XMLHttpRequest)

### axios API语法：

- axios([config])

  ```js
  // 发送 POST 请求
  axios({
    method: 'post',
    url: '/user/12345',
    data: {
      firstName: 'Fred',
      lastName: 'Flintstone'
    }
  });
  ```

  ```js
  // 获取远端图片
  axios({
    method:'get',
    url:'http://bit.ly/2mTM3nY',
    responseType:'stream'
  })
    .then(function(response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
  });
  ```

  

- axios(url,[config])

  ```js
  // 发送 GET 请求（默认的方法）
  axios('/user/12345');
  ```

### 请求方法的别名

-  axios.request(config)

-  axios.get(url[, config])

-  axios.delete(url[, config])

- axios.head(url[, config])

-  axios.options(url[, config])

- axios.post(url[, data[, config]])

-  axios.put(url[, data[, config]])

- axios.patch(url[, data[, config]])

在使用别名方法时，指定了 请求地址url/请求方式method/请求主体data 这些东西,在config中无需再次配置了，别名都是快捷写法，最后处理的方案还是基于axios([config])

axios返回的结果都是一个**promise实例**，意味着可以使用then,catch等方法：

### 请求配置

> 这些是创建请求时可以用的配置选项。只有 `url` 是必需的。如果没有指定 `method`，请求将默认使用 `get` 方法。



```js
// `url` 是用于请求的服务器 URL
  url: '/user',

// `method` 是创建请求时使用的方法
method: 'get', // default

// `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
// 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
baseURL: 'https://some-domain.com/api/',
  
// `headers` 是即将被发送的自定义请求头
headers: {'X-Requested-With': 'XMLHttpRequest'},
 
// `transformRequest` 允许在向服务器发送前，修改请求数据
// 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
// 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
transformRequest: [function (data, headers) {
  // 对 data 进行任意转换处理
  return data;
}],
  
 // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
 transformResponse: [function (data) {
   // 对 data 进行任意转换处理
   return data;
 }],
   
// `params` 是即将与请求一起发送的 URL 参数(GET请求)
// 必须是一个无格式对象(plain object)或 URLSearchParams 对象
params: {
  ID: 12345
},
  
// `paramsSerializer` 是一个负责 `params` 序列化的函数
// (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },
    
  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },
 // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断，触发xhr.ontimeout事件
  timeout: 1000,
 
// `withCredentials` 表示跨域请求时是否需要使用凭证(例如:cookie)
  withCredentials: false, // default

// `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
responseType: 'json', // default 
  
  // `onUploadProgress` 允许为上传处理进度事件，监听上传或者下载的进度，用的是xhr.upload.onprogress事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },
    
   // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },
  
// `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },
```

### 响应结构

某个请求的响应包含以下信息:

#### 返回成功时，`response`对象：

```js
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 服务器响应的头
  headers: {},

   // `config` 是为请求提供的配置信息
  config: {},
 // 'request'
  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance the browser
  request: {}
}
```

使用 `then` 时，你将接收下面这样的响应 :

```js
axios.get('/user/12345')
  .then(function(response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
```

#### 返回错误时，`reason`对象

```js
{
	config://发送请求的时候传递的配置信息,
	
	request://基于new XMLHttpRequest创建的xhr对象,
	
	isAxiosError://true/false,
	
	response://等同于成功返回的response对象，如果没有从服务器获取任何的结果，response是不存在的
}
```

**注意：**

在POST请求中，data中的数据

```js
axios.post('/user/login', {
    // 默认会把对象变为JSON字符串传递给服务器，{"account":"zhufengpeixun","password":"xxxxxx"}
    account: 'zhufengpeixun',
    password: 'xxxxxx'
}, {
    baseURL: 'http://127.0.0.1:8888',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    transformRequest: [function (data) {
        return Qs.stringify(data);
    }]
});
```

或者这样：

![img](https://i.loli.net/2020/09/21/7cp1D6ol8OuAJBg.png)

axios发送的数据格式和jquery ajax发送的默认数据格式不同，axios发送的是JSON格式`application/json`，是一个对象，而jq ajax发送的是`application/x-www-form-urlencoded`

### 使用 application/x-www-form-urlencoded方案

#### 浏览器

##### 使用qs库

这里需要在发送数据前，把数据处理，使用axios配置中的`transformRequest`

```js
transformRequest: [function (data) {
        return Qs.stringify(data);
    }]
```

或者直接修改data

```js
const qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
```

或者以另一种方式（ES6），

```js
import qs from 'qs';
const data = { 'bar': 123 };
const options = {
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  data: qs.stringify(data),
  url,
};
axios(options);
```

##### 使用URLSearchParams

```
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```

![img](https://i.loli.net/2020/09/21/RbDmrYEpavJVGf9.png)

#### Node.js

在node.js中，您可以使用querystring模块，如下所示：

```
const querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({ foo: 'bar' }));
```

您也可以使用qs库。

### aixos使用方法示例

#### GET请求

```js
axios.get('http://127.0.0.1:8888/user/login').then(response => {
    // console.log('成功:', response);
    return response.data;//拿到resopnse后，返回response中的data
}).then(data => {
    console.log('响应主体信息:', data);
}).catch(reason => {
    console.dir(reason);
});
```

#### POST请求

```js
axios.post('/user/login', {
    // 默认会把对象变为JSON字符串传递给服务器 {"account":"zhufengpeixun","password":"xxxxxx"}
    account: 'zhufengpeixun',
    password: 'xxxxxx'
}, {
    baseURL: 'http://127.0.0.1:8888',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    transformRequest: [function (data) {
        return Qs.stringify(data);
    }]
});
```

#### 使用async await

```js
async function fn() {
    try {
        let data = await axios.get('http://127.0.0.1:8888/user/login')
            .then(response => response.data);
        console.log('响应主体信息:', data);
    } catch (reason) {
        console.dir(reason);
    }
}
fn();
```

