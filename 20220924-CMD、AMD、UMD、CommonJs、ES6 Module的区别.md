# 20220924-CMD、AMD、UMD、CommonJs、ES6 Module的区别

​		接触前端开发之后，一直都有听说过什么CMD,AMD,CommonJs等等的名词，但是一直没有理解他们之间的区别，现在搜集总结了下他们的区别后，不得不说前端开发里各种东西太杂了，连JavaSctipt这门语言的模块化都没有一个统一，对于刚入门的新手确实不太友好。

## 整体的区别如图

![img](https://raw.githubusercontent.com/Michael-py001/imgUpload/matster/img/202209241039985.png)

## AMD

标签：`浏览器端` `require.js`

AMD: `Asynchronous Module Definition`的缩写，意思就是"`异步模块定义`"。

​		由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是大名鼎鼎RequireJS，实际上AMD 是 RequireJS 在推广过程中对模块定义的规范化的产出，也就是说：`require.js`是AMD的实现。

> RequireJS 的官方文档：[How to get started with RequireJS](https://requirejs.org/docs/start.html)

它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

RequireJS主要解决两个问题：

1. 多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
2. js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长

### AMD的用法

> 具体请看：[AMD · amdjs/amdjs-api Wiki (github.com)](https://github.com/amdjs/amdjs-api/wiki/AMD)

- `define` API

```js
  define(id?, dependencies?, factory);
```

- Examples

  1. 设置 ID 为“alpha”的模块，使用require，导出 ID 为“beta”的模块：

     ```js
     define("alpha", ["require", "exports", "beta"], function (require, exports, beta) {
            exports.verb = function() {
                return beta.verb();
                //Or:
                return require("beta").verb();
            }
        });
     ```

  2. 返回对象文本的匿名模块：

     ```js
        define(["alpha"], function (alpha) {
            return {
              verb: function(){
                return alpha.verb() + 2;
              }
            };
        });
     ```

  3. 无依赖模块可以定义直接对象文本：

     ```js
        define({
          add: function(x, y){
            return x + y;
          }
        });
     ```

  4. 使用简化的 CommonJS 包装定义的模块：

     ```js
        define(function (require, exports, module) {
          var a = require('a'),
              b = require('b');
     
          exports.action = function () {};
        });
     ```

  

## CMD

CMD 即`Common Module Definition`通用模块定义，CMD规范是国内发展出来的，就像AMD有个`requireJS`，CMD有个浏览器的实现`SeaJS`，SeaJS要解决的问题和requireJS一样，只不过在模块定义方式和模块加载（可以说运行、解析）时机上有所不同。

区别：

AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。

CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

类似的还有 CommonJS Modules/2.0 规范，是 BravoJS 在推广过程中对模块定义的规范化产出。还有不少⋯⋯这些规范的目的都是为了 JavaScript 的模块化开发，特别是在浏览器端的。目前这些规范的实现都能达成浏览器端模块化开发的目的。

### CMD和AMD的区别

　1. 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.

　　2. CMD 推崇依赖就近，AMD 推崇依赖前置。

// CMD默认推荐的是：

```js
define(function(require, exports, module) {   
    var a = require('./a')   
    a.doSomething()   
    // 此处略去 100 行   
    var b = require('./b') // 依赖可以就近书写   
    b.doSomething()   
    // ... 
})
```

// AMD 默认推荐的是：

```js
define(['./a', './b'], function(a, b) {  
    // 依赖必须一开始就写好    
    a.doSomething()    // 此处略去 100 行    
    b.doSomething()    
    ...
})
```

## CMD模块定义规范

在 CMD 规范中，一个模块就是一个文件。代码的书写格式如下：

```js
define(factory);
```

`define` 是一个全局函数，用来定义模块。

`define` 接受 `factory` 参数，`factory` 可以是一个函数，也可以是一个对象或字符串。

`factory` 为对象、字符串时，表示模块的接口就是该对象、字符串。比如可以如下定义一个 JSON 数据模块：

```
define({ "foo": "bar" });
```

也可以通过字符串定义模板模块：

```js
define('I am a template. My name is {{name}}.');
```

`factory` 为函数时，表示是模块的构造方法。执行该构造方法，可以得到模块向外提供的接口。`factory` 方法在执行时，默认会传入三个参数：`require`、`exports` 和 `module`：

```js
define(function(require, exports, module) {

  // 模块代码

});
```

### define `define(id?, deps?, factory)`

`define` 也可以接受两个以上参数。字符串 `id` 表示模块标识，数组 `deps` 是模块依赖。比如：

```js
define('hello', ['jquery'], function(require, exports, module) {

  // 模块代码

});
```

`id` 和 `deps` 参数可以省略。省略时，可以通过构建工具自动生成。

**注意**：带 `id` 和 `deps` 参数的 `define` 用法不属于 CMD 规范，而属于 [Modules/Transport](https://github.com/cmdjs/specification/blob/master/draft/transport.md) 规范。

### define.cmd `Object`

一个空对象，可用来判定当前页面是否有 CMD 模块加载器：

```js
if (typeof define === "function" && define.cmd) {
  // 有 Sea.js 等 CMD 模块加载器存在
}
```

### require `Function`

`require` 是 `factory` 函数的第一个参数。

### require `require(id)`

`require` 是一个方法，接受 [模块标识](https://github.com/seajs/seajs/issues/258) 作为唯一参数，用来获取其他模块提供的接口。

```js
define(function(require, exports) {

  // 获取模块 a 的接口
  var a = require('./a');

  // 调用模块 a 的方法
  a.doSomething();

});
```

**注意**：在开发时，`require` 的书写需要遵循一些 [简单约定](https://github.com/seajs/seajs/issues/259)。

### require.async `require.async(id, callback?)`

`require.async` 方法用来在模块内部异步加载模块，并在加载完成后执行指定回调。`callback` 参数可选。

```js
define(function(require, exports, module) {

  // 异步加载一个模块，在加载完成时，执行回调
  require.async('./b', function(b) {
    b.doSomething();
  });

  // 异步加载多个模块，在加载完成时，执行回调
  require.async(['./c', './d'], function(c, d) {
    c.doSomething();
    d.doSomething();
  });

});
```

**注意**：`require` 是同步往下执行，`require.async` 则是异步回调执行。`require.async` 一般用来加载可延迟异步加载的模块。

### require.resolve `require.resolve(id)`

使用模块系统内部的路径解析机制来解析并返回模块路径。该函数不会加载模块，只返回解析后的绝对路径。

```js
define(function(require, exports) {

  console.log(require.resolve('./b'));
  // ==> http://example.com/path/to/b.js

});
```

这可以用来获取模块路径，一般用在插件环境或需动态拼接模块路径的场景下。

exports `Object`

`exports` 是一个对象，用来向外提供模块接口。

```js
define(function(require, exports) {

  // 对外提供 foo 属性
  exports.foo = 'bar';

  // 对外提供 doSomething 方法
  exports.doSomething = function() {};

});
```

除了给 `exports` 对象增加成员，还可以使用 `return` 直接向外提供接口。

```js
define(function(require) {

  // 通过 return 直接提供接口
  return {
    foo: 'bar',
    doSomething: function() {}
  };

});
```

如果 `return` 语句是模块中的唯一代码，还可简化为：

```js
define({
  foo: 'bar',
  doSomething: function() {}
});
```

上面这种格式特别适合定义 JSONP 模块。

**特别注意**：下面这种写法是错误的！

```js
define(function(require, exports) {

  // 错误用法！！!
  exports = {
    foo: 'bar',
    doSomething: function() {}
  };

});
```

正确的写法是用 `return` 或者给 `module.exports` 赋值：

```js
define(function(require, exports, module) {

  // 正确写法
  module.exports = {
    foo: 'bar',
    doSomething: function() {}
  };

});
```

**提示**：`exports` 仅仅是 `module.exports` 的一个引用。在 `factory` 内部给 `exports` 重新赋值时，并不会改变 `module.exports` 的值。因此给 `exports` 赋值是无效的，不能用来更改模块接口。

### module `Object`

`module` 是一个对象，上面存储了与当前模块相关联的一些属性和方法。

### module.id `String`

模块的唯一标识。

```js
define('id', [], function(require, exports, module) {

  // 模块代码

});
```

上面代码中，`define` 的第一个参数就是模块标识。

### module.uri `String`

根据模块系统的路径解析规则得到的模块绝对路径。

```js
define(function(require, exports, module) {

  console.log(module.uri); 
  // ==> http://example.com/path/to/this/file.js

});
```

一般情况下（没有在 `define` 中手写 `id` 参数时），`module.id` 的值就是 `module.uri`，两者完全相同。

### module.dependencies `Array`

`dependencies` 是一个数组，表示当前模块的依赖。

### module.exports `Object`

当前模块对外提供的接口。

传给 `factory` 构造方法的 `exports` 参数是 `module.exports` 对象的一个引用。只通过 `exports` 参数来提供接口，有时无法满足开发者的所有需求。 比如当模块的接口是某个类的实例时，需要通过 `module.exports` 来实现：

```js
define(function(require, exports, module) {

  // exports 是 module.exports 的一个引用
  console.log(module.exports === exports); // true

  // 重新给 module.exports 赋值
  module.exports = new SomeClass();

  // exports 不再等于 module.exports
  console.log(module.exports === exports); // false

});
```

**注意**：对 `module.exports` 的赋值需要同步执行，不能放在回调函数里。下面这样是不行的：

```js
// x.js
define(function(require, exports, module) {

  // 错误用法
  setTimeout(function() {
    module.exports = { a: "hello" };
  }, 0);

});
```

在 y.js 里有调用到上面的 x.js:

```js
// y.js
define(function(require, exports, module) {

  var x = require('./x');

  // 无法立刻得到模块 x 的属性 a
  console.log(x.a); // undefined

});
```

### 小结

这就是 CMD 模块定义规范的所有内容。经常使用的 API 只有 `define`, `require`, `require.async`, `exports`, `module.exports` 这五个。其他 API 有个印象就好，在需要时再来查文档，不用刻意去记。

与 RequireJS 的 AMD 规范相比，CMD 规范尽量保持简单，并与 CommonJS 和 Node.js 的 Modules 规范保持了很大的兼容性。通过 CMD 规范书写的模块，可以很容易在 Node.js 中运行。

## CommonJs

​		CommonJS是2009年由JavaScript社区提出的包含了模块化的一个标准，后来被Node.js所采用并实现，也就是说我们在Node.js中用到的模块导入导出都是依照CommonJS标准来实现的。

### 1. 导出

​		我们可以把一个文件看成一个模块，每个模块之间是互相独立的，即不会互相影响。当需要使用到某个模块时，只需在文件中将目标模块导入即可

要想被其它模块导入首先需要导出需要向外暴露的变量或方法，在CommonJS中导出的语法有以下两种方式：

```js
// B.js
// 定义了函数show
function show() {
	console.log('show方法被调用')
}

// 定义了变量count
let count = 3

/*--------------  导出方法  --------------*/

// 第一种
module.exports = {
	show,
	count
}

// 第二种
exports.show = show
exports.count = count

```

上述代码中，两种导出方式是等价的。

第一种导出方式是将需要导出的函数或变量存储到 module.exports 里面，其中 module.exports 原本是一个空对象

第二种导出方式中，exports 在内部其实是指向了 module.exports，所以当我们执行 exports.变量 或 exports.函数 时，其实就相当于把变量或函数存储到 module.exports 中

> **注意：** 这里要特别强调的是，在使用第二种导出方式时，不能对 `exports` 进行重新赋值，否则就将 `module.exports` 直接全部覆盖了

### 2. 导入

再来看一下CommonJS的导入语法:

```js
// A.js
const bModule = require('./B.js')

console.log(bModule.count)  // 3

bModule.show()  // show方法被调用

```

​		从上述代码中可以看到，CommonJS是通过 require 方法来导入模块的，其参数为模块文件路径，要特别注意的是，我们导入模块后接收到的其实是一个对象，也就是 module.exports 的值，我们能从该对象中获取到所需的变量或函数

另外，比较特别的是，require 方法还可以接收一个表达式作为参数，代码如下

```js
let fileName = 'B.js'
const bModule = require('./' + fileName)
```

因此，我们是可以动态的改变并决定模块的加载导入路径的。

### 小结

- CommonJS规范主要用于服务端编程，加载模块是同步的，这并不适合在浏览器环境，因为同步意味着阻塞加载，浏览器资源是异步加载的，因此有了AMD CMD解决方案。
- 针对服务器端和针对浏览器端有什么本质的区别呢？服务器端一般采用同步加载文件，也就是说需要某个模块，服务器端便停下来，等待它加载再执行。而浏览器端要保证效率，需要采用异步加载，这就需要一个预处理，提前将所需要的模块文件并行加载好

## ES6 Module

如标题名写的，该模块标准是在ES6时才被提出的，此后JS才具有了模块化这一特性。**ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案**。

### 1. 导出

​		在ES6 Module 中，导出用到了关键字 `export` ，导出的方式也大致分为两种，分别是**命名导出** 、**默认导出**：

#### **第一种：** 命名导出：

```js
// B.js
/*--------  单个变量或函数导出  ----------*/
export function show() { console.log('show方法被调用') }

export let count = 3

/*--------  批量导出  ----------*/
function show() { console.log('show方法被调用') }

let count = 3

export {show, count}

```

上述代码分了两种情况，且这两种写法是等价的

第一种是单个的变量或函数导出，只需要直接在开头使用 `export` 关键字即可；命名变量可以使用`const`, `let`, `var`，函数可以用声明式函数：`function`，或者定义式函数:`const func = ()=>{}`

第二种情况是批量地把多个变量或函数导出，只需要把它们储存到一个对象中即可。

#### **第二种：** 默认导出

```js
// B.js
function show() { console.log('show方法被调用') }

// 命名导出变量count
export let count = 3

// 默认导出函数show
export default show

```

默认导出是在 `export` 关键词后面再跟上一个 `default` 表示导出的该变量或函数是匿名的。

> **注意：** 一个模块只能默认导出一次，否则就会报错，具体原因会在后面讲解。

### 2. 导入

ES6 Module 的导入用到的关键字是 `import` ，具体代码如下：

```js
// A.js
import {show, count} from './B.js'

show()   // show方法被调用

console.log(count)  // 3

```

ES6 Module的导入需要用一对 `{}` 大括号来接收我们需要导入的方法或函数。

> **注意：** 大括号中的变量或函数名必须与导出时的名称一模一样

那么如果我们想修改导入的变量或函数的名称，可以通过 as 关键词来命名，代码如下

```js
// A.js
import {show as print, count as number} from './B.js'

print()   // show方法被调用

console.log(number)  // 3
```

如果我们要想将所有的变量或函数都导入，可以通过 `*` 来整体导入，代码如下

```js
import * as bModule from './B.js'

bModule.show()  // show方法被调用

console.log(bModule.count)  // 3
```

`*`表示全部的意思，我们将其全部导入，并赋值给 `bModule`，这样我们就可以通过 `bModule` 获取想要的变量或对象了。

以上所说的都是针对命名导出的变量或函数，那么如何导入一个默认导出的变量或函数呢？

```js
// 将通过 export default 导出的变量导入
import print from './B.js'

print()  // show方法被调用
```


命名导出的变量都是通过`{}` 来接收的，那么去掉 `{} `，接收的就是默认导出的变量了，因为导出的变量是匿名的，因此我们可以随意地起个变量名用于接收

> 补充： 这里特别提一下，与CommonJS不同，ES6 Module 的导入文件路径是不支持表达式的

## CommonJS 与 ES6 Module 的区别

1. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。换句话说就是：对于模块的依赖，CommonJS是**动态的**，ES6 Module 是**静态的**，

2. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。

3. CommonJS 模块的require()是同步加载模块，ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。

### 区别一

对于模块的依赖，何为**动态**？何为**静态**？

动态是指对于模块的依赖关系建立在代码执行阶段；
静态是指对于模块的依赖关系建立在代码编译阶段；

上文提到，CommonJS导入时，`require` 的路径参数是支持表达式的，例如

```js
// A.js
let fileName = 'example.js'
const bModule = require('./' + fileName)
```

因为该路径在代码执行时是可以动态改变的，所以如果在代码编译阶段就建立各个模块的依赖关系，那么一定是不准确的，只有在代码运行了以后，才可以真正确认模块的依赖关系，因此说CommonJS是动态的。

而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。所以说ES6模块是静态的。

区别二
为了验证这一点，我准备用实例来演示一下

首先来验证CommonJS，代码如下

```js
// B.js
let count = 3

function change() {
    count ++    // 变量count + 1
    console.log('原count值为：', count);  // 打印B.js模块中count的值
}

module.exports = {
    count,
    change
}

// A.js
let count = require('./B.js').count 
let change = require('./B.js').change

console.log('改变前：', count);   
change()     // 调用模块B.js中的change方法，将原来的count + 1
console.log('改变后：', count); 

// 运行A.js文件的结果
改变前：3
原count值为：4
改变后：3

```

在上述代码中我们可以看到，在 A.js 文件中导入了 B.js 文件中的变量 count 和 函数 change，因为导入的 count 只是对原有值的一个拷贝，因此尽管我们调用了**函数 change 改变了 B.js 文件中变量 count 的值**，也**不会影响到 A.js 文件中的变量 count**。

根据这个结果得出结论：**CommonJS导入的变量是对原值的拷贝**

---

接下来再来验证一下ES6 Module，代码如下：

```js
// B.js
let count = 3

function change() {
    count ++        // 变量count + 1
    console.log(count);   // 打印B.js模块中count的值
}

export {count, change}

// A.js
import {count, change} from './B.js';

console.log('改变前：',count);

change()         // 调用模块B.js中的change方法，将原来的count + 1

console.log('改变后：', count);

// 运行A.js文件的结果
改变前：3
原count值为：4
改变后：4

```

相比较于CommonJS的结果，ES6 Module导入的变量 `count` 随着原值的改变而改变了

根据这个结果得出结论：ES6 Module导入的变量是对原值的引用

## UMD

UMD是AMD和CommonJS的糅合，AMD 浏览器第一的原则发展 异步加载模块。CommonJS 模块以服务器第一原则发展，选择同步加载，它的模块无需包装(unwrapped modules)。这迫使人们又想出另一个更通用的模式UMD （Universal Module Definition）。希望解决跨平台的解决方案。

既然要通用，怎么办呢？那就先判断是否支持node.js的模块，存在就使用node.js；再判断是否支持AMD（define是否存在），存在则使用AMD的方式加载。这就是所谓的UMD。

下面看下一些经典库的封装基本都是基于UMD方式的：

- jquery

  ```js
  (function(global, factory) {
  
  	"use strict";
  
  	if (typeof module === "object" && typeof module.exports === "object") {
  
  		// For CommonJS and CommonJS-like environments where a proper `window`
  		// is present, execute the factory and get jQuery.
  		// For environments that do not have a `window` with a `document`
  		// (such as Node.js), expose a factory as module.exports.
  		// This accentuates the need for the creation of a real `window`.
  		// e.g. var jQuery = require("jquery")(window);
  		// See ticket #14549 for more info.
  		module.exports = global.document ?
  			factory(global, true) :
  			function(w) {
  				if (!w.document) {
  					throw new Error("jQuery requires a window with a document");
  				}
  				return factory(w);
  			};
  	} else {
  		factory(global);
  	}
  
  	// Pass this if window is not defined yet
  }(typeof window !== "undefined" ? window : this, function(window, noGlobal) {
  
  	//........
  
  	//amd
  	if (typeof define === "function" && define.amd) {
  		define("jquery", [], function() {
  			return jQuery;
  		});
  	}
  
  	var
  	// Map over jQuery in case of overwrite
  		_jQuery = window.jQuery,
  
  		// Map over the $ in case of overwrite
  		_$ = window.$;
  
  	jQuery.noConflict = function(deep) {
  		if (window.$ === jQuery) {
  			window.$ = _$;
  		}
  
  		if (deep && window.jQuery === jQuery) {
  			window.jQuery = _jQuery;
  		}
  
  		return jQuery;
  	};
  
  	// Expose jQuery and $ identifiers, even in AMD
  	// and CommonJS for browser emulators (#13566)
  	if (!noGlobal) {
  		window.jQuery = window.$ = jQuery;
  	}
  	//return obj
  	return jQuery;
  }))
  ```

  jquery框架使用的是一个自执行函数包裹

  ```js
  (function(xx,oo){
  //...
  }(xx,oo))
  ```

- moment.js

  ```js
  ;(function (global, factory) {
     //三元表达式进行commonjs amd的检测
      typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
      typeof define === 'function' && define.amd ? define(factory) :
      global.moment = factory()
  }(this, (function () { 
  'use strict';
  //工厂函数
  })));
  ```

  moment和jquery的实现方式几乎一样 只是把amd cmd的检测放到了一起做判断，jquery是在工厂函数中进行amd的判断 命名冲突检测 全局暴露等操作；

- 简单案例：

  ```Js
  ((root, factory) => {
    if (typeof define === 'function' && define.amd) {
      //AMD
      define(['jquery'], factory);
    } else if (typeof exports === 'object') {
      //CommonJS
      var $ = requie('jquery');
      module.exports = factory($);
    } else {
      //都不是，浏览器全局定义
      root.testModule = factory(root.jQuery);
    }
  })(this, ($) => {
    //do something...  这里是真正的函数体
  });
  ```

  

## 各种模块规范如何使用?

- CommonJs的话，因为nodeJs就是它的实现，所以使用node就行，也不用引入其他包。

- AMD则是引入RequireJs。
  - 使用教程：[RequireJS in Node](https://requirejs.org/docs/node.html)
- CMD则是引入SeaJs。
  - 使用教程：[Sea.js - A Module Loader for the Web (seajs.github.io)](https://seajs.github.io/seajs/docs/#quick-start)

- UMD: 其实就是一种用来判断各模块使用，从而使用不同的加载的方式。
- ES6 Module：不需要使用包,不再是使用闭包和函数封装的方式进行模块化，而是从语法层面提供了模块化的功能，一般来说平日里写的 `ES6` 模块代码最终都会经由 `Babel`, `Typescript` 等工具处理成 CommonJS 代码来兼容各家浏览器。

## Node.js中使用ES6 Module

CommonJS 模块是 Node.js 专用的，与 ES6 模块不兼容。

1. 语法上面，两者最明显的差异是，CommonJS 模块使用require()和module.exports，ES6 模块使用import和export。

2. 从 Node.js v13.2 版本开始，Node.js 已经默认打开了 ES6 模块支持。

3. Node.js 要求 ES6 模块采用.mjs后缀文件名。也就是说，只要脚本文件里面使用import或者export命令，那么就必须采用.mjs后缀名。

4. Node.js 遇到.mjs文件，就认为它是 ES6 模块，默认启用严格模式，不必在每个模块文件顶部指定"use strict"。
5. 如果不希望将后缀名改成.mjs，可以在项目的package.json文件中，指定type字段为module。一旦设置了以后，该目录里面的 JS 脚本，就被解释用 ES6 模块。
6. 如果这时还要使用 CommonJS 模块，那么需要将 CommonJS 脚本的后缀名都改成.cjs
7. 如果没有type字段，或者type字段为commonjs，则.js脚本会被解释成 CommonJS 模块
8. 注意，ES6 模块与 CommonJS 模块尽量不要混用

## CommonJs 的require 从哪里来？

在编写 CommonJS 模块的时候，我们会使用 `require` 来加载模块，使用 `exports` 来做模块输出，还有 `module`，`__filename`, `__dirname` 这些变量，为什么它们不需要引入就能使用？

原因是 Node 在解析 JS 模块时，会先按文本读取内容，然后将模块内容进行包裹，在外层裹了一个 function，传入变量。再通过 `vm.runInThisContext` 将字符串转成 `Function`形成作用域，避免全局污染。

```js
let wrap = function(script) {
  return Module.wrapper[0] + script + Module.wrapper[1];
};
 
const wrapper = [
  '(function (exports, require, module, __filename, __dirname) { ',
  '\n});'
];
```

​		于是在 CommmonJS 的模块中可以不需要 require，直接访问到这些方法，变量。

​		参数中的 `module` 是当前模块的的 `module` 实例（尽管这个时候模块代码还没编译执行），`exports` 是 `module.exports` 的别名，最终被 `require` 的时候是输出 `module.exports` 的值。`require` 最终调用的也是 Module._load 方法。`__filename，__dirname` 则分别是当前模块在系统中的绝对路径和当前文件夹路径。

​		`CommonJS` 模块加载过程是同步阻塞性地加载，在模块代码被运行前就已经写入了 `cache`，同一个模块被多次 `require` 时只会执行一次，重复的 `require` 得到的是相同的 `exports` 引用。

## __esModule 是什么？干嘛用的？

​		使用转换工具处理 `ES6` 模块的时候，常看到打包之后出现 `__esModule` 属性，字面意思就是将其标记为 `ES6 Module`。这个变量存在的作用是为了方便在引用模块的时候加以处理。

​		例如 `ES6` 模块中的 `export default` 在转化成 CommonJS 时会被挂载到 `exports['default']` 上，当运行 `require('./a.js')` 时 是不能直接读取到 default 上的值的，为了和 ES6 中 `import a from './a.js'`的行为一致，会基于 `__esModule` 判断处理。

例子：

```js
// a.js
export default 1;
 
// main.js
import a from './a';
 
console.log(a);
```

转化后：

```js
// a.js
Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = 1;
 
// main.js
'use strict';
 
var _a = require('./a');
 
var _a2 = _interopRequireDefault(_a);
 
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
 
console.log(_a2.default);
```

## 参考文章

> [AMD 和 CMD 的区别有哪些？ - yanmuxiao - 博客园 (cnblogs.com)](https://www.cnblogs.com/yanmuxiao/p/8761603.html)
>
> [AMD · amdjs/amdjs-api Wiki (github.com)](https://github.com/amdjs/amdjs-api/wiki/AMD)
>
> [CMD 模块定义规范 · Issue #242 · seajs/seajs (github.com)](https://github.com/seajs/seajs/issues/242)
>
> [JavaScript模块化编程简介 · Issue #2 · gnipbao/iblog (github.com)](https://github.com/gnipbao/iblog/issues/2)
>
> [聊聊CommonJS与ES6 Module的使用与区别_「零一」的博客-CSDN博客](https://lpyexplore.blog.csdn.net/article/details/109740863)
>
> [AMD，CMD，CommonJs到底是何方神圣？以及三者之间的区别_weixin_43597801的博客-CSDN博客_amd commonjs](https://blog.csdn.net/weixin_43597801/article/details/125758687)
>
> [CommonJS 和 ES6 Module 究竟有什么区别？ - 掘金 (juejin.cn)](https://juejin.cn/post/6844904080955932680)

