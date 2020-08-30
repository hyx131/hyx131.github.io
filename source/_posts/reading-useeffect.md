---
title: The useEffect Hook
date: 2020-08-22 19:50:56
categories: Reading
tags: React
---

> Blogs under _'Reading'_ category mainly gives highlights/snippets of the topic post for my personal learning purposes (may also contain other learning notes on the side).
> For this topic in full, you can visit the original post here:
>> _Title_: A Complete Guide to useEffect
>> _Author_: [Dan Abramov](http://iqkui.com/)
>> _Path_: http://iqkui.com/a-complete-guide-to-useeffect/

&nbsp;  

### Rendering
Each render has its own props & state & event handlers & effect & ...everything.

```bash
// During first render
function Counter() {
  const count = 0; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// After a click, our function is called again
function Counter() {
  const count = 1; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// After another click, our function is called again
function Counter() {
  const count = 2; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```

When update the state (`setCount`), React calls component again with a different value for `count` (which is a _constant_ inside the function).
Each render = component gets called _again_ with a _different_ `count` value (which was set as a constant) for that particular render.

```bash
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}

// During first render
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 0 inside
  // ...
}

// After a click, our function is called again
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 1 inside
  // ...
}
```

Above example: although the function is async, but the handler will still "see" the value at the time it is being called. Because inside each render, props and state and any values using them (including event handlers) "belongs" to that particular render, thus variables in their scope will only "see" the value for that particular component call.

### Effect Cleanup

```bash
// First render, props are {id: 10}
function Example() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // Cleanup for effect from first render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}

// Next render, props are {id: 20}
function Example() {
  // ...
  useEffect(
    // Effect from second render
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // Cleanup for effect from second render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
```

Cleanup function is called before the next render, values it "sees" = values captured in current render.

### Synchronization
Not lifecycle. _React is about the destination, not journey_.
React synchronizes DOM according to current state & props. Same goes with `useEffect`: allow for synchronize things outside of React tree according to current props & state. This is different from the mental model of _mount/update/unmount_ (which is the journey), would fail to _synchronize_ the results if attempt to write an effect that behaves differently depending on whether the component renders for the first time or not.

### Diff Effects
Running all effects on _every_ render = not efficient (could also lead to infinite loops). Fix by doing the following:

```bash
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]); // Our deps

  const oldEffect = () => { document.title = 'Hello, Dan'; };
const oldDeps = ['Dan'];

const newEffect = () => { document.title = 'Hello, Dan'; };
const newDeps = ['Dan'];

// React can't peek inside of functions, but it can compare deps.
// Since all deps are the same, it doesn’t need to run the new effect.
```
Avoid re-running effect by providing dependency array, will check agains this array during every render, and run the effect if one (or more) of values in the array is different from the previous render.

### Dependency Honesty

1. include _all_ values insdie the component that are used inside the effect.

```bash
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

2. change effect code to reduce dependencies needed.

```bash
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```
For the `count` example, could remove `count` from dependency array by using the [functional updater form](https://reactjs.org/docs/hooks-reference.html#functional-updates) of `setState`.

### Decouple Updates from Actions

```bash
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```
Added another dependency `step`, if want to avoid effect from calling on changes to `step`, could use `useReducer` to remove `step` from dependency array.

```bash
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```
React guarantees the dispatch function to be constant throughout the component lifetime.
_(You may omit dispatch, setState, and useRef container values from the deps because React guarantees them to be static. But it also doesn’t hurt to specify them.)_
When setting a state variable depends on the current value of another state variable, replace both with `useReducer`. Consider `useReducer` for cases where `setSomething(something => ...)` is necessary.
`useReducer` allow to decouple expressing the “actions” that happened in your component from how the state updates in response to them (decouple the update logic from describing what happened).

### Functions in Effects

Move function directly into effect:

```bash
function SearchResults() {
  // ...
  useEffect(() => {
    // We moved these functions inside!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, []); // ✅ Deps are OK
  // ...
}
```
If use any state values inside the function, will need to add that state into the dependency array:

```bash
function SearchResults() {
  const [query, setQuery] = useState('react');

  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, [query]); // ✅ Deps are OK

  // ...
}
```

If functions does not use any values from within the component scope, can hoist the functions to remove them from dependency array:

```bash
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  // ...
}
```

Can also use `useCallback` hook:

```bash
function SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

`useCallback` = adding another layer of dependency checks -- make the function itself change when necessary (instead of calling effect on every change).
