---
  category: 专业知识
  tags:
    - Redux
  date: 2021-03-01
  title: 自己动手实现一个redux
  vssue-title: 自己动手实现一个redux
---
```js
function createStore(reducer, preloadedState, enhancer) {
  if (enhancer) {
    return enhancer(createStore)(reducer, preloadedState);
  }

  let currentState = preloadedState;
  let currentReducer = reducer;
  let listeners = [];

  function getState() {
    return currentState;
  }

  function dispatch(action) {
    currentState = currentReducer(currentState, action);
    listeners.forEach((fn) => fn());
  }

  function subscribe(listener) {
    listeners.push(listener);
    return function () {
      listeners = listeners.filter((fn) => fn !== listener);
    };
  }

  function replaceReducer(nextReducer) {
    reducer = nextReducer;
    // 强行调用一次reducer获取外部的默认state
    dispatch({ type: "@@redux/REPLACE" });
  }

  // 强行调用一次reducer获取外部的默认state
  dispatch({ type: "@@redux/INIT" });
  return {
    getState,
    dispatch,
    subscribe,
    replaceReducer,
  };
}

function combineReducers(reducers) {
  return function (state = {}, action) {
    let combinedState = {};
    Object.keys(reducers).forEach((key) => {
      combinedState[key] = reducers[key](state[key], action);
    });
    return combinedState;
  };
}

// 个人认为叫bindDispatch更合适一点
function bindActionCreators(actionCreators, dispatch) {
  let boundActionCreators = {};
  Object.keys(actionCreators).forEach((key) => {
    boundActionCreators[key] = (...args) => {
      dispatch(actionCreators[key](...args));
    };
  });
  return boundActionCreators;
}

function compose(...funcs) {
  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}

function applyMiddleware(...middlewares) {
  return function (createStore) {
    return function (reducer, preloadedState) {
      let store = createStore(reducer, preloadedState);
      const chain = middlewares.map((middleware) => middleware(store));
      let dispatch = compose(...chain)(store.dispatch);
      return {
        ...store,
        dispatch,
      };
    };
  };
}

if (require.main === module) {
  let defaultState = {
    a: 1,
  };
  let reducer = (state = defaultState, action) => {
    switch (action.type) {
      case "ADD": {
        const { a } = state;
        return {
          ...state,
          a: a + 1,
        };
      }
      default:
        return state;
    }
  };
  let store = createStore(reducer);

  console.group("get initial state");
  console.log(store.getState());
  console.groupEnd();

  console.group("get state after dispatch type ADD");
  store.dispatch({ type: "ADD" });
  console.log(store.getState());
  console.groupEnd();

  console.group("subscribe");
  const unsubscribe = store.subscribe(() => {
    console.log("listener1");
  });
  store.dispatch({ type: "ADD" });
  console.groupEnd();

  console.group("unsubscribe");
  unsubscribe();
  store.dispatch({ type: "ADD" });
  console.groupEnd();

  console.group("test combineReducers");
  let defaultState1 = {
    a: 1,
  };
  let reducer1 = (state = defaultState1, action) => {
    if (action.type === "reducer1") {
      return { ...state, a: state.a + 1 };
    }
    return state;
  };
  let defaultState2 = {
    b: 1,
  };
  let reducer2 = (state = defaultState2, action) => {
    if (action.type === "reducer2") {
      return { ...state, b: state.b + 1 };
    }
    return state;
  };

  let combinedReducer = combineReducers({
    reducer1,
    reducer2,
  });

  store = createStore(combinedReducer);
  store.dispatch({ type: "reducer1" });
  console.log(store.getState());
  store.dispatch({ type: "reducer2" });
  console.log(store.getState());
  console.groupEnd();

  console.group("applyMiddlewares");
  const logger = (store) => (dispatch) => (action) => {
    console.log("before", store.getState());
    dispatch(action);
    console.log("after", store.getState());
  };
  let storeWithLogger = createStore(
    reducer,
    defaultState,
    applyMiddleware(logger)
  );
  storeWithLogger.dispatch({ type: "ADD" });

  console.groupEnd();
}

```