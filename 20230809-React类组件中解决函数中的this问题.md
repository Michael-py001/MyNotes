# 20230809-React类组件中解决函数中的this问题

## 问题

下面沙箱中的按钮，点击后会报错，原因是`changeAge`中的this为undefined。

<iframe src="https://codesandbox.io/embed/optimistic-pine-tftm8w?fontsize=14&hidenavigation=1&module=%2Fsrc%2FApp.js&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="optimistic-pine-tftm8w"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 原因

在 React 中，类组件中的方法默认不会绑定 this，因此在调用类组件中的方法时，需要手动绑定 this 或使用箭头函数来绑定 this。

## 解决方案1：点击事件改为箭头函数

在上述的代码中，`changeAge` 方法中的 this 指向的是 `undefined`，因为它没有被正确绑定。为了解决这个问题，你可以使用箭头函数来定义 changeAge 方法，这样它就会自动绑定 this，例如：

```jsx
import React from 'react'

class Example extends React.Component {
  state = {
    name: '小明',
    age: '99'
  }
  //使用箭头函数
  changeAge = () => {
    console.log(this)
    this.setState({
      age: '100'
    });
  }
  render() {
    return <button onClick={this.changeAge}>{this.state.name}的年龄是{this.state.age}</button>
  }
}
export default Example
```

在这个例子中，我们使用箭头函数来定义 changeAge 方法，这样它就会自动绑定 this，而不需要手动绑定。这样，当你点击 button 时，changeAge 方法就能正确地访问 this.state 和 this.setState。

## 解决方案2：使用bind绑定this

除了使用箭头函数来绑定 this，还有其他几种方法可以解决这个问题。

一种方法是在构造函数中手动绑定 this，例如：

```jsx
import React from 'react'

class Example extends React.Component {
  constructor(props) {
    super(props);
    //使用bind绑定当前this
    this.changeAge = this.changeAge.bind(this);
  }
  state = {
    name: '小明',
    age: '99'
  }
  changeAge() {
    console.log(this)
    this.setState({
      age: '100'
    });
  }
  render() {
    return <button onClick={this.changeAge}>{this.state.name}的年龄是{this.state.age}</button>
  }
}
export default Example
```

在这个例子中，我们在构造函数中手动绑定了 changeAge 方法的 this，这样它就能正确地访问 this.state 和 this.setState。

## 解决方案3: render方法改为箭头函数

另一种方法是使用箭头函数来定义 render 方法，例如：

```jsx
import React from 'react'

class Example extends React.Component {
  state = {
    name: '小明',
    age: '99'
  }
  changeAge() {
    console.log(this)
    this.setState({
      age: '100'
    });
  }
  render = () => {
    return <button onClick={this.changeAge}>{this.state.name}的年龄是{this.state.age}</button>
  }
}
export default Example
```

需要注意的是，使用箭头函数来定义方法会导致该方法无法被继承，因此在某些情况下可能不适用。