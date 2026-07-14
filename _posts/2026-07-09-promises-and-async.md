---
layout: post
title:  "Promises, async and await"
date:   2026-07-09 00:00:00 +0000
categories:
---
I’m FINALLY at the part about connecting backends, promises and awaits.

<br>
**Callbacks:**
: Appears to be a fancy way people describe ‘a function that calls another function when necessary’. Like the anonymous function that is called during an `addEventListener`. Callbacks were used in Javascript for async code before promises became a thing.

There are two kinds of callbacks:
* **Synchronous callback:** Runs functions passed into it immediately, during the outer function’s execution. E.g. `forEach`, `map`
* **Asynchronous callback:** Runs functions passed into it after the outer function returns. E.g. `setTimeout`, `addEventListener`, Promise `.then`

<br>
**Promises:**
* Are a way to handle asynchronous code, it lets us wait for some code to finish before going to the next step
* Is an object, initialised by the `Promise` class, it requires a function as an input argument
* Runs whatever in the input argument function immediately
* There are two functions to run inside the input argument function of a promise, `resolve` and `reject`:
    - Both are functions for when the async work finishes, it becomes the function to pass the final value into `.then`
    - `resolve` is for when it succeeds, `reject` is when it fails
* Whatever after/outside the promise still runs immediately in its own thread (this allows Javascript to do multiple things concurrently)
* `.then` runs a function with the result of the promise after the promise is fulfilled
    * It always returns a new promise, which is why you can chain them together
    * Errors skip to `.catch`

Example:
{% highlight javascript %}
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('success!'); // this becomes the value in .then()
    } else {
      reject(new Error('failure'));
    }
  }, 1000);
});

myPromise.then(result => console.log(result));

Promise.all() lets us run multiple promises at the same time, and wait for all of them to finish
Promise.all([
  new Promise(resolve => loadProducts(() => resolve())),
  new Promise(resolve => loadCart(() => resolve()))
]).then(() => {
  renderOrderSummary()
  renderPaymentSummary()
})
{% endhighlight %}

<br>
**Async:**
* Returns a promise by wrapping whatever in the function in one
* Whatever the function returns gets converted into a ‘`resolve(returned value)`’
* Async lets us use `await`

<br>
I finished the supersimpledev tutorial today. Fortunately I haven’t forgotten React too much after this 5 day detour, and after 30 minutes of staring and console.log’ing my previous code for FSO, I’ve gotten comfortable with my code again.

The logical OR `||` operator in Javascript is so interesting. Like the logical AND (guard operator? They call it?), it returns actual values instead of pure true/false like in C.

```
OR Operator (||)
Const result = value1 || value2;
If value1 is true, the result will be value1. Else, the value will be value2.
```
A much shorter version of the ternary operator. And it plays into how Javascript has ‘falsy’ and ‘truthy’ values.

Falsy values are: `null`, `undefined`, `false`, `NaN`, `0`, `-0`, `0n`, `‘’` and `document.all`

Everything else computes to a truthy value. So an empty array or object is technically true in Javascript.

FSO uses the axios library to handle the RESTful requests to the backend. So no manually writing it out with `fetch` and `await`. I still find promises confusing, I need to get even more practice with it.

<br>

[Previous Post](../../../2026/07/07/useful-javascript.html) | [Next Post](../../../2026/07/09/derived-data-states.html)