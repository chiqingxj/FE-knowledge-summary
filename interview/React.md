## React
React 是用于构**建用户界面**的 JavaScript 库。

React 的理念是构建**快速响应**的大型 Web 应用程序。

## React 架构
**React15 架构**

+ **Reconciler**（协调器）—— 负责找出变化的组件

    每当有更新发生时，Reconciler 会做如下工作：

    + 调用函数组件、或 class 组件的 render 方法，将返回的 JSX 转化为虚拟 DOM

    + 将虚拟 DOM 和上次更新时的虚拟 DOM 对比

    + 通过对比找出本次更新中变化的虚拟 DOM

    + 通知 Renderer 将变化的虚拟 DOM 渲染到页面上

+ **Renderer**（渲染器）—— 负责将变化的组件渲染到页面上

    由于React支持跨平台，所以不同平台有不同的Renderer：

    + 负责在浏览器环境渲染的 Renderer —— ReactDOM。

    + ReactNative 渲染器，渲染 App 原生组件。
    + ReactTest 渲染器，渲染出纯Js对象用于测试。
    + ReactArt 渲染器，渲染到Canvas, SVG 或 VML (IE8)。

    在每次更新发生时，Renderer接到Reconciler通知，将变化的组件渲染在当前宿主环境。

    **React15 架构的缺点：**

    递归更新：由于递归执行，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16ms，用户交互就会卡顿。

    解决方案：用**可中断的异步更新**代替同步的更新。

**React16 架构**

+ **Scheduler**（调度器）—— 调度任务的优先级，高优任务优先进入Reconciler

  既然我们以浏览器是否有剩余时间作为任务中断的标准，那么我们需要一种机制，当浏览器有剩余时间时通知我们。

  部分浏览器已经实现了这个 API，这就是 requestIdleCallback (opens new window)。但是由于以下因素，React 放弃使用：

  + 浏览器兼容性
  + 触发频率不稳定，受很多因素影响。比如当我们的浏览器切换 tab 后，之前 tab 注册的requestIdleCallback 触发的频率会变得很低。

  基于以上原因，React 实现了功能更完备的 **requestIdleCallback polyfill**，这就是 Scheduler。除了在空闲时触发回调的功能外，Scheduler 还提供了多种调度优先级供任务设置。

  注意：Scheduler (opens new window) 是独立于 React 的库。

+ **Reconciler**（协调器）—— 负责找出变化的组件

  React16 的更新工作从递归变成了可以中断的循环过程。每次循环都会调用 **shouldYield** 判断当前是否有剩余时间。

  在React16中，Reconciler 与 Renderer 不再是交替工作。当 Scheduler 将任务交给 Reconciler 后，Reconciler 会为变化的虚拟 DOM 打上代表增/删/更新的标记。

  整个 Scheduler 与 Reconciler 的工作都在内存中进行。只有当所有组件都完成 Reconciler 的工作，才会统一交给 Renderer。

+ **Renderer**（渲染器）—— 负责将变化的组件渲染到页面上

  Renderer 根据 Reconciler 为虚拟 DOM 打上的标记，同步执行对应的 DOM 操作。


## JSX 语法
JSX，是一个 JavaScript 的语法扩展。在编译时会被 Babel 编译为 React.createElement 方法。

这也是为什么在每个使用JSX的JS文件中，你必须显式的声明：
```js
import React from 'react';
```

JSX 并不是只能被编译为 React.createElement 方法，你可以通过 @babel/plugin-transform-react-jsx 插件显式告诉 Babel 编译时需要将 JSX 编译为什么函数的调用（默认为 React.createElement）。

React.createElement
```js
export function createElement(type, config, children) {
  let propName;

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    // 将 config 处理后赋值给 props
    // ...
  }

  const childrenLength = arguments.length - 2;
  // 处理 children，会被赋值给props.children
  // ...

  // 处理 defaultProps
  // ...

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}

const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记这是个 React Element
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  };

  return element;
};
```

## React 生命周期
![16.4之后生命周期](https://upload-images.jianshu.io/upload_images/5287253-19b835e6e7802233.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

+ **挂载阶段**

    **constructor**

    构造函数最先执行，通常在构造函数中初始化 state 对象或者给自定义方法绑定 this

    **getDerivedStateFromProps**

    在 render 方法之前调用，并且在初始挂载和后续的每一次更新都会调用。它应该返回一个对象来更新 state，或者返回 null 表示不更新任何内容。

    注意：getDerivedStateFromProps 前面要加上 static 保留字，声明为静态方法，不然会被 react 忽略掉。

    **render**

    render 函数是纯函数，只返回需要渲染的东西，不应该包含其他的业务逻辑（render 里面不能执行 setState 等异步操作）。

    可以返回原生 DOM、React 组件、Fragment、Portals、字符串、数字、boolean、和 null 等内容。

    **componentDidMount**

    组件挂载到 DOM 后调用，且只会被调用一次。

    此时我们可以获取到 DOM 节点并操作，比如对 canvas，svg 的操作，服务器请求，订阅等副作用操作都可以写在里面。

+ **更新阶段**

    **getDerivedStateFromProps**

    功能同上

    **ShouldComponentUpdate(nextProps, nextState)**

    nextProps 和 nextState 表示新的属性和变化之后的 state。

    通过比较 nextProps，nextState 及当前组件的 this.props，this.state，返回 true 表示触发重新渲染，返回 false 表示不触发重新渲染，默认返回 true，以此可用来减少组件的不必要渲染，优化组件性能。

    **render**

    功能同上

    **getSnapshotBeforeUpdate(prevProps, prevState)**

    getSnapshotBeforeUpdate 调用于 render 之后，最近一次渲染输出(提交到 DOM 节点)之前，可以读取但无法使用 DOM 的时候。它使组件可以在可能更改之前从 DOM 捕获一些信息（例如滚动位置）。

    此生命周期返回的任何值都将作为参数传递给 componentDidUpdate()。

    **componentDidUpdate(prevProps, prevState)**

    此方法在组件更新后立即被调用，可以操作组件更新的 DOM，prevProps 和 prevState 这两个参数指的是组件更新前的 props 和 state。 

+ **卸载阶段**

    **componentWillUnmount**

    此方法在组件被卸载前或销毁时调用，可以在这里执行一些清理工作，比如清楚组件中使用的定时器，取消网络操作，清除 componentDidMount 中手动创建的 DOM 元素等，以避免引起内存泄漏。


## React 组件通信机制
+ props 父子、兄弟组件通信

+ Context 跨层级通信

+ Redux 全局状态管理


## React 状态逻辑复用
**Mixin**

​    不了解

**Render Props**



**HOC**



**React Hooks**




## 受控组件与非受控组件

在React中，所谓受控组件和非受控组件，是针对表单而言的。

+ 受控组件

    表单元素依赖于 state，表单元素需要默认值实时映射到 state 的时候，就是受控组件。

    + 表单元素的修改会实时映射到状态值上，此时就可以对输入的内容进行校验。
    + 受控组件只有继承 React.Component 才会有状态。
    + 受控组件必须要在表单上使用 onChange 事件来绑定对应的事件。

    注意：

    + 在受控组件中，如果没有给输入框绑定 onChange 事件，将会收到 react 的警告。
    + 输入框除了默认值，是无法输入其他任何参数的。

+ 非受控组件

    非受控组件即不受状态的控制，获取数据就是相当于操作 DOM。

    非受控组件的好处是很容易和第三方组件结合。

    React 中可以通过 ref 来获取原生 DOM。

## React 事件机制

**合成事件**
主要包括了一下三个方面：

1. 对原生事件的封装

   SyntheticEvent是react合成事件的基类，定义了合成事件的基础公共属性和方法。

   react会根据当前的事件类型来使用不同的合成事件对象，比如鼠标单机事件 - SyntheticMouseEvent，焦点事件-SyntheticFocusEvent等，但是都是继承自SyntheticEvent。

2. 对某些原生事件的升级和改造

  对于有些dom元素事件，我们进行事件绑定之后，react并不是只处理你声明的事件类型，还会额外的增加一些其他的事件，帮助我们提升交互的体验。


3. 对不同浏览器事件兼容的处理

**事件注册机制**

react 事件注册过程其实主要做了两件事：事件注册、事件存储。

+ 事件注册

  组件挂载阶段，根据组件内的声明的事件类型 onclick，onchange 等，给 document 上添加事件 addEventListener，并指定统一的事件处理程序 dispatchEvent。

+ 事件存储

  把 react 组件内的所有事件统一的存放到一个对象里，缓存起来，为了在触发事件的时候可以查找到对应的方法去执行。

**事件执行机制**

在事件注册阶段，最终所有的事件和事件类型都会保存到 listenerBank 中。

事件触发过程总结为主要下面几个步骤：

1. 进入统一的事件分发函数(dispatchEvent)

2. 结合原生事件找到当前节点对应的ReactDOMComponent对象

3. 开始 事件的合成

      3.1 根据当前事件类型生成指定的合成对象

   ​	3.2 封装原生事件和冒泡机制

   ​	3.3 查找当前元素以及他所有父级

   ​	3.4 在 listenerBank查找事件回调并合成到 event(合成事件结束)

4. 批量处理合成事件内的回调事件（事件触发完成 end）


## React Fiber 架构

**Fiber 的含义**

+ 作为架构来说，之前 React15 的 Reconciler 采用递归的方式执行，数据保存在递归调用栈中，所以被称为 stack Reconciler。React16的 Reconciler 基于 Fiber 节点实现，被称为 Fiber Reconciler。
+ 作为静态的数据结构来说，每个 Fiber 节点对应一个 React element，保存了该组件的类型（函数组件/类组件/原生组件...）对应的 DOM 节点等信息。
+ 作为动态的工作单元来说，每个 Fiber 节点保存了本次更新中该组件改变的状态、要执行的工作（需要被删除/被插入页面中/被更新...）。

**Fiber 的结构**
```javascript
function FiberNode(
    tag: WorkTag,
    pendingProps: mixed,
    key: null | string,
    mode: TypeOfMode,
) {
    // 作为静态数据结构的属性
    this.tag = tag; // Fiber 对应组件的类型 Function/Class/Host...
    this.key = key; // key 属性
    this.elementType = null; // 大部分情况同 type，某些情况不同，比如 FunctionComponent 使用 React.memo 包裹
    this.type = null; // 对于 FunctionComponent，指函数本身，对于 ClassComponent，指 class，对于 HostComponent，指 DOM 节点 tagName
    this.stateNode = null; // Fiber 对应的真实 DOM 节点

    // 用于连接其他 Fiber 节点形成 Fiber 树
    this.return = null; // 指向父级 Fiber 节点
    this.child = null; // 指向子 Fiber 节点
    this.sibling = null; // 指向右边第一个兄弟 Fiber 节点
    this.index = 0;

    this.ref = null;

    // 作为动态的工作单元的属性
    this.pendingProps = pendingProps;
    this.memoizedProps = null;
    this.updateQueue = null;
    this.memoizedState = null;
    this.dependencies = null;

    this.mode = mode;

    // 保存本次更新会造成的DOM操作
    this.effectTag = NoEffect; // 要执行DOM操作的具体类型
    this.nextEffect = null;

    this.firstEffect = null;
    this.lastEffect = null;

    // 调度优先级相关
    this.lanes = NoLanes;
    this.childLanes = NoLanes;

    // 指向该fiber在另一次更新时对应的fiber
    this.alternate = null;
}
```

**Fiber 架构工作原理**

Fiber 节点构成的 Fiber 树就对应 DOM 树。

那么如何更新DOM呢？这需要用到被称为“双缓存”的技术。

在内存中构建并直接替换的技术叫做**双缓存**。

比如 当我们用canvas绘制动画，每一帧绘制前都会调用ctx.clearRect清除上一帧的画面。

如果当前帧画面计算量比较大，导致清除上一帧画面到绘制当前帧画面之间有较长间隙，就会出现白屏。

为了解决这个问题，我们可以在内存中绘制当前帧动画，绘制完毕后直接用当前帧替换上一帧画面，由于省去了两帧替换间的计算时间，不会出现从白屏到出现画面的闪烁情况。

React 使用“双缓存”来完成 Fiber 树的构建与替换——对应着 DOM 树的创建与更新。

在 React 中最多会同时存在两棵 Fiber 树。

+ 当前屏幕上显示内容对应的 Fiber 树称为 current Fiber 树。current Fiber 树中的 Fiber 节点被称为 **current fiber**。
+ 正在内存中构建的 Fiber 树称为 **workInProgress Fiber** 树。workInProgress Fiber 树中的 Fiber 节点被称为**workInProgress fiber**。

urrent fiber 和 workInProgress fiber 通过 **alternate** 属性连接。

```javascript
currentFiber.alternate === workInProgressFiber;
workInProgressFiber.alternate === currentFiber;
```


React 应用的根节点通过 **current** 指针在不同 Fiber 树的 **rootFiber** 间**切换**来实现 Fiber 树的切换。

当 workInProgress Fiber 树构建完成交给 Renderer 渲染在页面上后，应用根节点的 current 指针指向workInProgress Fiber 树，此时 workInProgress Fiber 树就变为 current Fiber 树。

每次状态更新都会产生新的 workInProgress Fiber 树，通过 current 与 workInProgress 的替换，完成 DOM 更新。

具体 mount 和 update 时如何更新，参考这篇[文章](https://react.iamkasong.com/process/doubleBuffer.html)。

**render 阶段**

+ **beginWork**

    beginWork 的工作是传入当前 Fiber 节点，创建子 Fiber 节点。
    ![beginWork](https://react.iamkasong.com/img/beginWork.png)

+ **completeWork**

    completeWork 的工作是构建 DOM 树并执行 DOM 节点上的相关操作。
    ![completeWork](https://react.iamkasong.com/img/completeWork.png)

**commit 阶段**

+ **before mutation 阶段**
+ **mutation 阶段**
+ **layout 阶段**

commit 阶段的具体过程参考这篇[文章](https://react.iamkasong.com/renderer/prepare.html)。


## Diff 算法
在 render 阶段的 beginWork 中，对于 update 的组件，它会**将当前组件与该组件在上次更新时对应的 Fiber 节点比较**（也就是俗称的 Diff 算法），**将比较的结果生成新 Fiber 节点**。

**一个 DOM 节点在某一时刻最多会有 4 个节点与它相关**：

+ **current Fiber**：如果该DOM节点已在页面中，current Fiber代表该DOM节点对应的Fiber节点。

+ **workInProgress Fiber**：如果该DOM节点将在本次更新中渲染到页面中，workInProgress Fiber代表该DOM节点对应的Fiber节点。

+ **DOM 节点本身**

+ **JSX 对象**：即 ClassComponent 的 render 方法的返回结果，或 FunctionComponent 的调用结果。JSX 对象中包含描述 DOM 节点的信息。

**Diff 算法的本质**是对比 current Fiber 和 JSX 对象，生成 workInProgress Fiber。

**React 为了降低算法复杂度，将 diff 预设三个限制：**

+ 只对同级元素进行 Diff。如果一个 DOM 节点在前后两次更新中跨越了层级，那么 React 不会尝试复用它。

+ 两个不同类型的元素会产生出不同的树。如果元素由 div 变为 p，React 会销毁 div 及其子孙节点，并新建 p 及其子孙节点。
+ 开发者可以通过 key 属性来暗示哪些子元素在不同的渲染下能保持稳定。

**Diff 的实现过程**：

Diff 的入口函数 reconcileChildFibers 出发，该函数会根据 newChild（即JSX对象）类型调用不同的处理函数。

从同级的节点数量将 Diff 分为两类：

+ **单点 Diff**：当 newChild 类型为 object、number、string，代表同级只有一个节点。

    ![单点diff](https://react.iamkasong.com/img/diff.png)

    判断DOM节点是否可以复用是如何实现的

    React通过先判断key是否相同，如果key相同则判断type是否相同，只有都相同时一个DOM节点才能复用。

    这里有个细节需要关注下：

        当child !== null且key相同且type不同时执行deleteRemainingChildren将child及其兄弟fiber都标记删除。
        
        当child !== null且key不同时仅将child标记删除。

+ **多点 Diff**：当 newChild 类型为 Array，同级有多个节点。

    整体逻辑会经历**两轮**遍历：

    **第一轮遍历**：处理**更新**的节点(日常开发中，更新的频率是最高的)

    1. let i = 0，遍历 newChildren，将 newChildren[i] 与 oldFiber 比较，判断 DOM 节点是否可复用。

    2. 如果可复用，i++，继续比较 newChildren[i] 与 oldFiber.sibling，可以复用则继续遍历。

    3. 如果不可复用，分两种情况：
       + key 不同导致不可复用，立即跳出整个遍历，第一轮遍历结束。
       + key 相同 type 不同导致不可复用，会将 oldFiber 标记为 DELETION，并继续遍历。

    4. 如果 newChildren 遍历完（即i === newChildren.length - 1）或者 oldFiber 遍历完（即oldFiber.sibling === null），跳出遍历，第一轮遍历结束。

    **第二轮遍历**：处理剩下的**不属于更新**的节点

    对于第一轮遍历的结果，我们分别讨论：

    + **newChildren 与 oldFiber 同时遍历完**

      这是最理想的情况：只需在第一轮遍历进行组件更新。此时 Diff 结束。

    + **newChildren 没遍历完，oldFiber 遍历完**

      已有的 DOM节点 都复用了，意味着本次更新有新节点插入，我们只需要遍历剩下的 newChildren 为生成的 workInProgress fiber 依次标记 Placement。

    + **newChildren 遍历完，oldFiber 没遍历完**

      意味有节点被删除了，所以需要遍历剩下的 oldFiber，依次标记 Deletion。

    + **newChildren 与 oldFiber 都没遍历完**

        这就比较麻烦了，意味着有节点在这次更新中改变了位置。
        
        如何才能将同一个节点在**两次更新中对应**上呢？

        + 为了快速的找到 key 对应的 oldFiber，我们将所有还未处理的 oldFiber 存入以 key 为 key，oldFiber 为 value 的 Map 中。
        + 接下来遍历剩余的 newChildren，通过 newChildren[i].key 就能在 existingChildren 中找到 key 相同的 oldFiber。

        节点**是否移动**是以什么为**参照物**？

        以最后一个可复用的节点在 oldFiber 中的位置索引（用变量lastPlacedIndex表示）作为参照物。

        由于本次更新中节点是按 newChildren 的顺序排列。在遍历 newChildren 过程中，每个遍历到的可复用节点一定是当前遍历到的所有可复用节点中最靠右的那个，即一定在 lastPlacedIndex 对应的可复用的节点在本次更新中位置的后面。
        
        那么我们只需要比较遍历到的可复用节点在上次更新时是否也在 lastPlacedIndex 对应的 oldFiber 后面，就能知道两次更新中这两个节点的相对位置改变没有。
        
        我们用变量 oldIndex 表示遍历到的可复用节点在 oldFiber 中的位置索引。如果 oldIndex < lastPlacedIndex，代表本次更新该节点需要向右移动。
        
        lastPlacedIndex 初始为 0，每遍历一个可复用的节点，如果 oldIndex >= lastPlacedIndex，则lastPlacedIndex = oldIndex。


## React Hooks


## Redux
Redux 是 JavaScript 状态容器，提供**可预测化**的**状态管理**。

**三大原则**：

+ **单一数据源**
  
    整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。
    
+ **State 是只读的**

    唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。
    
+ **使用纯函数来执行修改**

    为了描述 action 如何改变 state tree ，你需要编写 reducers，reducers 是真正操作修改 state 的地方。

**Store**

Store 是将 action 和 reducer 联系到一起的对象。Redux 应用只有一个**单一**的 store。

Store 的**职责**：

+ 维持应用的 state
+ 提供 getState() 方法获取 state
+ 提供 dispatch(action) 方法更新 state
+ 通过 subscribe(listener) 注册监听器
+ 通过 subscribe(listener) 返回的函数注销监听器

**Action**

Action 是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。

一般来说通过 store.dispatch() 将 action 传到 store。

Action 本质上是 JavaScript 普通对象。我们约定，action 内必须使用一个字符串类型的 **type** 字段来表示将要执行的动作。

我们应该尽量减少在 action 中传递的数据。

bindActionCreators() 可以自动把多个 action 创建函数 绑定到 dispatch() 方法上。**Action 创建函数** 就是生成 action 的方法。

**异步 Action**

使用中间件（redux-thunk/redux-saga...）将同步的 action 创建函数与异步操作结合起来。

通过使用指定的 middleware，action 创建函数除了返回 action 对象外还可以返回函数。

**Reducer**

Reducer 指定了应用状态的变化是如何响应 actions 并发送到 store 的。

combineReducers() 所做的只是生成一个函数，这个函数来调用你的一系列 reducer，每个 reducer **根据它们的 key 来筛选出 state 中的一部分数据并处理**，然后这个生成的函数再将所有 reducer 的结果合并成一个大的对象。

**Redux 数据流**：

1. 调用 store.dispatch(action)

    可以在任何地方调用 store.dispatch(action)，包括组件中、XHR 回调中、甚至定时器中。

2. Redux store 调用传入的 reducer 对象

    Store 会把两个参数传入 reducer： 当前的 state 树和 action。

3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树

    Redux 原生提供 combineReducers() 辅助函数，来把根 reducer 拆分成多个函数，用于分别处理 state 树的一个分支。

4. Redux store 保存了根 reducer 返回的完整 state 树

    所有订阅 store.subscribe(listener) 的监听器都将被调用；监听器里可以调用 store.getState() 获得当前 state。

## React-Redux
Redux 官方提供的 React 绑定库，具有高效且灵活的特性。可以让 React 组件从 Redux store 中读取数据，并向 store 中 dispatch action 以更新数据。

**Provider**

React Redux 使用 React 的 **Context** 功能使 Redux store 可被深度嵌套的组件访问。 

从React Redux v6开始，通常由 React.createContext() 生成的单个默认 context 对象实例（称为ReactReduxContext）处理。

React Redux的 <Provider> 组件使用 <ReactReduxContext.Provider> 将 Redux store 和 current store state 保存在 context 中，而 connect 使用 <ReactReduxContext.Consumer> 读取这些值并处理更新。

**connect()**

```js
function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
```

connect() 为组件提供了 store 中所需的数据片段，以及可用于向 store dispatch action 的功能。它不会修改传递给它的组件，而是返回一个**新**的，已连接 store 的组件类，该类包装传入的组件。接受两个参数：

  + **mapStateToProps(state, ownProps?)**

    mapStateToProps 用于从 store 中选取组件所需的数据并以对象的形式返回。

    特点：

    + 每当 store 中 state 发生更改时都会调用 mapStateToProps（如果传入了 ownProps，props 发生改变也会调用 mapStateToProps）
    + mapStateToProps 接收整个 store 中的 state，并返回组件需要的数据对象。

    mapStateToProps 应该定义为函数形式：
    ```js
    function mapStateToProps(state, ownProps?)
    ```

  + **mapDispatchToProps(dispatch, ownProps?)**

    mapDispatchToProps 用于将 dispatch action 到 store。

    React Redux 提供了两种让组件 dispatch action 的方式：

    + 默认情况下，connect 生成的组件会接收 props.dispatch 并可以自己 dispatch action。
    + connect 可以接受 mapDispatchToProps，它可让组件在调用 dispatch 时创建函数，并将这些函数作为 props 传递给组件。

    mapDispatchToProps 可以采用函数和对象两种形式：

    + 如果是函数，在创建组件时将调用它一次。它将 dispatch 作为参数接收，并且应返回一个对象，该对象由 dispatch 用于 dispatch action 的函数组成。函数允许更多的自定义，获得对 dispatch 的访问权，并可以选择是否传入 ownProps
    + 如果是一个由 action 创建函数组成的对象，则每个 action 创建函数都将变成一个 prop 函数，该函数在被调用时会自动 dispatch action。此时组件不会接收 dispatch 作为 props。

  + **mergeProps(stateProps, dispatchProps, ownProps))**

    mergeProps 定义如何确定组件的最终的 props，返回 props 对象，默认为 {... ownProps，... stateProps，... dispatchProps}。

  + **options?: Object**

    ```js
    {
        context?: Object,
        pure?: boolean,
        areStatesEqual?: Function,
        areOwnPropsEqual?: Function,
        areStatePropsEqual?: Function,
        areMergedPropsEqual?: Function,
        forwardRef?: boolean,
    }
    ```

**useStore**
```js
const store = useStore()
```
useStore Hook 返回对传递给 <Provider> 组件的同一 Redux store 的引用。

**useSelector**
```js
const result : any = useSelector(selector : Function, equalityFn? : Function)
```
useSelector 的功能是**从 store 中获取数据**。在组件渲染和 dispatch action 时会被调用。

选择器可以返回任何结果，而不仅仅是对象。 选择器的返回值将用作useSelector（）钩子的返回值。

dispatch action 时，useSelector() 将对前一个 selector result 和 current result 进行浅比较。如果它们不相等，则将强制重新渲染组件。如果它们相同，则组件将不会重新渲染。

useSelector（）默认情况下使用严格的 === 引用相等检查，而不是浅相等。

**useDispatch**
```js
const dispatch = useDispatch()
```
useDispatch 返回 Redux store 中 dispatch 函数的引用。


## React-Router
<Router>

所有 Router 组件的通用底层接口。

<BrowserRouter>

使用 HTML5 history API（pushState，replaceState 和 popstate 事件）的 <Router>，使 UI 与 URL 保持同步。

```jsx
<BrowserRouter
  basename={optionalString}
  forceRefresh={optionalBool}
  getUserConfirmation={optionalFunc}
  keyLength={optionalNumber}
  children={...}
>
  <App />
</BrowserRouter>
```

<HashRouter>

<Router> 使用 URL 的哈希部分（即 window.location.hash）使您的 UI 与 URL 保持同步。

```jsx
<HashRouter
  basename={optionalString}
  getUserConfirmation={optionalFunc}
  hashType={optionalString}
  children={...}
>
  <App />
</HashRouter>
```

<Link>

<Link> 提供声明式，可访问的导航。

```jsx
<Link 
  to={string | object | function}
  replace={boolean}
  innerRef={function}
  component={React.Component}
  other={...}
>
  About
</Link>
```

<NavLink>

<Link> 的特殊版本，当它与当前 URL 匹配时，它将为呈现的元素添加样式属性。

```jsx
<Link 
  to={string | object | function}
  replace={boolean}
  innerRef={function}
  component={React.Component}
  activeClassName={string}
  activeStyle={object}
  exact={boolean}
  strict={boolean}
  isActive={function}
  location={object}
  aria-current={string}
  other={...}
>
  About
</Link>
```

<Redirect>

<Redirect> 将导航到新位置。新位置将覆盖历史记录堆栈中的当前位置，就像服务端重定向（HTTP 3xx）一样。

```jsx
<Redirect
  to={string | object}
  from={string}
  push={boolean}
  exact={boolean}
  strict={boolean}
  sensitive={bool}
>
  About
</Redirect>
```

<Route>

<Route> 负责当其路径与 URL 匹配时，呈现相应的 UI。

```jsx
<Route
  path={string | string[]}
  render
  exact={boolean}
  strict={boolean}
  location={object}
  sensitive={bool}
>
  About
</Route>
```

Route render 方法：

+ <Route component>
+ <Route render>
+ <Route children> function

Route props：

+ match
+ location
+ history

<Switch>

渲染第一个与 URL 匹配的 <Route>或<Redirect>

```jsx
<Switch
  location={object}
  children={node}
>
  <Route>
  ...
</Switch>
```

**history / useHistory**
useHistory 返回用于导航的历史记录实例。

**location / useLocation**
useLocation 返回代表当前 URL 的位置对象。

**match** 
**matchPath**

**useParams**

useParams 返回 UR L参数的键/值对的对象。使用它来访问当前 <Route> 的 match.params。

**useRouteMatch**

useRouteMatch 尝试以与<Route>相同的方式匹配当前 URL。在无需实际呈现<Route>的情况下访问匹配数据最有用。

**withRouter**

通过 withRouter 高阶组件访问历史对象的属性和最接近的<Route>匹配项。每次当渲染时，withRouter 都会将更新的 match, location, history props 传递给包装的组件。

## 常见面试题
1. **为什么 React16 要废除 componentWillMount，componentWillReceiveProps，componentWillUpdate 这三个生命周期？**

    因为 React 的 reconciler 架构由之前的 Stack reconciler 重构成为了现在的 Fiber reconciler。

    之前的 Stack reconciler 在处理 Virtual DOM 树的时候采用的是递归遍历且不中断的方法；而 Fiber reconciler 在处理 Fiber 树的时候可能会被系统的高优先级任务中断，导致重新遍历。

    React 15 中的 componentWillMount，componentWillReceiveProps，componentWillUpdate 以及 shouldComponentUpdate 都是运行在 commit 阶段的，更换成 Fiber 架构后可能会执行多次，导致结果不正确。

2. **setState 到底是同步还是异步？**

    setState 本身并不是一个异步方法。

    React 会将多个 setState 的调用合并为一个来执行，也就是说，当执行 setState 的时候，state 中的数据并不会马上更新，因此表现为一种异步更新的机制。

    另一个说法：setState 只在合成事件和钩子函数中表现为“异步”，因为合成事件和钩子函数的调用顺序在 state 更新之前，无法立即拿到更新后的值，形成了所谓的异步。

    如果新的 state 要依赖之前的 state，使用函数的方式改变state（setState 的第一个参数）。

    如何获取同步的获取更新之后的数据呢？

    + 回调函数

      ```
      this.setState({number:3}, ()=>{
      	console.log(this.state.number)
      })
      ```

    + setTimeout / setInterval

      ```
      state = {
          number: 1
      };
      componentDidMount(){
          setTimeout(()=>{
            this.setState({number: 3})；
            console.log(this.state.number)；
          },0)；
      }
      ```

    + 原生事件中修改状态

      ```
      state = {
          number: 1
      };
      componentDidMount() {
          document.body.addEventListener('click', this.changeVal, false);
      }
      changeVal = () => {
          this.setState({
            number: 3
          })；
          console.log(this.state.number)；
      }
      ```

3. **为什么要绑定 this，箭头函数和 constructor 中 bind this 有何不同？**

    React 事件中的回调函数是直接调用的，没有指定调用的组件，所以需要绑定 this。(源码 dispatchEvent 调用了 invokeGuardedCallback)。

    bind this 每个组件实例只会调用一次。

    箭头函数每次调用都要重新生成一个函数，开销相较要大些。

4. **React Hooks 闭包陷阱**

5. **为什么要求 key 唯一？**

6. **React 性能优化**

7. **Redux 性能优化**

    + Reselect
    + Immutable
    + 减少 store 更新事件的数量