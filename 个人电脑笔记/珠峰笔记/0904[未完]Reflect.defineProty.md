### Reflect.defineProperty

https://www.xxcheng.top/archives/377

```js
var obj = {
    name: "jpc",
    area: "浙江"
}
Reflect.defineProperty(obj, "age", {
    value: 19,
    writable: false, //是否可读取
    configurable: false, //是否可配置
    enumerable: false //是否可枚举
});
console.log(obj);
console.log(Reflect.deleteProperty(obj, "name")); //true
console.log(Reflect.deleteProperty(obj, "age")); //false -->configurable为false时不能删除
console.log(Reflect.deleteProperty(obj, "sex")); //true
console.log(obj); //{ area:"浙江" }
```

