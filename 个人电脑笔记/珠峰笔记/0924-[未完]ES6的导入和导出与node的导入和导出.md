在JavaScript ES6中，如果想要导入一个常量、函数、文件、模块等需要先在导入之前做导出的工作。导出的时候使用 export 或者 export default；导入的时候使用 import 或者 import {} 的方式。

# 导出：export 和 export default ；导入 import 和 import {}

在JavaScript ES6中，export与export default均可用于导出常量、函数、文件、模块等，你可以在其它文件或模块中通过 import + (常量 | 函数 | 文件 | 模块) 名的方式，将其导入，以便能够对其进行使用。

## export 和 export default 的区别

1.在一个文件或模块中，export可以有多个，export default仅有一个；
2.通过export方式导出，在导入时要使用 import { } ，export default则不需要；
\3. 输出的值数量有不同：
(1) 输出单个值，使用export default，
(2) 输出多个值，使用export，
(3) export default与普通的export不要同时使用，
4.export能直接导出变量表达式，export default不行。

#### 使用 export 方式导出并使用 import{} 方式导入

```javascript
//demo1 export方式
const Say = 'HelloWorld';
const Person = {
	name:Lbc,
	age:27
};


export Say;
export Person;
export var title = '他很帅'；
1234567891011
```

导入的时候则要对应使用 import {} 的方式

```javascript
//demo1 import {} 方式
import{Say,Person,title} from './demo1.js'
console.log(Person.name + '===' + Person.age)
console.log(title +'   '+ Say)
1234
```

#### 使用 export default 方式导出并使用 import 方式导入

```javascript
//demo1 export default 方式
const Person = {
	name:Lbc,
	age:27
};

export default Person
1234567
```

导入的时候则要对应使用 import 的方式

```javascript
//demo1 import方式
import Person from './demo2.js'
console.log(Person.name + '===' + Person.age)
123
```

------

## 注意：

1、export default 向外暴露的成员，可以使用任意变量来接收
2、在一个模块中，export default 只允许向外暴露一次
3、在一个模块中，可以同时使用export default 和export 向外暴露成员
4、使用export向外暴露的成员，只能使用{ }的形式来接收，这种形式，叫做【按需导出】
5、export可以向外暴露多个成员，同时，如果某些成员，在import导入时，不需要，可以不在{ }中定义
6、使用export导出的成员，必须严格按照导出时候的名称，来使用{ }按需接收
7、使用export导出的成员，如果想换个变量名称接收，可以使用as来起别名

# node的导入和导出

导入：`require("xxx")`

导出：`module.exports = 'xxx'`(字符串或者函数)