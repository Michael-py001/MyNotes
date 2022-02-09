# 20211224-find和filter的区别

## find

> find会依次遍历每个元素，当遇到符合条件的时候，立即返回该元素

```js
let arr = [2,4,5,6,8]

let result = arr.find(item=>item%2==0)

result ==> 2
```

## filter

> filter会返回所有符合条件的元素的一个数组集合

```js
let arr = [2,4,5,6,8]

let result = arr.f(item=>item%2==0)

result ==> [2, 4, 6, 8]
```

