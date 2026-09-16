---
layout: post
title:  "Finally... Context API. And also TanStack Query"
date:   2026-09-07 00:00:00 +0000
categories:
---
Part 6C of FSO is about TanStack Query and useContext (finally!)

## Custom Hooks

## Custom hooks don't share states between calls
Creating a custom hook with state within it, each time the hook is called like so:
```jsx
const { count, setCount } = useCounter()
```
It will create a new instance of the state. Which means declaring it in two files, but then using `setCount` in the second file will not update the state in the first one, as they are two different instances.

<br />

[Previous Post](../../../2026/09/05/useshallow-zustand.html) | Next Post