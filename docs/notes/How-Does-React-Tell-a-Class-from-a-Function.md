# How Does React Tell a Class from a Function 读后感

```javascript
// 使用 function 定义组件
function Greeting() {
  return <p>Hello</p>;
}

// 使用 class 定义组件
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

// 用户使用组件
<Greeting />;
```

对用户来说，使用的时候不区分，但是，React 内部需要区分：

```javascript
// 针对 function
const result = Greeting(props); // <p>Hello</p>

// 针对 class
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

由于 class 的引入，使得创建组件有了两种方式。但是，class 组件必须使用 new 来创建实例，这也要求了 React 内部需要处理这种情况。这就涉及到 **new 那一套东西**了。需要解决的问题可以总结为：

|          | new Person()                 | Person()                       |
| -------- | ---------------------------- | ------------------------------ |
| class    | ✅ this is a Person instance | 🔴 TypeError                   |
| function | ✅ this is a Person instance | 😳 this is window or undefined |

React 采用的方式也很巧妙，由于使用 class 定义组件，一定会继承 React.Component，那么就从这里入手。当然，又是**原型链那一套东西**。

class 的原型链和 function 的稍微有点不一样，由于 class 有继承的概念，所以实例的原型链和继承层级相似：

```text
// `extends` chain
Greeting
  → React.Component
    → Object (implicitly)

// `__proto__` chain， 通过 `_proto_` 获得原型
new Greeting()
  → Greeting.prototype
    → React.Component.prototype
      → Object.prototype
```

最终，伪代码类似于：

```javascript
let greeting = new Greeting();

console.log(greeting instanceof Greeting); // true
// greeting (🕵️‍ We start here)
//   .__proto__ → Greeting.prototype (✅ Found it!)
//     .__proto__ → React.Component.prototype
//       .__proto__ → Object.prototype

console.log(greeting instanceof React.Component); // true
// greeting (🕵️‍ We start here)
//   .__proto__ → Greeting.prototype
//     .__proto__ → React.Component.prototype (✅ Found it!)
//       .__proto__ → Object.prototype

console.log(greeting instanceof Object); // true
// greeting (🕵️‍ We start here)
//   .__proto__ → Greeting.prototype
//     .__proto__ → React.Component.prototype
//       .__proto__ → Object.prototype (✅ Found it!)

console.log(greeting instanceof Banana); // false
// greeting (🕵️‍ We start here)
//   .__proto__ → Greeting.prototype
//     .__proto__ → React.Component.prototype
//       .__proto__ → Object.prototype (🙅‍ Did not find it!)

/////////// 可以采用的方式 ///////////////
console.log(Greeting.prototype instanceof React.Component);
// greeting
//   .__proto__ → Greeting.prototype (🕵️‍ We start here)
//     .__proto__ → React.Component.prototype (✅ Found it!)
//       .__proto__ → Object.prototype
```

但是 React 没有采用上面的方式，而是使用了如下方法：

```javascript
// Inside React
class Component {}
Component.prototype.isReactComponent = {};

// We can check it like this
class Greeting extends Component {}
console.log(Greeting.prototype.isReactComponent); // ✅ Yes
```
