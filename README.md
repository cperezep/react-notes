# React

- [`React Hooks`](#reactHooks)
- [`React Patterns`](#reactPatterns)
- [`React Testing`](#reactTesting)

## React Hooks <a id="reactHooks"></a>

Normally an interactive application will need to hold state somewhere. In React, you use special functions called "hooks" to do this. Common built-in hooks include:

- [`React.useState`](#useState)
- [`React.useEffect`](#useEffect)
- [`React.useRef`](#useRef)
- [`React.useReducer`](#useReducer)
- [`React.useCallback`](#useCallback)
- [`React.useContext`](#useContext)
- [`React.useLayoutEffect`](#useLayoutEffect)
- [`React.useDebugValue`](#useDebugValue)

Each of these is a special function that you can call inside your custom React component function to store data (like state) or perform actions (or side-effects).


Each of the hooks has a unique API. Some return a value (like `React.useRef` and `React.useContext`), others return a pair of values (like `React.useState` and `React.useReducer`), and others return nothing at all (like `React.useEffect`).

### React.useState <a id="useState"></a>
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

### React.useEffect <a id="useEffect"></a>
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

### React.useRef <a id="useRef"></a>
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

### React.useReducer <a id="useReducer"></a>
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

### React.useCallback <a id="useCallback"></a>
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

### React.useContext <a id="useContext"></a>

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
6. You may notice that the context provider/consumers in React DevTools just display as `Context.Provider` and `Context.Consumer`. That doesn't do a good job differentiating itself from other contexts that may be in your app. Luckily, you can set the context `displayName` and it'll display that name for
the `Provider` and `Consumer`. Hopefully in the future this will happen automatically ([learn more](https://github.com/babel/babel/issues/11241)).
```javascript
const MyContext = React.createContext()
MyContext.displayName = 'MyContext'
```


### `React.useLayoutEffect` <a id="useLayoutEffect"></a>

`useEffect` and `useLayoutEffect` both of these can be used to do basically the same thing, but they have slightly different use cases. So here are some rules for you to consider when deciding which React Hook to use.

* `useEffect`: 99% of the time this is what you want to use. **The one catch** is that this runs after react renders your component and ensures that your effect callback does not block browser painting.

  However, if your effect is mutating the DOM (via a DOM node ref) and the DOM mutation will change the appearance of the DOM node between the time that it is rendered and your effect mutates it, then you don't want to use useEffect. You'll want to use useLayoutEffect. Otherwise the user could see a flicker when your DOM mutations take effect. **This is pretty much the only time you want to avoid useEffect and use useLayoutEffect instead**.

* `useLayoutEffect`: This runs synchronously immediately after React has performed all DOM mutations. This can be useful if you need to make DOM measurements (like getting the scroll position or other styles for an element) and then make DOM mutations or trigger a synchronous re-render by updating state.

  Your code runs immediately after the DOM has been updated, but before the browser has had a chance to "paint" those changes (the user doesn't actually see the updates until after the browser has repainted).

#### Example
We have a chatbox and when you click on `add message`, we automatically scroll to the bottom from chatbox. When you remove a message, then we automatically scroll you to the bottom. That way, we keep the user at the latest message of messages that we have in this little chatbox.

For example:
```javascript
function MessagesDisplay() {
  const containerRef = React.useRef();

  React.useEffect(() => {
    containerRef.current.scrollTop = containerRef.current.scrollHeight;
  });

  return (
    <div ref={containerRef} role="log">
      <ListMessages />
    </div>
  );
}
```

If we use `useEffect`, you'll also notice that the message shows up before the scroll happens. That's not a super-great experience for the users of our app.

Let's switch this over to useLayoutEffect, but we don't see the message appear before we scroll. That scroll happens at the same time the message appears. That's exactly the behavior that we want, and here's the real difference.

When we use layout effects, we can change stuff in the DOM before the browser has a chance to paint the screen. The browser only has to paint the screen once, and the user only will see that update that one time. Anytime you have a side effect that's going to manipulate the DOM in a way that's observable to the user, you want to do that in a useLayoutEffect.

To bring it down to a rule, you use useEffect almost all of the time, and you useLayoutEffect if the side effect that you are performing makes an observable change to the DOM that will require the browser to paint that update that you've made.

#### Summary

* `useLayoutEffect:` If you need to mutate the DOM and/or do need to perform measurements
* `useEffect:` If you don't need to interact with the DOM at all or your DOM changes are unobservable (seriously, most of the time you should use this).

#### Conclusion

It's all about defaults. The default behavior is to let the browser re-paint based on DOM updates before React runs your code. This means your code won't block the browser and the user sees updates to the DOM sooner. So stick with useEffect most of the time.

### `React.useDebugValue` <a id="useDebugValue"></a>

When you start writing custom hooks, it can be useful to give them a special label. This is especially useful to differentiate different usages of the same hook in a given component when you inspect it in React DevTools Browser Extension.

This is where `useDebugValue` comes in. You use it in a custom hook, and you call it like so:

```javascript
function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step})
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + step)
  return [count, increment]
}
```

So now when people use the `useCount` hook, they'll see the `initialCount` and `step` values for that particular hook. That gives us a bit of nicer experience when using the dev tools for our custom hooks.

As a reminder, this would not work inside a component. This is just to label custom hooks.

#### Second Parameter
`useDebugValue` also takes a second argument which is an optional formatter function, allowing you to do stuff like this if you like:

```javascript
const formatCountDebugValue = ({initialCount, step}) =>
  `init: ${initialCount}; step: ${step}`;

function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step}, formatCountDebugValue);
  const [count, setCount] = React.useState(0);
  const increment = () => setCount(c => c + step);
  return [count, increment];
}
```

The reason this exists is that let's assume that calculating this DebugValue was computationally expensive for one reason or another. We don't want to call that function and execute that computation if our users don't need that.

The only users who need this label are those developer users who are using this in their development workflow. The end-users will never have the DevTools open. They'll never see this DebugValue. Making this computation for that DebugValue is a waste of their resources.

If you ever have a computationally expensive value to use with your useDebugValue, then you pass all of the things that you need for calculating that value as a first argument. Then the second argument is a function that will accept those values and return the DebugValue you want to be assigned.


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

# React Patterns <a id="reactPatterns"></a>

- [`Context Module Functions`](#contextModuleFunctions)
- [`Compound Components`](#compoundComponents)
- [`Flexible Compound Components`](#flexibleCompoundComponents)
- [`Prop Collections and Getters`](#propCollectionsGetters)
- [`State Reducer`](#stateReducer)
- [`Control Props`](#controlProps)

## Context Module Functions <a id="contextModuleFunctions"></a>

Let's take a look at an example of a simple context and a reducer combo:

```javascript
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]
  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

export {CounterProvider, useCounter}
```

```javascript
// src/screens/counter.js
import {useCounter} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  // Take this logic and move it into one of these context of module functions, so you can simplify the code for the user of context.
  const increment = () => dispatch({type: 'increment'})
  const decrement = () => dispatch({type: 'decrement'})
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </div>
  )
}
```

```javascript
// src/index.js
import {CounterProvider} from 'context/counter'

function App() {
  return (
    <CounterProvider>
      <Counter />
    </CounterProvider>
  )
}
```

I want to focus in on the user of our reducer (the `Counter` component). Notice that they have to create their own `increment` and `decrement` functions which call `dispatch`. I don't think that's a super great API. It becomes even more of an annoyance when you have a sequence of `dispatch` functions that need to be called.

The first inclination is to create "helper" functions and include them in the context. You'll notice that we HAVE to put it in `React.useCallback` so we can list our "helper" functions in dependency lists:

```javascript
const increment = React.useCallback(() => dispatch({type: 'increment'}), [
  dispatch,
])
const decrement = React.useCallback(() => dispatch({type: 'decrement'}), [
  dispatch,
])
const value = {state, increment, decrement}
return <CounterContext.Provider value={value} {...props} />

// now users can consume it like this:

const {state, increment, decrement} = useCounter()
```

This isn't a _bad_ solution necessarily. But [as Dan Abramov says](https://twitter.com/dan_abramov/status/1125758606765383680):

> Helper methods are object junk that we need to recreate and compare for no
> purpose other than superficially nicer looking syntax.

What Dan recommends (and what Facebook does) is pass dispatch as we had originally. And to solve the annoyance we were trying to solve in the first place, they use importable "helpers" that accept `dispatch`. Let's take a look at how that would look:

```javascript
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]

  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

const increment = dispatch => dispatch({type: 'increment'})
const decrement = dispatch => dispatch({type: 'decrement'})

export {CounterProvider, useCounter, increment, decrement}
```

```javascript
// src/screens/counter.js
import {useCounter, increment, decrement} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={() => decrement(dispatch)}>-</button>
      <button onClick={() => increment(dispatch)}>+</button>
    </div>
  )
}
```

In some situations this pattern can not only help you reduce duplication, but it also [helps improve performance](https://twitter.com/dan_abramov/status/1125774170154065920) and helps you avoid mistakes in dependency lists.

The benefit of doing things this way is that when we have multiple dispatches that we're going to be calling, if we just leave that up to the user of our context, it's possible that they might miss a dispatch call. It's better to pass that user dispatch to this context module function so it can ensure that we're calling the dispatch in the right order.

A common thing that people will do instead is they'll put the "helpers" inside of their consuming hook or maybe even inside of the value, but then you have to worry about memorizing a whole bunch of stuff to make sure that you can use these functions inside of a useEffect dependency list or a useCallback dependency list. That can have a spidering effect across your entire code base.

In situations where you have multiple dispatch calls that you need to make for asynchronous UI updates like this, you can make a context module function that accepts that dispatch, as well as anything else that's needed, and that can manage to call dispatch correctly.

```javascript
// Helper function
async function updateUser(dispatch, user, updates) {
  dispatch({type: 'start update', updates});
  try {
    const updatedUser = await userClient.updateUser(user, updates);
    dispatch({type: 'finish update', updatedUser});
    return updatedUser;
  } catch (error) {
    dispatch({type: 'fail update', error});
    throw error;
  }
}
```

## Compound Components <a id="compoundComponents"></a>
Compound components are components that work together to form a complete UI. The classic example of this is `<select>` and `<option>` in HTML:

```html
<select>
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
</select>
```

The `<select>` is the element responsible for managing the state of the UI, and the `<option>` elements are essentially more configuration for how the select should operate (specifically, which options are available and their values).

Let's imagine that we were going to implement this native control manually. A naive implementation would look something like this:

```jsx
<CustomSelect
  options={[
    {value: '1', display: 'Option 1'},
    {value: '2', display: 'Option 2'},
  ]}
/>
```

This works fine, but it's less extensible/flexible than a compound components API. For example. What if I want to supply additional attributes on the `<option>` that's rendered, or I want the `display` to change based on whether it's selected? We can easily add API surface area to support these use cases,
but that's just more for us to code and more for users to learn. That's where compound components come in really handy!

**Real World Projects that use this pattern:**

- [`@reach/tabs`](https://reacttraining.com/reach-ui/tabs)

**Example**

We have a Toggle component that manages the state, and we want to render different parts of the UI however we want control over the presentation of the UI.
```javascript
import * as React from 'react';
import {Switch} from '../switch';

function Toggle({children}) {
  const [on, setOn] = React.useState(false);
  const toggle = () => setOn(!on);

  // React.Children.map: It works especially for React.Children because the children prop can be a single element, or it can be an array of elements.
  return React.Children.map(children, child => {
    return React.cloneElement(child, {on, toggle});
  });
}

/** <ToggleOn /> renders children when the on state is true */
const ToggleOn = ({on, children}) => (on ? children : null);

/** <ToggleOff /> renders children when the on state is false */
const ToggleOff = ({on, children}) => (on ? null : children);

/** <ToggleButton /> renders the <Switch /> with the on prop set to the on state and the onClick prop set to toggle. */
const ToggleButton = ({on, toggle}) => <Switch on={on} onClick={toggle} />;

function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <ToggleButton />
      </Toggle>
    </div>
  );
}

export default App;
```
The fundamental challenge face with an API like this is the state shared between the components is implicit, meaning that the developer using your component cannot actually see or interact with the state (on) or the mechanisms for updating that state (toggle) that are being shared between the components.

## Flexible Compound Components <a id="flexibleCompoundComponents"></a>
The last component can only clone and pass props to immediate children. So if we need some way for the compound components to implicitly accept the on state and toggle method regardless of where they're rendered within the Toggle
component's "posterity".

The way we do this is through context `React.createContext`.

**Real World Projects that use this pattern:**

- [`@reach/accordion`](https://reacttraining.com/reach-ui/accordion)

**Example**
```javascript
import * as React from 'react';
import {Switch} from '../switch';

const ToggleContext = React.createContext();
ToggleContext.displayName = 'ToggleContext';

function Toggle({children}) {
  const [on, setOn] = React.useState(false);
  const toggle = () => setOn(!on);

  const value = {on, toggle};
  return (
    <ToggleContext.Provider value={value}>{children}</ToggleContext.Provider>
  );
}

function useToggle() {
  const context = React.useContext(ToggleContext);
  if (!context) {
    throw new Error(`useToggle must be within a <Toggle />`);
  }
  return context;
}

function ToggleOn({children}) {
  const {on} = useToggle();
  return on ? children : null;
}

function ToggleOff({children}) {
  const {on} = useToggle();
  return on ? null : children;
}

function ToggleButton(props) {
  const {on, toggle} = useToggle();
  return <Switch on={on} onClick={toggle} {...props} />;
}

/* The fundamental difference between this code and the last one is that now we're going to allow people to render the compound components wherever they like in the render tree. */
function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <div>
          <ToggleButton />
        </div>
      </Toggle>
    </div>
  );
}

export default App;
```

## Prop Collections and Getters <a id="propCollectionsGetters"></a>
The Props Getters Pattern instead of exposing native props, we provide a shortlist of props getters. A getter is a function that returns many props, it has a meaningful name allowing the user to naturally link it to the right JSX element.

Lots of the reusable/flexible components and hooks that we'll create have some common use-cases and it'd be cool if we could make it easier to use our components and hooks the right way without requiring people to wire things up for common use cases.

In typical UI components, you need to take accessibility into account. For a button functioning as a toggle, it should have the `aria-pressed` attribute set to `true` or `false` if it's toggled on or off.

**Example**
```javascript
function useToggle() {
  const [on, setOn] = React.useState(false);
  const toggle = () => setOn(!on);

  const togglerProps = {
    'aria-pressed': on,
    onClick: toggle,
  };
  return {on, toggle, togglerProps};
}

function App() {
  const {on, togglerProps} = useToggle();
  return (
    <div>
      <Switch on={on} {...togglerProps} />
      <hr />
      <button aria-label="custom-button" {...togglerProps}>
        {on ? 'on' : 'off'}
      </button>
    </div>
  );
}
```

But, what happen if someone wants to use our `togglerProps` object, but they need to apply their own `onClick` handler? So that instead of having an object of props, we call a function to get the props. Then we can pass that function the props we want applied and that function will be responsible for composing the props together.

**Example**
```javascript
// useToggle.js
function useToggle() {
  const [on, setOn] = React.useState(false);
  const toggle = () => setOn(!on);

  // Destructure this so that I can grab the onClick if it's provided,
  // and then we'll take the rest of the props
  function getTogglerProps({onClick, ...props} = {}) {
    return {
      'aria-pressed': on,
      onClick: () => {
        onClick && onClick();
        toggle();
      },
      ...props,
    };
  }

  // That's actually what we would recommend. We don't typically use prop collections.
  // I defer to prop getters because they're more flexible in that they allow me to
  // compose things together.
  return {on, toggle, getTogglerProps};
}
```

```javascript
// Usage
function App() {
  const {on, getTogglerProps} = useToggle()
  return (
    <div>
      <Switch {...getTogglerProps({on})} />
      <hr />
      <button
        {...getTogglerProps({
          'aria-label': 'custom-button',
          onClick: () => console.info('onButtonClick'),
          id: 'custom-button-id',
        })}
      >
        {on ? 'on' : 'off'}
      </button>
    </div>
  )
}
```

Let's going to improve the previous hook making this fancy function called callAll(). This is going to take any number of functions, and then it's going to return a function that takes any number of arguments. We don't care what those arguments are. We'll take those functions. For each of those, we'll take that function. If that function exists, then we'll call that function with all the args. Basically, it's a function that We can call passing any number of functions that will return a function that calls all those functions.

```javascript
function callAll(...fns) {
  return (...args) => {
    fns.forEach(fn => {
      fn && fn(...args);
    });
  };
}

function useToggle() {
  const [on, setOn] = React.useState(false);
  const toggle = () => setOn(!on);

  function getTogglerProps({onClick, ...props} = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    };
  }

  return {on, toggle, getTogglerProps};
}
```

**Real World Projects that use this pattern:**

- [downshift](https://github.com/downshift-js/downshift) (uses prop getters)
- [react-table](https://github.com/tannerlinsley/react-table) (uses prop
  getters)
- [`@reach/tooltip`](https://reacttraining.com/reach-ui/tooltip) (uses prop
  collections)

## State Reducer <a id="stateReducer"></a>
The benefit of the state reducer pattern is in the fact that it allows [Inversion of Control](https://kentcdodds.com/blog/inversion-of-control) which is basically a mechanism for the author of the API to allow the user of the API to control how things work internally.

By inverting control of state updates with the state reducer pattern, I was able to enable their use case as well as any other use case people could possibly want when they want to change how it operates internally. Inversion of control is an enabling computer science principle and the state reducer pattern is an awesome implementation of that idea that translates even better to hooks than it did to regular components.

### Using a State Reducer with Hooks

The concept goes like this:

1. End user does an action
2. Dev calls dispatch
3. Hook determines the necessary changes
4. Hook calls dev's code for further changes 👈  this is the inversion of control part
5. Hook makes the state changes

**Example 1**
```javascript
// useCounter.js
const actionTypes = {
  increment: 'INCREMENT',
  decrement: 'DECREMENT',
}

function internalReducer({ count }, { type, payload }) {
  switch (type) {
    case actionTypes.increment:
      return {
        count: Math.min(count + 1, payload.max)
      };
    case actionTypes.decrement:
      return {
        count: Math.max(0, count - 1)
      };
    default:
      throw new Error(`Unhandled action type: ${type}`);
  }
};

// Custom Hook Pattern
/** This hook is accessible by the user and exposes several internal logics (States, Handlers), allowing him to have better control over your component */
function useCounter({ initial, max }, reducer = internalReducer) {
  const [{ count }, dispatch] = useReducer(reducer, { count: initial });

  const handleIncrement = () => {
    dispatch({ type: actionTypes.increment, payload: { max } });
  };

  const handleDecrement = () => {
    dispatch({ actionTypes.decrement });
  };

  return {
    count,
    handleIncrement,
    handleDecrement
  };
}

useCounter.reducer = internalReducer;


export { useCounter, actionTypes }
```


```javascript
// Usage
import { useCounter, actionTypes } from "./useCounter";

function Counter() {
  // Custom reducer
  const reducer = (state, action) => {
    switch (action.type) {
      case actionTypes.decrement:
        return {
          count: Math.max(0, state.count - 2) // The decrement delta was changed for 2 (Default is 1)
        };
      default:
        return useCounter.reducer(state, action);
    }
  };

  const { count, handleIncrement, handleDecrement } = useCounter(
    { initial: 0, max: 10 },
    reducer
  );

  return (
    <div>
      <button onClick={handleDecrement}>Decrement</button>
      <div>{count}</div>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  )
}

function App() {
  return <Counter />
}

ReactDOM.render(<App />, document.getElementById('root'))
```

**Example 2**
```javascript
// useToggle.js
const actionTypes = {
  toggle: 'TOGGLE',
  reset: 'RESET',
};

function toggleReducer(state, {type, initialState}) {
  switch (type) {
    case actionTypes.toggle: {
      return {on: !state.on};
    }
    case actionTypes.reset: {
      return initialState;
    }
    default: {
      throw new Error(`Unsupported type: ${type}`);
    }
  }
}

function useToggle({initialOn = false, reducer = toggleReducer} = {}) {
  const {current: initialState} = React.useRef({on: initialOn});
  const [state, dispatch] = React.useReducer(reducer, initialState);
  const {on} = state;

  const toggle = () => dispatch({type: actionTypes.toggle});
  const reset = () => dispatch({type: actionTypes.reset, initialState});

  return {on, reset, toggle};
}

export {useToggle, toggleReducer, actionTypes};
```

```javascript
// Usage
import {useToggle, toggleReducer, actionTypes} from './useToggle';

function App() {
  const [timesClicked, setTimesClicked] = React.useState(0);
  const clickedTooMuch = timesClicked >= 4;

  // "If the action.type is 'toggle' AND I clicked 4 times, then I can return this right here."
  // Otherwise, I'll return the toggleReducer with that state and that action.
  function toggleStateReducer(state, action) {
    if (action.type === actionTypes.toggle && timesClicked >= 4) {
      return {on: state.on};
    }
    return toggleReducer(state, action);
  }

  const {on, toggle, reset} = useToggle({
    reducer: toggleStateReducer,
  });

  return (
    <div>
      <Switch
        onClick={() => {
          toggle();
          setTimesClicked(count => count + 1);
        }}
        on={on}
      />
      {clickedTooMuch ? (
        <div data-testid="notice">
          Whoa, you clicked too much!
          <br />
        </div>
      ) : timesClicked > 0 ? (
        <div data-testid="click-count">Click count: {timesClicked}</div>
      ) : null}
      <button
        onClick={() => {
          reset();
          setTimesClicked(0);
        }}
      >
        Reset
      </button>
    </div>
  );
}

export default App;
```

The most advanced pattern in terms of inversion of control. It gives an advanced way for the user to change how your component operates internally.

The code is similar to `Custom Hook Pattern`, but in addition the user defines a reducer which is passed to the hook. This reducer will overload any internal action of your component.
This makes our hook WAY more flexible, but it also means that the way we update state is now part of the API and if we make changes to how that happens, then it could be a breaking change for users.
All your internal component’s actions are now accessible from the outside and can be overridden. It's totally worth the trade-off for complex hooks/components, but it's just good to keep that in mind.

**Source**
* [The State Reducer Pattern with React Hooks](https://kentcdodds.com/blog/the-state-reducer-pattern-with-react-hooks)
* [5 Advanced React Patterns](https://javascript.plainenglish.io/5-advanced-react-patterns-a6b7624267a6)

## Control Props <a id="controlProps"></a>
Sometimes, people want to be able to manage the internal state of our component from the outside. The state reducer allows them to manage what state changes are made when a state change happens, but sometimes people may want to make state changes themselves. We can allow them to do this with a feature called "Control Props."

This pattern transforms your component into a [controlled component](https://reactjs.org/docs/forms.html#controlled-components). An external state is consumed as a “single source of truth” allowing the user to insert custom logic that will modify the default component behavior.

```javascript
function MyCapitalizedInput() {
  const [capitalizedValue, setCapitalizedValue] = React.useState('')

  return (
    <input
      value={capitalizedValue}
      onChange={e => setCapitalizedValue(e.target.value.toUpperCase())}
    />
  )
}
```

In this case, the "component" that's implemented the "control props" pattern is the `<input />`. Normally it controls state itself (like if you render `<input />` by itself with no `value` prop). But once you add the `value` prop, suddenly the `<input />` takes the back seat and instead makes "suggestions" to you via the `onChange` prop on the state updates that it would normally make itself.

This flexibility allows us to change how the state is managed and it also allows us to programmatically change the state whenever we want to.

**Example 1**
```javascript
// Counter.js
function Counter({ value = null, initialValue = 0, onChange }) {
  const [count, setCount] = useState(initialValue);

  const isControlled = value !== null && !!onChange;

  const getCount = () => (isControlled ? value : count);

  const handleCountChange = (newValue) => {
    isControlled ? onChange(newValue) : setCount(newValue);
  };

  const handleIncrement = () => {
    handleCountChange(getCount() + 1);
  };

  const handleDecrement = () => {
    handleCountChange(Math.max(0, getCount() - 1));
  };

  return (
    <div>
      <button onClick={handleDecrement}>Decrement</button>
      <div>{getCount()}</div>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}

export { Counter };
```

```javascript
// Usage
import { Counter } from "./Counter";

function App() {
  const [count, setCount] = useState(0);

  const handleChangeCounter = (newCount) => {
    setCount(newCount);
  };
  return (
    <Counter value={count} onChange={handleChangeCounter} />
  );
}

ReactDOM.render(<App />, document.getElementById('root'));
```

**Example 2**
```javascript
// useToggle.js
const actionTypes = {
  toggle: 'toggle',
  reset: 'reset',
};

function toggleReducer(state, {type, initialState}) {
  switch (type) {
    case actionTypes.toggle: {
      return {on: !state.on};
    }
    case actionTypes.reset: {
      return initialState;
    }
    default: {
      throw new Error(`Unsupported type: ${type}`);
    }
  }
}

function useToggle({
  initialOn = false,
  reducer = toggleReducer,
  onChange,
  on: controlledOn,
} = {}) {
  const {current: initialState} = React.useRef({on: initialOn});
  const [state, dispatch] = React.useReducer(reducer, initialState);

  const isControlled = controlledOn != null;
  const on = isControlled ? controlledOn : state.on;

  function dispatchWithOnChange(action) {
    if (!isControlled) {
      dispatch(action);
    }
    // onChange only is executed it's provided
    onChange?.(reducer({...state, on}, action), action);
  }

  const toggle = () => dispatchWithOnChange({type: actionTypes.toggle});
  const reset = () => dispatchWithOnChange({type: actionTypes.reset, initialState});

  return { on, reset, toggle };
}

export {useToggle, toggleReducer, actionTypes};
```

```javascript
import { useToggle } from './useToggle';

// Toggle.js
function Toggle({on: controlledOn, onChange}) {
  const {on, toggle} = useToggle({on: controlledOn, onChange});

  return <Switch onClick={toggle} on={on} />;
}

export { Toggle };
```

```javascript
// Usage
import { actionTypes } from './useToggle';
import { Toggle } from './Toggle';

function App() {
  const [bothOn, setBothOn] = React.useState(false);
  const [timesClicked, setTimesClicked] = React.useState(0);

  function handleToggleChange(state, action) {
    if (action.type === actionTypes.toggle && timesClicked > 4) {
      return;
    }
    setBothOn(state.on);
    setTimesClicked(c => c + 1);
  }

  function handleResetClick() {
    setBothOn(false);
    setTimesClicked(0);
  }

  return (
    <div>
      <div>
        <Toggle on={bothOn} onChange={handleToggleChange} />
        <Toggle on={bothOn} onChange={handleToggleChange} />
      </div>
      {timesClicked > 4 ? (
        <div data-testid="notice">
          Whoa, you clicked too much!
          <br />
        </div>
      ) : (
        <div data-testid="click-count">Click count: {timesClicked}</div>
      )}
      <button onClick={handleResetClick}>Reset</button>
      <hr />
      <div>
        <div>Uncontrolled Toggle:</div>
        <Toggle />
      </div>
    </div>
  );
}

export default App;
```

**Real World Projects that use this pattern:**

- [downshift](https://github.com/downshift-js/downshift)
- [`@reach/listbox`](https://reacttraining.com/reach-ui/listbox)


# React Testing <a id="reactTesting"></a>

Everything we do with testing our React components is walking the line of trade-offs of getting our tests to resemble the way our software is actually used and having something that's reasonably possible for testing.

When we think about how things are used, we need to consider who the users are:

1. The end user that's interacting with our code (clicking buttons/etc)
2. The developer user that's actually using our code (rendering it, calling our functions, etc.)

Often a _third_ user creeps into our tests and we want to avoid them as much as possible: [The Test User](https://kentcdodds.com/blog/avoid-the-test-user).

When it comes to React components, our developer user will render our component with `ReactDOM` and in some cases they'll pass props and/or wrap it in a context provider. The end user will click buttons
and assert on the output.

## *Example*
We have a simple counter component. The test will sure that it starts out saying "Current count: 0" and that when the user clicks "Increment" it'll increase the count and when they click "Decrement" it'll
decrease the count.

To do this, we'll need to create a DOM node, add it to the body, and render the component to that DOM node. You'll also need to clean up the DOM when your test is finished so the next test has a clean DOM to interact with.
```typescript
import ReactDOM from 'react-dom';
import Counter from '../../components/counter';

beforeEach(() => {
  // cleaning up your environment between each one of your tests, so that your tests can run in total isolation of each other.
  document.body.innerHTML = '';
});

test('counter increments and decrements when the buttons are clicked', () => {
  const div = document.createElement('div');
  document.body.append(div);

  ReactDOM.render(<Counter />, div);
  console.log(document.body.innerHTML);

  // TypeScript doesn't trust the DOM very much, so you'll need to verify
  // things are what they should be to make TypeScript happy here.
  const [decrement, increment] = Array.from(div.querySelectorAll('button'));
  if (!decrement || !increment) {
    throw new Error('decrement and increment not found');
  }
  if (!(div.firstChild instanceof HTMLElement)) {
    throw new Error('first child is not a div');
  }

  const message = div.firstChild.querySelector('div');
  if (!message) {
    throw new Error(`couldn't find message div`);
  }

  console.log(message.textContent);
  expect(message.textContent).toBe('Current count: 0');

  increment.click();
  expect(message.textContent).toBe('Current count: 1');

  decrement.click();
  expect(message.textContent).toBe('Current count: 0');

  // clean up the DOM when the test is finished so the next test has a clean DOM to interact with
  // If you don't cleanup, then it could impact other tests and/or cause a memory leak
  // If the previous expect fails this will throws an error, meaning this next line of code does not run. Then is better approach if you clean DOM in the beforeEach method
  // div.remove();
});
```

### *Using a dispatchEvent*
  Using `.click` on a DOM node works fine, but what if you wanted to fire an event that doesn't have a dedicated method (like mouseover). Rather than use `button.click()`, try using `button.dispatchEvent`: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent

```typescript
test('counter increments and decrements when the buttons are clicked', () => {
  ...

  // Using the mouseEvent constructor, and the dispatchEvent API, this aligns our test a little bit closer to the user's experience when they're actually using our component.
  const incrementClickEvent = new MouseEvent('click', {
    // That means that it will bubble up, which is important because React uses event delegation, and a bubbling is required for event delegation to work.
    bubbles: true,
    // Cancelable because that's how the event is going to be by default when the user clicks on the button
    cancelable: true,
    // Which turns this into a left-click.
    button: 0,
  });

  increment.dispatchEvent(incrementClickEvent);
  expect(message.textContent).toBe('Current count: 1');

  const decrementClickEvent = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    button: 0,
  });

  decrement.dispatchEvent(decrementClickEvent);
  expect(message.textContent).toBe('Current count: 0');

  ...
})
```

## Testing Library
[React Testing Library](https:/testing-library.com/react) is the React implementation of the [DOM Testing Library](https://testing-library.com). Testing Library comes with a ton of really useful features.

Here's a simple example of how to use this:

```javascript
import {render, fireEvent, screen} from '@testing-library/react'

test('it works', () => {
  const {container} = render(<Example />)
  // container is the div that your component has been mounted onto.

  const input = container.querySelector('input')
  fireEvent.mouseEnter(input) // fires a mouseEnter event on the input

  screen.debug() // logs the current state of the DOM (with syntax highlighting!)
})
```

Notice the lack of `cleanup` functionality. That's thanks to
`@testing-library/react`'s [auto-cleanup feature](https://testing-library.com/docs/react-testing-library/api#cleanup)

Another automatic feature of React Testing Library is its handling of [React's `act` function](https://reactjs.org/docs/test-utils.html#act). If you've ever seen a warning about something not being wrapped in `act`, that's what we're talking about. As mentioned in the React docs, React Testing Library is recommended for avoiding the issues `act` is warning you about. You can learn more about this from my blog post [Fix the "not wrapped in act(...)" warning](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning)

```typescript
import {render, fireEvent} from '@testing-library/react';
import Counter from '../../components/counter';

// That cleanup is going to be managed for us by React Testing Library because we're using render.
/* beforeEach(() => {
  document.body.innerHTML = ''
}) */

test('counter increments and decrements when the buttons are clicked', () => {
  // const div = document.createElement('div')
  // document.body.append(div)
  // ReactDOM.render(<Counter />, div);

  // Note that React Testing Library's render doesn't need you to pass a `div`
  // so you only need to pass one argument. render returns an object with a
  // bunch of utilities on it. For now, let's just grab `container` which is
  // the div that React Testing Library creates for us.
  const {container} = render(<Counter />);

  // const [decrement, increment] = div.querySelectorAll('button');
  // const message = div.firstChild.querySelector('div');
  const [decrement, increment] = Array.from(
    container.querySelectorAll('button'),
  );
  if (!decrement || !increment) {
    throw new Error('decrement and increment not found');
  }
  if (!(container.firstChild instanceof HTMLElement)) {
    throw new Error('first child is not a div');
  }

  const message = container.firstChild.querySelector('div');
  if (!message) {
    throw new Error(`couldn't find message div`);
  }

  expect(message.textContent).toBe('Current count: 0');

  /* const incrementClickEvent = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    button: 0,
  });
  increment.dispatchEvent(incrementClickEvent); */
  fireEvent.click(increment);
  expect(message.textContent).toBe('Current count: 1');

  /* const decrementClickEvent = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    button: 0,
  });
  decrement.dispatchEvent(decrementClickEvent); */
  fireEvent.click(decrement);
  expect(message.textContent).toBe('Current count: 0');
});
```

### Avoid Implementation Details
One of the most important things to remember about testing our software the way it is used is to avoid testing implementation details. "Implementation details" is a term referring to how an abstraction accomplishes a certain outcome. Thanks to the expressiveness of code, you can typically accomplish the same outcome using completely different implementation details.

The implementation of your abstractions does not matter to the users of your abstraction and if you want to have confidence that it continues to work through refactors then **neither should your tests.**

Here's a React example of this:

```javascript
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}
```

Here's one way you might access that `button` to click and assert on it:

```javascript
const {container} = render(<Counter />)
container.firstChild // <-- that's the button
```

However, what if we changed it a bit:

```javascript
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <span>
      <button onClick={increment}>{count}</button>
    </span>
  )
}
```

Our tests would break!

The only difference between these implementations is one wraps the button in a `span` and the other does not. The user does not observe or care about this difference, so we should write our tests in a way that passes in either case.

So here's a better way to search for that button in our test that's implementation detail free and refactor friendly:

```javascript
render(<Counter />)
screen.getByText('0') // <-- that's the button
// or (even better) you can do this:
screen.getByRole('button', {name: '0'}) // <-- that's the button
```

```typescript
import {render, screen, fireEvent} from '@testing-library/react';
import Counter from '../../components/counter';

test('counter increments and decrements when the buttons are clicked', () => {
  render(<Counter />);
  const decrement = screen.getByRole('button', {name: /decrement/i});
  const increment = screen.getByRole('button', {name: /increment/i});
  const message = screen.getByText(/current count/i);

  // Notice that React Testing Library queries do null-checks for you automatically, so you can ditch all this null-checking stuff.
  expect(message).toHaveTextContent('Current count: 0');

  fireEvent.click(increment);
  expect(message).toHaveTextContent('Current count: 1');

  fireEvent.click(decrement);
  expect(message).toHaveTextContent('Current count: 0');
});
```

There are several different queries that you can use. [Here](https://testing-library.com/docs/queries/about#priority) in the documentation, we described which one of these you should focus on first because you can get the same elements with different kinds of queries.

There's also a cool site called [testing-playground.com](https://testing-playground.com/). Where you can actually put in some HTML and it will render that out over here and give you what query you should be using for that particular element.

**Read more about**
* [Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details)
* [Avoid the Test User](https://kentcdodds.com/blog/avoid-the-test-user)

### Using @testing-library/jest-dom
Testing Library also has a suite of assertions that can be installed with Jest. They're already added to this project, so you can switch from Jest's built-in assertions to more specific assertions which will give you better error messages. [`@testing-library/jest-dom`](http://testing-library.com/jest-dom).

```typescript
import {render, fireEvent} from '@testing-library/react';
import Counter from '../../components/counter';

test('counter increments and decrements when the buttons are clicked', () => {
  render(<Counter />);
  const decrement = screen.getByRole('button', {name: /decrement/i});
  const increment = screen.getByRole('button', {name: /increment/i});
  const message = screen.getByText(/current count/i);

  // Notice that React Testing Library queries do null-checks for you automatically, so you can ditch all this null-checking stuff.
  expect(message).toHaveTextContent('Current count: 0');

  fireEvent.click(increment);
  expect(message).toHaveTextContent('Current count: 1');

  fireEvent.click(decrement);
  expect(message).toHaveTextContent('Current count: 0');
});
```

### Using @testing-library/user-event
Clicking buttons is also a bit of an implementation detail. We're firing a single event, when we actually should be firing several other events like the user does. When a user clicks a button, they first have to
move their mouse over the button which will fire some mouse events. They'll also mouse down and mouse up on the input and focus it! Lots of events!

If we want to be truly implementation detail free, then we should probably fire all those same events too. Luckily for us, Testing Library has us covered with `@testing-library/user-event`. This may one-day be baked directly into `@testing-library/dom`, but for now it's in a separate package.

When the user clicks on things, they are firing all kinds of events like pointer events, mouse events. If they're using keyboard, then they're going to be doing key events. It would be great if instead of just firing the click event, we've fired all of those events to ensure that our test is resembling the way that our software is going to be used in production as closely as possible.

The only difference between fireEvent and userEvent is that userEvent is going to fire a bunch of different events that are associated with this typical user interaction of a click. UserEvent does fire a mouseDown and a mouseUp, as well as several other events that are typically fired when a user clicks on a particular area of the UI.

UserEvent is built on top of testing-library's fireEvent to fire all of those events. UserEvent has a bunch of methods on it that you can use to trigger typical interactions that your users will perform with your application. In general, you want to defer to userEvent if you can, before you reach for fireEvent. That way you can keep your test free of implementation details.

```typescript
import {render, screen} from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Counter from '../../components/counter';

test('counter increments and decrements when the buttons are clicked', () => {
  render(<Counter />);
  const decrement = screen.getByRole('button', {name: /decrement/i});
  const increment = screen.getByRole('button', {name: /increment/i});
  const message = screen.getByText(/current count/i);

  // Notice that React Testing Library queries do null-checks for you automatically, so you can ditch all this null-checking stuff.
  expect(message).toHaveTextContent('Current count: 0');

  // The only difference between fireEvent and userEvent is that userEvent is going to fire a bunch of different events that are associated
  // with this typical user interaction of a click. UserEvent does fire a mouseDown and a mouseUp, as well as several other events that are
  // typically fired when a user clicks on a particular area of the UI.
  userEvent.click(increment);
  expect(message).toHaveTextContent('Current count: 1');

  userEvent.click(decrement);
  expect(message).toHaveTextContent('Current count: 0');
});
```

Once you're done, look around in the code of `@testing-library/user-event`'s [`click` method](https://github.com/testing-library/user-event/blob/673c2257ef63f69dd46b9fe2d4bbc3146db1426b/src/click.ts#L116-L139).
It's pretty interesting! You'll notice that it's using `fireEvent` under the hood. For example, a single click on a button will fire these events (in this order):

- `pointerover`
- `pointerenter`
- `mouseover`
- `mouseenter`
- `pointermove`
- `mousemove`
- `pointerdown`
- `mousedown`
- `focus`
- `focusin`
- `pointerup`
- `mouseup`
- `click`

### Form Testing
The users spend a lot of time interacting with forms and many of them are among the most important parts of our application (like the "checkout" form of an e-commerce app or the "login" form of most apps). Because of this, it's pretty critical to have confidence that those continue to work over time.

You need to ensure that the user can find inputs in the form, fill in their information, and validate that when they submit the form the submitted data is correct.

```typescript
import {render, screen} from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Login from './components/login';

test('submitting the form calls onSubmit with username and password', () => {
  let submittedData: unknown;
  const handleSubmit = (data: unknown) => (submittedData = data);
  render(<Login onSubmit={handleSubmit} />);
  const username = 'chucknorris';
  const password = 'i need no password';

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.type(screen.getByLabelText(/password/i), password);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  expect(submittedData).toEqual({username, password});
});
```

#### *Using a jest mock function*
Jest has built-in "mock" function APIs. Rather than creating the `submittedData` variable, try to use a mock function and assert it was called correctly:

```typescript
test('submitting the form calls onSubmit with username and password', () => {
  // Jest mock function that will keep track of those things for us
  const handleSubmit = jest.fn();
  render(<Login onSubmit={handleSubmit} />);
  const username = 'chucknorris';
  const password = 'i need no password';

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.type(screen.getByLabelText(/password/i), password);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  expect(handleSubmit).toHaveBeenCalledWith({username, password});
  expect(handleSubmit).toHaveBeenCalledTimes(1);
});
```

#### *Generate test data*
An important thing to keep in mind when testing is simplifying the maintenance of the tests by reducing the amount of unrelated cruft in the test. You want to make it so the code for the test communicates what's important and what is not important.

Specifically, if we have these values:

```javascript
const username = 'chucknorris'
const password = 'i need no password'
```

Does the code behave differently when the username is `chucknorris`? Do we have special logic around that? Without looking at the implementation I cannot be completely sure. What would be better is if the code communicated that the actual value is irrelevant. But how do you communicate that? A code comment? Let's generate the value!

There's a package we can use for this called [faker](https://www.npmjs.com/package/faker). You can get a random username and password from `faker.internet.userName()` and `faker.internet.password()`. Import that and generate the username and password.

```typescript
import type {LoginFormValues} from '../../components/login';

function buildLoginForm(overrides?: Partial<LoginFormValues>) {
  return {
    username: internet.userName(),
    password: internet.password(),
    overrides,
  };
}

test('submitting the form calls onSubmit with username and password', () => {
  const handleSubmit = jest.fn();
  render(<Login onSubmit={handleSubmit} />);
  // We might not want a randomly generated password and we'd instead want a specific password.
  const {username, password} = buildLoginForm({password: 'Qwerty123*'});

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.type(screen.getByLabelText(/password/i), password);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  expect(handleSubmit).toHaveBeenCalledWith({username, password});
  expect(handleSubmit).toHaveBeenCalledTimes(1);
});
```

#### *Using Test Data Bot*
There's a library for generating test data: [`@jackfranklin/test-data-bot`](https://www.npmjs.com/package/@jackfranklin/test-data-bot). It provides a few nice utilities (including the ability to provide overrides like our own function does).

```typescript
import {build, fake, sequence} from '@jackfranklin/test-data-bot';

const loginFormBuilder = build<LoginFormValues>({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
    // email: sequence(x => `jack${x}@email.com`),
  },
});

test('submitting the form calls onSubmit with username and password', () => {
  const handleSubmit = jest.fn();
  render(<Login onSubmit={handleSubmit} />);
  const {username, password} = loginFormBuilder();

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.type(screen.getByLabelText(/password/i), password);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  expect(handleSubmit).toHaveBeenCalledWith({username, password});
  expect(handleSubmit).toHaveBeenCalledTimes(1);
});
```

### Mocking HTTP Requests
Testing the frontend code interacts with the backend is important. It's how the user uses our applications, so it's what our tests should do as well if we want the maximum confidence. However, there are several challenges that come with doing that. The setup required to make this work is non-trivial. It is definitely important that we test that integration, but we can do that with a suite of solid E2E tests using a tool like [Cypress](https://cypress.io).

For our Integration and Unit component tests, we're going to trade-off some confidence for convenience and we'll make up for that with E2E tests. So for all of our Jest tests, we'll start up a mock server to handle all of the `window.fetch` requests we make during our tests.

> Because window.fetch isn't supported in JSDOM/Node, we have the `whatwg-fetch`
> module installed which will polyfill fetch in our testing environment which
> will allow MSW to handle those requests for us. This is setup automatically in
> our jest config thanks to `react-scripts`.

To handle these fetch requests, we're going to start up a "server" which is not actually a server, but simply a request interceptor. This makes it really easy to get things setup (because we don't have to worry about finding an available port for the server to listen to and making sure we're making requests to the right port) and it also allows us to mock requests made to other domains.

We'll be using a tool called [MSW](https://mswjs.io/) for this. Here's an example of how you can use msw for tests:

```typescript
// __tests__/fetch.test.tsx
import {rest} from 'msw'
import {setupServer} from 'msw/node'
import {render, waitForElementToBeRemoved, screen} from '@testing-library/react'
import {userEvent} from '@testing-library/user-event'
import Fetch from '../fetch'

const server = setupServer(
  rest.get('/greeting', (req, res, ctx) => {
    return res(ctx.json({greeting: 'hello there'}))
  }),
  // Other handler
  rest.post<Record<string, string>, LoginResponse>(
    '/login',
    async (req, res, ctx) => {
      // That way, we feel a little bit more confident because the mock resembles the real world more closely.
      // This allows us to add some additional tests later to verify what the UI does when we get a 400 response because we didn't submit a password.
      if (!req.body.password) {
        return res(ctx.status(400), ctx.json({message: 'password required'}));
      }
      if (!req.body.username) {
        return res(ctx.status(400), ctx.json({message: 'username required'}));
      }
      return res(ctx.json({username: req.body.username}));
    },
  )
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

test('loads and displays greeting', async () => {
  render(<Fetch url="/greeting" />)

  userEvent.click(screen.getByText('Load Greeting'))

  await waitForElementToBeRemoved(() => screen.getByText('Loading...'))

  expect(screen.getByRole('heading')).toHaveTextContent('hello there')
  expect(screen.getByRole('button')).toHaveAttribute('disabled')
})

test('handles server error', async () => {
  server.use(
    rest.get('/greeting', (req, res, ctx) => {
      return res(ctx.status(500))
    }),
  )

  render(<Fetch url="/greeting" />)

  userEvent.click(screen.getByText('Load Greeting'))

  await waitForElementToBeRemoved(() => screen.getByText('Loading...'))

  expect(screen.getByRole('alert')).toHaveTextContent('Oops, failed to fetch!')
  expect(screen.getByRole('button')).not.toHaveAttribute('disabled')
})
```

That should give you enough to go on, but if you'd like to check out the docs.
[MSW](https://mswjs.io/)

#### Server Request Handlers
One of the cool things about MSW is that it works not only in the test environment in Node as well as in our development environment in the browser. It's often more reliable, works offline, doesn't require a lot of environment setup,
and allows us to start writing UI for APIs that aren't finished yet. It means that as awesome because it's able to share those server handlers for our development environment in the browser. All we had to do that is put our server handlers in a shared file, which we import in both our development environment, as well as our test environment, and we can have this nice shared experience between the both of them.

**Example**
```typescript
// src/test/server-handlers.ts
import {rest} from 'msw'

const delay = process.env.NODE_ENV === 'test' ? 0 : 1500

type LoginResponse = {username: string} | {message: string}

const handlers = [
  // <RequestBodyType, ResponseBodyType, RequestParamsType>
  rest.post<Record<string, string>, LoginResponse>(
    'https://auth-provider.example.com/api/login',
    async (req, res, ctx) => {
      if (!req.body.password) {
        return res(
          ctx.delay(delay),
          ctx.status(400),
          ctx.json({message: 'password required'}),
        )
      }
      if (!req.body.username) {
        return res(
          ctx.delay(delay),
          ctx.status(400),
          ctx.json({message: 'username required'}),
        )
      }
      return res(ctx.delay(delay), ctx.json({username: req.body.username}))
    },
  ),
]

export {handlers}
```
Development Environment in Node

```typescript
// src/test/server.ts
import {setupWorker} from 'msw'
import {handlers} from './server-handlers'
import {homepage} from '../../package.json'

setupWorker(...handlers)

const fullUrl = new URL(homepage)

const server = setupWorker(...handlers)

server.start({
  quiet: true,
  serviceWorker: {
    url: fullUrl.pathname + 'mockServiceWorker.js',
  },
})
```

Test Environment in the browser

```typescript
import {render, screen, waitForElementToBeRemoved} from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import {build, fake} from '@jackfranklin/test-data-bot';
import {setupServer} from 'msw/node';
import {handlers} from 'test/server-handlers';
import Login from '../../components/login-submission';
import {LoginFormValues} from '../../components/login';

const buildLoginForm = build<LoginFormValues>({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
  },
});

const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterAll(() => server.close());

test(`logging in displays the user's username`, async () => {
  render(<Login />);
  const {username, password} = buildLoginForm();

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.type(screen.getByLabelText(/password/i), password);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  // screen.debug();

  // as soon as the user hits submit, we render a spinner to the screen. That
  // spinner has an aria-label of "loading" for accessibility purposes, so
  // wait for the loading spinner to be removed using waitForElementToBeRemoved
  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i));
  // https://testing-library.com/docs/dom-testing-library/api-async#waitforelementtoberemoved

  // screen.debug();

  // once the login is successful, then the loading spinner disappears and
  // we render the username.
  // assert that the username is on the screen
  expect(screen.getByText(username)).toBeInTheDocument();
});

test(`omitting the password results in an error`, async () => {
  render(<Login />);
  const {username} = buildLoginForm();

  userEvent.type(screen.getByLabelText(/username/i), username);
  // not going to fill in the password
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i));

  // screen.debug();

  expect(screen.getByRole('alert')).toHaveTextContent('password required');
});
```

#### Inline Snapshots for error messages
Snapshot tests are a very useful tool whenever you want to make sure your UI does not change unexpectedly.

A typical snapshot test case renders a UI component, takes a snapshot, then compares it to a reference snapshot file stored alongside the test. The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

Inline snapshots behave identically to external snapshots (.snap files), except the snapshot values are written automatically back into the source code. This means you can get the benefits of automatically generated snapshots without having to switch to an external file to make sure the correct value was written.

It's not good practice hard coding error messages because if the error message wherever to change.

Instead, we can use a special assertion to take a "snapshot" of the error message and Jest will update our code for us. Use `toMatchInlineSnapshot` rather than an explicit assertion on that error element.

Example:

First, you write a test, calling .toMatchInlineSnapshot() with no arguments:

```typescript
test(`omitting the password results in an error`, async () => {
  render(<Login />);
  const {username} = buildLoginForm();

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i));

  expect(screen.getByRole('alert').textContent).toMatchInlineSnapshot();
});
```

The next time you run Jest, the *alert* will be evaluated, and a snapshot will be written as an argument to toMatchInlineSnapshot:

```typescript
test(`omitting the password results in an error`, async () => {
  render(<Login />);
  const {username} = buildLoginForm();

  userEvent.type(screen.getByLabelText(/username/i), username);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i));

  // This is one feature that I use for error messages all the time. I think it's super helpful as an assertion for this kind of scenario
  expect(screen.getByRole('alert').textContent).toMatchInlineSnapshot(
    `"password required"`,
  );
});
```

That's all there is to it! You can even update the snapshots with --updateSnapshot or using the u key in --watch mode.

[Snapshot Testing](https://jestjs.io/docs/en/snapshot-testing)

#### One-off Server Handlers
How would we test a situation where the server fails for some unknown reason? There are plenty of situations where we want to test what happens when the _server_ misbehaves. But we don't want to code those scenarios in our application-wide server handlers for two reasons:

1. It clutters our application-wide handlers. Lots of the same problems of CSS applies here: people are afraid to modify or delete any code because they're uncertain what other code will break as a result.
2. The indirection makes the tests harder to understand.

[Read more about the benefits of colocation](https://kentcdodds.com/blog/colocation).

So instead, we want one-off server handlers to be written directly in the test that needs it. This is what MSW's `server.use` API is for. It allows you to add server handlers after the server has already started. And the `server.resetHandlers()` allows you to remove those added handlers between tests to preserve test isolation and restore the original handlers.

```typescript
...

// We'll say after each one of these, we're going to clean up all of the runtime handlers. Just remove them all and go back to what we had when we first set up this server.
afterEach(() => server.resetHandlers());

test(`unknown server error displays the error message`, async () => {
  const testErrorMessage = 'Something is wrong';
  // Keep in mind here is we've added a server handler for this request, effectively overriding the existing request.
  // Things are working OK right here because this is the last test in our file, but if I move this up to the very
  // top and make it the first test in our file, then sure this test will pass just fine, but my other tests are totally going to fail.

  // The reason that they fail is because every time we make this POST request, we're going to respond with a 500. We need to clear this out somehow.
  // One way we can do that is say server reset handlers in afterEach test
  server.use(
    rest.post<Record<string, string>, LoginResponse>(
      'https://auth-provider.example.com/api/login',
      async (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({message: testErrorMessage}));
      },
    ),
  );
  render(<Login />);
  userEvent.click(screen.getByRole('button', {name: /submit/i}));

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i));

  expect(screen.getByRole('alert')).toHaveTextContent(testErrorMessage);
});

...
```

### Context and Custom Render Method

A common question when testing React components is what to do with React components that use context values. If you take a step back and consider the guiding testing philosophy of writing tests that resemble the way our software is used, then you'll know that you want to render your component with the provider:

```javascript
render(
  <ContextProvider>
    <ComponentToTest />
  </ContextProvider>,
)
```

The one problem with this is if you want to re-render the `<ComponentToTest />` (for example, to give it new props and test how it responds to updated props), then you have to include the context providers:

```javascript
const {rerender} = render(
  <ContextProvider>
    <ComponentToTest />
  </ContextProvider>,
)

rerender(
  <ContextProvider>
    <ComponentToTest newProp={true} />
  </ContextProvider>,
)
```

This is kind of annoying, so instead, you can provide a `wrapper` option and that will ensure that rerenders are wrapped as well:

```typescript
function Wrapper({children}: {children: React.ReactNode}) {
  return <ContextProvider>{children}</ContextProvider>
}

const {rerender} = render(<ComponentToTest />, {wrapper: Wrapper})

rerender(<ComponentToTest newProp={true} />)
```

#### Example

```typescript
import * as React from 'react';
import {render, screen} from '@testing-library/react';
import {ThemeProvider} from '../../components/theme';
import EasyButton from '../../components/easy-button';

test('renders with the light styles for the light theme', () => {
  function Wrapper({children}: {children: React.ReactNode}) {
    return <ThemeProvider initialTheme="light">{children}</ThemeProvider>;
  }
  render(<EasyButton>Easy</EasyButton>, {wrapper: Wrapper});
  const button = screen.getByRole('button', {name: /easy/i});
  expect(button).toHaveStyle(`
     background-color: white;
     color: black;
   `);
});

test('renders with the dark styles for the dark theme', () => {
  function Wrapper({children}: {children: React.ReactNode}) {
    return <ThemeProvider initialTheme="dark">{children}</ThemeProvider>;
  }
  render(<EasyButton>Easy</EasyButton>, {wrapper: Wrapper});
  const button = screen.getByRole('button', {name: /easy/i});
  expect(button).toHaveStyle(`
     background-color: black;
     color: white;
   `);
});
```

[Wrapper API](https://testing-library.com/docs/react-testing-library/api#wrapper)

This `Wrapper` could include providers for all your context providers in your app: Router, Theme, Authentication, etc.

To take it further, you could create your own custom render method that does this automatically:

```javascript
import {render as rtlRender} from '@testing-library/react'
// "rtl" is short for "react testing library" not "right-to-left"

function render(ui, options) {
  return rtlRender(ui, {wrapper: Wrapper, ...options})
}

// then in your tests, you don't need to worry about context at all:
const {rerender} = render(<ComponentToTest />)

rerender(<ComponentToTest newProp={true} />)
```

#### Example

```typescript
// Extending a type via intersections
type renderWithProvidersProps = RenderOptions & {
  theme?: string;
};

// ...options: This object not only supports the theme that I have for my particular use case
// but also anything else that the render function from React Testing Library gives me.
// Example: That would allow me to configure other things like if I wanted to test the hydration
// functionality or whatever else I want to do.
// renderWithTheme(<EasyButton>Easy</EasyButton>, {theme: 'light', hydrate: true });
function renderWithTheme(ui: React.ReactElement, {theme, ...options}: renderWithProvidersProps = {}) {
  function Wrapper({children}: {children: React.ReactNode}) {
    // We need to check if theme is Theme type
    if (theme !== 'light' && theme !== 'dark') {
      throw new Error('Invalid theme');
    }
    return <ThemeProvider initialTheme={theme}>{children}</ThemeProvider>;
  }

  return render(ui, {wrapper: Wrapper, ...options});
}

test('renders with the light styles for the light theme', () => {
  renderWithTheme(<EasyButton>Easy</EasyButton>, {theme: 'light'});
  const button = screen.getByRole('button', {name: /easy/i});
  expect(button).toHaveStyle(`
     background-color: white;
     color: black;
   `);
});

test('renders with the dark styles for the dark theme', () => {
  renderWithTheme(<EasyButton>Easy</EasyButton>, {theme: 'dark'});
  const button = screen.getByRole('button', {name: /easy/i});
  expect(button).toHaveStyle(`
     background-color: black;
     color: white;
   `);
});
```

From there, you can put that custom render function in your own module and use your custom render method instead of the built-in one from React Testing Library. Learn more about this from the docs:

[Setup](https://testing-library.com/docs/react-testing-library/setup)

#### Example

```typescript
// test/test-utils.tsx

// Module that mports render from React Testing Library render.
// It creates its own render function and then it re-exports everything from React Testing Library.
// It can be your own version of React Testing Library.
import {render as rtlRender, RenderOptions} from '@testing-library/react';
import {ThemeProvider} from 'components/theme';

// Extending a type via intersections
type renderWithProvidersProps = RenderOptions & {
  theme?: string;
};

function render(
  ui: React.ReactElement,
  {theme, ...options}: renderWithProvidersProps = {},
) {
  function Wrapper({children}: {children: React.ReactNode}) {
    if (theme !== 'light' && theme !== 'dark') {
      throw new Error('Invalid theme');
    }
    return <ThemeProvider initialTheme={theme}>{children}</ThemeProvider>;
  }

  return rtlRender(ui, {wrapper: Wrapper, ...options});
}

export * from '@testing-library/react';
// override React Testing Library's render with our own
// They should all be using this render method because those providers are an implementation detail of each one of your components.
// They should just have all of the same providers that they're going to have when you ship the actual app.
export {render};
```

## Add-Ons

* [Closure](https://whatthefork.is/closure)
* [Imperative vs Declarative Programming](https://ui.dev/imperative-vs-declarative-programming/)
