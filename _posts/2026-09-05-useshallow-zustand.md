---
layout: post
title:  "Zustand's useShallow and How to test Zustand stores"
date:   2026-09-05 00:00:00 +0000
categories:
---
## useShallow
We can manipulate data within Zustand's store itself, keeping state altering logic within the store, and cleaning up logic from components. It is possible to implement logic within the selectors themselves like so:
```jsx
export const useNotes = () => useNoteStore(state => {
  const { notes, filter } = state
  if (filter === 'important') return notes.filter(n => n.important)
  if (filter === 'nonimportant') return notes.filter(n => !n.important)
  return notes
})
```
However, this results in an infinite rendering loop whenever the filter is set.

On default Zustand compares the selector's return value using the `===` (reference equality) operator, which checks if the memory address is the same. Since using an immutable javascript method that returns a new object/array (such as `.map`), React will always interpret it as a new state and trigger another re-render, creating an infinite loop.

`useShallow` replaces the `===` comparison with a shallow comparison. It manually compares each array element one by one to check if the content has changed. If it has not, it returns the old array reference instead of the new one.

The result is as so:
```jsx
import { useShallow } from 'zustand/react/shallow'

export const useNotes = () => useNoteStore(useShallow(({ notes, filter }) => {
  if (filter === 'important') return notes.filter(n => n.important)
  if (filter === 'nonimportant') return notes.filter(n => !n.important)
  return notes
}))
```
<br />

## Middleware
Zustand supports middlewares, which can be added to stores easily without touching the store's inner logic. It 'wraps' around the store manually, unlike Node's flat list method (though underneath, Node middlewares also wrap around the app).

Here is a logger middleware:
```js
const logger = (config) => (set, get) => config(
  (...args) => {
    console.log('prev state', get());
    set(...args);
    console.log('next state', get());
  },
  get
);
```
And here is how it's used:
```js
const useNoteStore = create(logger((set, get) => ({
  notes: [],
  filter: '',
  actions: {
    // ...
  }
})))
```
The middleware itself looks arcane. It's three functions wrapped around each other. Here's how the logger looks like unwrapped in a (hopefully) slightly less confusing way:
```js
const logger = function (config) { // config is the original function passed into create()
                                   // to create useCounterStore
  return function (set, get) { // logger returns this different (set, get) => ({ that returns...
    return config( // The original config, but a fake set argument put in
      (...args) => {
        console.log('prev state', get())
        set(...args) // the real set argument used here to actually set ...args
        console.log('next state', get())
      },
      get // the get here is the real one from the outer function
    )
  }
}
```
Here's an alternative diagram that explains it:
```
create(logger(config))
        └────┬────┘
             │
  logger(config) runs immediately,
  returns a NEW function (let's call it wrapped)


create(wrapped)
   Zustand calls wrapped(realSet, realGet)
       │
       └─→ wrapped calls config(fakeSet, realGet)
                  │
                  └─→ your original config runs,
                      returns { count: 0, increment: ... }
                      but any set() call inside it
                      is actually fakeSet
```
Just mind bending.
<br />

## Testing Zustand Stores
You could just directly use the stores like so:
```js
import { beforeEach, describe, expect, it } from 'vitest'
import useCounterStore from './store'

beforeEach(() => {
  useCounterStore.setState({ counter: 0 })
})

describe('counter store', () => {
  it('initial state is 0', () => {
    expect(useCounterStore.getState().counter).toBe(0)
  })

  it('increment increases counter by 1', () => {
    useCounterStore.getState().actions.increment()
    expect(useCounterStore.getState().counter).toBe(1)
  })
})
```
But if more complex logic has been implemented through custom hooks (such as retrieving notes from the backend upon start up), it may be necessary to write tests that utilise the hooks. 
```js
import { beforeEach, describe, expect, it } from 'vitest'
import { renderHook, act } from '@testing-library/react'
import useCounterStore, { useCounter, useCounterControls } from './store'

beforeEach(() => {
  useCounterStore.setState({ counter: 0 })
})

describe('counter hooks', () => {
  it('useCounter returns initial value of 0', () => {
    const { result } = renderHook(() => useCounter())
    expect(result.current).toBe(0)
  })

  it('increment updates counter', () => {
    const { result: counter } = renderHook(() => useCounter())
    const { result: controls } = renderHook(() => useCounterControls())

    act(() => controls.current.increment())
    expect(counter.current).toBe(1)
  })
})
```
`renderHook` gives the hook a dummy component function to render inside, since hooks expect to hook onto a React Component. So we can't call `useCounter()` from any `.js` file and expect it to work. It returns an object of the current state of the hook.

State updates are wrapped around `act()`, which ensures React to finish all queued state changes and re-renders before continuing.

### More mocking
Below is a mock version of `noteService`, the service responsible for communicating with the server. `vi.mock` replaces the `./services/notes` module with its own version. 
```js
vi.mock('./services/notes', () => ({
  default: {
    getAll: vi.fn(),
    createNew: vi.fn(),
    update: vi.fn(),
  }
}))

import noteService from './services/notes'
```
So after, when `noteService` is imported, it actually imports the mocked version instead of the real deal. And any hooks that use `noteService` will be interfacing with the mocked version as well.
```js
it('add appends a new note', async () => {
const newNote = { id: 2, content: 'New note', important: false }
noteService.createNew.mockResolvedValue(newNote)
const { result } = renderHook(() => useNoteActions())

await act(async () => { //async because the add function is async itself
    await result.current.add('New note')
})

const { result: notesResult } = renderHook(() => useNotes())
expect(notesResult.current).toContainEqual(newNote)
})
```
The `add` action within `useNoteActions()` awaits a return value from the server and uses it to update the store's state, so the mock needs to resolve a real note so `add` works properly.

In order to do that, `.mockResolvedValue(newNote)` configures the mocked `createNew` function (a `vi.fn()`) to return a Promise that resolves to `newNote`, mocking the server response.

<br />

[Previous Post](../../../2026/09/04/flux-zustand.html) | Next Post