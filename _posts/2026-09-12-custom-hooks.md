---
layout: post
title:  "Custom Hooks"
date:   2026-09-12 00:00:00 +0000
categories:
---
Part 7 of FSO is about custom hooks and other extra bits.

## Custom Hooks
WIP

## Custom hooks don't share states between calls
Creating a custom hook with state within it, each time the hook is called like so:
```jsx
const { count, setCount } = useCounter()
```
It will create a new instance of the state. Which means declaring it in two files, but then using `setCount` in the second file will not update the state in the first one, as they are two different instances.

<br />

[Previous Post](../../../2026/09/07/context.html) | Next Post