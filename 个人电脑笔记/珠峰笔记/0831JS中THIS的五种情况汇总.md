### JS中THIS的五种情况汇总

1. 在DOM操作中，给当前元素的某个事件行为绑定方法，当事件触发，函数被执行，函数中的this一般都是当前操作的元素。

   【排除】：IE6-8中，基于attachEvent进行DOM2事件绑定，方法中的this是window。

   

5. 我们可以基于Function.prototype上的call/apply/bind方法强制改变函数中的this指向。【例外】：对箭头函数没用，因为它没有this。

2. 函数执行，看函数前面是否有"点"，有"点"，"点"前面是谁this就是谁，没有"点"，this就是window。

- JS严格模式下，没有"点"，this是undefined
- 匿名函数(自执行函数/回调函数)中，this一般是window/undefined，除非有特殊处理。
- 括号表达式中有特殊处理

```js
const fn = function () {
    console.log(this);
};
let obj = {
    name: 'obj',
    fn: fn
};
//括号表达式
(obj.fn)(); //this->obj
// 括号表达式中有多项，也只取最后一项：但是this是window，而不是之前的obj
(10, 20, obj.fn)(); //this->window

//看有没有点，没有就是window
fn(); //this->window/undefined
obj.fn(); //this->obj

//自执行函数中的this是window
(function () {
    console.log(this); //window/undefined
})();

//回调函数中的this一般是window
[1, 2].sort(function (a, b) {
    console.log(this); //window
});
//如果forEach里传了第二个参数，情况就不一样了
[1].forEach(function (item, index) {
    // forEach内部做了特殊的处理，传递的第二个参数是为了改变回调函数中的this指向的
    console.log(this); //obj
}, obj);
```

4. 构造函数执行"new xxx"，函数体中的this是当前的实例

```js
function Fn() {
    this.x = 100;
}
Fn.prototype.sum = function () {};
Fn(); //this->window
let f = new Fn(); //this->f
f.sum(); //this->f
f.__proto__.sum(); //this->f.__proto__ 
```



5. ES6中的箭头函数(或者基于{ }形参的块级上下文)里面没有this，如果代码中遇到this也不是这个函数自己的，而是它所在上下文中的this。

```js
let obj = {
    name: 'obj',
    // fn:function(){}
    fn() {
        // this->obj

        /!* setTimeout(function () {
            // this->window 相当于给window添加属性，和obj没啥关系
            this.name = "lili";
        }, 1000); *!/

        // 需求：1S后执行函数，把obj.name改为lili
        /!* let that = this;//把函数中的this存储起来
        setTimeout(function () {
            // that存储的是上级上下文中的this
            that.name = 'lili';
        }, 1000); *!/

        // 简单的办法：使用箭头函数
        setTimeout(() => {
            // this->用的是上级上下文中的this,也就是obj
            this.name = 'lili';
            console.log(this);
        }, 1000);
    }
};
obj.fn();
```

