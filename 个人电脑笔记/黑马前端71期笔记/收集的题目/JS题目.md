# 1.

```js
    function getPersonInfo(one, two, three) {
      console.log(one);
      console.log(two);
      console.log(three);
    }
    const person = "wl";
    const age = 21;
    const sex = "女"
    getPersonInfo`${person} is ${age} years old ${sex}`; 
    //["", " is ", " years old ", "", raw: Array(4)]
    //wl
    //21
```

如果使用标记模板字面量，第一个参数的值总是包含字符串的数组。其余的参数获取的是传递的表达式的值！
不使用模板字符串就要这么写`person` + “is” + `age` + “years old” + `sex`
`person`要被替换到第一项和第二项中间,但是`person`前面没有字符串了,所以会补上一个"",`age`,`sex`同理.

```js
    function getPersonInfo(one, two, three) {
      console.log(one);
      console.log(two);
      console.log(three);
    }
    const person = "wl";
    const age = 21;
    const sex = "女"
    getPersonInfo`${person} is years old ${sex}`;
    //["", " is years old ", "", raw: Array(3)]
    //wl
    //21
```

因为`age`被替换成"“所以会被认为"is”,"years old"😂

# 2.

```js
    function Person(firstName, lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
    }
    const wl = Person("wang", "lei")
    const wy = new Person("wang", "yan")  
    console.log(wl);//undefined
    console.log(wy);//Person {firstName: "wang", lastName: "lei"}
```

没有`new`时`this`的指向是window,使用`new`时`this`的指向是新空对象`Person`