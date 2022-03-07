比较会隐式调用toString或valueOf方法
a.toString 调用的是Object.prototype.toString (检测数据类型)
所以我们可以自己写一个toString或者valueOf

# 1.

```js
  let a = {
    i: 1,
    toString() {
      return this.i++
    }
  }
  if (a == 1 && a == 2 && a == 3) {
    console.log("OK");//OK
  }
或
let a = {
    i: 1,
    valueOf() {
      return this.i++
    }
  }
  if (a == 1 && a == 2 && a == 3) {
    console.log("OK");
  }
```

第一次比较 调用a.toString返回1,第二次比较i调用a.toString返回2,第二次比较i调用a.toString返回3.

# 2.

我们可以使用vue2.0的数据劫持:Object.defineProperty()
我们先给window添加一个"a",获取a的时候会触发get().

```js
let i = 1;
  Object.defineProperty(window, "a", {
    get() {
      return i++
    }
  })
  if (a == 1 && a == 2 && a == 3) {
    console.log("OK"); //OK
  }
```

# 3.

```js
let a = [1, 2, 3]
  a.join = a.shift
  if (a == 1 && a == 2 && a == 3) {
    console.log("OK"); //OK
  }
```

第一次比较,返回a.shift第一个元素1,比较完后会删除第一个元素,此时a=[2,3]
第二次比较,返回a.shift第一个元素2,比较完后会删除第一个元素,此时a=[3]
第三次比较,返回a.shift第一个元素3,比较完后会删除第一个元素,此时a=[3]

数组的toString 方法返回一个字符串，经调用 join() 方法连接数组每一个返回的字符串