Redux DevTools Instrumentation
==============================

Redux enhancer used along with [Redux DevTools](https://github.com/gaearon/redux-devtools) or [Remote Redux DevTools](https://github.com/zalmoxisus/remote-redux-devtools).

### Installation

```
npm install --save-dev redux-devtools-instrument
```

### Usage

Add the store enhancer:

##### `store/configureStore.js`

```js
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import devTools from 'remote-redux-devtools';
import reducer from '../reducers';

// Usually you import the reducer from the monitor
// or apply with createDevTools as explained in Redux DevTools
const monitorReducer = (state = {}, action) => state; 

export default function configureStore(initialState) {
  const enhancer = compose(
    applyMiddleware(...middlewares),
    // other enhancers and applyMiddleware should be added before the instrumentation
    instrument(monitorReducer, { maxAge: 50 })
  );
  
  // Note: passing enhancer as last argument requires redux@>=3.1.0
  return createStore(reducer, initialState, enhancer);
}
```

### API

`instrument(monitorReducer, [options])`

- arguments
  - **monitorReducer** *function* called whenever an action is dispatched ([see the example of a monitor reducer](https://github.com/gaearon/redux-devtools-log-monitor/blob/master/src/reducers.js#L13)).
  - **options** *object*
    - **maxAge** *number* - maximum allowed actions to be stored on the history tree, the oldest actions are removed once `maxAge` is reached. 
- as the result enhance the store by adding a `liftedStore` object which state (`store.liftedStore.getState()`) contains the following
  - **actionsById** *object* containing the history of actions (as objects) 
    - **action** *object* dispatched to the store
    - **type** *string* - instrumentation action type
    - **timestamp** *number* - unix time stamp indicating when the action was dispatched (reducer started)
    - **duration** *[DOMHighResTimeStamp](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp)* - number of milliseconds spent for executing the reducer.
  - **stagedActionIds** *array* of actions ids (from `actionsById`) which should be shown on the monitor.
  - **skippedActionIds** *array* of skipped (canceled) actions ids which should be shown as toggled in the monitor.
  - **nextActionId** *number* - id of the next action which will be added to `actionsById`.
  - **computedStates** *array* containing the history of states (as objects) 
    - **state** *object* - the result state after dispatching the action with the id corresponding to the current index 
    - **error** *string* - error message in case there's an exception while processing the current reducer.
  - **committedState** *object* - used as initial state after committing (squashing) actions or when `maxAge` is reached.
  - **currentStateIndex** *number* - index of current state in `computedStates` array, which will be returned for `store.getState()`. Also used for jumping to a specific state (moving back and forth).

### License

MIT
