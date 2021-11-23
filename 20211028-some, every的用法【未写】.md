# 20211028-some, every的用法

> 参考文章： https://www.cnblogs.com/redsquirrel/p/11546534.html
>

## some

some会遍历数组的每一个元素，遇到满足条件的之后，后面不再遍历，返回true。

```js
let arr = [3,8,9,16,25]
    const check = (age) => {
      console.log(age)
      return age > 5
    }
    // let isGood = arr.some((item,index)=>{
    //   console.log("[item]->",item,"[index]-->",index);
    //   return item>5
    // })
    let isGood = arr.some(check)
    console.log(isGood)
```

