# Redux 实战-读书笔记

## Redux 介绍

### 什么是状态

状态存在于组件或应用程序中。Redux 利用纯函数，将应用程序的状态存储在一个 JavaScript 对象中，应用程序的状态是完全可预见的。

### 什么是 Flux

Flux 抛弃了 MVC 架构模式中经常采用的双向数据绑定，转而采用单向数据流。涉及的主要概念有：

- action(s)
- dispatcher
- store(s)
- view(s)

### 什么是 Redux

> Redux 是 JavaScript 应用程序的可预测状态的容器。

主要概念有：

- store：应用程序的所有状态都存储在单个对象中，同时也是暴露给用户的接口
  - `getState()`
  - `dispatch(action)`
  - `subscribe(listener)`
- action(s)

  ```javascript
  const action = {
    type: "CREATE_POST",
    payload: {},
  };
  ```

- reducer(s)

  ```javascript
  const reducer = (previousState, action) => nextState;
  ```

3 个原则：

- 单一数据源，也就是只有一个 store
- 状态是只读的，状态只能通过发起 action 进行更改
- 改变由纯函数进行，通过 reducer 接收 action，更改 store，reducer 是纯函数

## 优点

- 可预测性
- 开发者体验良好
- 可测试性
- 学习曲线平滑
- 包体积小

## 缺点

- 很多模板代码
- 很难规划一个全局的 store

## 扩展（TODO）

- GraphQL
  - Apollo
  - Relay
- MobX
