# 20230116-Antd Vue中Form表单自定义校验规则使用

## 校验规则的格式

rules格式必须是个数组，里面包含每个校验规则的对象。每一个对象代表一个校验规则。

```js
rules:[
        { required: true, message: '请输入短信验证码' },
        { type: 'number', len: 4, message: '请输入正确的验证码' },
      ],
```

## 同时触发多条规则

触发多条时，会显示多条提示。

```js
 rules:[
        { required: true, message: '请输入短信验证码' },
        { type: 'number', len: 4, message: '请输入正确的验证码' },
        { type: 'number', message: '验证码必须是数字' },
      ],
```

![image-20230116103738135](https://s2.loli.net/2023/01/16/pLybYrzCaIP8dWm.png)

## input控件校验number类型时的细节处理

因为input输入框取出的值都是string类型，所以如果设置校验type:'number'时，即使input设置了type:'number'也会报错。

方法有两种：

- 校验规则里面设置transform，把vallue转成Number类型
- 设置input的v-model:value.number，vue自动转换float浮点值

```js
rules:[
        { required: true, message: '请输入短信验证码' },
        { len: 4, message: '请输入正确的验证码' },
        { type: 'number', message: '验证码必须是数字', transform(value) {
              return !!value ? Number(value) : value;
           } }
      ],
```

![image-20230116104155817](https://s2.loli.net/2023/01/16/amZTlwUSq6fVKPL.png)

