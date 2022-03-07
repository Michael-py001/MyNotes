#### var VS let

+ **let/const不存在变量提升机制**，带var/function的存在变量提升，而

+ **let不允许重复声明**；且不论基于什么方式声明的变量，只要在当前上下文中有了，则不允许再基于let/const声明；

+ **let存在暂时性死区**：如果使用Let声明了一个变量，那么在这之前不能赋值

  <img src="https://i.loli.net/2020/08/27/qRPIMd8OfzrbU23.png" alt="img" style="zoom:50%;" />

+ **let会产生块级作用域**

  - 除函数或者对象的大括号之外，如果括号中出现 let/const/function 则会产生块级私有上下文

  - 当前块级上下文也只是对let/const/function他们声明的变量有作用

- **var是函数作用域**
  - 只有在函数内任何地方用var声明了变量，那么在函数内任何地方都可以访问
  - 如果没有用var声明的变量，会隐式用var声明，且是全局变量，在window可以访问到

#### let VS const

  let和const声明的都是变量，const声明的变量是不允许指针重新指向的（但是存储的值是可以改变的，例如：存储的值是引用数据类型，可以基于地址改变堆内存中的信息）

```js
const name = 'lili';
name = 'alili';//报错
//
const li = {name:"lili"};
li.name = 'lala'-->可以修改值

```

