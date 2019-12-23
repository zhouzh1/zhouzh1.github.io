---
title: redux源代码阅读
tags: [JavaScript Redux]
category: FrontEnd
filename: read source code of redux
createdTime: 2019-12-20 13:59
---
Redux是一个常用的组件状态管理库，通过将应用的状态数据集中存储并且限制可以更改状态数据的路径，使得开发者可以更好的追踪以及预测应用状态的变化过程，Redux的源代码不到1000行，是一个学习源代码分析的不错对象.

在浏览器中通过`script`标签加载[](https://unpkg.com/redux@4.0.4/dist/redux.js)后，脚本会在全局作用域下添加`Redux`变量
![redux object screenshot](/site_assets/E57038ED-4FB7-4eca-AE2D-56F675EFF327.png)

对应的源代码如下
```js
/**
 * 先探测脚本所处的运行环境和加载机制，然后将redux的方法和属性挂载到对应的作用域
 * 1. Node.js环境下和AMD加载机制下挂载到模块的导出对象
 * 2. script标签加载挂载到window对象
 */
(function (global, factory) {
	// Node.js环境
	typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
		// 浏览器环境（通过AMD方式加载）
		typeof define === 'function' && define.amd ? define(['exports'], factory) :
			// 浏览器环境（通过script标签加载）
			(global = global || self, factory(global.Redux = {}));
}(this, function (exports) {

    /* other code... */

    exports.__DO_NOT_USE__ActionTypes = ActionTypes;
	exports.applyMiddleware = applyMiddleware;
	exports.bindActionCreators = bindActionCreators;
	exports.combineReducers = combineReducers;
	exports.compose = compose;
	exports.createStore = createStore;

	Object.defineProperty(exports, '__esModule', { value: true });
}));
```
### __DO_NOT_USE__ActionTypes
```js
/**
 * 生成以小数点分割的随机数
 */
var randomString = function randomString() {
	return Math.random().toString(36).substring(7).split('').join('.');
};

/**
 * 这些action type是redux内部的保留关键字
 * 不可以在业务代码中使用这些action type
 */
var ActionTypes = {
    INIT: "@@redux/INIT" + randomString(),
    REPLACE: "@@redux/REPLACE" + randomString(),
    PROBE_UNKNOWN_ACTION: function PROBE_UNKNOWN_ACTION() {
        return "@@redux/PROBE_UNKNOWN_ACTION" + randomString();
    }
};
```

### compose
```js
/**
 * compose是一个工具函数，它接收一系列的函数作为数组，并且将这些函数形成一个调用栈，其中最右侧的函数位于栈顶，最左侧的函数位于栈底
 * 栈顶的函数可以接收任意个参数，并且将其返回值作为下一个函数的参数，以此类推，最终栈底的函数的返回值为最终的结果
 * @param {Array<Function>} functions 函数数组
 * @returns {Function}
 */
function compose(func1, func2, func3) {
    for (var _len = arguments.length, funcs = new Array(_len), _key = 0; _key < _len; _key++) {
        funcs[_key] = arguments[_key];
    }

    if (funcs.length === 0) {
        return function (arg) {
            return arg;
        };
    }

    if (funcs.length === 1) {
        return funcs[0];
    }

    return funcs.reduce(function (a, b) {
        return function () {
            return a(b.apply(void 0, arguments));
        };
    });
}
```

### createStore
`createStore`用于创建store，store就是redux用于存储应用状态数据的地方，其本质是一个对象，store拥有`getState`方法，用来获取应用状态数据，`dispatch`方法用来触发reducer的计算从而更新应用状态，`subscribe`方法用于订阅状态更新的处理函数，每次调用`dispatch`后redux都会执行这些函数.
![store screenshot](/site_assets/2CEE1E5D-E6F1-45f1-9292-FF87C5CEA046.png)

```js
/**
 * 创建store
 * @param {Function} reducer reducer函数用于计算state，每一次调用store.dispatch，redux都会调用reducer函数并传入当前的state和对应的action，
 * reducer函数会根据action的type属性和payload计算最新的state，对于未匹配到的action类型，reducer函数应该返回默认的或者当前的state，
 * reducer函数不能返回一个undefined值，但是可以返回null，因为一个应用任何时刻都有一个确定的状态，不可能是未定义的
 * @param {String} preloadedState preloadedState是一个序列化的对象字符串，用于表示默认的state，如果应用使用了服务端渲染，那么在浏览器端创建store的时候，
 * 应该将在服务端计算出的state经过序列化后传给preloadState
 * @param {Function} enhancer enhancer是一个高阶函数，以createStore函数为参数，并返回增强的'enhancedCreateStore'函数，开发者可以用enhancer给createStore函数
 * 添加额外的处理逻辑
 * @returns {Object}
 */
function createStore(reducer, preloadedState, enhancer) {
    var _ref2;

    /**
     * 参数有效性检查
     * enhancer函数只能有一个
     */
    if (typeof preloadedState === 'function' && typeof enhancer === 'function' || typeof enhancer === 'function' && typeof arguments[3] === 'function') {
        throw new Error('It looks like you are passing several store enhancers to ' + 'createStore(). This is not supported. Instead, compose them ' + 'together to a single function.');
    }

    // 参数容错处理
    if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
        enhancer = preloadedState;
        preloadedState = undefined;
    }

    // enhancer必须得是一个函数
    if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') {
            throw new Error('Expected the enhancer to be a function.');
        }

        // 调用enhancer函数并用增强版的'createStore'函数创建store
        return enhancer(createStore)(reducer, preloadedState);
    }
    
    // reducer必须得是一个函数
    if (typeof reducer !== 'function') {
        throw new Error('Expected the reducer to be a function.');
    }

    // 存储当前的reducer函数
    var currentReducer = reducer;
    // 存储应用的当前state
    var currentState = preloadedState;
    // 存储目前存在的状态更新订阅函数
    var currentListeners = [];
    var nextListeners = currentListeners;
    // 表示应用state此刻是否正处于被更新中，防止两个更新操作同时进行
    var isDispatching = false;

    /**
     * 数组浅拷贝，使得对currentListeners数组的变更更安全
     */
    function ensureCanMutateNextListeners() {
        if (nextListeners === currentListeners) {
            nextListeners = currentListeners.slice();
        }
    }

    /**
     * 回去应用的当前state
     * @returns {Object}
     */
    function getState() {
        if (isDispatching) {
            throw new Error('You may not call store.getState() while the reducer is executing. ' + 'The reducer has already received the state as an argument. ' + 'Pass it down from the top reducer instead of reading it from the store.');
        }

        return currentState;
    }

    /**
     * 订阅一个state发生更新的处理函数
     * @param {Function} listener 
     * @returns {Function} 执行此函数可以取消当前订阅
     */
    function subscribe(listener) {
        // 参数校验，listener必须是一个函数
        if (typeof listener !== 'function') {
            throw new Error('Expected the listener to be a function.');
        }

        // 不能在状态更新(reducer函数执行)的过程中订阅处理函数
        if (isDispatching) {
            throw new Error('You may not call store.subscribe() while the reducer is executing. ' + 'If you would like to be notified after the store has been updated, subscribe from a ' + 'component and invoke store.getState() in the callback to access the latest state. ' + 'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.');
        }

        // 函数订阅成功
        var isSubscribed = true;
        // 函数入队
        ensureCanMutateNextListeners();
        nextListeners.push(listener);
        return function unsubscribe() {
            if (!isSubscribed) {
                return;
            }

            // 不能在状态更新(reducer函数执行)的过程中取消订阅
            if (isDispatching) {
                throw new Error('You may not unsubscribe from a store listener while the reducer is executing. ' + 'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.');
            }

            isSubscribed = false;
            ensureCanMutateNextListeners();
            var index = nextListeners.indexOf(listener);
            nextListeners.splice(index, 1);
        };
    }

    /**
     * dispatch函数是redux中唯一可以更新state的途径，结合dispatch和reducer，开发者可以对应用状态进行记录、预测、重放等操作
     * @param {Object} action action必须有一个type属性，用于指示reducer如何计算最新的state
     */
    function dispatch(action) {
        // action是一个对象，但不能是数组、NULL等值
        if (!isPlainObject(action)) {
            throw new Error('Actions must be plain objects. ' + 'Use custom middleware for async actions.');
        }

        // action必须存在一个明确的type属性
        if (typeof action.type === 'undefined') {
            throw new Error('Actions may not have an undefined "type" property. ' + 'Have you misspelled a constant?');
        }

        // 不能同时存在两个state更新进程，否则哪个进程优先级更高呢？
        if (isDispatching) {
            throw new Error('Reducers may not dispatch actions.');
        }

        try {
            // 锁住，此次更新结束前后续所有的更新操作都丢弃
            isDispatching = true;
            // 调用reducer函数计算最新的state，reducer函数的参数为当前state和对应的action
            currentState = currentReducer(currentState, action);
        } finally {
            // 更新完毕，释放锁
            isDispatching = false;
        }

        // 更新完毕，按订阅顺序依次执行处理函数
        var listeners = currentListeners = nextListeners;

        for (var i = 0; i < listeners.length; i++) {
            var listener = listeners[i];
            listener();
        }

        // 返回action，提供给redux中间件进行后续处理
        return action;
    }

    /**
     * 应用运行期间更换reducer函数
     * @param {Function} nextReducer
     */
    function replaceReducer(nextReducer) {
        if (typeof nextReducer !== 'function') {
            throw new Error('Expected the nextReducer to be a function.');
        }

        currentReducer = nextReducer; 

        // REPLACE action和INIT action的目的一样，是为了计算默认的state
        dispatch({
            type: ActionTypes.REPLACE
        });
    }

    /**
     * 提供一种机制供类似rx.js这样的observable/reactive库使用
     */
    function observable() {
        var _ref;

        var outerSubscribe = subscribe;
        return _ref = {
            subscribe: function subscribe(observer) {
                if (typeof observer !== 'object' || observer === null) {
                    throw new TypeError('Expected the observer to be an object.');
                }

                function observeState() {
                    if (observer.next) {
                        observer.next(getState());
                    }
                }

                observeState();
                var unsubscribe = outerSubscribe(observeState);
                return {
                    unsubscribe: unsubscribe
                };
            }
        }, _ref[result] = function () {
            return this;
        }, _ref;
    } 

    // redux内部dispatch一个INIT的action，目的是为了计算应用最初的默认state
    dispatch({
        type: ActionTypes.INIT
    });

    return _ref2 = {
        dispatch: dispatch,
        subscribe: subscribe,
        getState: getState,
        replaceReducer: replaceReducer
    }, _ref2[result] = observable, _ref2;
}
```

### combineReducers
`combineReducers`函数用于组合reducer，在大型web应用中，state的结构比较复杂，对应reducer函数也比较庞大，出于提升可维护性的目的，按照业务逻辑对state进行拆分成多个子state，每个子state对应一个子reducer函数，在创建store的时候，先调用`combineReducers`函数将各个子reducer函数组合成一个总体的reducer函数.

```js
/**
 * combineReducers函数的使用例子
 */
const { combineReducers, createStore } = Redux;

// trivial child reducer
function reducerA(state, action) {
    return 'a';

}

// another trivial child reducer
function reducerB(state, action) {
    return 'b';
}

// attention: 应用state最终也拥有childA和childB两个属性
const rootReducer = combineReducers({
    childA: reducerA,
    childB: reducerB
});

const store = createStore(rootReducers);
```

```js
/**
 * 检查各个子reducer函数是否会返回undefined值，前面说了，reducer函数可以返回null，但是不能返回undefined
 */
function assertReducerShape(reducers) {
    Object.keys(reducers).forEach(function (key) {
        var reducer = reducers[key];
        // dispatch INIT action
        var initialState = reducer(undefined, {
            type: ActionTypes.INIT
        });

        if (typeof initialState === 'undefined') {
            throw new Error("Reducer \"" + key + "\" returned undefined during initialization. " + "If the state passed to the reducer is undefined, you must " + "explicitly return the initial state. The initial state may " + "not be undefined. If you don't want to set a value for this reducer, " + "you can use null instead of undefined.");
        }

        // dispatch一些其他的action再确认一下reducer的合法性
        if (typeof reducer(undefined, {
            type: ActionTypes.PROBE_UNKNOWN_ACTION()
        }) === 'undefined') {
            throw new Error("Reducer \"" + key + "\" returned undefined when probed with a random type. " + ("Don't try to handle " + ActionTypes.INIT + " or other actions in \"redux/*\" ") + "namespace. They are considered private. Instead, you must return the " + "current state for any unknown actions, unless it is undefined, " + "in which case you must return the initial state, regardless of the " + "action type. The initial state may not be undefined, but can be null.");
        }
    });
}

/**
 * 检查应用当前的state和reducers的对象结构是否一致，如果不一致，控制台给出警告提示
 * @param {Object} inputState 应用当前的state
 * @param {Object} reducers reducers对象
 * @param {action} action 当前被dispatch的action
 * @param {Object} unexpectedKeyCache 应用运行过程中state和reducers之间不一致的key都会缓存在这里
 */
function getUnexpectedStateShapeWarningMessage(inputState, reducers, action, unexpectedKeyCache) {
    var reducerKeys = Object.keys(reducers);
    var argumentName = action && action.type === ActionTypes.INIT ? 'preloadedState argument passed to createStore' : 'previous state received by the reducer';

    if (reducerKeys.length === 0) {
        return 'Store does not have a valid reducer. Make sure the argument passed ' + 'to combineReducers is an object whose values are reducers.';
    }

    if (!isPlainObject(inputState)) {
        return "The " + argumentName + " has unexpected type of \"" + {}.toString.call(inputState).match(/\s([a-z|A-Z]+)/)[1] + "\". Expected argument to be an object with the following " + ("keys: \"" + reducerKeys.join('", "') + "\"");
    }

    // 正常情况下，应用state中有的属性，reducers对象中也要有
    var unexpectedKeys = Object.keys(inputState).filter(function (key) {
        return !reducers.hasOwnProperty(key) && !unexpectedKeyCache[key];
    });
    unexpectedKeys.forEach(function (key) {
        unexpectedKeyCache[key] = true;
    });
    if (action && action.type === ActionTypes.REPLACE) return;

    if (unexpectedKeys.length > 0) {
        return "Unexpected " + (unexpectedKeys.length > 1 ? 'keys' : 'key') + " " + ("\"" + unexpectedKeys.join('", "') + "\" found in " + argumentName + ". ") + "Expected to find one of the known reducer keys instead: " + ("\"" + reducerKeys.join('", "') + "\". Unexpected keys will be ignored.");
    }
}

/**
 * 组合reducer函数
 * @param {Object} reducers reducers是一个对象，对象的key对应各个子state的key，value为各个子state对应的reducer函数
 * @returns {Function} 最终组合而成的应用reducer函数
 */
function combineReducers(reducers) {
    var reducerKeys = Object.keys(reducers);
    var finalReducers = {};

    // 复制reducers对象到闭包变量中
    for (var i = 0; i < reducerKeys.length; i++) {
        var key = reducerKeys[i];

        {
            // 错误检测，剔除无效的子reducer
            if (typeof reducers[key] === 'undefined') {
                warning("No reducer provided for key \"" + key + "\"");
            }
        }

        if (typeof reducers[key] === 'function') {
            finalReducers[key] = reducers[key];
        }
    }

    var finalReducerKeys = Object.keys(finalReducers); // This is used to make sure we don't warn about the same
    // keys multiple times.

    var unexpectedKeyCache;

    {
        unexpectedKeyCache = {};
    }

    var shapeAssertionError;

    try {
        // 检查各个reducer是否合法
        assertReducerShape(finalReducers);
    } catch (e) {
        shapeAssertionError = e;
    }

    return function combination(state, action) {
        // state默认是一个空对象
        if (state === void 0) {
            state = {};
        }

        // 存在不合法的reducer，抛出错误
        if (shapeAssertionError) {
            throw shapeAssertionError;
        }

        {
            // 检测state和reducers之间的不一致
            var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache);

            if (warningMessage) {
                warning(warningMessage);
            }
        }

        // 这个变量指示dispatch后应用state是否确实发生了改变
        var hasChanged = false;
        var nextState = {};

        // 依次执行每个子reducer函数，参数为对应的子state和action，返回值为新的子state
        for (var _i = 0; _i < finalReducerKeys.length; _i++) {
            var _key = finalReducerKeys[_i];
            var reducer = finalReducers[_key];
            var previousStateForKey = state[_key];
            var nextStateForKey = reducer(previousStateForKey, action);
            // 说了，reducer的返回值不能是undefined
            if (typeof nextStateForKey === 'undefined') {
                var errorMessage = getUndefinedStateErrorMessage(_key, action);
                throw new Error(errorMessage);
            }

            nextState[_key] = nextStateForKey;
            // attention: reducer函数在更新state的过程中，必须要返回一个新的对象，不能直接对参数state做更改后再返回，原因看下面
            hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
        }

        // 原因在这里，如果每个reducer都不返回新的对象，那么redux返回的是旧的应用state，最后应用状态未得到更新，此次dispatch操作就默默的失败了，无任何告警提示
        return hasChanged ? nextState : state;
    };
}
```

### applyMiddleware
`applyMiddleware`函数用于给redux添加中间件，所谓redux中间件，本质上是一个函数，用于重载store的dispatch函数调用并且添加自定义操作，但是最终依然会调用redux的dispatch方法更新state，例如`redux-thunk`这个常用的中间件，使得开发者可以dispatch一个函数类型的action，其内部原理是在重载后的dispatch函数内部判断了action的类型，如果是函数类型，则执行该函数并且将redux的dispatch和getState函数作为参数传入，开发者在函数内部根据函数执行结果调用redux的dispatch函数，如果是Object类型，则直接调用redux的dispatch函数.

```js
/**
 * redux-thunk源代码，其实就是一个简单的函数
 */
```

```js
/**
 * applyMiddlware
 */
```





