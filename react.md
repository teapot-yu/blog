## react核心思想
简单来说，就是virtual dom & react diff。
我们都知道在前端开发中，js运行很快，dom操作很慢，而react充分利用了这个前提。在react中render的执行结果是树形结构的javascript对象，当数据(state || props)发生变化时，会生成一个新的树形结构的javascript对象，这两个javascript对象我们可以称之为virtual dom。然后对比两个virtual dom，找出最小的有变化的点，这个对比的过程我们称之为react diff，将这个变化的部分（patch）加入到一个队列中，最终批量更新这些patch到dom中。

## react执行render和setState进行渲染时主要有两个阶段

* 调度阶段（Reconciler）：React 会自顶向下通过递归, 用新数据生成一颗新树，遍历虚拟dom，diff新老virtual dom树，搜集具体的UI差异，找到需要更新的元素(Patch)，放到更新队列中。
* 渲染阶段（Renderer）：遍历更新队列，通过调用宿主环境的API（比如 DOM、Native、WebGL）实际更新渲染对应元素。

## 引入虚拟dom的好处是什么？
* js运行很快，dom操作很慢。配合react diff算法，通过对比virtual Dom，可以快速找出真实dom的最小变化，这样前端其实是不需要去关注那个变化的点，把这个变化交给react来做就好，同时你也不必自己去完成属性操作、事件处理、DOM更新，React会替你完成这一切，这让我们更关注我们的业务逻辑而非DOM操作，基于以上两点可大大提升我们的开发效率。
* 跨浏览器、跨平台兼容。react基于virtual dom自己实现了一套自己的事件机制，自己模拟了事件冒泡和捕获的过程，采用了事件代理，批量更新等方法，抹平了各个浏览器的事件兼容性问题。跨平台virtual dom为React带来了跨平台渲染的能力。以React Native为例子。React根据virtual dom画出相应平台的ui层，只不过不同平台画的姿势不同而已。


### react对性能的提升
关于提升性能，很多人说virtual dom可以提升性能，这一说法实际上是很片面的。因为我们知道，直接操作dom是非常耗费性能的，但是即使我们用了react，最终依然要去操作真实的dom。而react帮我们做的事情就是尽量用最佳的方式有操作dom。如果是首次渲染，virtual dom不具有任何优势，甚至它要进行更多的计算，消耗更多的内存。
react本身的优势在于react diff算法和批处理策略。react在页面更新之前，提前计算好了如何进行更新和渲染DOM,实际上，这个计算过程我们在直接操作DOM时，也是可以自己判断和实现的，但是一定会耗费非常多的精力和时间，而且往往我们自己做的是不如React好的。所以，在这个过程中React帮助我们"提升了性能"。
所以，我更倾向于说，virtual dom帮助我们提高了开发效率，在重复渲染时它帮助我们计算如何更高效的更新，而不是它比DOM操作更快。

## 什么是jsx？
我们在实现一个React组件时可以选择两种编码方式，第一种是使用JSX编写，第二种是直接使用React.createElement编写。实际上，上面两种写法是等价的，jsx只是为React.createElemen方法的语法糖，最终所有的jsx都会被babel转换成React.createElement。
但是请注意，babel在编译时会判断jsx中组件的首字母，当首字母为小写时，其被认定为原生dom标签，createElement的第一个变量被编译为字符串。当首字母为大写时，其被认定为自定义组件，createElement的第一个变量被编译为对象。

## react的生命周期是怎样的？
在react16中，废弃了三个will属性componentWillMount，componentWillReceiveProps，comonentWillUpdate，但是目前还未删除，react17计划会删除，同时通过UNSAFF_前缀向前兼容。
在 React 中，我们可以将其生命周期分为三个阶段。
#### 挂载阶段
* constructor()
  组件在挂载前，会调用它的构造函数，在构造函数内部必须执行一次super(props)，否则不能在constructor内部使用this，constructor通常用于给this.state初始化内部状态，为事件处理函数绑定this。
* static getDerivedStateFromProps(newProps,prevState)
  是一个静态方法，父组件传入的newProps和当前组件的prevState进行比较，判断时需要更新state，返回值用作更新state，如果不需要则返回null。在render()方法之前调用，并且在初始挂载和后续更新时调用。
* render()
  render()是组件中唯一必须实现的方法。需要返回以下类型，React元素、数组、fragments、Portals、字符串或者、值类型、布尔类型或null。同时render函数应该是纯函数。不能够调用setState。
* componentDidMount()

#### 更新阶段
* static getDerivedStateFromProps(props,state)
* shouldComponentUpate()
  当props或者state发生变化时，会在渲染前调用。根据父组件的props和当前的state进行对比，返回true/false。决定是否触发后续的 UNSAFE_componentWillUpdate()，render()和componentDidUpdate()。。
* render()
* getSnapshotBeforeUpdate(prevProps,prevSteate)
 在render()之后componentDidUpdate()之前调用。此方法的返回值(snaphot)可作为componentDidUpdate()的第三个参数使用。如不需要返回值则直接返回null。
* componentDidUpdate(prevProps, prevState, snapshot)
  该方法会在更新完成后立即调用。首次渲染不会执行此方法，当组件更新后，可以在此处对dom进行操作。可以在此阶段使用setState，触发render()但必须包裹在一个条件语句里，以避免死循环。

#### 卸载阶段
* componentWillUnmount()
  会在组件卸载和销毁之前直接调用。此方法主要用来执行一些清理工作，例如：定时器，清除事件绑定，取消网络请求。此阶段不能调用setState，因为组件永远不会重新渲染。

## react diff解决什么问题？是怎样的实现思路？
react diff会帮助我们计算出virtual dom中真正变化的部分，并只针对该部分进行实际dom操作，而非重新渲染整个页面，从而保证了每次操作更新后页面的高效渲染。传统diff算法通过循环递归对节点进行依次对比，效率低下，算法复杂度达到 O(n^3)。react diff基于一下三个策略实现了O(n)的算法复杂度。
* Web UI中dom节点跨层级的移动操作特别少，可以忽略不计。
* 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。
* 对于同一层级的一组子节点，它们可以通过唯一id进行区分。
基于以上三个前提策略，React分别对tree diff、component diff以及element diff 进行算法优化，事实也证明这三个前提策略是合理且准确的，它保证了整体界面构建的性能。

## react中key的作用
首先说一下element diff的过程。比如有老的集合（A，B，C，D）和新的集合（B，A，D，C），我们考虑在不增加空间复杂度的情况下如何以O（n）的时间复杂度找出老集合中需要移动的元素。
在react里的思路是这样的，遍历新集合，初始化lastIndex=0（代表访问过的老集合中最右侧的位置），表达式为max(prev.mountIndex, lastIndex),如果当前节点在老集合中的位置即(prev.mountIndex)比lastIndex大说明当前访问节点在老集合中就比上一个节点位置靠后则该节点不会影响其他节点的位置，因此不用添加到差异队列中，即不执行移动操作，只有当访问的节点比 lastIndex 小时，才需要进行移动操作。
部分源码为
```
var lastIndex = 0;
var nextIndex = 0;
for (name in nextChildren) {
    var prevChild = prevChildren && prevChildren[name]; // 老节点
    var nextChild = nextChildren[name]; // 新节点
    if (prevChild === nextChild) { // 如果新节点存在老节点集合里
      // 移动节点
      this.moveChild(prevChild, nextIndex, lastIndex);
      lastIndex = Math.max(prevChild._mountIndex, lastIndex);
      prevChild._mountIndex = nextIndex;
    } else {
      if (prevChild) { // 如果不存在在
        lastIndex = Math.max(prevChild._mountIndex, lastIndex);
        // 删除节点
        this._unmountChild(prevChild);
      }
      // 初始化并创建节点
      this._mountChildAtIndex(
        nextChild, nextIndex, transaction, context
      );
    }
    nextIndex++;
}

// 移动节点
moveChild: function(child, toIndex, lastIndex) {
  if (child._mountIndex < lastIndex) {
    this.prepareToManageChildren();
    enqueueMove(this, child._mountIndex, toIndex);
  }
}
```


## React 16有哪些新特性？

* render支持返回数组和字符串
* Error Boundaries
* createPortal
* rollup减小文件体积
* fiber
* Fragment
* createRef
* Strict Mode

## React Fiber是什么？解决什么问题？
React Fiber是React对核心算法的一次重新实现。
在协调阶段阶段，以前由于是采用的递归的遍历方式，这种也被称为Stack Reconciler，主要是为了区别Fiber Reconciler取的一个名字。这种方式有一个特点： 一旦任务开始进行，就无法中断，那么js将一直占用主线程，一直要等到整棵virtual dom树计算完成之后，才能把执行权交给渲染引擎，那么这就会导致一些用户交互、动画等任务无法立即得到处理，就会有卡顿，非常的影响用户体验。
页面是一帧一帧绘制出来的，当每秒绘制的帧数（FPS）达到60时，页面是流畅的，小于这个值时，用户会感觉到卡顿。1秒60帧，所以每一帧分到的时间是1000/60 ≈ 16ms。所以我们书写代码时力求不让一帧的工作量超过 16ms。如果任意一个步骤所占用的时间过长，超过16ms了之后，用户就能看到卡顿。
#### Fiber如何实现
简单来说就是时间分片 + 链表结构。而fiber就是维护每一个分片的数据结构。
Fiber利用分片的思想，把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，在每个小片执行完之后，就把控制权交还给React负责任务协调的模块，如果有紧急任务就去优先处理，如果没有就继续更新，这样就给其他任务一个执行的机会，唯一的线程就不会一直被独占。
因此，在组件更新时有可能一个更新任务还没有完成，就被另一个更高优先级的更新过程打断，优先级高的更新任务会优先处理完，而低优先级更新任务所做的工作则会完全作废，然后等待机会重头再来。所以 React Fiber把一个更新过程分为两个阶段：
* 第一个阶段 Reconciliation Phase，Fiber会找出需要更新的DOM，这个阶段是可以被打断的。
* 第二个阶段 Commit Phase，是无法别打断，完成dom的更新并展示。

## 什么是高阶组件
高阶组件（HOC）是React中用于复用组件逻辑的一种高级技巧。HOC自身不是React API的一部分，它是一种基于 React 的组合特性而形成的设计模式。具体而言，高阶组件是参数为组件，返回值为新组件的函数。
请注意，HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将组件包装在容器组件中来组成新组件。HOC 是纯函数，没有副作用。
我理解的高阶组件是，将组件以参数的方式传递给另外一个函数，在该函数中，对组件进行包装，封装了一些公用的组件逻辑，实现组件的逻辑复用，该函数被称为高阶组件。但是请注意，高阶组件不应修改传入的组件行为。

## 什么是渲染属性
术语 “render prop” 是指一种技术，用于使用一个值为函数的 prop 在 React 组件之间的代码共享。
带有渲染属性(Render Props)的组件需要一个返回 React 元素并调用它的函数，而不是实现自己的渲染逻辑。
我理解的渲染属性是，将组件以props的方式传递给另外一个组件，在后者中封装了复用逻辑，同时产出前者需要的参数，也就是让后者决定前者何时进行渲染。

## 什么是React Hooks，它是为了解决什么问题？说一下它的实现原理！
React Hooks 是 React 16.7.0-alpha 版本推出的新特性，它可以让你在不编写class的情况下使用state以及其他的 React特性。React Hooks要解决的问题是状态共享，是继render-props和hoc之后的第三种状态共享方案，不会产生JSX嵌套地狱问题。这个状态指的是状态逻辑，所以称为状态逻辑复用会更恰当，因为只共享数据处理逻辑，不会共享数据本身。
#### 简单实现
```
let memoizedState = []; // hooks 存放在这个数组
let cursor = 0; // 当前 memoizedState 下标

function useState(initialValue) {
  memoizedState[cursor] = memoizedState[cursor] || initialValue;
  const currentCursor = cursor;
  function setState(newState) {
    memoizedState[currentCursor] = newState;
    render();
  }
  return [memoizedState[cursor++], setState]; // 返回当前 state，并把 cursor 加 1
}

function useEffect(callback, depArray) {
  const hasNoDeps = !depArray;
  const deps = memoizedState[cursor];
  const hasChangedDeps = deps
    ? !depArray.every((el, i) => el === deps[i])
    : true;
  if (hasNoDeps || hasChangedDeps) {
    callback();
    memoizedState[cursor] = depArray;
  }
  cursor++;
}
```

## React为什么要在构造函数中调用super(props)，为什么要bind（this）？
super代表父类的构造函数，javascript规定如果子类不调用super是不允许在子类中使用this的，这不是React的限制，而是javaScript的限制，同时你也必须给super传入props，否则React.Component就没法初始化this.props
在 React 的类组件中，当我们把事件处理函数引用作为回调传递过去，事件处理程序方法会丢失其隐式绑定的上下文。当事件被触发并且处理程序被调用时，this的值会回退到默认绑定，即值为 undefined，这是因为类声明和原型方法是以严格模式运行。

## 说一下react事件机制？
#### react为什么要用自己的事件机制
* 减少内存消耗，提升性能，不需要注册那么多的事件了，一种事件类型只在document上注册一次。
* 统一规范，解决 ie 事件兼容问题，简化事件逻辑。
* 对开发者友好。
### react的合成事件
SyntheticEvent是react合成事件的基类，定义了合成事件的基础公共属性和方法。react会根据当前的事件类型来使用不同的合成事件对象，比如鼠标单机事件 - SyntheticMouseEvent，焦点事件-SyntheticFocusEvent等，但是都是继承自SyntheticEvent。在合成事件中主要做了以下三件事情。
* 对原生事件的封装
* 对某些原生事件的升级和改造
* 不同浏览器事件兼容的处理
#### 事件注册
组件挂载阶段，根据组件内的声明的事件类型-onclick，onchange等，给document上添加事件addEventListener，并指定统一的事件处理程序dispatchEvent。
通过virtual dom的props属性拿到要注册的事件名，回调函数，通过listenTo方法使用原生的addEventListener进行事件绑定。
#### 事件存储
事件存储，就是把react组件内的所有事件统一的存放到一个二级map对象里，缓存起来，为了在触发事件的时候可以查找到对应的方法去执行。先查找事件名，然后找对对应的组件id相对应的事件。如下图：
![8081b073fb2c06f047538b75cc97fc6f.png](evernotecid://AE784552-2477-4700-990B-17A5AAE0A09B/appyinxiangcom/22566670/ENResource/p29)

## setState是异步的？为什么要这么做？setState执行机制？

由执行机制看，setState本身并不是异步的，而是在调用setState时，如果react正处于更新过程，当前更新会被暂存，等上一次更新执行后再执行，这个过程给人一种异步的假象。
```
ReactComponent.prototype.setState = function(partialState, callback) {
  //  将setState事务放进队列中
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};
enqueueSetState: function (publicInstance, partialState) {
     // 获取当前组件的instance
    var internalInstance = getInternalInstanceReadyForUpdate(publicInstance, 'setState');

     // 将要更新的state放入一个数组里
     var queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

     //  将要更新的component instance也放在一个队列里
    enqueueUpdate(internalInstance);
}
function enqueueUpdate(component) {
  // 如果没有处于批量创建/更新组件的阶段，则处理update state事务
  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
  // 如果正处于批量创建/更新组件的过程，将当前的组件放在dirtyComponents数组中
  dirtyComponents.push(component);
}
```
这里的partialState可以传object,也可以传function,它会产生新的state以一种Object.assgine（）的方式跟旧的state进行合并。
由这段代码可以看到，当前如果正处于创建/更新组件的过程，就不会立刻去更新组件，而是先把当前的组件放在dirtyComponent里，所以不是每一次的setState都会更新组件。这段代码就解释了我们常听说的：setState是一个异步的过程，它会集齐一批需要更新的组件然后一起更新。而batchingStrategy 又是个什么东西呢?
ReactDefaultBatchingStrategy.js
```
var ReactDefaultBatchingStrategy = {
  // 用于标记当前是否出于批量更新
  isBatchingUpdates: false,
  // 当调用这个方法时，正式开始批量更新
  batchedUpdates: function (callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;

    ReactDefaultBatchingStrategy.isBatchingUpdates = true;

    // 如果当前事务正在更新过程在中，则调用callback，既enqueueUpdate
    if (alreadyBatchingUpdates) {
      return callback(a, b, c, d, e);
    } else {
    // 否则执行更新事务
      return transaction.perform(callback, null, a, b, c, d, e);
    }
  }
};
```
## react-router原理
前端路由的原理思路大致上都是相同的，即实现在无刷新页面的条件下切换显示不同的页面。而前端路由的本质就是页面的URL发生改变时，页面的显示结果可以根据URL的变化而变化，但是页面不会刷新。目前实现前端路由有两种方式：
#### 通过Hash实现前端路由
路径中hash值改变，并不会引起页面刷新，同时我们可以通过hashchange事件，监听hash的变化，从而实现我们根据不同的hash值展示和隐藏不同UI显示的功能，进而实现前端路由。
#### 通过H5的history实现前端路由
HTML5的History接口，History对象是一个底层接口，不继承于任何的接口。History接口允许我们操作浏览器会话历史记录。
而history的pushState和repalce方法可以实现改变当前页面显示的url，但都不会刷新页面。

未完待续~
参考文档：

[react生命周期详解](https://juejin.im/post/5ddcea315188256eaa0ebcf5)
[React diff](https://zhuanlan.zhihu.com/p/20346379)
[react 16新特性](https://zhuanlan.zhihu.com/p/52016989)
[react fiber1](https://www.cxymsg.com/guide/fiber.html#%E4%BB%8E-react-%E5%85%83%E7%B4%A0%E5%88%B0-fiber-%E8%8A%82%E7%82%B9)
[react fiber2](https://github.com/HuJiaoHJ/blog/issues/7#)
[react hooks](https://github.com/brickspert/blog/issues/26)
[react 事件机制](https://toutiao.io/posts/28of14w/preview)
[setState机制1](https://cloud.tencent.com/developer/article/1431167)
[setState机制2](https://imweb.io/topic/5b189d04d4c96b9b1b4c4ed6)
[react-router原理](https://segmentfault.com/a/1190000016435538)
[集合](https://juejin.im/post/5d5f44dae51d4561df7805b4#heading-10)

