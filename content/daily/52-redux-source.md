---
title: "Redux 源码"
date: 2022-06-23T13:52:54+08:00
lastmod: 2022-06-23T13:52:54+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Redux 源码"
tags: ["JavaScript", "React", "Redux"]
author: "xuyou"
---

先来看一个基本的 redux 使用：

```js
const initState = {
  count: 0
};

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "DECREMENT":
      return { ...state, count: state.count - 1 };
    case "RESET":
      return { ...state, count: 0 };
    default:
      return state;
  }
}

const store = createStore(reducer, initState);

// 订阅变化，结合 react 的话页面更新的逻辑在这里处理
store.subscribe(() => {
  console.log(store.getState(), "subscribe");
});

// 派发 action
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'DECREMENT' });
store.dispatch({ type: 'RESET' });
```

基于上述可以实现一个简单的 redux：

```js
function createStore(reducer, initState) {
  let state = initState;
  let listeners = [];

  function subscribe(callback) {
    listeners.push(callback);
  }

  function dispatch(action) {
    state = reducer(state, action);

    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i];
      listener();
    }
  }

  function getState() {
    return state;
  }

  return {
    getState,
    subscribe,
    dispatch
  };
}
```

再下来实现结合 react 的 `connect` 方法：

```js
function connect(mapStateToProps, mapDispatchToProps) {
  return function (WrappedComponent) {
    return function NewComponent(...selfProps) {
      const context = useContext(ReduxContext); // 需要借助 context，从 context 注入
      const { store } = context;
      const [stateProps, setStateProps] = useState({});

      function update() {
        const newState = mapStateToProps(store.getState());
        const newDispatch = mapDispatchToProps(store.dispatch);
        setStateProps({ ...newState, ...newDispatch });
      }

      useEffect(() => {
        const unsubscribe = store.subscribe(() => update());
        update();
      }, []);

      return <WrappedComponent {...selfProps} {...stateProps} />;
    };
  };
}
```

看到这里需要借助 context：

```jsx
const store = createStore(reducer, initState);
const ReduxContext = React.createContext();

function Provider(props) {
  const { store, children } = props;
  return (
    <ReduxContext.Provider value={{ store }}>{children}</ReduxContext.Provider>
  );
}
```

使用时就可以像下面这样：

```jsx
const App = (props) => {
  return (
    <div>
      <button onClick={props.increase}>点击加1</button>
      <button onClick={props.decrease}>点击减1</button>
      <button onClick={props.reset}>重置</button>
      <h1>{props.count}</h1>
    </div>
  );
};

const mapStateToProps = (state) => {
  return {
    count: state.count
  };
};

const mapDispatchToProps = (dispatch) => ({
  increase: () => dispatch({ type: "INCREMENT" }),
  decrease: () => dispatch({ type: "DECREMENT" }),
  reset: () => dispatch({ type: "RESET" })
});

const NewApp = connect(mapStateToProps, mapDispatchToProps)(App);

ReactDOM.render(
  <Provider store={store}>
    <NewApp />
  </Provider>,
  document.getElementById("app")
);
```