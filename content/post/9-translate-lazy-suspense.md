---
title: "React 18 中的 Suspense SSR 架构"
date: 2022-03-16T20:54:44+08:00
lastmod: 2022-03-16T20:54:44+08:00
draft: false
keywords: ["SSR"]
description: "是一篇翻译 -- React 18 中的 Suspense SSR 架构"
tags: ["SSR", "React"]
author: "xuyou"
---

> 原文标题: New Suspense SSR Architecture in React 18
>
> 原文链接: https://github.com/reactwg/react-18/discussions/37

<!--more-->

React 18 将包括对服务端渲染（SSR）性能的架构改进。最重要的新的 API 是 `renderToPipeableStream`。可以在[这里](https://github.com/reactwg/react-18/discussions/22)看到相关的 API 介绍。主要涉及到的之前的 API 是 `<Suspense>`。本篇是对新架构设计以及它解决的问题的介绍。

## 太长不看版

服务端渲染（SSR）即在服务端用 React 组件生成 HTML，并且将 HTML 直接返回给用户。SSR 允许用户在 JavaScript 资源加载和执行之前看到页面内容。

SSR 通常有以下几步：

1. 在服务端，获取 app 所需要的数据；
2. 在服务端，生成 HTML 并返回；
3. 在客户端，加载应用所需 JavaScript 代码；
4. 在客户端，把服务端生成的 HTML 与 JavaScript 逻辑结合（即 “hydration”）。

重点在于 **每一步需要在上一步完成之后才可以开始**。所以当某些部分比剩余部分逻辑慢的时候，就会影响整个应用加载的效率。

React 18 允许我们使用 `<Suspense>` **将应用分成独立的单元** ，这些单元可以单独完成这些步骤而且不会影响到应用的其他部分。因此用户可以快速看到应用的内容并且与之交互。即逻辑最慢的部分不会“拖慢”速度快的部分。这里有个[demo](https://codesandbox.io/s/kind-sammet-j56ro?file=/src/App.js)。

## 什么是 SSR

当用户访问我们的应用时，作为开发者当然希望尽快呈现出可交互的页面：

![full page](../../static-img/9-translate-suspense/1.png)

使用绿色代表可交互的页面，即 JavaScript 事件处理程序也都添加结束。

这个页面在 JavaScript 代码加载结束之前，页面是不能响应用户交互的。这些代码包括 React 和业务代码。所以大多数时间都是在等待代码下载。

如果不使用 SSR，在 JavaScript 代码加载的时候用户只能看到一个空白页面：

![blank page](../../static-img/9-translate-suspense/2.png)

因为空白页面交互不友好所以推荐使用 SSR。但是 SSR 也存在一个问题：无法马上可交互，除非是内置链接或者表单处理项。然而，SSR 可以让用户在 JavaScript 加载的时候看到页面：

![non-interactive page](../../static-img/9-translate-suspense/3.png)

这里使用灰色代替无法交互的部分。这段时间 JavaScript 代码还没有加载完，所以点击按钮并不会有任何响应。但对于内容繁重的网站 SSR 非常有用，因为它可以让网络连接较差的用户在加载 JavaScript 的同时开始阅读或查看内容。

当 React 和应用代码加载结束后，React 会将事件处理程序添加到 HTML 上，React 会在内存中渲染组件树，但不会为它生成 DOM 节点，而是将所有逻辑附加到现有的 HTML 中。

**渲染组件并且添加事件的过程被称为“hydration”**。因为这个过程像是在用交互性和事件处理程序的“水”注入到“干燥”的 HTML 中。在注水之后，我们的页面就会响应事件和状态更新，就像下图：

![full page](../../static-img/9-translate-suspense/1.png)

SSR 不会使应用程序更快地可交互，它只是尽快展示出应用无法交互的版本，所以用户可以在 JS 代码加载的时候看到静态页面。这个技巧对于网络连接不佳的用户来说体验有很大的不同，并提高了整体的感知性能。由于更容易索引和更快的速度，它还可以帮助我们进行搜索引擎排名。

> 提示：不要混淆 SSR 和 Server Components。Server Components 是一个实验中的特性，大概率不会成为初始化 React 18 中的一部分，可以在[这里](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)看到更多关于 Server Components 的内容。 Server Components 是 SSR 的补充，将成为推荐的数据获取方法的一部分，但这篇文章不涉及到相关知识。

## 当前的 SSR 存在哪些问题

### 需要在展示页面之前获取所有数据

目前我们了解到 SSR 的一个问题是不允许组件“等待数据”。在渲染页面之前，需要先获取到所有的数据。例如我们想渲染一篇有评论的博文，评论数据自然是很重要的，所以我们想要在服务端的 HTML 返回中就包括评论组件。但是如果数据库或 API 层较慢，我们就需要从中做选择。可以选择服务端生成的 HTML 中不包括评论组件，那用户就只能在 js 资源加载后看到评论信息。或者就选择服务端生成的 HTML 中包括评论组件，页面的导航栏、侧边栏就都需要等到评论数据返回才能渲染。

### 注水需要等待资源加载完成

JavaScript 代码加载结束后，React 会开始给 HTML 绑定事件来使页面可交互。React 将在渲染组件时“遍历”服务器生成的 HTML，并将事件处理程序添加上去。为了使事件正确绑定到元素上，浏览器中组件生成的树必须与服务器生成的树匹配。造成的结果就是 **必须等待所有 JavaScript 加载结束，才能开始为其中任何一个组件添加事件绑定**。

假设评论组件有非常复杂的交互逻辑，并且需要事件来加载相应的 JavaScript 代码。我们就又面临了两个选择：将评论内容在服务器生成 HTML，以便尽早向用户展示。但是因为只能一次性完成注水，所以在加载评论组件的代码之前，不能开始为导航栏、侧边栏和帖子内容添加交互响应。另一个选择就是可以做代码分割来进行分开加载，那就需要将评论内容从服务端渲染的 HTML 中移除。否则 React 将不知道如何处理这块 HTML 并在注水期间将其删除。

### 需要在注水结束后才可以进行交互

这是注水过程本身的问题。因为 React 注水是一次性的，意味着一旦开始绑定事件，就不会中断。因此，**必须等待所有组件都被注水，然后才能与其中的任何一个部分进行交互**。

继续我们的评论组件，假设它具有繁重的渲染逻辑，可能在我们的电脑上渲染速度很快，但是在某些低端设备上并不便宜，甚至可能会锁定屏幕几秒钟。当然理想情况下，我们根本不会在客户端上有这样的逻辑，但是对于某些重事件和交互的逻辑，这种情况也是有可能发生的。因此，一旦水合开始，用户就无法与导航栏、侧边栏或帖子内容进行交互，直到整个树被注水结束。对于用户期望的导航操作而言，这种体验是非常差的，因为 React 正在注水，会无法响应跳出页面的操作。

### 如何解决这些问题

这些问题之间有一个共同点。它们迫使我们在提前做某事（但会因为阻碍所有工作而影响用户体验）或延迟做某事（会因为浪费时间而影响用户体验）之间做出选择。

这是因为有一个“瀑布”：服务端获取数据 -> 服务端生成 HTML -> 加载代码（客户端）-> 注水（客户端）。在应用程序的前一个阶段完成之前，下一个阶段都不能开始。这就是它效率低下的原因。我们的解决方案是将工作分开，以便我们 **可以为屏幕的一部分而不是整个应用程序执行每个阶段**。

这并不是什么独特的想法：例如 [Marko](https://tech.ebayinc.com/engineering/async-fragments-rediscovering-progressive-html-rendering-with-marko/) 是一个以这种思路实现的 JavaScript 框架。挑战在于如何使这样的模式与 React 编程模型结合起来。React 一开始引入 `<Suspense>` 组件的作用是在客户端支持懒加载。但目标是其实是将其与服务器渲染集成并解决目前的已知问题。

让我们学习一下如何在 React 18 中使用 `<Suspense>` 来解决这些问题。

## React 18：流式 HTML 和选择性补水

Suspense 解锁了 React 18 中的两个主要 SSR 功能：

- 服务端**流式 HTML**：可以使用新的 `renderToPipeableStream` API 替换 `renderToString` 来开启这个特性，像[这里](https://github.com/reactwg/react-18/discussions/22)描述的一样。
- 客户端**选择性补水**：可以使用 [`createRoot`](https://github.com/reactwg/react-18/discussions/5) 并且将需要选择性注水的内容用 `<Suspense>` 包起来。

要了解这些功能的作用以及它们如何解决上述问题，让我们回到我们的示例。

### 获取数据之前流式渲染 HTML

当下的 SSR，渲染 HTML 和注水是 “要么全部要么全不”。首先渲染所有的 HTML：

```jsx
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
   </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section>
    <!-- Comments -->
    <p>First comment</p>
    <p>Second comment</p>
  </section>
</main>
```

客户端最终会获取这样的页面：

![non-interactive page](../../static-img/9-translate-suspense/3.png)

然后等加载完所有代码后会给应用注水：

![full page](../../static-img/9-translate-suspense/1.png)

但是 React 18 给了我们新的可能性。我们可以把应用的一部分包在 `<Suspense>` 中。

例如，我们可以包裹注释块并告诉 React 在组件准备好之前，应该显示 `<Spinner />` 组件：

```jsx
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

通过把 `<Comments>` 包裹到 `<Suspense>` 中，我们告诉 React **不需要等待评论即为页面的其余部分流式传输 HTML**。并且浏览器会在这个位置显示一个占位符（一个旋转图）作为替代。

![spin page](../../static-img/9-translate-suspense/4.png)

并且现在的 HTML 结构中是看不到评论组件的：

```jsx
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
   </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section id="comments-spinner">
    <!-- Spinner -->
    <img width="400" src="spinner.gif" alt="Loading..." />
  </section>
</main>
```

故事到现在并没有结束。当服务器的评论数据准备好后，React **会将额外的 HTML 发送到同一个流中，以及一个最小的内联 `<script>` 标签以将该 HTML 放在“正确的位置”**：

```jsx
<div hidden id="comments">
  <!-- Comments -->
  <p>First comment</p>
  <p>Second comment</p>
</div>
<script>
  // This implementation is slightly simplified
  document.getElementById('sections-spinner').replaceChildren(
    document.getElementById('comments')
  );
</script>
```

结果，即使在 React 本身加载到客户端之前，迟到的评论 HTML 也会正常加载:

![non-interactive page](../../static-img/9-translate-suspense/3.png)

这就解决了我们的第一个问题。现在我们不需要在展示任何元素前获取所有的 HTML。如果屏幕的某些部分拖慢了初始 HTML，我们不必在延迟所有 HTML 或将其从 HTML 中剔除之间做出选择。我们可以只允许该部分稍后在 HTML 流中渲染。

**与传统的 HTML 流式传输不同，它不必按照自上而下的顺序发生**。例如，如果侧边栏需要一些数据，可以将其包装在 Suspense 中，React 将使用一个占位符并继续渲染帖子。然后，当侧边栏 HTML 准备就绪时，React 会将其与将其插入正确位置的 `<script>` 标签并一起流式传输 -- 即使帖子的 HTML 已经发送。React 不要求以任何特定顺序加载数据，我们指定 loading 图应该出现的位置，其余的由 React 计算。

### 在所有代码加载前开始注水

我们可以提前传输 HTML，但是始终存在一个问题。直到 JavaScript 代码加载结束之前，我们都不能开始注水。如果代码量很大，这需要花费一些时间。

为了避免大包，我们会使用 “代码分割”：通常是指定一段代码不需要同步加载，这样打包器会把它们分割到独立的 `<script>` 标签中。

我们可以通过 `React.lazy` 将评论组件从主包中拆分出来：

```jsx
import { lazy } from "react";

const Comments = lazy(() => import("./Comments.js"));

// ...

<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>;
```

在以前，这种写法不会在服务端渲染时任何作用，但是在 React 18 中，`<Suspense>` 组件允许我们 **在评论组件的代码加载结束前给别的部分注水**。

从用户的角度来看，最初他们会看到无法交互的 HTML 内容：

![spin page](../../static-img/9-translate-suspense/4.png)

![non-interactive page](../../static-img/9-translate-suspense/3.png)

然后 React 开始注水，**现在评论组件的代码还没有准备完成，但是也没有关系**：

![partial-active page](../../static-img/9-translate-suspense/5.png)

这是 **选择性注水** 的结果。通过把评论组件包裹到 `<Suspense>` 中，React 不会妨碍剩下的部分流式传输，甚至不会妨碍注水。意味着第二个问题也解决了：我们不需要等待所有代码加载才能开始注水。 React 可以在加载部分时对其进行水合。

React 会在组件代码加载完后开始对这部分 HTML 注水：

![full page](../../static-img/9-translate-suspense/1.png)

多亏了选择性注水特性，大量的 JS 并不会妨碍页面的其余部分变得可交互。

### 在所有 HTML 流式传输之前对页面进行注水

我们在使用流式传输特性时，不需要担心页面是否会以不期待的顺序渲染。例如，当 HTML 正在传输时，可能也需要一段时间才能加载：

![spin page](../../static-img/9-translate-suspense/4.png)

如果这部分 JavaScript 代码在所有 HTML 返回之前就加载结束，也会直接注水，不会等待所有 HTML 传输完毕：

![active and spin page](../../static-img/9-translate-suspense/6.png)

当评论组件 HTML 加载结束，它会以不能交互的方式暂时呈现：

![active and inactive page](../../static-img/9-translate-suspense/7.png)

最后当评论组件的 JavaScript 代码也加载结束，页面就会完全可交互：

![full page](../../static-img/9-translate-suspense/1.png)

### 所有组件注水前页面可交互

当我们把组件包裹到 `<Suspense>` 中后，还有一个体验上的提升，现在注水不会阻碍别的部分与用户的交互。比如当页面加载评论组件时，我们触发了侧边栏的点击：

![click](../../static-img/9-translate-suspense/8.png)

在 React18 中，Suspense 包裹的组件注水会发生在浏览器响应用户交互的间隙。基于这点，**点击事件会被立马响应，浏览器也不会在低性能设备注水时卡顿**。例如，用户在等待页面注水时可以点击导航栏跳转页面~

在之前的例子中，我们只把评论组件包在了 Suspense 中，所以剩下的部分注水是一次性的。然而我们也可以包多层 Suspense。就像下面这样，我们把侧边栏也包在了 Suspense 中：

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<Spinner />}>
    <Sidebar />
  </Suspense>
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

现在这两个组件都可以在包含导航栏和帖子的初始 HTML 之后从服务器流式传输。包两个 Suspense 当然也会对注水有一定影响。假设此时服务端的 HTML 都已加载，但它们的代码尚未加载：

![two suspense](../../static-img/9-translate-suspense/9.png)

当侧边栏和评论代码都加载结束，React 会尝试给两部分注水，从它在树中较早找到的 Suspense 边界开始（上面的示例中，会从侧边栏先开始）。

![hydrate suspense](../../static-img/9-translate-suspense/10.png)

但是比如现在用户开始和评论组件交互，此时 JavaScript 代码还在加载：

![click suspense](../../static-img/9-translate-suspense/11.png)

React 会记录此次点击，并且会提高评论组件注水的优先级因为它拥有较高的优先级：

![high priority suspense](../../static-img/9-translate-suspense/12.png)

当评论组件注水结束后，React 会“回放”记录的点击事件（通过重新派发），并且让组件响应交互。之后 React 没有别的高优先级任务要做，会给侧边栏注水：

![low priority suspense](../../static-img/9-translate-suspense/13.png)

这解决了我们的第三个问题。因为可以选择性注水，所以我们不需要“在交互之前等待所有组件注水完成”，**React 会尽早开始注水，并且会将用户交互的部分优先级提高。**如果我们在整个应用程序中使用 Suspense，选择性补水的好处会变得更加明显，边界将变得更加细化：

![more parts](../../static-img/9-translate-suspense/14.png)

在上图中，用户在注水开始时点击了评论组件的第一个，React 会提高整个父 Suspense 的优先级，但是会跳过不相关的兄弟节点。**这会造成注水是及时的错觉，因为在交互作用中的组件会被先注水。**React 后续会继续给组件其他部分注水。

在开发中，我们可能会将 Suspense 添加到应用的根组件附近：

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<BigSpinner />}>
    <Suspense fallback={<SidebarGlimmer />}>
      <Sidebar />
    </Suspense>
    <RightPane>
      <Post />
      <Suspense fallback={<CommentsGlimmer />}>
        <Comments />
      </Suspense>
    </RightPane>
  </Suspense>
</Layout>
```

在上面的代码中，初始 HTML 可能包含导航栏内容，但其余部分将在加载相关代码后立即流式传输并部分水化，优先考虑用户与之交互的部分。

## 例子

React 准备了一个 [demo](https://codesandbox.io/s/kind-sammet-j56ro?file=/src/App.js) 来演示 Suspense 架构怎么生效，它可以人为减慢的，所以你可以在 server/delays.js 中调整延迟：

- `API_DELAY` 让您可以延长在服务器上获取评论的时间，展示如何提前发送 HTML 的其余部分。
- `JS_BUNDLE_DELAY` 让你延迟 `<script>` 标签的加载，展示评论小部件的 HTML 是如何“弹出”的，甚至在 React 和你的应用程序包被下载之前。
- `ABORT_DELAY` 让您看到服务器“放弃”并在服务器上获取时间过长时将渲染移交给客户端。

## 结论

React 18 为 SSR 提供了两个主要的特性：

- **流式 HTML** 会尽早开始生成 HTML，将 HTML 流式传输以获取其他内容以及将它们放置在正确位置的 `<script>` 标签。
- **选择性注水** 会让我们在其余的 HTML 和 JavaScript 代码完全下载之前尽早开始注水。还会提高用户正在与之交互的部分的优先级，从而产生即时注水的错觉。

这些特性解决了 React 中 SSR 的三个长期存在的问题：

- **在发送 HTML 之前，不再需要等待所有数据都在服务器上获取完。** 相反，一旦有足够的时间来显示应用程序的外壳，服务端就会开始发送 HTML，并在准备好 HTML 时流式传输其余的 HTML。
- **不再需要等待所有 JavaScript 加载完毕才能开始补水。** 相反，我们可以将代码拆分与服务器渲染结合使用。服务器 HTML 将被保留，React 将在相关代码加载时对其进行水合。
- **您不再需要等待所有组件水合开始与页面交互。** 相反，可以依靠选择性注水来确定与用户交互的组件的优先级，并尽早补水。

`<Suspense>` 组件可作为所有这些功能的选择加入。这些改进本身在 React 内部是自动的，我们希望它们能够与大多数现有的 React 代码一起使用。 这证明了以声明方式表达加载状态的能力。 从 `if (isLoading)` 到 `<Suspense>` 看起来可能没什么大的变化，但正是它解锁了所有这些改进。
