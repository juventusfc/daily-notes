# Why Do We Write super(props)? 读后感

Dan 在这篇文章中，讲了 React 引进 class 组件的历史，引申出写 class 组件经常写到的代码：

```javascript
class Checkbox extends React.Component {
  constructor(props) {
    super(props); // 🤖
    this.state = { isOn: true };
  }
  // ...
}
```

关于这一行代码 `super(props);`，有两点需要注意。

## this 不能在 super 之前使用

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // 🔴 This is disallowed, read below why
    super(name);
  }
  greetColleagues() {
    alert("Good morning folks!");
  }
}
```

上面的代码在 super 之前使用了 this，这就会有一个问题：

如果由于需求变动， greetColleagues 改为：

```javascript
greetColleagues() {
  alert('Good morning folks!');
  alert('My name is ' + this.name + ', nice to meet you!');
}
```

这种情况下，由于 this.name 压根还没有设置，会导致 bug。

## 必须调用 super(props)

```javascript
// Component 定义
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}

// 生成实例后，会重新赋值 props
const instance = new YourComponent(props);
instance.props = props;
```

从 React 源码看，生成实例后，会实例重新赋值 props，也就是说，React 不强制用户必须调用 ``super(props)`。但是，最佳实践使调用 `super(props)`，因为如果不调用，在如下的情况下会引起 bug：

```javascript
class Button extends React.Component {
  constructor(props) {
    super(); // 😬 We forgot to pass props
    console.log(props); // ✅ {}
    console.log(this.props); // 😬 undefined
  }
  // ...
}
```
