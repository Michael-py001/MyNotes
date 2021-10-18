## 请求参数的使用

请求方法：`this.$Request({api_name},{data},{options}) `、`this.$Post()`、`this.$Get()`

options参数如下：

| 字段            | 类型    | 默认值     | 说明                                                         |
| --------------- | ------- | ---------- | ------------------------------------------------------------ |
| header          | Object  | {}         | 请求头，也可以在全局配置                                     |
| loading         | Boolean | false/true | Get 方法默认是false，Post 方法默认是true                     |
| loadingText     | String  | 加载中     | 加载显示类型                                                 |
| successCode     | Array   | []         | 定义成功返回的code，默认是全局配置 ，成功的code不会做提示处理，也会做正常的返回 |
| ignoreToastCode | Array   | []         | 默认读取全局配置、忽略toast提示code                          |
| forceReturn     | Boolean | false      | 是否强制返回，true代表无论什么请求结果，都成功返回数据       |
| codeHandle      | Object  | {}         | 根据返回的code执行不同的函数，CodeHandle.js文件是全局配置    |

## successCode实际应用

设置successCode,把错误的返回码也返回，在函数里处理不同code，

```js
  this.$Post('Common.login', {
          phone: this.phone,
          code: this.code,
          login_code: this.$route.query.code
        }, { successCode: [400, 402, 200] }).then(res => {
          if (res.code === 200) {
            this.$SetStorage('daocheng_domestic_token', res.data.access_token, 'local')
            this.$SetStorage('daocheng_domestic_user', res.data, 'local')
            this.SaveUser(res.data)
            if (res.data.is_register * 1 === 1) {
              this.$Bus.$emit(this.$EmitType['OpenRegisterSucc'], { text: '注册成功，道诚欢迎您！' })
            }
            this.GoNav()
          } else if (res.code === 402) {
            this.GetAuthUrl()
          } else {
            this.$ShowToast(res.msg)
          }
        })
```



## 上传统一接口

每个项目的上传图片接口都一样：`/Upload/UploadPublicApi/uploadFile`

## 

