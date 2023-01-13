# 20230113-【踩坑】antd Vue表单校验number数字类型时校验不通过的问题

问题复现：

```html
<a-input
   v-model:value="data"
   type="number"
   :placeholder="请输入"
></a-input>
```

对应的校验规则：

```js
 {
    required:true,
    message:"请输入",
    type:'number',
 }
```

校验期望是number类型，input标签上也写了是`type="number"`类型，但是表单校验就是不通过，一直报红。

![image-20230113170914421](https://s2.loli.net/2023/01/13/HONztYGKS7pgyVq.png)

后面经过一番搜索才知道，要在input标签上手动转换值的类型才行。原理就是input框默认取得的值都是string类型，虽然设置了type="number"，有了number输入框的一些特性，但是input里面的值存的还是string类型，所以表单校验时从input框取出value值时还是得到string类型的值。

## 解决办法：

### 1、加上.number

`v-model:value.number="data"`，这样做Vue内部会把数据转换为数字类型并赋值给变量。如果该值无法被 `parseFloat()` 处理，那么将返回原始值。

```html
<a-input
   v-model:value.number="data"
   type="number"
   :placeholder="请输入"
></a-input>
```

### 2、规则中加上transform转换

```js
{
    required:true,
    message:"请输入",
    type:'number',
    transform(value) {
       return value !== "" ? Number(value) : "";
    }
 }
```

![image-20230113172956031](https://s2.loli.net/2023/01/13/C8HmE15nBIdUQva.png)



[antd文档 校验规则](https://2x.antdv.com/components/form-cn#API)

![image-20230113175429519](https://s2.loli.net/2023/01/13/754TczLSNbiFKe3.png)