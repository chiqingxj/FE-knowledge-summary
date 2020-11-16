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
Mixin

Render Props

HOC

React Hooks


## 受控组件与非受控组件

在React中，所谓受控组件和非受控组件，是针对表单而言的。

+ 受控组件

    表单元素依赖于状态，表单元素需要默认值实时映射到状态的时候，就是受控组件。

    + 表单元素的修改会实时映射到状态值上，此时就可以对输入的内容进行校验。

    + 受控组件只有继承 React.Component 才会有状态。

    + 受控组件必须要在表单上使用 onChange 事件来绑定对应的事件。

    注意：

    + 在受控组件中，如果没有给输入框绑定 onChange 事件，将会收到 react 的警告。
    + 输入框除了默认值，是无法输入其他任何参数的。

+ 非受控组件

    非受控组件即不受状态的控制，获取数据就是相当于操作DOM。

    非受控组件的好处是很容易和第三方组件结合。

    React 中 可以通过 ref 来获取原生 DOM。

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




## 常见面试题
1. **为什么 React v16+ 要废除 componentWillMount，componentWillReceiveProps，componentWillUpdate 这三个生命周期？**

    因为 React 的 reconciler 架构由之前的 Stack reconciler 重构成为了现在的 Fiber reconciler。

    之前的 Stack reconciler 在处理 Virtual DOM 树的时候采用的是递归遍历且不中断的方法；而 Fiber reconciler 在处理 Fiber 树的时候可能会被系统的高优先级任务中断，导致重新遍历。

    React 15 中的 componentWillMount，componentWillReceiveProps，componentWillUpdate 以及 shouldComponentUpdate 都是运行在 commit 阶段的，更换成 Fiber 架构后可能会执行多次，导致结果不正确。

2. **setState 到底是同步还是异步？**

    setState 本身并不是一个异步方法。

    React 会将多个 setState 的调用合并为一个来执行，也就是说，当执行 setState 的时候，state 中的数据并不会马上更新，表现为一种异步更新的机制。

3. **为什么要绑定 this，箭头函数和 constructor 中 bind this 有何不同？**

    React 事件中的回调函数是直接调用的，没有指定调用的组件，所以需要绑定 this。(源码 dispatchEvent 调用了 invokeGuardedCallback)。

    bind this 每个组件实例只会调用一次。

    箭头函数每次调用都要重新生成一个函数，开销相较要大些。