---
title: Redux principles
tags: [Notebooks/Redux]
---

### Redux principles

  The state of the application is stored in a single object called state tree
  
  State tree is read-only, every time you want to modify the state tree you need to call an action
  
  Actions are plain objects describing the changes

  An action **should** have a type property:

    { type: "INCREMENT" }
    { type: "DECREMENT" }

  and its required properties

    { type: "INCREMENT", index: 3 }
    { type: "SET_FILTER", filter: "ACTIVE" }
  
#### Pure functions
    
  Functions that, given the same parameters, will return the same value
  
    function square(x) {
      return x * x;
    }

    
 Pure functions also don't modify the values passed to them value value
 
    function squareAll(items) {
      return items.map(square)
      // returns a new array
    }
    
 On the opposite, impure functions may call the database or the network, they may have side effects, they may operate on the DOM, and they may override the values that you pass to them.
    
    
#### The Reducer

  The state mutations in your app need to be described as a pure function, that takes the previous state:
   
    0
     
 and the action being dispatched:
 
    { type: "INCREMENT" }
  
  and returns the next state of your app:
  
    1
    
  this function is called the **Reducer**
  
    const counter = (state = 0, action) => {
      switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
      }
    }
    
    expect(
      counter(0, { type: 'INCREMENT' })
    ).toEqual(2);
    
    expect(
      counter(1, { type: 'DECREMENT' })
    ).toEqual(0);
    
    expect(
      counter(undefined, {})
    ).toEqual(0);
    
    
#### The Store

  The store binds together the 3 principles of Redux. It holds the current application state tree, lets you dispatch actions and when you create it, you need to specify the reducer that tells how the state will be updated with actions.
  
    import { createStore } from redux;
    const store = createStore(counter);
    
  The store has 3 important methods:
  
    store.getState() // 0
    
  The first method is called `getState`. It retrieves the current `state` of the Redux store.
  Here we are getting 0 as it is the defined initial state of our application.
  
  Then we have `dispatch`. It lets you dispatch actions to change the state.
  
    store.dispatch({ type: 'INCREMENT' });
    store.getState(); // 1
    
  The third store method is `subscribe`. It lets you register a callback that the redux store will always call every time an action has been dispatched so you can update the UI of your app to reflect the current state.
  
    const render = () => {
      document.body.innerText = store.getState();
    };
    
    store.subscribe(render);
    render()
    
    document.addEventListener('click', () => {
      store.dispatch({ type: 'INCREMENT' });
    });
    
  The `createStore` method, if implemented from scratch, should look similar to this:
  
    const createStore = (reducer) => {
      let state;
      let listeners = [];
      
      const getState = () => state;
      
      const dispatch = (action) => {
        state = reducer(state, action);
        listeners.forEach(listener => listener());
      };
      
      const subscribe = (listener) => {
        listeners.push(listener);
        return () => {
          listeners = listeners.filter(l => l !== listener);
          // this is a way of removing the listener (unsubscribing) from the store
        };
      };
      
      dispatch({});
      // we want the store to have its initial state
    }
    
#### Reducer Composition

  The app's state tree can get quite big and complex after a while as different contextual states will be stored. Reducer Composition is a pattern that helps with scaling the Reducer function by having other more specific Reducer functions that handle parts of the state and together compose the whole state of the application.
  
  Taking a simple TODO app for example, the initial state should look like:
   
    {
      todos: [],
      visibilityFilter: '',
    }
  
  and its reducer:
  
    const todo = (
      state = {
        todos: [],
        visibilityFilter: '',
      },
      action
    ) => {
      switch (action.type) {
        case 'ADD_TODO':
          return {
            ...state,
            todos: [
              ...state.todos,
              {
                id: action.id,
                text: action.text,
                completed: false,
              },
            ],
          };
        case 'TOGGLE_TODO':
          return {
            ...state,
            todos: todos.map(todo => {
              todo.id !== action.id
                ? todo
                : { ...todo, completed: !todo.completed }
            }),
          };  
        case 'SET_VISIBILITY_FILTER': 
          return {
            ...state,
            filter: action.filter,
          }
        default:
          return state;
      }
    }
    
  There's a lot going on here and this reducer function built like this ends up being a bit heavy in the eyes. What could be done to improve its readability and scalability is to segment it:
  
    const todos = (state = [], action) => {
      switch (action.type) {
        case 'ADD_TODO':
          return [
              ...state,
              {
                id: action.id,
                text: action.text,
                completed: false,
              },
            ],
          };
        case 'TOGGLE_TODO':
          return state.map(todo => {
            todo.id !== action.id
              ? todo
              : { ...todo, completed: !todo.completed }
          });
        default:
          return state;
    };
  
  this being the reducer that deals with the possible actions that mutate the `todos` array,
  
    const visibilityFilter = (state = 'SHOW_ALL', action) => {
      switch (action.type) {
        case 'SET_VISIBILITY_FILTER':
          return action.filter;
        default:
          return state;
      }
    };
  
  and this is the one that deals with setting a new visibility filter.
  
  These more specific and contained reducers can be combined using the **reducer composition pattern** to create a new reducer that composes its results into a single state object.
  
    const todoApp = (state = {}, action) => {
      return {
        todos: todos(
          state.todos,
          action,
        ),
        visibilityFilter: visibilityFilter(
          state.visibilityFilter,
          action,
        ),
      }
    };
