# 『 JavaScript 』
## 数据类型
**数据类型**

+ **原始类型**：number，string，boolean，null，undefined，symbol，bigint
+ **引用类型**：object (包括 array，function，reg，math，date...)

**JS 中变量的存储方式**

原始类型存储在栈中，引用类型的内容存储在堆内存中，栈内存中存储指向对象的指针。

**类型检测**

+ **typeof**

    只能精准检测原始类型，不能准确的检测出引用类型的具体类型。

    注意： typeof null === 'object', typeof func === 'function', typeof 所能得到的值只有 'number', 'string', 'boolean', 'undefined', 'object', 'function'。
  
+ **instanceof**

    实质是通过检测原型链上原型对象来判断的。

    注意：instanceof 只能检测对象，且所有对象 instanceof Object 都会返回 true，且有原型链断裂的风险。

+ **constructor**

    和 instanceof 的原理类似。

    注意：有原型链断裂的风险。

+   **Object.prototype.toString.call()**

    最稳健的方法


## 作用域与闭包
**执行上下文(Execution Context)**

JS代码执行时，会存在以下三种执行环境：**全局环境**，**函数环境**，**eval 函数环境**。

浏览器首次加载 JS 代码时，默认情况下会进入全局执行上下文，若在全局环境中调用函数时，会创建一个新的函数执行上下文，并将它压入调用栈的栈顶。

如果在这个函数内部再调用一个函数，同样会再创建一个新的函数执行上下文并压入栈顶，浏览器将始终执行位于栈顶的执行上下文，且一旦函数执行完成，将会将该函数的执行上下文从栈中弹出。

每个函数调用都会创建一个新的执行上下文，甚至是对自身的调用。

在 JS 解释器内部，对执行上下文的每次调用都会有两个阶段：

1. **创建阶段**，在执行任何代码之前 

   在创建阶段，会做以下工作：**创建作用域链 -> 创建变量、函数、参数(变量对象) -> 确定 this 的值**

2. 代码**执行阶段**

   在执行阶段，会做以下工作：**为函数中的变量赋值并执行代码**

  可以在概念上对执行上下文表示为具有三个属性的对象：
  ```javascript
  execContextObject = {
      scopeChain: {
          // 变量对象 + 所有的父级执行上下文中的变量对象
      },
      variableObject: {
          // 参数，变量，内部函数声明
      },
      this: {}
  }
  ```
  以下是解释器如何执行代码的伪概述：

1. 找到调用函数的代码

2. 在执行功能代码之前，创建执行上下文

3.  进入创建阶段：

   ① 初始化范围链

   ② 创建变量对象：

   + 创建 arguments 对象，检查参数的上下文，初始化名称和值并创建引用副本上下文中的函数声明：
     + 对于找到的每个函数，在变量对象中创建一个属性，即确切的函数名称，该属性在内存中具有指向该函数的引用指针
     + 如果函数名称已经存在，则引用指针值将被覆盖

   + 扫描上下文中的变量声明：
      + 对于找到的每个变量声明，在变量对象中创建一个属性，即变量名称，并将该值初始化为 undefined
      + 如果变量名称已经存在于变量对象中，则什么也不做，然后继续扫描 (故同时声明同名的函数和变量，得到的是函数)

   ③ 确定上下文中 this 的值

4. 激活/代码执行阶段：

    在上下文中运行/解释功能代码，并在逐行执行代码时分配变量值。

**作用域**

作用域是指**程序源代码中定义变量的区域**。

作用域规定了如何查找变量，也就是**确定当前执行代码对变量的访问权限**。

JS 中的作用域是**词法作用域**，而不是动态作用域。在代码编写时就已经确定，作用域的**具体内容就是当前执行上下文的变量对象**。

每个函数都有一个与之对应的执行上下文，该执行上下文包含一个变量对象(VO)，每个执行上下文的作用域链属性是当前上下文的变量对象 + 所有父级执行上下文的变量对象的集合。

**闭包**

闭包就是**内部函数始终可以访问其外部函数的变量和参数**，即使在外部函数完成 return 后也是如此。

**何时使用闭包？**

+ 封装 -> 模块：公开公共接口，隐藏内部实现
+  回调
+ 作为参数

**闭包的缺点：**

+ 导致作用域链过长
+ 可能导致循环引用


## 原型与继承
**原型与原型链**

每个对象都有一个 **__proto__** 属性，指向它的原型对象。

每个函数都有一个 **prototype** 属性，指向它实例的原型对象。

instance.__proto__.__proto__... 一直到 **null**，这一系列的原型对象构成一条原型链。

**继承**

手写实现


## this 与 new
**this 关键字**

this 是 JavaScript 语言中的一个关键字，表示**当前执行代码的环境对象**，只存在于全局和函数中。

**this 的值**

1. 全局作用域下，this 指向全局对象，浏览器中为 window 对象

2. 独立函数调用时，this 指向全局对象 window

3. 对象方法调用时，this 指向当前对象

4. 构造函数中，this 指向 new 的对象实例

5. call，apply，bind中，this 指向传入的第一个参数对象

6. DOM 事件绑定，onClick 和 addEventListener 中 this 默认指向绑定事件的元素，而使用 attachEvent 中的 this 默认指向 window

7. 箭头函数中，this 与所在词法环境 this 保持一致。

**总结**：1 ~ 6 中，this 都是动态绑定的，7 中的 this 是静态词法确定的。

**new 原理**

JavaScript 调用 new 的过程：

1. 生成一个新的空对象

2. 将空对象链接到原型对象

3. 绑定 this

4. 返回新对象

注意：

​	如果构造函数的 **return 返回的是个对象**的话，new 的时候会返回该对象；

​	return 的是 undefined 或者原始类型的话就返回生成的对象实例。

手写模拟实现 new

手写实现 call apply bind


## Array String Object 常用 API
**Array**

Array.prototype.push()：将一个或多个元素添加到数组的末尾，并返回该数组的新长度。

Array.prototype.pop()：从数组中删除最后一个元素，并返回该元素的值。此方法更改数组的长度。

Array.prototype.unshift()：将一个或多个元素添加到数组的开头，并返回该数组的新长度。

Array.prototype.shift()：从数组中删除第一个元素，并返回该元素的值。此方法更改数组的长度。

Array.prototype.splice(start[, deleteCount[, item1[, item2[, ...]]]])：通过删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容。此方法会改变原数组

Array.prototype.slice([begin[, end]])：返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝（包括 begin，不包括end）。原始数组不会被改变。

Array.prototype.concat()：用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。

Array.prototype.reverse()：将数组中元素的位置颠倒，并返回该数组。数组的第一个元素会变成最后一个，数组的最后一个元素变成第一个。该方法会改变原数组。

Array.prototype.sort()：用原地算法对数组的元素进行排序，并返回数组。默认排序顺序是在将元素转换为字符串，然后比较它们的 UTF-16 代码单元值序列时构建的。

Array.prototype.flat([depth])：会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。

Array.prototype.includes()：用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false。

Array.prototype.indexOf()：返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。

Array.prototype.join()：将一个数组（或一个类数组对象）的所有元素连接成一个字符串并返回这个字符串。如果数组只有一个项目，那么将返回该项目而不使用分隔符。

Array.from()：从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。

Array.prototype.filter(cb, thisArg)：创建一个新数组, 其包含通过所提供函数实现的测试的所有元素，无则为 []。

Array.prototype.some(cb, thisArg)：测试数组中是不是至少有1个元素通过了被提供的函数测试。它返回的是一个Boolean类型的值。

Array.prototype.forEach(cb, thisArg)：对数组的每个元素执行一次给定的函数。

Array.prototype.map(cb, thisArg)：创建一个新数组，其结果是该数组中的每个元素是调用一次提供的函数后的返回值。

Array.prototype.reduce(cb, initialVal)：对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。

**String**

String.prototype.toString()：返回指定对象的字符串形式。

String.prototype.charAt(index)：从一个字符串中返回指定的字符。

String.prototype.concat(str, str1 ...):将一个或多个字符串与原字符串连接合并，形成一个新的字符串并返回。

String.prototype.match(regexp)：检索返回一个字符串匹配正则表达式的结果。

String.prototype.matchAll(regexp)：返回一个包含所有匹配正则表达式的结果及分组捕获组的迭代器。

String.prototype.replace()：返回一个由替换值（replacement）替换部分或所有的模式（pattern）匹配项后的新字符串。模式可以是一个字符串或者一个正则表达式，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。如果pattern是字符串，则仅替换第一个匹配项。原串不会改变。

String.prototype.substring()：返回一个字符串在开始索引到结束索引之间的一个子集, 或从开始索引直到字符串的末尾的一个子集。

String.prototype.trim()：会从一个字符串的两端删除空白字符。

**Object**

Object.assign()：将所有可枚举属性的值从一个或多个源对象分配到目标对象。它将返回目标对象。

Object.create(obj)：创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

Object.defineProperty(obj, prop, descriptor)：会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

Object.defineProperties(obj, props)：直接在一个对象上定义新的属性或修改现有属性，并返回该对象。

Object.entries(obj)：返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in 循环遍历该对象时返回的顺序一致。

Object.freeze(obj)：可以冻结一个对象。一个被冻结的对象再也不能被修改；即不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结一个对象后该对象的原型也不能被修改。freeze() 返回和传入的参数相同的对象。

Object.getOwnPropertyNames(obj)：返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。

Object.getPrototypeOf(obj)：返回指定对象的原型（内部[__proto__]属性的值。

Object.is(val1, val2)：判断两个值是否为同一个值。

Object.keys(obj)：会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致。

Object.prototype.isPrototypeOf(obj)：用于测试一个对象是否存在于另一个对象的原型链上。

Object.ptorotype.hasOwnProperty(props)：会返回一个布尔值，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键）。


## 事件循环(Event Loop)
**浏览器事件循环**

  1. 开始整段 script 脚本作为第一个宏任务执行
  2. 执行过程中同步代码直接执行，宏任务进入宏任务队列，微任务进入微任务队列
  3. 当前宏任务执行完出队，检查微任务队列，如果有则依次执行，直到微任务队列为空
  4. 执行浏览器 UI 线程的渲染工作
  5. 检查是否有Web worker任务，有则执行
  6. 执行队首新的宏任务，回到2，依此循环，直到宏任务和微任务队列都为空

**node 事件循环**

1. timer 阶段
2. I/O 异常回调阶段
3. 空闲、预备状态(第2阶段结束，poll 未触发之前)
4. poll 阶段
5. check 阶段
6. 关闭事件的回调阶段

**浏览器事件循环和 node 事件循环的区别**

两者最主要的区别在于浏览器中的微任务是在每个相应的宏任务中执行的，而nodejs中的微任务是在不同阶段之间执行的。

**宏任务和微任务**

+ 宏任务：script，setTimeout，setInterval，setImmediate，I/O，用户交互操作，UI渲染。
+ 微任务：Promise.then/Promise.reject，Mutation Observer，以 Promise 为基础开发的其他技术，V8 垃圾回收过程，Object.observe(已废弃)。

**process.nextTick**

  process.nextTick 是一个独立于 eventLoop 的任务队列。

  在每一个 eventLoop 阶段完成后会去检查这个队列，如果里面有任务，会让这部分任务优先于微任务执行。


## 异步解决方案
**callback**

缺点：callback 表达异步流程的方式是非线性的、非顺序的，这使得人脑正确推导这样的代码难度很大，容易形成"**回调地狱**"。

**Promise**

Promise 对象代表一个**异步操作**，有三种状态：

+ pending（进行中）
+ fulfilled（已成功）
+ rejected（已失败）

对象的状态不受外界影响，**一旦确定不会再更改**。

Promise 对象是一个**构造函数**，用来生成 Promise 实例。

Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve和 reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

  ```javascript
const promise = new Promise((resolve, reject) => {
    let option; // 表示异步操作

    option.result ? resolve(val) : reject(err); // 异步操作成功调用 resolve，失败调用 reject
})
  ```

  Promise.prototype.**then**(() => {})，它的作用是为 Promise 实例添加状态改变时的回调函数。

  Promise.prototype.**catch**(() => {})，捕获异步操作或代码运行中的错误。

  Promise.prototype.**finally**(() => {})，用于指定不管 Promise 对象最后状态如何，都会执行的操作。

  Promise.**resolve**()，将现有对象转换成为 Promise 对象。

  Promise.**resolve**()，也会返回一个新的 Promise 实例，该实例的状态为rejected。

  Promise.**all**([promise1, promise2...])，用于将多个 Promise 实例，包装成一个新的 Promise 实例。
```javascript
const p = Promise.all([p1, p2, p3]);
```
只有 p1、p2、p3 的状态都变成 fulfilled，p 的状态才会变成 fulfilled，此时 p1、p2、p3 的返回值组成一个数组，传递给 p 的回调函数。

只要 p1、p2、p3 之中有一个被 rejected，p 的状态就变成 rejected，此时第一个被 reject 的实例的返回值，会传递给 p 的回调函数。

Promise.**race**([promise1, promise2...])，同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
```javascript
const p = Promise.race([p1, p2, p3]);
```
只要 p1、p2、p3 之中有一个实例率先改变状态，p 的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。

Promise.**allSettled**([promise1, promise2...])，接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是 fulfilled 还是 rejected，包装实例才会结束。
```javascript
const p = Promise.allSettled([p1, p2, p3]);
```
该方法返回的新的 Promise 实例，一旦结束，状态总是 fulfilled，不会变成 rejected。

Promise.**any**([promise1, promise2...])，该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。

只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；

如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

```javascript
const p = Promise.any([p1, p2, p3]);
```

**co 库 + Generator 函数**

Generator 函数是一个普通函数，但是有两个特征：

+ function 关键字与函数名之间有一个星号
+ 函数体内部使用 yield 表达式，定义不同的内部状态

利用协程完成 Generator 函数，用 co 库让代码依次执行完，同时以同步的方式书写，也让异步操作按顺序执行。
  ```javascript
  co(function* () {
    const r1 = yield readFilePromise('1.json');
    const r2 = yield readFilePromise('2.json');
    const r3 = yield readFilePromise('3.json');
    const r4 = yield readFilePromise('4.json');
  })
  ```

**async/await**

async 函数是 Generator 函数的语法糖。

async 的函数都**默认返回一个 Promise 对象**，而更重要的是 async + await 也能让异步代码以同步的方式来书写。

await 是一个让出线程的标志：

+ await 标志着 js 会去先执行一遍后面紧跟的函数，然后马上让出线程，跳出整个 async，执行本轮执行周期里面的任务；

+ 等待本轮的宏任务（同步任务）执行完成之后，再回来 async 里等待之前 await 的函数的返回值，如果是返回值异步 promise，那么会把它塞入 promise.resolve() 微任务（异步任务执行栈）；

+ 等待它前面的异步任务执行完毕之后，再得到 await promise.resolve() 的值，然后才是去执行 await 后面的逻辑，如果 await 函数体返回的不是异步 promise，那么就直接去执行其后面的逻辑。


## DOM 事件机制
**事件流**

捕获阶段 -> 处于目标阶段 -> 冒泡阶段

在页面中点击一个元素，事件是从这个元素的祖先元素中逐层传递下来的，这个阶段为事件的捕获阶段。

当事件传递到这个元素之后，又会把事件逐成传递回去，直到根元素为止，这个阶段是事件的冒泡阶段。

我们为一个元素绑定一个点击事件的时候，可以指定是要在捕获阶段绑定或者换在冒泡阶段绑定。

当 addEventListener 的第三个参数为 true 的时候，代表是在捕获阶段绑定，当第三个参数为 false 或者为空的时候，代表在冒泡阶段绑定。

**事件委托**

事件委托：把一个元素响应事件（click、keydown......）的函数委托到另一个元素。

优点：减少内存消耗。


## 深浅拷贝
**浅拷贝**

只复制对象的一层，原始类型直接复制，引用类型复制其指针。

**深拷贝**

对目标的完全拷贝。

**浅拷贝实现方法**

+ Object.assign()
+ Array.prototype.concat() (浅拷贝数组)
+ Array.prototype.slice() (浅拷贝数组)
+ ... 展开运算符 (浅拷贝 iterator 对象)
+ 手写实现

**深拷贝实现方法**

+ JSON.parse(JSON.stringify())

  缺点：无法解决循环引用问题；无法拷贝特殊对象(RegExp、Date...)；无法拷贝函数； 

+ 手写实现


## 防抖与节流
**防抖**

在事件被触发 n 秒后再执行回调，如果在这 n 秒内又被触发，则重新计时，先计算时间后执行。

**节流**

n 秒内回调函数只会被执行一次，先执行后计算。


## 常见面试题

1. **如何判断两个变量相等？**

   可以通过比较 == === Object.is() 之间的区别来说明。

2. **null 和 undefined 之间的区别**

   null 表示"没有对象"，即该处不应该有值。

   undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义。
   
3. **如何解析对象属性？**

   JS 解释器尝试解析属性或标识符时，将首先使用作用域链来定位对象。

   找到对象后，再遍历该对象的原型链以查找属性名称。
   
4. **{}、new Object()、Object.create() 三者间的区别**

   字面量和 new 关键字创建的对象是 Object 的实例，原型指向 Object.prototype，继承内置对象 Object。

   Object.create(arg, pro) 创建的对象的原型取决于arg，arg为null，新对象是空对象，没有原型，不继承任何对象；arg为指定对象，新对象的原型指向指定对象，继承指定对象。
   
5. **event.target 和 event.currentTarget 的区别**

   + event.target 在事件流的目标阶段
   + event.currentTarget 在事件流的捕获，目标及冒泡阶段。

   只有当事件流处在目标阶段的时候，两者的指向才是一样的。

   而当处于捕获和冒泡阶段的时候，event.target 指向触发事件的标签元素，event.currentTarget 指向事件绑定的标签元素。