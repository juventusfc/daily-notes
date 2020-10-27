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
6. 添加交互
   1. 视图 dispatch action，action 由 action creator 创建
   2. reducer 执行，更新 store 中的 state
   3. 视图更新

### dispatch 同步 action 一次

我们先来看 Redux 中发生了什么：

```javascript
// Redux 源码
function dispatch(action) {
  // 执行 reducer
  try {
    isDispatching = true;
    currentState = currentReducer(currentState, action);
  } finally {
    isDispatching = false;
  }

  // 执行已订阅的函数
  const listeners = (currentListeners = nextListeners);
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i];
    listener();
  }

  return action;
}
```

从 Redux 源码可以知道，在执行 dispatch 函数时，会调用 reducer，而 reducer 会返回一个新的 state 当作 store 的 state。

我们接着来看看 React 这边发生了什么：

由于 react-redux 的存在，当 store 中的 state 发生变化，会立即执行 mapStateToProps 方法，将最新的 state 传入，也就是说会改变 React 组件的 props。

但是，React 并不保证马上重新渲染，这是 React 的机制，跟 Redux 无关。

这里简单讲一下 react-redux 的工作原理。

在 react-redux 这个包里，主要涉及 2 个 API：

- Provider，使 store 当作应用的 Context，使其任何子组件都能访问 Redux store
- connect，将 React 以及 Redux 连接起来。如果某个组件添加了 connect，表示这个组件与 Redux store 连接了起来，这个组件称为容器型组件。

  ```javascript
  function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
  ```

  从原理上来说，connect 是一个高阶组件，它 subscribe 了 store，也就是说，store 中任何 state 的改变，它都能知道（发布-订阅模式）。然后，这个高阶组件通过对比 state 的变化，决定是否要执行 setState，而 setState 会引起高阶组件的重新渲染，从而引起被封装组件的重新渲染。

所以，整个时序可以描述为：

`const LinkFilter = connect(mapStateToProps, mapDispatchToProps)(Link)` 点击一次

1. Link 上调用 store.dispatch
2. reducer 计算得到新的 state
3. 通知 LinkFilter,Redux store state 有更新
4. LinkFilter 执行 setState，引起重新渲染
5. 由于 React 中，父组件的重现渲染会引起子组件的重新渲染，Link 也会重新渲染

### dispatch 同步 action 多次

上面讲了 dispatch 同步 action 一次的情况。现在我们分析下连着 dispatch 同步 action 多次会发生什么。

整个时序可以描述为：

`const LinkFilter = connect(mapStateToProps, mapDispatchToProps)(Link)` 点击两次

1. Link 上调用 store.dispatch
2. reducer 计算得到新的 state
3. 通知 LinkFilter,Redux store state 有更新
4. LinkFilter 执行 setState
5. 🤷 引起重新渲染
6. 🤷 由于 React 中，父组件的重现渲染会引起子组件的重新渲染，Link 也会重新渲染
7. Link 上调用 store.dispatch
8. reducer 计算得到新的 state
9. 通知 LinkFilter,Redux store state 有更新
10. LinkFilter 执行 setState
11. 🤷 引起重新渲染
12. 🤷 由于 React 中，父组件的重现渲染会引起子组件的重新渲染，Link 也会重新渲染

注意，如果点击两次的时间间隔很短，由于 setState 的存在，第 5、6 和第 11、12 步中的重新渲染可能会合并。

## 调试 Redux 应用程序

由于 Redux 采用了单向数据流的设计方式，store state 的任何变化，都可追溯至 action，作为前端，自然会有将这个过程可视化的想法。Redux DevTools 应运而生。更妙的是，它提供了浏览器插件版本，可以集成进浏览器开发工具。

关于热加载，我并不喜欢用，所以书上的那些就不看了。

## 使用 API

这一章主要讲 dispatch 异步 action 的情况。

由于存在同步 action 和异步 action，所以有同步 action 创建器以及异步 action 创建器。

同步 action 创建器：

```javascript
// 返回 plain object
function fetchTasksSucceeded(tasks){
  return {
    type:'FETCH_TASKS_SUCCEEDED'，
    payload:[]
  }
}
```

异步 action 创建器：

```javascript
// 返回一个函数，这个函数会被 middleware 识别出
function fetchTasks() {
  return (dispatch) => {
    api.fetchTasks().then((res) => {
      dispatch(fetchTasksSucceeded(res.data));
    });
  };
}
```

在不改变 reducer 的情况下，就引申出了中间件 redux-thunk，它使得 Redux 有了 dispatch 异步 action 的能力。

```javascript
dispatch(fetchTasksSucceeded()); // dispatch 同步 action
dispatch(fetchTasks()); // dispatch 异步 action
```

完整的 fetchTasks 步骤如下：

1. 视图 dispatch 异步 action： fetchTasks()
2. 中间件接管这个异步 action，执行里面的代码

   ```javascript
   function fetchTasks() {
     return (dispatch) => {
       dispatch(FETCH_TASKS_STARTED);

       axios
         .fetch()
         .then(() => dispatch(FETCH_TASKS_SUCCEEDED))
         .catch(() => dispatch(FETCH_TASKS_FAILED));
     };
   }
   ```

3. dispatch 同步 action： FETCH_TASKS_STARTED ，reducer 执行更改 store，页面更新
4. 执行异步请求
5. 执行其他代码
6. 异步请求返回，dispatch 同步 action： FETCH_TASKS_SUCCEEDED 或 FETCH_TASKS_FAILED，reducer 执行更改 store，页面更新

## 中间件

Redux 借用了 Express.js 中的中间件概念，是位于 action 和 store 之间的代码，也就是说，作用于 dispatch action 之后，reducer 执行之前。从设计上来说，相当于封装了原始 dispatch 功能，使 dispatch 功能更强大。

常用的中间件有：

- redux-thunk：支持异步 action
- logger：action 记录

  ```javascript
  // 中间件基本套路
  const logger = (store) => (next) => (action) => {
    console.log("dispatching", action);
    let result = next(action);
    console.log("next state", store.getState());
    return result;
  };
  ```

- 数据分析中间件
- API 中间件。与服务器交互的逻辑都是类似的：

  - 1 个异步 action
  - 3 个同步 action：started、succeeded、failed
  - 3 个同步 action 对应的 reducer

  当有很多这种请求时，写模板代码太麻烦了。为了减少这种模板代码，可以将这些逻辑提取到 API 中间件中，这样，只需要写：

  - 1 个异步 action
  - 3 个同步 action 对应的 reducer

  但是，需不需要这么提取，还是需要看项目和人员。

## 处理复杂的副作用

## 扩展（TODO）

- GraphQL
  - Apollo
  - Relay
- MobX
- ImmutableJS
- Toy-Redux
- Toy-React
- Toy-React-Redux
- Toy-Redux-Thunk
- Toy-Redux-Saga
- 生成器
- 迭代器
