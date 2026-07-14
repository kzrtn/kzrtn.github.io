---
layout: post
title:  "Useful Javascript Arrays and its copy-by-reference quirks"
date:   2026-07-07 00:00:00 +0000
categories:
---
### Useful Array Methods
My ear infection meant staying home instead of going to work. And that means... More code time!

Useful array methods:
`.find` helps find an element in an array.
`.filter` helps filter elements in an array (returns a filtered array)
`.reject` does the same thing as filter, but eliminates elements that fit the filter requirement
`.reduce` uses an accumulator (I really need to find better words for this)

I need to look into what `useEffect()` (in React) is for exactly... I still have no idea what its purpose is, how to use it and when to use it.

I also should really look into promises, await and async in Javascript.
The `.then` keyword is still confusing for me...


### Copy-by-reference
I did not expect to get tripped up so quickly on how Javascript does copy-by-reference.
{% highlight javascript %}
document.querySelectorAll('.js-add-to-cart')
  .forEach((button) => {
    button.addEventListener('click', () => {
      const productName = button.dataset.productName
      let matchingItem;

      cart.forEach(item => {
        if (item.productName === productName) matchingItem = item
      })

      if (matchingItem) {
        matchingItem.quantity += 1
      } else {
        cart.push({
          productName: productName,
          quantity: 1
        })
      }
      console.log(cart)
    })
  })
{% endhighlight %}
I was following a tutorial on vanilla Javascript (brushing up and need to get to understanding promises with SuperSimpleDev) and I was puzzled by how we created a new variable (`matchingItem`), then assigned it one of the objects in the cart object array, and then modifying `matchingItem` also modifies the element in the cart object array itself!

Wow! That’s so much more confusing than I expected! No wonder React has so much fuss about “do NOT modify the original value, use states and always copy by value”. It’s so easy to modify values directly in this language. `const` doesn’t allow direct modification but it allows... this... whatever this is...

So, `.forEach` in javascript goes through elements in an array directly (with no return value in each loop nor at the end of it all), while `.map` maps a new element to the array per element, and will finally return a whole new copy of the array.

One thing that I feel nice about is how much I’ve been having a blast using the array functions I’ve learned. Even though this is vanilla javascript and he never uses these in his code, I use them myself to make my code much shorter.
{% highlight javascript %}
const totalPriceCents = cart.reduce((accumulator, cartItem) =>
  accumulator + ((products.find(product =>
    product.id === cartItem.productId)).priceCents * cartItem.quantity
), 0)
{% endhighlight %}
It’s shorter but it’s not too clean or readable. Oops.

Continuing this javascript tutorial side quest, I’m learning about Jasmine and testing frameworks.

Note to self: I need to look up the `this` key word in Javascript. And how its usage differs in classes, functions and arrow functions.

<br>

[Previous Post](../../../2026/07/01/react-states.html) | [Next Post](../../../2026/07/09/promises-and-async.html)