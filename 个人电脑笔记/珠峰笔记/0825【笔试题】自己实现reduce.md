### 自己实现reduce



```js
function reduce(arr, callback, init) {
    arr = arr.slice(0);
    let result = init; //把init存起来，不改变init
    for (let i = 0; i < arr.length; i++) {
        if (init === undefined && i === 0) {
            // init没有传递值（只有第一轮特殊）
            result = arr[0];
            let item = arr[1],
                index = 1;
            if (!item) return result; //如果数组只有一项，则直接返回result
            result = callback(result, item, index); //函数里面是写的return result + item
            i++; //没有传Init的时候，result是数组第一项，index从1开始，但是i当前还是0，下一轮是1，但是我们需要的是第二轮从第三项开始，
            //所以这里i++多加1
            continue;
        }
        // init传递值了
        let item = arr[i],
            index = i; //传递init值了，index就等于i，从0开始
        result = callback(result, item, index); //返回的是我们在function (result, item, index){...}里写的返回值->return result+item
    }
    return result;
}
let arr = [10, 20, 30, 40, 50];
//没有传递值的情况
let result = reduce(arr, function (result, item, index) {
    console.log(index);
    return result + item;
});
console.log(result);
//传递init初始值的情况
result = reduce(arr, function (result, item, index) {
    console.log(index);
    return result + item;
}, 100);
console.log(result);
```

**简易版**

```js
Array.prototype.myreduce = function (fn, initVal) {
    let result = initVal,
        i = 0;
    if (typeof initVal === 'undefined') {
        result = this[i]
        i++;
    }
    while (i < this.length) {
        result = fn(result, this[i])
    }
    return result
}
```

