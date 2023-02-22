---
title: "Redux 异步"
date: 2023-02-19T16:28:00+08:00
lastmod: 2023-02-19T16:28:00+08:00
draft: false
toc: true
keywords: ["daily"]
description: "Redux 异步"
tags: ["JavaScript", "Redux", "React"]
author: "xuyou"
---

首先可见[这个问答](https://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux?noredirect=1&lq=1)，即 Redux 明明可以处理异步场景，为什么还需要中间件呢？

<!--more-->

然后来自 [Dan Abramov](https://stackoverflow.com/users/458193/dan-abramov) 的回答，**直接处理异步场景其实并没有任何问题** ，只是说在大型应用的开发中，对于我们需要复用 action 的场景，这种方式可能会不方便。在[这个回答](https://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559#35415559)中举了个🌰来解释不方便在哪里。

正文从此开始：

## 直接处理异步场景

比如我们直接使用 redux 来处理异步场景：

```js
store.dispatch({ type: 'SHOW_NOTIFICATION', text: 'You logged in.' })
setTimeout(() => {
  store.dispatch({ type: 'HIDE_NOTIFICATION' })
}, 5000)
```

在调用了 `connect()` 后的组件中则像下面这样：

```js
this.props.dispatch({ type: 'SHOW_NOTIFICATION', text: 'You logged in.' })
setTimeout(() => {
  this.props.dispatch({ type: 'HIDE_NOTIFICATION' })
}, 5000)
```

区别就在于在组件中获得的 `dispatch` 是从 props 注入的。

如果不想在不同的组件中复制粘贴相同的 action，我们通常会将他们抽离出来作为一个方法作为 action creator，而不是 dispatch 一个对象：

```js
// actions.js
export function showNotification(text) {
  return { type: 'SHOW_NOTIFICATION', text }
}
export function hideNotification() {
  return { type: 'HIDE_NOTIFICATION' }
}

// component.js
import { showNotification, hideNotification } from '../actions'

this.props.dispatch(showNotification('You just logged in.'))
setTimeout(() => {
  this.props.dispatch(hideNotification())
}, 5000)
```

或者如果我们之前已经使用 `connect()` 绑定过这两个方法了：

```js
this.props.showNotification('You just logged in.')
setTimeout(() => {
  this.props.hideNotification()
}, 5000)
```

目前我们还没有用过任何中间件或者其他高级技巧，我们也实现了异步处理。

## 提取出 action creator

上述🌰可以很好的处理简单场景，但是我们也会发现一些小问题：

- 当我们想在任何地方展示通知栏的时候都需要复制这一段代码；
- 通知栏并没有带唯一 ID，所以当我们快速多次触发展示通知栏时会出现竞态条件：当前一个 timeout 结束的时候会派发 `HIDE_NOTIFICATION`，会错误地隐藏第二个通知栏。

为了解决上述问题，我们通常会抽离出一个方法处理 timeout 和 action 的逻辑。这个方法大概会像下面这样：

```js
// actions.js
function showNotification(id, text) {
  return { type: 'SHOW_NOTIFICATION', id, text }
}
function hideNotification(id) {
  return { type: 'HIDE_NOTIFICATION', id }
}

let nextNotificationId = 0
export function showNotificationWithTimeout(dispatch, text) {
  // 维护一个 nextNotificationId 让 reducer 可以忽略非当前 ID 的 HIDE_NOTIFICATION 的 action
  // 或者我们也可以存储一个 timerId 便于调用 clearTimeout()
  const id = nextNotificationId++
  dispatch(showNotification(id, text))

  setTimeout(() => {
    dispatch(hideNotification(id))
  }, 5000)
}
```

现在组件内部就可以调用 `showNotificationWithTimeout` 来解决复制代码和竞态的问题：

```js
// component.js
showNotificationWithTimeout(this.props.dispatch, 'You just logged in.')

// otherComponent.js
showNotificationWithTimeout(this.props.dispatch, 'You just logged out.')
```

会注意到第一个参数是 `this.props.dispatch`，通常我们的组件是可以获取到 `dispatch` 的，但是这里需要的是一个外部方法来处理 `dispatch`，就需要作为参数传入。

如果我们的 `store` 是一个单例，那就可以直接引入并调用：

```js
// store.js
export default createStore(reducer)

// actions.js
import store from './store' // ##

// ...

let nextNotificationId = 0
export function showNotificationWithTimeout(text) {
  const id = nextNotificationId++
  store.dispatch(showNotification(id, text)) // ##

  setTimeout(() => {
    store.dispatch(hideNotification(id))
  }, 5000)
}

// component.js
showNotificationWithTimeout('You just logged in.')

// otherComponent.js
showNotificationWithTimeout('You just logged out.')
```

这样处理看起来简单多了，但是 **并不推荐** ，最重要的原因是这样会限制 `store` 必须为一个单例，这样会令[服务端渲染](https://redux.js.org/usage/server-rendering)比较难实现，因为在服务端每一个请求需要拥有自己的 `store`，这样才可以让不同的用户拥有不同的预获取数据。

单例的 `store` 也会加大单元测试的难度，测试 action 的时候很难 mock store，因为会引用具体的真实的 `store`。我们甚至不能从外部重置 `store` 状态。

所以官方不鼓励模块内导出一个单例的 `store`，除非我们可以确认 app 不会做服务端渲染：

所以回到之前的版本：

```js
// actions.js

// ...

let nextNotificationId = 0
export function showNotificationWithTimeout(dispatch, text) {
  const id = nextNotificationId++
  dispatch(showNotification(id, text))

  setTimeout(() => {
    dispatch(hideNotification(id))
  }, 5000)
}

// component.js
showNotificationWithTimeout(this.props.dispatch, 'You just logged in.')

// otherComponent.js
showNotificationWithTimeout(this.props.dispatch, 'You just logged out.')
```

这个版本当前解决了复制代码和竞态的问题。

## thunk 中间件

对于简单的应用，这个方法可以满足需求，并不需要使用中间件。

在一些大型应用中，我们就会发现不方便的地方。

首先，我们需要在使用这个方法的地方传入 dispatch，这让我们[分隔容器组件和展示组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)变得很困难，因为任何触发异步 action 的组件都必须接收 dispatch 作为参数，这样才能继续往下传。也不能把这个方法绑定到 `connect()` 上，因为 `showNotificationWithTimeout()` 并没有返回 Redux action 。所以它不是一个真正的 action creator。

此外，区分哪个方法是同步的 action creator 例如 `showNotification()` 还是异步的 action 帮助方法例如 `showNotificationWithTimeout()` 也是很麻烦的，需要小心区分并且对他们进行不同的处理。

这就是为什么我们需要找到一个合适的方法 **给辅助函数提供 dispatch 参数，并且帮助 Redux 区分出哪些是与同步 action 完全不同的异步 action creator**。

接下来我们来看一下 Redux Thunk 是怎么解决这个问题的。

```js
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'

const store = createStore(
  reducer,
  applyMiddleware(thunk)
)

// 可以识别普通的 对象格式的 action
store.dispatch({ type: 'INCREMENT' })

// 使用 thunk 中间件，也可以识别函数类型的 action
store.dispatch(function (dispatch) {
  // ... 可以触发多次
  dispatch({ type: 'INCREMENT' })
  dispatch({ type: 'INCREMENT' })
  dispatch({ type: 'INCREMENT' })

  setTimeout(() => {
    // ... 甚至是异步
    dispatch({ type: 'DECREMENT' })
  }, 1000)
})
```

当开启中间件了之后，**如果我们派发了一个函数**，Redux thunk 中间件会将 `dispatch` 作为参数传给这个函数。中间件会处理这样的函数 action 所以不需要担心我们的 reducer 会收到奇怪的函数参数。

目前好像看起来并不是非常有用的功能，但是他让我们可以像定义一个普通的 action creator 那样去定义`showNotificationWithTimeout()`：

```js
// actions.js
function showNotification(id, text) {
  return { type: 'SHOW_NOTIFICATION', id, text }
}
function hideNotification(id) {
  return { type: 'HIDE_NOTIFICATION', id }
}

let nextNotificationId = 0
export function showNotificationWithTimeout(text) {
  return function (dispatch) { // ##
    const id = nextNotificationId++
    dispatch(showNotification(id, text))

    setTimeout(() => {
      dispatch(hideNotification(id))
    }, 5000)
  }
}
```

需要注意的是这里的方法和上一节的 `showNotificationWithTimeout()` 其实是很像的，区别在于他返回了一个函数，并且函数的第一个参数是 `dispatch`。

在组件中怎么使用呢？如下：

```js
// component.js
showNotificationWithTimeout('You just logged in.')(this.props.dispatch)
```

这里调用了异步的 action creator 来得到他返回的函数，并将 dispatch 参数传入。

这里看起来似乎比之前的方法更尴尬，为什么要这样做呢？

**当启用 Redux Thunk 中间件后，任何时候我们想要派发一个方法而不是一个纯 action 对象，中间件会直接传递 `dispatch` 作为第一个参数。**

所以我们可以像这样使用：

```js
// component.js
this.props.dispatch(showNotificationWithTimeout('You just logged in.'))
```

最后，对于组件来说，dispatch 一个异步的 action 看起来和 dispatch 一个普通的同步 action 看起来并没有啥区别。这是个好现象，因为组件就不应该关心那些动作到底是同步的还是异步的，我们已经将它抽象出来了。

现在 Redux 已经可以区分这些特殊的 action creator，就可以在任何可以使用普通的 action creator 的地方使用他们了。比如，我们可以直接在 connect() 中使用：

```js
// actions.js
function showNotification(id, text) {
  return { type: 'SHOW_NOTIFICATION', id, text }
}
function hideNotification(id) {
  return { type: 'HIDE_NOTIFICATION', id }
}

let nextNotificationId = 0
export function showNotificationWithTimeout(text) {
  return function (dispatch) {
    const id = nextNotificationId++
    dispatch(showNotification(id, text))

    setTimeout(() => {
      dispatch(hideNotification(id))
    }, 5000)
  }
}

// component.js
import { connect } from 'react-redux'

// ...
this.props.showNotificationWithTimeout('You just logged in.')

// ...
export default connect(
  mapStateToProps,
  { showNotificationWithTimeout }
)(MyComponent)
```

## Thunk 中获取 State

通常我们的 reducer 中也会包含修改 state 的业务逻辑，然而 Redux 只会在 action 被派发时才会更新 state，那就会面临一个问题，即当我们在副作用（例如远程调用一个 API）中条件触发异步 action creator 的话应该怎么处理？

如果不使用 thunk 中间件，我们可以在组件中这样写：

```js
// component.js
if (this.props.areNotificationsEnabled) {
  showNotificationWithTimeout(this.props.dispatch, 'You just logged in.')
}
```

但是我们提取 action creator 的目的就是为了将这些重复的逻辑集中处理。幸运的是，Redux Thunk 提供了一个读取当前 store state 的方法。那就是除了传入 dispatch 参数外，他还会传入 getState 作为第二个参数，这样 thunk 就可以读取 store 的当前状态了。

```js
let nextNotificationId = 0
export function showNotificationWithTimeout(text) {
  return function (dispatch, getState) {
    // 这里可以提前退出
    // Redux 不关心这里的返回值，没返回值也没关系
    if (!getState().areNotificationsEnabled) {
      return
    }

    const id = nextNotificationId++
    dispatch(showNotification(id, text))

    setTimeout(() => {
      dispatch(hideNotification(id))
    }, 5000)
  }
}
```

但是不要滥用这种方法！当有缓存数据可用时，它有利于避免 API 调用，但是最好不要在这种方法的基础上构建业务逻辑。如果我们只是用 getState 来做条件判断是否要 dispatch action，可以考虑将这些逻辑放到 reducer 里面去。

## 下一步

现在我们已经对 thunk 的工作原理有了一个基本的概念，可以点击[这里](https://redux.js.org/introduction/examples#async)看看更多的🌰。

我们会发现很多例子里 thunk 中间件都返回了 Promise，这个不是必须的，但是用起来却很方便。Redux 并不关心 thunk 返回了什么值，但是他会将这个值通过外层的 dispatch() 透传出来。所以我们可以在 thunk 中返回一个 Promise 并且等他完成：

```js
dispatch(someThunkReturningPromise()).then(...)
```

也可以将一个复杂的 thunk action creator 拆分成几个更小的 thunk action creator。这是因为 thunk 提供的 dispatch 也可以接收 thunk，所以你可以一直嵌套的 dispatch thunk。而且结合 Promise 的话可以更好的控制异步流程。

在一些更复杂的应用中，我们可能会发现异步控制流程很难通过 thunk 表示。比如，重试失败的请求，使用 token 进行重新授权认证，或者在一步一步的引导流程中，使用这种方式可能会很繁琐，而且容易出错。如果你有这些需求，你可以考虑下一些更高级的异步流程控制库，比如 [Redux Saga](https://github.com/yelouafi/redux-saga) 或者 [Redux Loop](https://github.com/raisemarketplace/redux-loop)。可以评估下，看哪个更适合业务需求。

最后，如果我们并没有真实的需求，不要使用任何库(包括 thunk)。我们的实现都是要看需求的，也许这个简单的方案就能满足：

```js
store.dispatch({ type: 'SHOW_NOTIFICATION', text: 'You logged in.' })
setTimeout(() => {
  store.dispatch({ type: 'HIDE_NOTIFICATION' })
}, 5000)
```
