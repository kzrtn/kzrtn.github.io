---
layout: post
title:  "My Prayers Answered: Flux-architecture and Zustand"
date:   2026-09-04 00:00:00 +0000
categories:
---
In [my last blog post](../../../2026/08/30/react-router.html) I lamented over React’s prop drilling problem and how React Router did not fix or help the problem whatsoever.

But turns out, no!

I AM SO GLAD THAT FLUX-ARCHITECTURE IS A THING.

## Flux Architecture
In [Flux](https://facebookarchive.github.io/flux/docs/in-depth-overview/), states are separated entirely from the React components. They’re like global variables (but are read-only) in the sense that they live completely separate from everything, and components subscribe to the store and pull a copy of the state for rendering.

Basically, three principles in Flux architecture:
1. **Separation:** states must live outside of the component tree
2. **Read-only access:** Components cannot mutate the state directly
3. **Subscription-based sync:** Components pull their own copy after being notified, instead of multiple components reading the same shared memory live

Flux provides a standard way for how and where the application state is kept and for making changes to it. This avoids race conditions, as all writes are serialised through the Dispatcher, one at a time. This stops two updates from being interleaved with each other, a race condition MVC is prone to.

Other than forcing state changes into a serial queue system, Flux also tidies up React programs, as states don’t need to be passed down through props anymore, as individual components can import (subscribe) to stores independently.

![Image](/assets/images/mvc-flux.png)
<br />
<br />

## Zustand
FSO introduces [Zustand](https://zustand.docs.pmnd.rs/), a Flux-like alternative to Flux. It’s gaining popularity for its ease of use, Flux and [Redux](https://redux.js.org/) have a lot of boilerplate code, while Zustand simply cuts down everything down to just the store and view:

![Image](/assets/images/zustand.png)
<br />

A Zustand store is created with their `create` function:
```javascript
import { create } from 'zustand'

const useCounterStore = create(set => ({
  counter: 0,
  actions: {
    increment: () => set(state => ({ counter: state.counter + 1 })),
    decrement: () => set(state => ({ counter: state.counter - 1 })),
    zero: () => set(() => ({ counter: 0 })),
  }  
}))
```
The `create` function creates a store function containing the states and actions (functions that modifies the states with `set()`).

For easy access to the states, they can be selected and exported like so:
```javascript
export const useCounter = () => useCounterStore(state => state.counter)
export const useCounterControls = () => useCounterStore(state => state.actions)
```
All of these are custom React hooks. 

### But what is a hook?
(WIP)

<br />

[Previous Post](../../../2026/08/30/react-router.html) | Next Post