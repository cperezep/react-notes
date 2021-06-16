## React Hooks

Normally an interactive application will need to hold state somewhere. In React, you use special functions called "hooks" to do this. Common built-in hooks include:

- `React.useState`
- `React.useEffect`
- `React.useRef`
- `React.useReducer`
- `React.useCallback`
- `React.useContext`

Each of these is a special function that you can call inside your custom React component function to store data (like state) or perform actions (or side-effects).


Each of the hooks has a unique API. Some return a value (like `React.useRef` and `React.useContext`), others return a pair of values (like `React.useState` and `React.useReducer`), and others return nothing at all (like `React.useEffect`).

### React.useState
An example of a component that uses the `useState` hook and an onClick event handler to update that state:

```jsx
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(count + 1)
  return <button onClick={increment}>{count}</button>
}
```

`React.useState` is a function that accepts a single argument. That argument is the initial state for the instance of the component. In this case, the state will start as `0`.

`React.useState` returns a pair of values. It does this by returning an array with two elements (and we use destructuring syntax to assign each of those values to distinct variables). The first of the pair is the state value and the second is a function we can call to update the state. We can name these variables whatever we want. Common convention is to choose a name for the state variable, then prefix `set` in front of that for the updater function.

State can be defined as: data that changes over time. So how does this work over time? When the button is clicked, our `increment` function will be called at which time we update the `count` by calling `setCount`.

When we call `setCount`, that tells React to re-render our component. When it does this, the entire `Counter` function is re-run, so when `React.useState` is called this time, the value we get back is the value that we called `setCount` with. And it continues like that until `Counter` is unmounted (removed from the application), or the user closes the application.

#### Lazy state initialization
```jsx
function Counter() {
  const [count, setCount] = React.useState(() => 0)
  const increment = () => setCount(previousCount => previousCount + 1)
  return <button onClick={increment}>{count}</button>
}
```
If you were to add a console.log to the function body of the Counter function, you would find that the function is run every time you click the button. This makes sense because your Counter function is run during every render phase and clicking the button triggers the state update which triggers the re-render. One thing you need to keep in mind is that if the function body runs, that means all the code inside it runs as well. Which means any variables you create or arguments you pass are created and evaluated every render. This is normally not a big deal because JavaScript engines are very fast and can optimize for this kind of thing. So something like this would be no problem:

```javascript
const initialState = 0
const [count, setCount] = React.useState(initialState)
```

However, what if the initial value for your state is computationally expensive?
```javascript
const initialState = calculateSomethingExpensive(props)
const [count, setCount] = React.useState(initialState)
```

Remember that the only time React needs the initial state is `initially` Meaning, it only really needs the initial state on the first render. But because our function body runs every time there's a re-render of our component, we end up running that code on every render, even if its value is not used or needed.

Creating a function is fast. Even if what the function does is computationally expensive. So you only pay the performance penalty when you call the function. So if you pass a function to useState, React will only call the function when it needs the initial value (which is when the component is initially rendered).

This is called "lazy initialization." It's a performance optimization. You shouldn't have to use it a whole lot, but it can be useful in some situations, so it's good to know that it's a feature that exists and you can use it when needed. I would say I use this only 2% of the time. It's not really a feature I use often.

### React.useEffect
`React.useEffect` is a built-in hook that allows you to run some custom code after React renders (and re-renders) your component to the DOM. It accepts a
callback function which React will call after the DOM has been updated:

```javascript
React.useEffect(() => {
  // your side-effect code here.
  // this is where you can make HTTP requests or interact with browser APIs.
  // This code always will be executed and depend on everything 
});
```

One important thing to note about the `useEffect` hook is that you cannot return anything other than the cleanup function. This has interesting implications with regard to async/await syntax:

```javascript
// this does not work, don't do this:
React.useEffect(async () => {
  const result = await doSomeAsyncThing()
  // do something with the result
})
```

The reason this doesn't work is because when you make a function async, it automatically returns a promise (whether you're not returning anything at all, or explicitly returning a function). This is due to the semantics of async/await syntax. So if you want to use async/await, the best way to do that is like so:

```javascript
React.useEffect(() => {
  async function effect() {
    const result = await doSomeAsyncThing()
    // do something with the result
  }
  effect()
})
```
This ensures that you don't return anything but a cleanup function.

It's typically just easier to extract all the async code into a utility function which I call and then use the promise-based `.then` method instead of using async/await syntax:

```javascript
React.useEffect(() => {
  doSomeAsyncThing().then(result => {
    // do something with the result
  })
})
```

#### Effect dependencies
The callback we're passing to `React.useEffect` is called after _every_ render of our component (including re-renders). Our component can re-render for various reasons (for example, when a parent component in the application tree gets re-rendered). React.useEffect allows you to pass a second argument called the "dependency array" which signals to React that your effect callback function should be called when (and only when) those dependencies change.

```javascript
React.useEffect(() => {
  // This code only will be executed in initial renderering and when 'dependencie' changes.
}, [dependencie]);
```

```jsx
React.useEffect(() => {
  // This code only will be executed once time, not depend on anything
}, []);
```

### React.useRef
`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument (initialValue). The returned object will persist for the full lifetime of the component.
```javascript
function MyDiv() {
  const myRef = React.useRef();
  React.useEffect(() => {
    const myDiv = myRef.current;
    // myDiv is the div DOM node!
    console.log(myDiv);
  }, [])
  return <div ref={myRef}>hi</div>;
}
```
Remember that when you do: `<div>hi</div>` that's actually syntactic sugar for a `React.createElement` so you don't actually have access to DOM nodes in your render method. In fact, DOM nodes aren't created at all until the `ReactDOM.render` method is called. Your component's render method is really just responsible for creating and returning React Elements and has nothing to do with the DOM in particular.

If you pass a ref object to React with `<div ref={myRef} />`, React will set its `.current` property to the corresponding DOM node whenever that node changes. Essentially, `useRef` is like a “box” that can hold a mutable value in its `.current` property.

However, `useRef()` is useful for more than the `ref` attribute. It’s [handy for keeping any mutable value around](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables) similar to how you’d use instance fields in classes.

This works because `useRef()` creates a plain JavaScript object. The only difference between `useRef()` and creating a `{current: ...}` object yourself is that `useRef` will give you the same ref object on every render.

Keep in mind that `useRef` doesn’t notify you when its content changes. Mutating the `.current` property doesn’t cause a re-render. If you want to run some code when React attaches or detaches a ref to a DOM node, you may want to use a callback ref instead.

### React.useReducer
Sometimes you want to separate the state logic from the components that make the state changes. In addition, if you have multiple elements of state that typically change together, then having an object that contains those elements of state can be quite helpful. This is where `useReducer` comes in really handy. `useReducer` is usually preferable to useState when you have complex state logic that involves multiple sub-values or when the next state depends on the previous one. 

Accepts a reducer of type `(state, action) => newState`, and returns the current state paired with a `dispatch` method. (If you’re familiar with Redux, you already know how this works.)

```jsx
const initialState = {count: 0};

function countReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {count: state.count + 1};
    case 'DECREMENT':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(countReducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'DECREMENT'})}>-</button>
      <button onClick={() => dispatch({type: 'INCREMENT'})}>+</button>
    </>
  );
}
```

#### Lazy initialization
You can also create the initial state lazily. To do this, you can pass an init function as the third argument. The initial state will be set to init(initialArg).

It lets you extract the logic for calculating the initial state outside the reducer. This is also handy for resetting the state later in response to an action:

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function countReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {count: state.count + 1};
    case 'DECREMENT':
      return {count: state.count - 1};
    case 'RESET':
      return init(action.payload);
    default:
      throw new Error(`Unsupported action type: ${action.type}`);
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(countReducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({ type: 'RESET', payload: initialCount })}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'DECREMENT'})}>-</button>
      <button onClick={() => dispatch({type: 'INCREMENT'})}>+</button>
    </>
  );
}
```

Here are two really helpful blog posts comparing `useState` and `useReducer`:

- [Should I useState or useReducer?](https://kentcdodds.com/blog/should-i-usestate-or-usereducer)
- [How to implement useState with useReducer](https://kentcdodds.com/blog/how-to-implement-usestate-with-usereducer)

### React.useCallback
```javascript
React.useEffect(() => {
  window.localStorage.setItem('count', count)
}, [count]) // <-- that's the dependency list
```

The dependency list is how React knows whether to call your callback (and if you don't provide one then React will call your callback every
render). It does this to ensure that the side effect you're performing in the callback doesn't get out of sync with the state of the application.

But what happens if I use a function in my callback?

```javascript
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, []) // <-- what goes in that dependency list?
```

We could just put the `count` in the dependency list and that would actually work, but what would happen if we changed `updateLocalStorage`?

```javascript
const updateLocalStorage = () => window.localStorage.setItem(key, count)
```

Would we remember to update the dependency list to include the `key`? This can be a pain to keep track of dependencies. Especially if the function that we're using in our `useEffect` callback is coming to us from props (in the case of a custom component) or arguments (in the case of a custom hook).

Instead, it would be much easier if we could just put the function itself in the dependency list:

```javascript
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage]) // <-- function as a dependency
```

The problem with that though is because `updateLocalStorage` is defined inside the component function body, it's re-initialized every render, which means it's brand new every render, which means it changes every render, which means, you guessed it, our callback will be called every render!

**This is the problem `useCallback` solves**. And here's how to solve it

```javascript
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count], // <-- That's a dependency list!
)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage])
```

What that does is we pass React a function and React gives that same function back to us, but with a catch. On subsequent renders, if the elements in the
dependency list are unchanged, instead of giving the same function back that we give to it, React will give us the same function it gave us last time.

So while we still create a new function every render (to pass to `useCallback`), React only gives us the new one if the dependency list changes.

`useCallback` will return a memoized version of the callback that only changes if one of the dependencies has changed. `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

🔥 "Why don't we just wrap every function in `useCallback`?" You can read about this here [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback).

🔥 "Value stability" and "Memoization" [Memoization and React](https://epicreact.dev/memoization-and-react)

### React.useContext

Sharing state between components is a common problem. The best solution for this is to [lift your state](https://reactjs.org/docs/lifting-state-up.html). This requires [prop drilling](https://kentcdodds.com/blog/prop-drilling) which is not a problem, but there are some times where prop drilling can cause a real pain.

To avoid this pain, we can insert some state into a section of our react tree, and then extract that state anywhere within that react tree without having to explicitly pass it everywhere. This feature is called `context`. In some ways it's like global variables, but it doesn't suffer from the same problems (and maintainability nightmares) of global variables thanks to how the API works to make the relationships explicit.

```javascript
import * as React from 'react'

const CountContext = React.createContext()

function CountDisplay() {
  const count = React.useContext(CountContext)
  return <div>Count is: {count}</div>
}

ReactDOM.render(
  <CountContext.Provider value="20">
    <CountDisplay />
  </CountContext.Provider>,
  document.getElementById('root'),
);
// renders <div>Count is: 20</div>
```

`<CountDisplay />` could appear anywhere in the render tree, and it will have access to the `value` which is passed by the `CountContext.Provider` component.

Note that It doesn't have an initial value for the CountContext. If I wanted an initial value, e.g. `React.createContext({count: 0})`. But I don't include a default value and that's intentional. The defaultValue is only useful in a situation like this:

```javascript
function CountDisplay() {
  const {count} = React.useContext(CountContext);

  return <div>{count}</div>;
}

ReactDOM.render(<CountDisplay />, document.getElementById('⚛️'));
```

Because we don't have a default value for our CountContext, we'll get an error when we're destructuring the return value of useContext. This is because our default value is undefined and you cannot destructure undefined.

None likes runtime errors, so the reaction may be to add a default value to avoid the runtime error. However, what use would the context be if it didn't have an actual value? If it's just using the default value that's been provided, then it can't really do much good. 99% of the time that we're going to be creating and using context in applications, we want our context consumers (those using useContext) to be rendered within a provider which can provide a useful value.

> Note, there are situations where default values are useful, but most of the time they're not necessary or useful.

[The React docs](https://reactjs.org/docs/context.html#reactcreatecontext) suggest that providing a default value "can be helpful in testing components in isolation without wrapping them.

Most of the time, It is not recommended using a default value because it's probably a mistake to try and use context outside a provider.

#### Custom Provider Component

For this context to be useful at all we need to use the Provider and expose a component that provides a value. Our component will be used like this:

```javascript
function App() {
  return (
    <CountProvider>
      <CountDisplay />
      <Counter />
    </CountProvider>
  );
}

ReactDOM.render(<App />, document.getElementById('⚛️'));
```

So, the component will be implemented like this:

```javascript
import * as React from 'react'

const CountContext = React.createContext();

function countReducer(state, action) {
  switch (action.type) {
    case 'increment': {
      return {count: state.count + 1};
    }

    case 'decrement': {
      return {count: state.count - 1};
    }

    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

function CountProvider({children}) {
  const [state, dispatch] = React.useReducer(countReducer, {count: 0});
  const value = {state, dispatch};

  return <CountContext.Provider value={value}>{children}</CountContext.Provider>;

}

export {CountProvider}
```

> This does not mean it has to be this complicated every time! Feel free to use useState instead useReducer if that suits your scenario. In addition, some providers are going to be short and simple like this, and others are going to be MUCH more involved with many hooks.

#### Using a Custom Consumer Hook

The idea is to say if there's not a context, then I can throw a new error that says counter must be rendered within the count provider. At least the error message that we get in here is more descriptive.

```javascript
import * as React from 'react'

const CountContext = React.createContext();

function countReducer(state, action) {
  switch (action.type) {
    case 'increment': {
      return {count: state.count + 1};
    }

    case 'decrement': {
      return {count: state.count - 1};
    }

    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

function CountProvider({children}) {
  const [state, dispatch] = React.useReducer(countReducer, {count: 0});
  const value = {state, dispatch};

  return <CountContext.Provider value={value}>{children}</CountContext.Provider>;
}

function useCount() {
  const context = React.useContext(CountContext)
  if (context === undefined) {
    throw new Error('useCount must be used within a CountProvider')
  }

  return context
}

export {CountProvider, useCount}
```

First, the useCount custom hook uses React.useContext to get the provided context value from the nearest CountProvider. However, if there is no value, then we throw a helpful error message indicating that the hook is not being called within a function component that is rendered within a CountProvider. This is most certainly a mistake, so providing the error message is valuable. #FailFast.

```javascript
import * as React from 'react';
import { useCount } from 'count-context';

function CountDisplay() {
  const {count} = useCount()
  return <div>{`The current count is ${count}`}</div>
}

```

```javascript
import { CountProvider } from 'count-context';

function App() {
  return (
    <CountProvider>
      <CountDisplay />
      <Counter />
    </CountProvider>
  );
}

ReactDOM.render(<App />, document.getElementById('⚛️'));
```

Note that NOT exporting CountContext is intentional. We expose only one way to provide the context value and only one way to consume it. This allows to ensure that people are using the context value the way it should be and it allows to provide useful utilities for consumers.

#### Notes

1. You shouldn't be reaching for context to solve every state sharing problem that crosses your desk.
2. Context does NOT have to be global to the whole app, but can be applied to one part of your tree.
3. You can (and probably should) have multiple logically separated contexts in your app.
4. Context also has the unique ability to be scoped to a specific section of the React component tree. A common mistake of context (and generally any “application” state) is to make it globally available anywhere in your application when it’s actually only needed to be available in a part of the app (like a single page). Keeping a context value scoped to the area that needs it most has improved performance and maintainability characteristics.
5. Keep in mind that while context makes sharing state easy, it's not the only solution to Prop Drilling pains and it's not necessarily the best solution either. React's composition model is powerful and can be used to avoid issues with prop drilling as well. Learn more about this from [Michael Jackson on Twitter](https://twitter.com/mjackson/status/1195495535483817984)


### React Hook Flow
Understanding the order in which React hooks are called can be really helpful in using React hooks effectively. This [demo](https://agitated-cori-c5df24.netlify.app) will explore the lifecycle of a function component with hooks with colorful console log statements so it can help to visualize the way that your React components mount and update and unmount and the order in which things are run.

## Concepts

### Lifting State Up
A common question from React beginners is how to share state between two sibling components. The answer is to ["lift the state"](https://reactjs.org/docs/lifting-state-up.html) which basically amounts to finding the lowest common parent shared between the two components and placing the state management there, and then passing the state and a mechanism for updating that state down into the components that need it.

### [Colocating State](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster/)
The principle of colocation is: `Place code as close to where it's relevant as possible`
Try to keep your state as close to where it's relevant as possible. As you make changes over time, maintain that practice of, "We don't need this state over here anymore, so let's just co-locate that state to where it needs to be." Ask yourself "do I really need the modal's status (open/closed) state to be in Redux?" (the answer is probably "no"). Colocate your state and you'll find yourself with a faster, simpler codebase.

### [Derived State](https://kentcdodds.com/blog/dont-sync-state-derive-it)
- **Managed State:** State that you need to explicitly manage
- **Derived State:** State that you can calculate based on other state


### Use Status
We could make things much simpler by having some state to set the explicit status of our components. Our components can be in the following "states":

- `idle`: no request made yet
- `pending`: request started
- `resolved`: request successful
- `rejected`: request failed

Try to use a status state by setting it to these string values rather than
relying on existing state or booleans.

Learn more about this concept here:
https://kentcdodds.com/blog/stop-using-isloading-booleans

```javascript
function MyComponent() {
  const [status, setStatus] = React.useState('idle');

  React.useEffect(() => {
    setStatus('pending');

    fetchAPI(dependencie)
      .then(result => {
        setStatus('resolved');
      })
      .catch(error => {
        setStatus('rejected');
      });
  }, [dependencie]);

  if (status === 'idle') {
    return 'Submit something!';
  } else if (status === 'pending') {
    return 'Loading...';
  } else if (status === 'rejected') {
    throw error;
  } else if (status === 'resolved') {
    return 'Info';
  }
}
```

### Store state in an object
Normally it is not a problem when you’re calling a bunch of state updaters in a row, but each call to our state updater can result in a re-render of our component. React normally batches these calls so you only get a single re-render, but it’s unable to do this in an asynchronous callback (like promise success and error handlers).

```javascript
function MyComponent() {
  const [data, setData] = React.useState(null);
  const [error, setError] = React.useState(null);
  const [status, setStatus] = React.useState('idle');

  React.useEffect(() => {
    setStatus('pending');
    fetchAPI(dependencie)
      .then(result => {
        setStatus('resolved');
        setData(result);
      })
      .catch(error => {
        setStatus('rejected');
        setError(error);
      });
  }, [dependencie]);
}
```

We can took all three elements of state and putting them into a single object. Then instead of setting those in individual setters, we set them in a single setter. That helps us avoid issues were calling a state updater function in an asynchronous handler like this is going to trigger multiple re-renders. I wouldn't recommend always putting your state into a single object like this, because that's not often a problem.

```javascript
function MyComponent() {
  const [state, setState] = React.useState({
    status: 'idle',
    data: null,
    error: null,
  });
  const {status, pokemon, error} = state;

  React.useEffect(() => {
    setState({ status: 'pending' });
    fetchAPI(dependencie)
      .then(result => {
        setState({ data: result, status: 'resolved' });
      })
      .catch(error => {
        setState({ error, status: 'rejected' });
      });
  }, [dependencie]);
}
```

A better alternative is using `useReducer` can solve this problem really elegantly, but we can still accomplish this by storing our state as an object that has all the properties of state we're managing.

#### Using a custom hook
```javascript
function asyncReducer(state, action) {
  switch (action.type) {
    case 'pending': {
      return {status: 'pending', data: null, error: null};
    }
    case 'resolved': {
      return {status: 'resolved', data: action.data, error: null};
    }
    case 'rejected': {
      return {status: 'rejected', data: null, error: action.error};
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

function useAsync(asyncCallback, initialState) {
  const [state, dispatch] = React.useReducer(asyncReducer, {
    status: 'idle',
    data: null,
    error: null,
    ...initialState,
  });

  React.useEffect(() => {
    const promise = asyncCallback();
    if (!promise) {
      return;
    }
    dispatch({type: 'pending'});
    promise.then(
      data => {
        dispatch({type: 'resolved', data});
      },
      error => {
        dispatch({type: 'rejected', error});
      },
    );
  }, [asyncCallback]);

  return state;
}

function MyComponent() {
  // Before the problem is fetchAPI function is going to be redefined on every render. That means that every render that's used useAsync is going to call this function, even if the dependencie did not change. That could absolutely be a problem.
  // That is exactly what useCallback does. React is going to make sure that this callback that we get assigned to async callback will be the same one that we've created in the first place until the dependencie changes.
  const asyncCallback = React.useCallback(() => {
    return fetchAPI(dependencie);
  }, [dependencie]);
  
  const state = useAsync(
    asyncCallback,
    {prop1: initialValue1, prop2: initialValue2},
  );
}
```

### useAsync custom hook (improved)
```javascript
function asyncReducer(state, action) {
  switch (action.type) {
    case 'pending': {
      return {status: 'pending', data: null, error: null};
    }
    case 'resolved': {
      return {status: 'resolved', data: action.data, error: null};
    }
    case 'rejected': {
      return {status: 'rejected', data: null, error: action.error};
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

function useAsync(initialState) {
  const [state, dispatch] = React.useReducer(asyncReducer, {
    status: 'idle',
    data: null,
    error: null,
    ...initialState,
  });

  // Dispatch basically sends the type of action to the reducer function to perform its job, which, of course, is updating the state.
  // It means that the hook is going to render again, and if the run function is not memoized so it gonna render in a loop.
  // So, we need to use useCallback
  const run = React.useCallback(promise => {
    if (!promise) {
      return;
    }
    dispatch({type: 'pending'});
    promise.then(
      data => {
        dispatch({type: 'resolved', data});
      },
      error => {
        dispatch({type: 'rejected', error});
      },
    );
    // I'm not getting any weird underlines for this. We are using this dispatch, but that is coming from useReducer, and ESLint plugin knows that this dispatch will never change. useReducer ensures that for us, we'll never get a new dispatch function, even as these functions are re-rendering, so we don't need to include dispatch in there
  }, []);

  return {...state, run};
}

function MyComponent() {
  // Providing a run function that we are responsible for memoizing.
  const {data, status, error, run} = useAsync({prop1: initialValue1, prop2: initialValue2});

  React.useEffect(() => {
    run(fetchAPI(dependencie));
  }, [dependencie, run]);
}
```

### Error Boundaries
In the past, JavaScript errors inside components used to corrupt React’s internal state and cause it to emit cryptic errors on next renders. These errors were always caused by an earlier error in the application code, but React did not provide a way to handle them gracefully in components, and could not recover from them.

A JavaScript error in a part of the UI shouldn’t break the whole app. To solve this problem for React users, React 16 introduces a new concept of an “error boundary”.

Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

There’s an npm package we can use. It’s called [react-error-boundary](https://github.com/bvaughn/react-error-boundary). This component provides a simple and reusable wrapper that you can use to wrap around your components. Any rendering errors in your components hierarchy can then be gracefully handled.

Reading [this blog post](https://kentcdodds.com/blog/use-react-error-boundary-to-handle-errors-in-react) will help you understand what react-error-boundary does for you.

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({error, resetErrorBoundary}) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  const [data, setData] = React.useState('');

  // ErrorBoundary will call onReset when it is reset.
  function handleReset() {
    // reset the state of your app so the error doesn't happen again
    setData('');
  }

  // react-error-boundary supports this with the resetKeys prop.
  // You pass an array of values to resetKeys and if the ErrorBoundary is in an
  // error state and any of those values change, it will reset the error boundary.
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={handleReset}
      resetKeys={[data]}
    >
      <ComponentThatMayError />
    </ErrorBoundary>
  );
}
```

## Add-Ons

* [Closure](https://whatthefork.is/closure)
