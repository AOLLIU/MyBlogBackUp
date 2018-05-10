---
title: React组件化思想与Flux架构模式
date: 2016-8-1 23:56:25
tags:
    - 跨平台开发
---


# React 
React倾向于做传统MVC架构中的View

### 一,划分组件
 - React中划分组件的依据是单一责任原则,组件划分的越细,负责的事越少,维护越简单,逻辑越清晰

### 二.编写静态版本
 - React一个重要原则就是,让组件尽可能的是无状态的,大多数情况下,组件自身是没有状态的,只是从父组件接受一下属性,根据这些属性进行渲染,父组件自身维护了一些状态,通过props来传给子组件,而使UI发生变化.

 <!--more-->
 
### 三.添加状态 
- 状态需要包含那些仅与自身有关(不需要父组件决定),并且在组件的回调函数中会发生变化(指的是用户的行为导致变化),并且会体现在UI上的信息

### 四.通讯
- 我们不是把一些小的组件直接组合起来形成一个完整的App,而是将这些组件又组合到一个组合的集合,这样做就是希望这个组件集合体负责业务逻辑,整合信息,避免在小组件中UI渲染和业务逻辑的杂糅,提高的通用性,降低了后期业务逻辑发生变化带来的维护成本

**如何通讯**
- React完成父-子组件通讯通过props完成,子-父通讯通过绑定一个回调函数来完成,但是祖孙,兄弟,或者灭有明显关系的情况如何处理?
- Flux架构推荐我们使用事件订阅的方式来完成这样的通讯,具体完成方式见后面拓展


# Flux
- React相当于MVC中的V,Flux相当于添加M和C,他是一套架构模式
- 主要内容:(比喻有点儿不恰当,但是方便理解)
  - Dispatcher:处理动作分发,维护Store直接的依赖关系,相当于一个物流中心,主管分发货物
  - Store:数据和逻辑部分,相当于一个商店,卖东西
  - Views:React组件,可以看做是controller-view,作为试图同时响应事件,相当于购物者,买东西
  - Action:提供给dispatcher传递数据给store,可以看成是货物

## 核心概念:单向数据流
- Action ->Dispatcher->Store->View->Action
- 大致流程如下:
  - 首先由action,定义一些方法,用来提供给dispatcher,相当于有工场生产商品,提供给物流中心
  - 用户通过与view交互触发事件,相当于用户打电话说我要什么东西
  - Dispatcher会分发触发的Action给所有注册的Store的回调函数,相当于物流中心不知道用于具体在哪,只能把货物同时给了在他这里注册过的商店(不很恰当)
  - Store回调函数根据接受到的Action更新自身数据之后会触发一个change事件通知View数据改变了,相当于货物到达商店时,商店先把自己的货架更新了,然后打电话告诉用户来货了
  - View会监听这个change事件,拿到对应的新数据并调用setState更新组件UI,相当于用户一直在等这个电话,收到货到的电话时,就去商店拿到对应的货物,然后开开心心的拿着东西去打游戏了
- *所有的状态都是由Store维护,通过Action传递数据,构成了上述单向数据循环,所以在应用中各部分分工明确,高度解耦*  
  
### 一.Dispatcher
- **一个应用只需要一个 dispatcher 作为分发中心，管理所有数据流向，分发动作给 Store，没有太多其他的逻辑**
- **dispatcher 只是一个粘合剂，剩余的 Store、View、Action 就需要按具体需求去实现了。**

- Dispatcher分发动作给 Store 注册的回调函数，这和一般的订阅/发布模式不同的地方在于：
  - 1.回调函数不是订阅到某一个特定的事件/频道，每个动作会分发给所有注册的回调函数
  - 2.回调函数可以指定在其他回调之后调用

- Dispatcher 提供的 API :
  - 1.register(function callback): string 注册回调函数，返回一个 token 供在 waitFor() 使用
  - 2.unregister(string id): void 通过 token 移除回调
  - 3.waitFor(array ids): void 在指定的回调函数执行之后才执行当前回调。这个方法只能在分发动作的回调函数中使用
  - 4.dispatch(object payload): void 分发动作 payload 给所有注册回调
  - 5.isDispatching(): boolean 返回 Dispatcher 当前是否处在分发的状态

### 二.Action
- 首先要创建动作，通过定义一些 action creator 方法来创建，这些方法用来暴露给外部调用，通过 dispatch 分发对应的动作，所以 action creator 也称作 dispatcher helper methods 辅助 dipatcher 分发.

```
var AppDispatcher = require('../dispatcher/AppDispatcher');
var TodoConstants = require('../constants/TodoConstants');

var TodoActions = {

  /**
   * @param  {string} text
   */
  create: function(text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_CREATE,
      text: text
    });
  },

  /**
   * @param  {string} id The ID of the ToDo item
   * @param  {string} text
   */
  updateText: function(id, text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_UPDATE_TEXT,
      id: id,
      text: text
    });
  },

  /**
   * Toggle whether a single ToDo is complete
   * @param  {object} todo
   */
  toggleComplete: function(todo) {
    var id = todo.id;
    var actionType = todo.complete ?
        TodoConstants.TODO_UNDO_COMPLETE :
        TodoConstants.TODO_COMPLETE;

    AppDispatcher.dispatch({
      actionType: actionType,
      id: id
    });
  },

  /**
   * Mark all ToDos as complete
   */
  toggleCompleteAll: function() {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_TOGGLE_COMPLETE_ALL
    });
  },

  /**
   * @param  {string} id
   */
  destroy: function(id) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_DESTROY,
      id: id
    });
  },

  /**
   * Delete all the completed ToDos
   */
  destroyCompleted: function() {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_DESTROY_COMPLETED
    });
  }

};

module.exports = TodoActions;
```
AppDispatcher 直接继承自 Dispatcher.js，在这个例子中没有提供什么额外的功能。TodoConstants 定义了动作的类型名称常量。

类似 create、updateText 就是 action creator，这两个动作会通过 View 上的用户交互触发。除了用户交互会创建动作，服务端接口调用也可以用来创建动作，比如通过Ajax请求的一些初始数据也可以创建动作提供给 dispatcher，再分发给 store 使用这些初始数据。

可以看到所谓动作就是用来封装传递数据的，动作只是一个简单的对象，包含两部分：payload（数据）和 type（类型），type 是一个字符串常量，用来标识动作。

### 三.Store
- Stores 包含应用的状态和逻辑，不同的Store管理应用中不同部分的状态。

```
var AppDispatcher = require('../dispatcher/AppDispatcher');
var EventEmitter = require('events').EventEmitter;
var TodoConstants = require('../constants/TodoConstants');
var assign = require('object-assign');

var CHANGE_EVENT = 'change';

var _todos = {};

/**
 * Create a TODO item.
 * @param  {string} text The content of the TODO
 */
function create(text) {
  // Hand waving here -- not showing how this interacts with XHR or persistent
  // server-side storage.
  // Using the current timestamp + random number in place of a real id.
  var id = (+new Date() + Math.floor(Math.random() * 999999)).toString(36);
  _todos[id] = {
    id: id,
    complete: false,
    text: text
  };
}

/**
 * Update a TODO item.
 * @param  {string} id
 * @param {object} updates An object literal containing only the data to be
 *     updated.
 */
function update(id, updates) {
  _todos[id] = assign({}, _todos[id], updates);
}

/**
 * Update all of the TODO items with the same object.
 * @param  {object} updates An object literal containing only the data to be
 *     updated.
 */
function updateAll(updates) {
  for (var id in _todos) {
    update(id, updates);
  }
}

/**
 * Delete a TODO item.
 * @param  {string} id
 */
function destroy(id) {
  delete _todos[id];
}

/**
 * Delete all the completed TODO items.
 */
function destroyCompleted() {
  for (var id in _todos) {
    if (_todos[id].complete) {
      destroy(id);
    }
  }
}

var TodoStore = assign({}, EventEmitter.prototype, {

  /**
   * Tests whether all the remaining TODO items are marked as completed.
   * @return {boolean}
   */
  areAllComplete: function() {
    for (var id in _todos) {
      if (!_todos[id].complete) {
        return false;
      }
    }
    return true;
  },

  /**
   * Get the entire collection of TODOs.
   * @return {object}
   */
  getAll: function() {
    return _todos;
  },

  emitChange: function() {
    this.emit(CHANGE_EVENT);
  },

  /**
   * @param {function} callback
   */
  addChangeListener: function(callback) {
    this.on(CHANGE_EVENT, callback);
  },

  /**
   * @param {function} callback
   */
  removeChangeListener: function(callback) {
    this.removeListener(CHANGE_EVENT, callback);
  }
});

// Register callback to handle all updates
AppDispatcher.register(function(action) {
  var text;

  switch(action.actionType) {
    case TodoConstants.TODO_CREATE:
      text = action.text.trim();
      if (text !== '') {
        create(text);
        TodoStore.emitChange();
      }
      break;

    case TodoConstants.TODO_TOGGLE_COMPLETE_ALL:
      if (TodoStore.areAllComplete()) {
        updateAll({complete: false});
      } else {
        updateAll({complete: true});
      }
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_UNDO_COMPLETE:
      update(action.id, {complete: false});
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_COMPLETE:
      update(action.id, {complete: true});
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_UPDATE_TEXT:
      text = action.text.trim();
      if (text !== '') {
        update(action.id, {text: text});
        TodoStore.emitChange();
      }
      break;

    case TodoConstants.TODO_DESTROY:
      destroy(action.id);
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_DESTROY_COMPLETED:
      destroyCompleted();
      TodoStore.emitChange();
      break;

    default:
      // no op
  }
});

module.exports = TodoStore;
```
 在 Store 注册给 dispatcher 的回调函数中会接受到分发的 action，因为每个 action 都会分发给所有注册的回调，所以回调函数里面要判断这个 action 的 type 并调用相关的内部方法处理更新 action 带过来的数据（payload），再通知 view 数据变更。

   Store 里面不会暴露直接操作数据的方法给外部，暴露给外部调用的方法都是 Getter 方法，没有 Setter 方法，唯一更新数据的手段就是通过在 dispatcher 注册的回调函数。

### 四.View
**View就是React组件,从Store获取状态(数据),绑定change事件处理,**

```

var Footer = require('./Footer.react');
var Header = require('./Header.react');
var MainSection = require('./MainSection.react');
var React = require('react');
var TodoStore = require('../stores/TodoStore');

/**
 * Retrieve the current TODO data from the TodoStore
 */
function getTodoState() {
  return {
    allTodos: TodoStore.getAll(),
    areAllComplete: TodoStore.areAllComplete()
  };
}

var TodoApp = React.createClass({

  getInitialState: function() {
    return getTodoState();
  },

  componentDidMount: function() {
    TodoStore.addChangeListener(this._onChange);
  },

  componentWillUnmount: function() {
    TodoStore.removeChangeListener(this._onChange);
  },

  /**
   * @return {object}
   */
  render: function() {
    return (
      <div>
        <Header />
        <MainSection
          allTodos={this.state.allTodos}
          areAllComplete={this.state.areAllComplete}
        />
        <Footer allTodos={this.state.allTodos} />
      </div>
    );
  },

  /**
   * Event handler for 'change' events coming from the TodoStore
   */
  _onChange: function() {
    this.setState(getTodoState());
  }

});

module.exports = TodoApp;
```

一个 View 可能关联多个 Store 来管理不同部分的状态，得益于 React 更新 View 如此简单（setState），复杂的逻辑都被 Store 隔离了。


