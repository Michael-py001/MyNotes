# 20230901-antdesign-vue表单中input输入框校验问题

最近在使用自己封装的JS配置式表单组件时，发现一个input输入框的校验问题。

在使用接口数据回填后，点击保存校验，发现几个input输入框的值出现了校验错误，只要重新输入，错误提示又会消失。

![image-20230904104603482](https://s2.loli.net/2023/09/04/kegN1vbtuxrOEcl.png)

一开始以为是组件的validate校验方法出了问题，还有就是以为数据回填不准确，但是一直没有解决问题。最后翻了下网上的文章才找到了灵感，原来是rules校验规则中，如果是input输入框，则默认会带上`type:'string'`，默认为string类型。又联想到input输入框输入的值默认就是string类型，所以这是对应的。而接口返回的数据是number类型，我想到了大概率这就是导致提示报错的原因了。

比如接口返回的数据是：

```js
data:{
 money:100
}
```

由于money值的类型是number，和input输入框要求是string类型互斥，所以会报错。

需要手动把这个值改成字符串：

```js
data:{
	money:'100'
}
```

果然，当我手动把接口数据转成string类型后，报错就消失了。

```js
if (_props.type === 'input') {
  let value = injectFormData[_props.name] || '';
  emit('setFormData', { name: _props.name, value: String(value) });
  return String(value);
}
```

