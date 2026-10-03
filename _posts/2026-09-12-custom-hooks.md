---
layout: post
title:  "Custom Hooks"
date:   2026-09-12 00:00:00 +0000
categories:
---
Part 7 of FSO is about custom hooks and other extra bits. Let's start with the basics.

## What is a Hook?
A hook is a special function that lets a function component use React features, such as states, side effects, and context, that persist across renders.

They have conventions to follow:
1. Hooks should only be called from React functional components (not inside of conditional statements, etc).
2. The names should start with 'use'.
3. Hooks can call other hooks.
4. Hooks follow the [Rules of Hooks](https://legacy.reactjs.org/docs/hooks-rules.html).

## Why Hooks?
For the same reasons you want regular functions:
1. **Simplify complex code.** Move complicated logic out of a component and into a function, so the component stays easy to read.
2. **DRY (Don't Repeat Yourself).** Logic that appears in multiple components can be extracted into one custom hook and reused.
3. **Single Responsibility.** If a component is getting large, it likely has too many responsibilities. Extracting some of them into custom hooks is one way to break it up.
4. **Access to React features.** But, hooks give also function components a way to hold state across renders (`useState`, `useRef`) and to synchronize with things outside React, like the DOM, timers, or network requests (`useEffect`).

## The Common Built-in React Hooks
<details markdown="1">
<summary>useState</summary>
For saving a variable that needs to persist across renders. Setting the variable also triggers a render refresh for the component that state is called in.
```jsx
const [count, setCount] = useState(0)
```
</details>

<details markdown="1">
<summary>useEffect</summary>
React works in a few phases:
1. **Render:** React calls `App()` and gets a tree of plain objects describing the DOM (also known as the virtual DOM).
2. **Reconcile:** React compares the new tree to the previous one to evaluate what has changed.
3. **Commit:** React applies the changes to the real DOM.
4. **Effects:** `useEffect` callbacks run, and `useRef` refs point to real DOM nodes.

```jsx
useEffect(function () {
  // ... do something here
  return cleanupFunction
}, [dependencyArray])
```

So `useEffect` is for running side effects *after* React has rendered and committed. Things like fetching data, subscribing to `window`/`document` events, or dealing with timers.

The **dependency array** controls when the effect runs:
- `[]`: once, after the first render.
- `[a, b]`: after the first render, and again after any render where `a` or `b` changed.
- No array: after every render.

This is also what makes it safe to set state inside an effect. A fetch on mount with `[]` runs once, so it won't loop and cause an infinite render loop error.

If the effect sets something up (a listener, a timer), it can return a **cleanup function**, which React runs before the next effect and when the component unmounts.
</details>

<details markdown="1">
<summary>useRef</summary>
Returns an object with a single property (`.current`), and you can read and write the value there. It's like a `useState` when it comes to persisting values across renders, but mutating the value of a `useRef` will not trigger a re-render.
```jsx
const timerRef = useRef(null)
```

You can also pass the ref to a JSX element's `ref` prop, and after the commit step React sets `.current` to the real DOM node:
```jsx
function SearchBox() {
  const inputRef = useRef(null)

  useEffect(() => {
    inputRef.current.focus()
  }, [])

  return <input ref={inputRef} />
}
```
`useRef` is useful for holding a mutable value that shouldn't cause re-renders, like a timer ID, an interval, or the previous value of a prop or holding DOM nodes.
</details>

<details markdown="1">
<summary>useContext</summary>
I've already had a whole section dedicated to it previously [here](../../../2026/09/07/context.html). FSO part 7 covers `useMemo` and `useCallback`, so lets go over them.
</details>

<details markdown="1">
<summary>useMemo</summary>

Is a way to cache the result of a function between renders. It accepts a function that performs the computation and a dependency array. React only re-runs the punction when one of the dependencies changes, otherwise it returns the previously cached result.
```jsx
const [filter, setFilter] = useState('')

const filtered = useMemo(() => {
  return ITEMS.filter(item => {
    expensiveCalculation()
    return item.includes(filter)
  })
}, [filter])
```

`useMemo` is a performance optimisation, so it should not be reached for by default. Premature memoisation adds complexity without benefit when the computation is fast. Only add it when a particular calculation is a confirmed bottleneck.
</details>

<details markdown="1">
<summary>React.memo</summary>

Caches the rendered output of an entire component. It's not a hook but a higher-order component. It is being covered here because it complements `useMemo`. When a component is wrapped in `React.memo`, React skips re-rendering the component if its props has not changed since the last render.
```jsx
const MyComponent = React.memo(({ value }) => {
  console.log('rendered')
  return <div>{value}</div>
})
```
Note that `React.memo` only checks props. If the component uses a context value or its own state, it will still re-render when those change.
</details>

<details markdown="1">
<summary>useCallback</summary>

It works similarly to `useMemo` but it's for caching functions. As functions defined in a component are recreated as new objects on every render, if it is passed as `props` into a component wrapped in `React.memo`, it would render `React.memo` pointless, as the component will always see a changed prop and re-render anyway. This also happens if it is listed as a dependency of `useEffect` or `useMemo`. It'll cause these two to re-run on every render.

```jsx
const handleDelete = useCallback((id) => {
  setNotes(notes => notes.filter(note => note.id !== id))
}, []) // no external dependencies: this function never needs to change
```
Wrapping a function in `useCallback` will return the same function each time unless any of the dependencies in its dependency array changes.

Like `useMemo`, only use it when there is a concrete problem, such as a memoised child re-rendering unnecessarily or a `useEffect` running too often because of a function dependency. Otherwise it makes code harder to read without delivering a performance benefit.
</details>

## Creating Custom Hooks
Hooks are just regular Javascript functions that uses other hooks and follows the Rule of Hooks after all.

Here's a simple counter hook:
```jsx
const useCounter = () => {
  const [value, setValue] = useState(0)
  const increase = () => setValue(value + 1)
  const decrease = () => setValue(value - 1)
  const zero = () => setValue(0)

  return {
    value,
    increase,
    decrease,
    zero
  }
}
```

Now we can use it as follows:
```jsx
const App = () => {
  const counter = useCounter()

  return (
    <div>
      <div>{counter.value}</div>
      <button onClick={counter.increase}>plus</button>
      <button onClick={counter.decrease}>minus</button>      
      <button onClick={counter.zero}>zero</button>
    </div>
  )
}
```

## Spread syntax
The spread syntax allows an iterable (array or object) to be expanded like so:
```js
const array = [1, 2, 3];
const obj = { ...array }; // { 0: 1, 1: 2, 2: 3 }
```
We can actually use the spread syntax to assign attributes to elements too:
```jsx
const name = useField('text')

return (
  <div>
    <form>
      name: 
      <input  {...name} /> 
      <br/> 
    </form>
  </div>
)
```

## Custom hooks don't share states between calls
Creating a custom hook with state within it, each time the hook is called like so:
```jsx
const counter = useCounter()
```
It will create a new instance of the state. Which means declaring it in two files, but then using `counter.increase()` in the second file will not update the state in the first one, as they are two different instances.

<br />

[Previous Post](../../../2026/09/07/context.html) | Next Post