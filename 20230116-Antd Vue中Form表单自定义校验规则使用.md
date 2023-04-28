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

> 官方文档：[yiminghe/async-validator: validate form asynchronous (github.com)](https://github.com/yiminghe/async-validator)
>

```js
rules:[
        { required: true, message: '请输入短信验证码' },
        { len: 4, message: '请输入正确的验证码' },
        { type: 'number', message: '验证码必须是数字', transform(value) {
              return !!value ? Number(value) : value;
           } }
      ],
```

### 关于不同数据类型长度的判断

> #### Length
>
> To validate an exact length of a field specify the `len` property. For `string` and `array` types comparison is performed on the `length` property, for the `number` type this property indicates an exact match for the `number`, ie, it may only be strictly equal to `len`.

意思是`string` 和 `array`使用`length`判断长度，数字类型只能用`len`属性。

**注意：**不能把`type: 'number'`和`len: 4`写在同一条规则里面，否则会出现Bug

![image-20230116104155817](https://s2.loli.net/2023/01/16/amZTlwUSq6fVKPL.png)

## pattern使用

```Js
rules: [{
          //只能输入大于0的数字
          pattern: /^([1-9][0-9]*)$/,
          message: '请输入大于0的数字'
        }],
    pattern: /^1[3456789]\d{9}$/, //校验电话
```

## 数字格式化

```js
{
        el: 'a-input-number',
        label: '招标控制价',
        name: 'tenderPrice',
        type: 'number',
        colspan: 12,
        props: {
          formatter: value => `${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ','),
          parser: value => value.replace(/\$\s?|(,*)/g, ''),
          setp: 0.1,
          min: 0,
          precision: 2,
        },
        rules: [{ required: true, message: '请输入招标控制价' }],
      },
```

![image-20230417101622905](https://s2.loli.net/2023/04/17/rcN6bOVqWfhLDit.png)
