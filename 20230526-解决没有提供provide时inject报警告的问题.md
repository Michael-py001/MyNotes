# 20230526-解决没有提供provide时inject报警告的问题

```js
const injectData = inject("formData")
```

![image-20230526160117289](https://s2.loli.net/2023/05/26/mvjHy3zxM4gdi2A.png)

解决办法：

给`inject`的第二个参数(默认值)添加null，即可去掉报错提示

```js
const injectData = inject("formData",null)
```

