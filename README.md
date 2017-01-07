# Flux Standard Action Ducks

## Proposal

Note: this is a modified version of the [original ducks proposal](https://github.com/erikras/ducks-modular-redux) that also uses the [Flux Standard Action](https://github.com/acdlite/flux-standard-action) specification.

### Rules

A duck module...

1. MUST `export` all of the following:
    - A `reducer` function.
    - An `actionCreators` object.
2. MAY `export` any of the following:
    - An `actionTypes` object.
    - An `initialState` object.
3. MUST have action types defined as `UPPER_SNAKE_CASE` constants, with each value structured as follows: `package-or-app-name/duck-name/ACTION_TYPE`

### Usage

Import your ducks and export a combined `initialState` object and `reducer` function:

```javascript
// ducks/index.js

import { combineReducers } from 'redux';

import * as widgetsDuck from './widgets';
import * as loginDuck from './login';

export const initialState = {
  widgets: widgetsDuck.intialState,
  login: loginDuck.initialState,
};

export const reducer = combineReducers({
  widgets: widgetsDuck.reducer,
  login: loginDuck.reducer,
});
```

Combine the ducks reducer with any third-party reducers you want to use and create your store:

```javascript
// store/index.js

import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { reducer as formReducer } from 'redux-form';
import { reducer, initialState } from '../ducks';

const rootReducer = combineReducers({
  app: reducer,
  form: formReducer,
});

const enhancer = applyMiddleware(thunk);

export default createStore(rootReducer, initialState, enhancer);
```

### Examples

#### Simple duck

```javascript
// ducks/widgets.js

const LOAD = 'my-app/widgets/LOAD';
const CREATE = 'my-app/widgets/CREATE';
const UPDATE = 'my-app/widgets/UPDATE';
const DELETE = 'my-app/widgets/DELETE';

export const actionTypes = { LOAD, CREATE, UPDATE, DELETE };

export const actionCreators = {
  loadWidgets() {
    return {
      type: LOAD,
    };
  },
  createWidget(widget) {
    return {
      type: CREATE,
      payload: widget,
    };
  },
  updateWidget(widget) {
    return {
      type: UPDATE,
      payload: widget,
    };
  },
  removeWidget(widget) {
    return {
      type: REMOVE,
      payload: widget,
    };
  },
};

export function reducer(state = {}, action = {}) {
  switch (action.type) {
    // do reducer stuff
    default: {
      return state;
    }
  }
}
```

#### Async duck

```javascript
// ducks/login.js

const REQUEST = 'my-app/login/REQUEST';
const SUCCESS = 'my-app/login/SUCCESS';
const FAILURE = 'my-app/login/FAILURE';

export const actionTypes = { REQUEST, SUCCESS, FAILURE };

// Note that these are only used internally
// and therefore do not need to be exported.
function loginRequest() {
  return {
    type: REQUEST,
  }
}

function loginSuccess(data) {
  return {
    type: SUCCESS,
    payload: data,
  }
}

function loginFailure(error) {
  return {
    type: FAILURE,
    payload: error,
    error: true,
  }
}

export const actionCreators = {
  // Async action creator (requires Redux Thunk middleware)
  login(email, password) {
    return dispatch => {
      dispatch(loginRequest());
        
      const fetchInit = {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      };

      return fetch('/login', fetchInit).then(
        data => dispatch(loginSuccess(data)),
        error => dispatch(loginFailure(error)),
      );
    };
  },
};

export const initialState = {
  user: null,
};

export function reducer(state = initialState, action = {}) {
  // do reducer stuff
  default: {
    return state;
  }
}
```

#### Using bound action creators in a component

```javascript
/* @jsx */
// components/MyComponent/index.js

import React, { PureComponent } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import { actionCreators } from '../ducks/widgets';
import WidgetList from './WidgetList';

@connect
export default class MyComponent extends PureComponent {
  render() {
    const { dispatch } = this.props;
    const boundActionCreators = bindActionCreators(actionCreators, dispatch);
    
    return (
      <WidgetList {...boundActionCreators} />
    );
  }
}
```
