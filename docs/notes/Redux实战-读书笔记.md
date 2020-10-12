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

### 优点

- 可预测性
- 开发者体验良好
- 可测试性
- 学习曲线平滑
- 包体积小

### 缺点

- 很多模板代码
- 很难规划一个全局的 store

## 第一个 Redux 应用程序

需求，设计一个类似 Trello 的任务看板应用 Parsnip。

1. 根据需求，设计应用的 state

   ```javascript
   const state = {
     tasks: [
       {
         id: uuid(),
         title: "",
         description: "",
         status: STATUS.TODO,
       },
     ],
   };
   ```

2. 使用 [create-react-app](https://create-react-app.dev/docs/getting-started) 搭建项目框架
3. 新建无状态组件（展示型组件） Task、TaskList 以及有状态组件 TaskPage，注意，这一步只写静态页面
4. 配置 Redux
5. 使用 react-redux 连接 React 以及 Redux

   - 提供 Provider 组件，使其任何子组件都能访问 Redux store
   - connect 函数，将 React 以及 Redux 连接起来。如果某个组件添加了 connect，表示这个组件与 Redux store 连接了起来，这个组件称为容器型组件。

     ```javascript
     function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
     ```

6. 添加交互
   1. 视图 dispatch action，action 由 action creator 创建
   2. reducer 执行，更新 store 中的 state
   3. 视图更新

## 扩展（TODO）

- GraphQL
  - Apollo
  - Relay
- MobX
- ImmutableJS
- Toy-Redux
